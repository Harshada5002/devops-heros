# Session 11: Kubernetes Networking & Services

## Overview

This session explores Kubernetes networking and Service discovery. It demonstrates all five Service types, compares Kubernetes workload objects, explains Fully Qualified Domain Names (FQDNs), and investigates how CoreDNS resolves DNS queries within a Kubernetes cluster.

## Objectives

- Deploy and test ClusterIP, NodePort, LoadBalancer, ExternalName, and Headless Services.
- Understand how Services provide network access to Pods.
- Compare Deployment, ReplicaSet, DaemonSet, StatefulSet, and Service.
- Learn Kubernetes DNS naming conventions and FQDNs.
- Understand CoreDNS architecture, configuration, and troubleshooting.
- Capture terminal outputs and document observations.
- Push YAML manifests, documentation, and screenshots to GitHub.

## Tools and Technologies

- Kubernetes
- Minikube
- kubectl
- Docker
- NGINX
- CoreDNS
- Windows PowerShell
- YAML
- Git and GitHub

## Project Directory

```text
session-11-kubernetes-services/
├── services/
│   ├── clusterip.yaml
│   ├── nodeport.yaml
│   ├── loadbalancer.yaml
│   ├── externalname.yaml
│   └── headless.yaml
├── fqdn/
│   └── README.md
├── coredns/
│   └── README.md
├── screenshots/
│   ├── 01-clusterip-deployment.png
│   ├── 02-clusterip-service.png
│   ├── 03-nodeport-deployment.png
│   ├── 04-nodeport-test.png
│   ├── 05-loadbalancer-deployment.png
│   ├── 06-loadbalancer-test.png
│   ├── 07-externalname-service.png
│   ├── 08-externalname-test.png
│   ├── 09-headless-service.png
│   ├── 10-headless-endpoints.png
│   ├── 11-deployment-vs-replicaset.png
│   ├── 12-workload-comparison.png
│   ├── 13-replicaset-vs-service.png
│   ├── 14-fqdn-dns-test.png
│   ├── 15-coredns-pods.png
│   ├── 16-coredns-configuration.png
│   ├── 17-dns-resolution-test.png
│   └── 18-coredns-troubleshooting.png
└── README.md
```

## Task 1: Kubernetes Services

A Kubernetes Service provides a stable way to access a group of Pods or, in the case of ExternalName, an external DNS name. Services use selectors and endpoints where applicable to direct traffic to their destinations.

### 1. ClusterIP Service

**Purpose:** Exposes an application through an internal cluster IP. It is normally accessible only from within the cluster.

Create `services/clusterip.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-clusterip
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-clusterip
  template:
    metadata:
      labels:
        app: web-clusterip
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-clusterip
spec:
  type: ClusterIP
  selector:
    app: web-clusterip
  ports:
    - port: 80
      targetPort: 80
```

Apply and verify:

```powershell
kubectl apply -f services/clusterip.yaml
kubectl get deployment web-clusterip
kubectl get pods -l app=web-clusterip
kubectl get service web-clusterip
kubectl get endpointslices -l kubernetes.io/service-name=web-clusterip
```

Test connectivity from a temporary Pod:

```powershell
kubectl run curl-test --image=curlimages/curl:8.12.1 --restart=Never --rm -it -- curl -sS http://web-clusterip
```

**Expected observation:** The Service receives a cluster IP and forwards requests to the selected NGINX Pods.

![01-clusterip-deployment.png](screenshots/01-clusterip-deployment.png)

![02-clusterip-service.png](screenshots/02-clusterip-service.png)

### 2. NodePort Service

**Purpose:** Exposes a Service through a port on the cluster nodes, making it possible to access the application from outside the cluster when network access is available.

Create `services/nodeport.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-nodeport
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-nodeport
  template:
    metadata:
      labels:
        app: web-nodeport
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-nodeport
spec:
  type: NodePort
  selector:
    app: web-nodeport
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```

Apply and verify:

```powershell
kubectl apply -f services/nodeport.yaml
kubectl get pods -l app=web-nodeport
kubectl get service web-nodeport
kubectl describe service web-nodeport
```

For Minikube, obtain the reachable service URL:

```powershell
minikube service web-nodeport --url
```

