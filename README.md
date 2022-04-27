# CKA Preparation
Preparation for Certified Kubernetes Administrator

- [Kubernetes Architecture](#kubernetes-architecture)
    - [Control Plane](#control-plane)
    - [Worker Nodes](#worker-nodes)
- [Concepts](#concepts)
- [Setting up a Cluster](#setting-up-a-cluster)
    - [Initializing the Control Plane](#initializing-the-control-plane)
    - [Pod networking](#pod-networking)
    - [Joining worker nodes](#joining-worker-nodes)
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

## Setting up a Cluster

In order to set up a Kubernetes Cluster locally, we can use tools such as [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/) or [Minikube](https://minikube.sigs.k8s.io/docs/start/). However, the exam requires you to do it the hard way using ```kubeadm```. 

The minimum requirements for a kubernetes machine are:
- One or more machines running a deb/rpm-compatible Linux OS; for my setup i used Ubuntu. 
- 2 GiB or more of RAM per machine--any less leaves little room for your apps.
- At least 2 CPUs on the machine that you use as a control-plane node.
- Full network connectivity among all machines in the cluster. 

### Initializing the Control Plane

To install the necessary tools, i used the following commands:
```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

sudo echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl
```

Then, download the required images for kubeadm:
```bash
sudo kubeadm config images pull
``` 

Start the control plane:
```
sudo kubeadm init
```
After it finishes initializing, a message should appear with instructions on how to configure ```kubectl``` to interact with the cluster and add a worker node into the cluster:
```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a Pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  /docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>

```
### Pod networking
By default, Kubernetes does not come with a sollution to pod networking, so we must choose and deploy a CNI (Container Network Interface) into the cluster so that Pods can communicate with each other. Also, the Cluster DNS service (CoreDNS) will only start if a network is installed.

In this setup, i used [Weave-net](https://www.weave.works/oss/net/):
```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
After the apply, we can verify that weave-net and CoreDNS pods are running using the command:
```
kubectl get pods -n kube-system
```
### Joining worker nodes
To add new nodes into the cluster, do the following steps:
- SSH to the machine
- Become root (e.g. sudo su -)
- Install a runtime if needed
- Run the command that was output by kubeadm init. For example:

```bash
kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>
```

## Managing the cluster

### RBAC

RBAC (Role Based Access Control) is a method of regulating access to Kubernetes resources based on roles. It uses the ```rbac.authorization.k8s.io``` API Group, which has four objects: _Role, ClusterRole, RoleBinding_ and _ClusterRoleBinding_.

#### Role and ClusterRoles
In RBAC, an _Role_ represents a set of rules applied to specified resources in a namespace. _ClusterRoles_ are similar, with the difference that you don't need to specify a namespace, which means you can create a set of rules that applies to the entire cluster.
- It is important to know that there are no _Deny_ rules in RBAC, as Kubernetes follows the ```deny-all``` model. All access is denied by default until you add a expection.

Here's an example of a Role that grants read access to pods:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group (apiVersion: v1)
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

Here's an example of a Cluster that grants read access to nodes (a cluster-scoped resource):
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "watch", "list"]
```