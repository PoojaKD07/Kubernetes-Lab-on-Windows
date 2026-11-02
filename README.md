# Kubernetes Lab on Windows – NGINX Deployment, Scaling & Self-Healing

## Project Overview

This project demonstrates the implementation of a local three-node Kubernetes cluster on Windows using Docker Desktop and kind (Kubernetes IN Docker).

The lab focuses on deploying an NGINX web application, managing multiple replicas, observing Pod distribution across worker nodes, scaling the application, and testing Kubernetes self-healing capabilities.

The cluster consists of one control-plane node and two worker nodes, with all Kubernetes nodes running as Docker containers.

## Objectives

- Set up a local Kubernetes environment on Windows.
- Create a three-node Kubernetes cluster using kind.
- Understand the role of control-plane and worker nodes.
- Deploy an NGINX application using a Kubernetes Deployment.
- Monitor Pod status and distribution across worker nodes.
- Scale the application from three replicas to six replicas.
- Test Kubernetes self-healing by manually deleting a Pod.
- Understand how Kubernetes maintains the desired application state.

## Technologies Used

| Technology | Purpose |
|---|---|
| Windows 10/11 | Host operating system |
| WSL 2 | Linux environment for Docker Desktop |
| Docker Desktop | Runs Kubernetes node containers |
| Kubernetes | Container orchestration |
| kind | Creates a local Kubernetes cluster |
| kubectl | Communicates with the Kubernetes cluster |
| NGINX | Web server application |
| YAML | Cluster configuration |

## System Architecture

The lab uses a three-node Kubernetes architecture.

```text
                 Windows Laptop
                       |
                PowerShell
                       |
               Docker Desktop
                       |
                 KIND Cluster
                       |
          +------------+------------+
          |                         |
   Control Plane                Worker Nodes
          |                         |
  kind-control-plane       +---------+---------+
                           |                   |
                      kind-worker        kind-worker2
                           |                   |
                       NGINX Pods          NGINX Pods
```

**Control Plane:** Manages the cluster, maintains its state, and coordinates scheduling.

**Worker Nodes:** Execute application workloads by running Kubernetes Pods.

**Pods:** Smallest deployable units in Kubernetes that contain application containers.

## Implementation – Six Lab Stages

### 1. Installation of Required Tools

The following tools were installed and configured:

- WSL 2
- Docker Desktop
- kubectl
- kind

Installation commands:

```powershell
wsl --install
winget install -e --id Kubernetes.kubectl
winget install -e --id Kubernetes.kind
```

Docker Desktop was configured to use the WSL 2 backend.

### 2. Verification of Tools

The installed tools were verified using:

```powershell
docker version
docker run hello-world
kubectl version --client
kind version
```

These commands verified Docker functionality, container execution, kubectl installation, and kind installation.

### 3. Creation of Kubernetes Cluster Configuration

A configuration file named `kind-config.yaml` was created to define the cluster structure.

**Configuration:**

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

This configuration specifies:
- One control-plane node.
- Two worker nodes.

### 4. Creation and Verification of the Cluster

The Kubernetes cluster was created using:

```powershell
kind create cluster --config kind-config.yaml
```

Cluster connectivity was checked using:

```powershell
kubectl cluster-info --context kind-kind
```

Node status was verified using:

```powershell
kubectl get nodes
```

Additional verification:

```powershell
docker ps
kubectl get pods -n kube-system -o wide
```

The cluster contains three nodes:

| Node | Role |
|---|---|
| kind-control-plane | Control Plane |
| kind-worker | Worker |
| kind-worker2 | Worker |

All three nodes were successfully initialized and reached the Ready state.

### 5. Deployment of NGINX Application

An NGINX Deployment named `web` was created with three replicas.

```powershell
kubectl create deployment web --image=nginx --replicas=3
```

Deployment status was checked using:

```powershell
kubectl get deployments
```

Pod information was retrieved using:

```powershell
kubectl get pods -o wide
```

The command displays:
- Pod names
- Readiness status
- Running status
- Restart count
- Pod IP addresses
- Worker-node placement

Individual Pod details were inspected using:

```powershell
kubectl describe pod <actual-pod-name>
```

The actual Pod name must be used without angle brackets.

**Observation:** The three NGINX Pods were successfully scheduled across the two worker nodes.

### 6. Scaling and Self-Healing

#### A. Scaling from 3 to 6 Replicas

