# Introduction to Kubernetes

## Kubernetes Overview
Open-source conainer orchestration tool designed to automate deploying, scaling, and operating containerized applications.

- Kubernetes is a distriubted system. Multiple machines (physical or virtual) maybe used to build a cluster. 
- Kubernetes places containers on machine based on schedules and availability.
- Kubernetes can move containers as machines are added/removed.
- Can use different container runtimes. It is container runtime agnostic.
- Modular, extensive design.

#### Using Kubernetes

- Declarative Configuration
- Deploy Containers fast
- Configure networking
- Scale and Expose services

#### Kubernetes for Operations
Kubernetes offers multiple features for Operations people.

- Automatically recover failed machines
- Built-in Support for Machine Maintenance
- Join Clusters with Federation

#### Kubernetes Feature Highlights
- Automated deployment rollout and rollback
- Seamless horizontal scaling
- Secrets Management
- Service discovery and load balancing
- Simple log collection
- Stateful application support
- Persistent volume management

**AWS's alternative of Kubernets:** Amazon ECS (Elastic Container Service). This can be managed by your EC2 or by AWS Fargate.

## Deploying Kubernetes

#### Single-Node Kubernetes Clusters
Some options to deploy Kubernetes is - 

- Docker (includes a port for running single-node docker clusters)
- Minikube 
- Kubeadm (only for Linux machines, will start kubernetes on the host machine, instead of a virtual machine like the prior two methods)

#### Single-Node Kubernets Clusters for Continuous Integration

- Create ephemeral clusters that start quickly and are in a clean state.
- Kubernetes-in-Docker (KinD) is made for this case.

#### Multi-Node Kubernetes Clusters

- For production workloads
- Horizontal scaling
- Tolerate Node Failures

## Kubernetes Architecture

- The top of the Kubernetes architecture is a **Cluster**. Cluster referes to all of the machines collectively and can be thought of as the entire running system.
- Nodes are the machines in the cluster. Nodes can be categorized as **workers** or **masters**.
- Worker nodes include software to run containers managed by the Kubernetes control plane.
- Master nodes run the control plane.
- The control plane is a set of APIs and software that Kubernetes users interact with. The APIs and software are referred to as master components.

#### Scheduling
Control plane schedules containers onto nodes. Scheduling decisions consider required CPU and other factors. Scheduling here refers to the decision process of placing containers onto node.

#### Kubernetes Pods

- Pods are group of Containers. Pods may include one or more containers. All containers in a pod run on the same node. 
- Pods are the smallest building block of Kubernetes.
- More complex and useful abstractions built on top of Pods.

#### Kubernetes services
- Services define networking rules for exposing groups of Pods - To other pods, To the cluster and To the internet

#### Kubernetes Deployments
- Manage deploying configuration changes to running Pods. They control rollout and rollback of Pods.
- Horizontal scaling.

## Interacting with Kubernetes

- **The Kubernetes API Server:** Modify cluster state information by sending requests to the Kubernetes API server. The API server is a master componentt that acts as the frontend of the server. It is possible but not common to work directly with the API server.
- **Client Libraries:** Client libraries can handle the tediousness of working with the REST API for you. It handles authenticating and managing individual REST API requests and responses.
- **Kubernetes Command-Line Tool (Kubectl):** This is by far the most common tool to interact with Kubernetes. With this, you can issue high-level commands that are translated into REST API calls. You can also acccess clusters that are both local and remote. Kubectl also manages all kinds of resources, and provides debugging and introspection features.

#### Example Kubectl commands

- ```kubectl create``` to create resources (Pods, Services etc.). We can either specify what we want to create in the commands or in the files. The files are usually in ```.yaml``` format and are called manifests.
- ```kubectl delete``` to delete resources
- ```kubectl get``` to get a list of resources of a given type. For example - ```kubectl get pods``` gets all the pods in the current namespace
- ```kubectl describe``` to get print a detailed info about a resource(s). For example - ```kubectl describe <resource-type> <resource-name>``` 
- ```kubectl logs``` to print container logs

## Pods

- Basic building blocks in Kubernetes
- Contain one or more containers
- Pod containers all share a container network
- One IP address per pod

#### What's in a Pod Declaration
- Container Image
- Container Ports
- Container Restart Policy
- Resource Limits

#### Manifest Files
All kinds of Pod configurations are written in a manifest file, typicall written in ```.yaml```. Manifests can describe all kinds of resources. 

- ```apiVersion``` tells kubernetes which api version is going to be used 
- ```kind``` tells what kind of resource is going to be used
- ```metadata``` provides relevant information abou the resource. For example - name of the resource
- ```spec``` contains resource specific properties


#### Manifests in Action
Manifests are send to kubernetes api server to take actions on. ```kubectl create``` sends manifest to Kubernetes API Server.

API server does the following for Pod manifests -

- Select a node with sufficient resources
- Schedule Pod onto a node
- Node pulls Pod's container image
- Start Pod's container

## Services

- A service defines networking rules for accessing Pods in the cluster and from the internet. You can use labels to select a group of pods.
- Service has a fixed IP address.
- Distribute requests across Pods in the group.

#### Example Manifest
```yaml
apiVersion: v1
kind: Service
metadata:
    labels:
        app: webserver
    name: webserver
spec:
    ports:
    - port: 80
    selector:
        app: webserver
    type: NodePort
```

## Deployments

- Represent multiple replicas of a Pod. If a replica is deleted, Kubernetes will automatically create a replica for you
- Describe a desired state the Kubernetes needs to achieve
- Deployment Controller master component converges actual state to the desired state.