Use the returned URL in a browser or HTTP client. If Minikube runs with the Docker driver on Windows, direct node IP access may not work; use the URL provided by Minikube.

**Expected observation:** The Service exposes port 80 through NodePort 30080 on the node networking path.

![03-nodeport-deployment.png](screenshots/03-nodeport-deployment.png)

![04-nodeport-test.png](screenshots/04-nodeport-test.png)

### 3. LoadBalancer Service

**Purpose:** Exposes a Service using an external load balancer when the cluster environment supports one.

Create `services/loadbalancer.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-loadbalancer
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-loadbalancer
  template:
    metadata:
      labels:
        app: web-loadbalancer
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: web-loadbalancer
  ports:
    - port: 80
      targetPort: 80
```

Apply and verify:

```powershell
kubectl apply -f services/loadbalancer.yaml
kubectl get deployment web-loadbalancer
kubectl get service web-loadbalancer
kubectl describe service web-loadbalancer
```

For Minikube, open another PowerShell terminal and run:

```powershell
minikube tunnel
```

Keep the tunnel running. In the original terminal, check the Service again:

```powershell
kubectl get service web-loadbalancer -w
```

Test the assigned external IP using an HTTP client once it is available.

**Expected observation:** A supported load-balancer implementation provisions or advertises an external address. Without a compatible implementation, the external IP may remain pending.

![05-loadbalancer-deployment.png](screenshots/05-loadbalancer-deployment.png)

![06-loadbalancer-test.png](screenshots/06-loadbalancer-test.png)

### 4. ExternalName Service

**Purpose:** Maps a Kubernetes Service name to an external DNS name using a DNS CNAME record. It does not create or select Pods.

Create `services/externalname.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: google-dns
spec:
  type: ExternalName
  externalName: example.com
```

Apply and inspect:

```powershell
kubectl apply -f services/externalname.yaml
kubectl get service google-dns
kubectl describe service google-dns
kubectl get service google-dns -o yaml
```

Test DNS resolution from a temporary Pod:

```powershell
kubectl run dns-test --image=busybox:1.36 --restart=Never --rm -it -- nslookup google-dns
```

The Service name resolves through a CNAME to `example.com`. External connectivity also depends on the cluster's DNS configuration and network access.

**Expected observation:** ExternalName has no cluster IP or Pod endpoints; it provides a DNS alias.

![07-externalname-service.png](screenshots/07-externalname-service.png)

![08-externalname-test.png](screenshots/08-externalname-test.png)

### 5. Headless Service

**Purpose:** Provides DNS records for individual backing Pods instead of assigning a virtual cluster IP. It is commonly used for StatefulSets and direct endpoint discovery.

Create `services/headless.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-headless
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web-headless
  template:
    metadata:
      labels:
        app: web-headless
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web-headless
spec:
  clusterIP: None
  selector:
    app: web-headless
  ports:
    - port: 80
      targetPort: 80
```

Apply and verify:

```powershell
kubectl apply -f services/headless.yaml
kubectl get pods -l app=web-headless -o wide
kubectl get service web-headless
kubectl get endpointslices -l kubernetes.io/service-name=web-headless
```

Test DNS resolution:

```powershell
kubectl run dns-test --image=busybox:1.36 --restart=Never --rm -it -- nslookup web-headless
```

**Expected observation:** The Service has no virtual cluster IP. DNS returns the IP addresses of the backing Pods when ready endpoints and DNS records are available.

![09-headless-service.png](screenshots/09-headless-service.png)

![10-headless-endpoints.png](screenshots/10-headless-endpoints.png)

## Task 2: Kubernetes Object Comparison

### Deployment vs ReplicaSet

| Feature | Deployment | ReplicaSet |
|---|---|---|
| Purpose | Manages application releases and desired state | Maintains a desired number of matching Pods |
| Pod management | Manages Pods through ReplicaSets | Creates or deletes Pods directly |
| Scaling | Supports replica scaling | Supports replica scaling |
| Rolling updates | Supports rolling updates and rollbacks | Does not independently provide Deployment-style rollout management |
| Relationship | Owns and manages ReplicaSets | Usually created and managed by a Deployment |