The Deployment was scaled to six replicas using:

```powershell
kubectl scale deployment web --replicas=6
```

The Deployment and Pods were verified using:

```powershell
kubectl get deployments
kubectl get pods -o wide
```

**Observation:** Six NGINX Pods reached the Running state, with all six showing `1/1` readiness.

The Pods were distributed across both worker nodes.

#### B. Testing Self-Healing

To test Kubernetes self-healing, one running Pod was manually deleted.

Command used:

```powershell
kubectl delete pod <actual-pod-name>
```

The Pod selected in the experiment was:

```powershell
kubectl delete pod web-5fc9f4bf66-826lz
```

The deletion was confirmed by Kubernetes.

The Deployment controller is responsible for creating a replacement Pod to maintain the desired replica count.

Verification commands:

```powershell
kubectl get pods
kubectl get deployments
```

**Expected result:** Kubernetes creates a replacement Pod with a new name, restoring the Deployment to six ready replicas.

## Results and Observations

| Experiment | Result |
|---|---|
| Kubernetes cluster creation | Successful |
| Control-plane node | Ready |
| Worker nodes | Two Ready nodes |
| Initial NGINX deployment | 3 replicas |
| Initial Pod status | Running |
| Scaling operation | Increased from 3 to 6 |
| Six-replica verification | All 6 Pods Running |
| Pod scheduling | Distributed across two workers |
| Manual Pod deletion | Successfully deleted |
| Self-healing | Replacement expected; final verification screenshot to be added |

### Pod Distribution Observation

During the six-replica experiment, the observed distribution was:

| Worker Node | Number of Pods |
|---|---:|
| kind-worker | 3 |
| kind-worker2 | 3 |
| **Total** | **6** |

The distribution represents the observed execution. Kubernetes scheduling can produce different placements in another run.

## Screenshots and Experimental Evidence

### 1. Initial Pod Creation

The screenshot captures the initial state where Pods were in `ContainerCreating`.

This is a temporary startup state while containers are being initialized.

![Initial Pod Creation](screenshots/01-initial-pod-startup.png)

### 2. Scaling and Pod Distribution

The screenshot demonstrates:
- Scaling the Deployment from three to six replicas.
- Six NGINX Pods reaching `1/1 Running`.
- Pod distribution across two worker nodes.
- Successful manual deletion of one Pod.

![Scaling and Pod Deletion](screenshots/02-scaling-and-pod-deletion.png)

### 3. Self-Healing Verification

After deleting one Pod, execute:

```powershell
kubectl get pods -o wide
kubectl get deployments
```

Capture the output after the replacement Pod is created and the Deployment returns to `6/6` ready replicas.

Save the screenshot as:

`screenshots/03-self-healing-replacement.png`

This provides complete visual evidence of the self-healing experiment.

## Key Kubernetes Concepts Learned

| Concept | Understanding |
|---|---|
| Cluster | Collection of nodes managed by Kubernetes |
| Control Plane | Manages and coordinates the cluster |
| Worker Node | Runs application workloads |
| Pod | Smallest deployable unit in Kubernetes |
| Deployment | Manages application replicas |
| Scheduler | Assigns Pods to suitable nodes |
| Scaling | Changes the number of application replicas |
| Self-Healing | Restores the desired state when a managed Pod is deleted |

## Troubleshooting

| Problem | Solution |
|---|---|
| Access denied while creating a folder | Use Desktop or another writable directory instead of System32 |
| YAML parsing error | Check line breaks, indentation, and remove copied line numbers |
| Docker connection error | Start Docker Desktop and wait for Engine running |
| kubectl or kind not recognized | Reopen PowerShell after installation |
| Pod shows ContainerCreating | Wait for container initialization and check again |
| ImagePullBackOff | Check internet connectivity and inspect Pod Events |
| PowerShell `<` operator error | Replace placeholders with actual names without angle brackets |

## Conclusion

This lab successfully demonstrates the fundamental workflow of Kubernetes container orchestration on a Windows environment.

A three-node Kubernetes cluster was created using kind, and an NGINX application was deployed with three replicas. The application was subsequently scaled to six replicas, and Pod scheduling across worker nodes was observed.

A manual Pod deletion experiment was also performed to study Kubernetes self-healing behavior.

Through this implementation, the lab provides practical understanding of cluster architecture, container orchestration, Deployment management, replica scaling, Pod scheduling, and automated workload recovery.
