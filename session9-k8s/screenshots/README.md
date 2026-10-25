# Session 9: Kubernetes Fundamentals

## Overview

This session focuses on Kubernetes fundamentals, including Minikube installation, cluster status verification, Kubernetes architecture, basic Kubernetes objects, and hands-on practice with Kubernetes commands.

## Objectives

1. Install and configure Minikube.
2. Verify the Kubernetes cluster status.
3. Understand Kubernetes architecture and its components.
4. Explore basic Kubernetes objects and commands.
5. Perform hands-on exercises using the Kubernetes Basics tutorial.
6. Capture screenshots and push the completed work to GitHub.

## Tools and Technologies

- Kubernetes
- Minikube
- kubectl
- Docker
- Windows PowerShell
- Git and GitHub

---

## Task 1: Install and Configure Minikube

Minikube is a tool that runs a local Kubernetes cluster, making it useful for learning and development.

### Commands Used

Check the installed versions:

```powershell
minikube version
kubectl version --client
docker --version
```

Start the Kubernetes cluster using Docker:

```powershell
minikube start --driver=docker
```

Verify the installation:

```powershell
minikube status
kubectl cluster-info
```

### Screenshot: Minikube Installation

![Minikube Installation](screenshots/01-minikube-installation.png)

---

## Task 2: Verify Kubernetes Cluster Status

After starting Minikube, verify that the cluster components are running and that Kubernetes resources can be accessed.

### Commands Used

```powershell
minikube status
kubectl cluster-info
kubectl get nodes
kubectl get pods -A
```

### Explanation

- `minikube status` displays the status of Minikube components.
- `kubectl cluster-info` displays information about the Kubernetes control plane and cluster services.
- `kubectl get nodes` lists the available cluster nodes.
- `kubectl get pods -A` lists pods across all namespaces.

### Screenshot: Cluster Status

![Kubernetes Cluster Status](screenshots/02-cluster-status.png)

---

## Task 3: Explore Kubernetes Architecture

Kubernetes is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications.

### 3.1 Control Plane Components

The control plane manages the overall state of the Kubernetes cluster.

| Component | Description |
|---|---|
| API Server | Exposes the Kubernetes API and processes requests. |
| etcd | Stores cluster state and configuration data. |
| Scheduler | Selects a suitable node for newly created pods. |
| Controller Manager | Runs controllers that help maintain the desired cluster state. |

### 3.2 Worker Node Components

Worker nodes run application workloads.

| Component | Description |
|---|---|
| Kubelet | Ensures that assigned pods are running as expected. |
| Container Runtime | Runs containers, such as containerd. |
| Kube-proxy | Provides network forwarding functionality when used in the cluster networking setup. |
| Pods | Smallest deployable Kubernetes units, containing one or more containers. |

### 3.3 How Kubernetes Works

1. A user submits a request through `kubectl`.
2. The API Server receives and processes the request.
3. The desired cluster state is stored in etcd.
4. The Scheduler assigns pods to suitable worker nodes.
5. The Kubelet communicates with the container runtime to run the containers.
6. Controllers continuously work to maintain the desired state.

### Screenshot: Kubernetes Architecture

![Kubernetes Architecture](screenshots/03-kubernetes-architecture.png)

---

## Task 4: Learn Basic Kubernetes Objects and Commands

Kubernetes objects represent the desired state of applications and other resources within a cluster.

### 4.1 Basic Kubernetes Objects

| Object | Purpose |
|---|---|
| Pod | Runs one or more containers. |
| Deployment | Manages replicated pods and application updates. |
| Service | Provides a stable network endpoint for accessing pods. |
| Namespace | Organizes resources within a cluster. |
| ConfigMap | Stores non-sensitive configuration data. |
| Secret | Stores sensitive configuration data; appropriate access controls are still required. |

### 4.2 Common Kubernetes Commands

Check nodes:

```powershell
kubectl get nodes
```

List pods:

```powershell
kubectl get pods
kubectl get pods -A
```

List deployments:

```powershell
kubectl get deployments
```

List services:

```powershell
kubectl get services
```

List namespaces:

```powershell
kubectl get namespaces
```

Inspect a resource:

```powershell
kubectl describe node <node-name>
```

View cluster information:

```powershell
kubectl cluster-info
```

### Screenshot: Basic Kubernetes Objects and Commands

![Basic Kubernetes Objects and Commands](screenshots/04-basic-objects-commands.png)

---

## Task 5: Perform the Kubernetes Basics Tutorial

This practical exercise demonstrates how to create a Deployment, inspect pods, expose an application, scale replicas, and remove resources.

### Step 1: Create a Deployment

```powershell
kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
```

### Step 2: Verify the Deployment and Pods

```powershell
kubectl get deployments
kubectl get pods
kubectl describe deployment hello-minikube
```

### Step 3: Expose the Application

```powershell
kubectl expose deployment hello-minikube --type=NodePort --port=8080
kubectl get services
minikube service hello-minikube
```

The final command opens the application service using Minikube's service access mechanism.

### Step 4: Scale the Deployment

Increase the number of pod replicas to three:

```powershell
kubectl scale deployment hello-minikube --replicas=3
kubectl get deployments
kubectl get pods
```

Scaling allows Kubernetes to maintain multiple instances of the application.

### Step 5: Check Deployment Status

```powershell
kubectl rollout status deployment/hello-minikube
kubectl rollout history deployment/hello-minikube
```

These commands display the rollout status and deployment revision history.

### Step 6: Clean Up Resources

After completing the exercise, remove the Service and Deployment:

```powershell
kubectl delete service hello-minikube
kubectl delete deployment hello-minikube
```

### Screenshot: Kubernetes Basics Tutorial

![Kubernetes Basics Tutorial](screenshots/05-kubernetes-basics-tutorial.png)

---

## Additional Screenshot: Docker Image Pull

The following screenshot documents the Docker image pull activity included in the project.

![Docker Image Pull](screenshots/docker_pull.png)

---

## Important Kubernetes Troubleshooting Commands

Check Minikube status:

```powershell
minikube status
```

View Minikube logs:

```powershell
minikube logs
```

Check all pods:

```powershell
kubectl get pods -A
```

Inspect a pod:

```powershell
kubectl describe pod <pod-name>
```

View container logs:

```powershell
kubectl logs <pod-name>
```

Restart a stopped local cluster:

```powershell
minikube start --driver=docker
```

Replace placeholder values such as `<pod-name>` with the actual names returned by Kubernetes commands.

---

## What I Learned

- Installing and starting a local Kubernetes cluster using Minikube.
- Checking cluster status and verifying Kubernetes connectivity.
- Understanding the responsibilities of control plane and worker node components.
- Exploring Pods, Deployments, Services, and Namespaces.
- Using `kubectl` to inspect and manage cluster resources.
- Creating and scaling a Deployment.
- Exposing an application through a Service.
- Cleaning up Kubernetes resources after completing practical exercises.

---



---

## GitHub Submission

Run the following commands from the root of the repository:

```powershell
git status
git add README.md screenshots/
git commit -m "Add Kubernetes fundamentals homework"
git push
```

If the Git repository has not been connected to GitHub, configure the remote URL and branch before pushing.


---

