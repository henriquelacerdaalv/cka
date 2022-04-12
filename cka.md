# CKA Preparation
Preparation for Certified Kubernetes Administrator

- [Kubernetes Architecture](#kubernetes-architecture)
    - [Control Plane](#control-plane)
    - [Worker Nodes](#worker-nodes)
- [Concepts](#concepts)
## Kubernetes Architecture
Kubernetes is a platform for managing containerized workloads and services. It facilitates operations like load balancing, storage provisioning, application rollback and configuration management. It originated at Google, based on a tool called Borg. 

A Kubernetes cluster consists of a control plane and worker nodes. 

### Control Plane
The control plane takes care of making global decisions about the cluster such as scheduling workloads and respoding to cluster events.

- **API server**:
    The API server exposes a REST API which administrators can use to interact with the control plane using tools like ```kubectl```.
- **Scheduler**:
    Responsible for scheduling pods into the cluster's worker nodes, based on available resources and node statuses.
- **Controller Manager**:
    Makes sure the cluster is in it's desired state, defined in etcd.
- **etcd**: A distributed key-value datastore that Kubernetes uses to store the cluster configuration.

### Worker Nodes

The worker nodes are responsible for running workloads.  
- **Kubelet**: An agent that runs of every worker node, responsible for managing pods and making sure they are running and healthy. 
- **Kube-proxy**: Handles pod networking in the node, allowing network communications from inside and outside of the cluster.
- **Container Runtime**: The software that runs containers inside a pod.

## Concepts
It's important to know a few concepts to better understand how Kubernetes schedules and runs workloads.

- **Pod**: Is the smallest managable unit of Kubernetes. A pod is a group of one or more containers with shared storage and network resources.
- **Controller**: Responsible for interacting with the API Server and orchestrating objects managed by the controller. Examples are: Deployments, ReplicaSets, DaemonSets. 