A Deployment is the higher-level controller. It creates ReplicaSets for application revisions and coordinates updates between them. The ReplicaSet ensures that the required number of matching Pods exists.

![11-deployment-vs-replicaset.png](screenshots/11-deployment-vs-replicaset.png)

### Deployment vs DaemonSet vs StatefulSet

| Feature | Deployment | DaemonSet | StatefulSet |
|---|---|---|---|
| Use cases | Stateless web apps and APIs | Node-level agents and monitoring | Stateful databases and clustered services |
| Pod creation | Creates a chosen number of replicas | Creates Pods on eligible nodes | Creates Pods with stable identities |
| Scaling | Set replica count | Changes as eligible nodes change; can also be constrained | Adjust replica count while respecting StatefulSet behavior |
| Networking | Usually accessed through a Service | Uses ordinary Pod networking; often exposes node-level functionality | Supports stable network identity through a governing Service |
| Storage | Often ephemeral; persistent volumes are also possible | Optional persistent storage | Supports persistent volume claims per Pod |
| Examples | NGINX, frontend API | Logging agent, node exporter | PostgreSQL cluster, ZooKeeper |

![12-workload-comparison.png](screenshots/12-workload-comparison.png)

### ReplicaSet vs Service

| Feature | ReplicaSet | Service |
|---|---|---|
| Responsibility | Maintains the desired number of matching Pods | Provides a stable network endpoint |
| Scaling | Creates or removes Pods to meet replica count | Does not scale Pods |
| Networking | Does not provide a stable client-facing endpoint | Routes traffic to selected ready endpoints |
| Traffic handling | Not a traffic-routing controller | Uses selectors and endpoint information to direct traffic |

**Why is a Service required?**

Pod IP addresses can change when Pods are replaced. A Service provides a stable name and virtual IP for clients to use instead of tracking individual Pod addresses.

### How Traffic Reaches Pods

1. A client sends a request to a Service's DNS name or IP.
2. The Service's virtual IP and networking implementation direct the request toward an eligible endpoint.
3. Traffic reaches a selected Pod.
4. The Pod's container handles the request and returns a response.

![13-replicaset-vs-service.png](screenshots/13-replicaset-vs-service.png)

## Task 3: FQDN

See [`fqdn/README.md`](fqdn/README.md) for the complete explanation of FQDNs, Kubernetes Service DNS, namespace-based naming, Pod-to-Service communication, and examples.

![14-fqdn-dns-test.png](screenshots/14-fqdn-dns-test.png)

## Task 4: CoreDNS

See [`coredns/README.md`](coredns/README.md) for CoreDNS architecture, service discovery, DNS query resolution, configuration, and troubleshooting procedures.

![15-coredns-pods.png](screenshots/15-coredns-pods.png)

![16-coredns-configuration.png](screenshots/16-coredns-configuration.png)

![17-dns-resolution-test.png](screenshots/17-dns-resolution-test.png)

![18-coredns-troubleshooting.png](screenshots/18-coredns-troubleshooting.png)

## Useful Troubleshooting Commands

```powershell
kubectl get services
kubectl get pods -o wide
kubectl get endpointslices
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get pods -n kube-system
kubectl get configmap coredns -n kube-system -o yaml
kubectl logs -n kube-system -l k8s-app=kube-dns
```

The CoreDNS label selector and ConfigMap name are common defaults, but check your cluster if they differ.

## Cleanup

Delete the resources created by the Service exercises:

```powershell
kubectl delete -f services/clusterip.yaml
kubectl delete -f services/nodeport.yaml
kubectl delete -f services/loadbalancer.yaml
kubectl delete -f services/externalname.yaml
kubectl delete -f services/headless.yaml
```

## GitHub Submission

Run these commands from the project root:

```powershell
git status
git add README.md services/ fqdn/ coredns/ screenshots/
git commit -m "Complete Kubernetes networking and services session"
git push
```

## Learning Resources

- [Kubernetes Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- [Service Types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
- [DaemonSets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
- [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
- [Debugging DNS Resolution](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)

## Conclusion

This session demonstrates how Kubernetes Services enable application discovery and networking, how workload controllers manage Pods, and how FQDNs and CoreDNS support communication within the cluster. The YAML examples and command outputs provide practical experience with Kubernetes networking fundamentals.