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

## Multi-Container Pods
To demonstrate this we are going to take an example - 

- A simple application that increments and prints a count. 
- The application consists **4 containers split across 3 tiers**. 
- The Application tier is a Node.js server container. 
- There is a Data tier that contains a Redis cache. 
- The Support tier contains the Poller (that polls the server with GET requests) and Counter (that sends POST requests). 
- All containers configured using the environment variables. 

#### Kubernetes Namespaces

- We can use namespaces to seperate the resources according to users, environments, or applications.
- Role-based access control (RBAC) to secure access per Namespace.
- Using namespace is the best practice.

**Example Namespace manifest file** -

```yaml
apiVersion: v1
kind: Namespace
metadata:
    name: microservice
    labels:
        app: counter
```
Now create Namepace with ```kubectl create``` command.

**Multi Container Application Manifest**
```yaml
apiVersion: v1
kind: Pod
metadata:
    name: app
spec:
    containers:
        - name: redis
          image: redis:latest
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 6379
        
        - name: server
          image: 
          ports:
            - containerPort: 8080
          env:
            - name: REDIS_URL
              value: redis://localhost:6379
        
        - name: counter
          image:
          env:
            - name: API_URL
              value: http://localhost:8080

        - name: poller
          image:
          env:
            - name: API_URL
              value: http://localhost:8080
```

Now if you want to create this ```multi-container pod``` with the ```microservice``` namespace, run this command -
```bash
kubectl create -f <manifest-name> -n <namespace-name>
``` 

If you want to get the pod, type - ```kubectl get -n <namespace> pod <pod-name>```

## Service Discovery
We will take the example that we did in the last lesson. But in this case - the tiers are not in the same pod, but the three tiers are split into three different pods. So they need to communicate. This is where Service comes in.

**Why Services?**

- Supports multi-Pod design
- Provides static endpoint for each tier
- Handles Pod IP changes
- They also have Load Balancing. They also distribute loads in the group of pods.

To handle this we need to have App tier service in front of the Server tier and Data tier service infront of the Redis tier.

With YAML you can delcare multiple resources in a single manifest by seperating the resources by ```---```. Just like -
```yaml
apiVersion: v1
kind: Service
metadata:
    name: data-tier
    labels:
        app: microservices
spec:
    ports:
    - port: 6379
      protocol: TCP
      name: redis
    selector:
      tier: data
    type:
---
apiVersion: v1
kind: Pod
metadata:
    name: data-tier
    labels:
        app: microservices
        tier: data
spec:
    containers:
        -  name: redis
           ports:
               - containerPort: 6379
    
```

If you want to create a service for a specific tier, you can do - 
```bash 
kubectl describe service -n <service> <tier name>
```

#### Service Discovery Meachnisms

- Environment Variables
    - Services address automatically injected in containers
    - Environment variables follow naming conventions based on service name
- DNS 
    - DNS records automatically created in cluster's DNS
    - Containers automatically configured to query cluster DNS

So in conclusion -

- Environment variables and DNS allow Pods to discover Services
- Environment variables for Services available at Pod creation time and in the same Namespace
- DNS dynamically updated and accross Namespaces.

## Deployments

- Represent multiple replicas of a Pod. Pods in deployment are identical and you can create Pods using the manifest. If a replica is deleted, Kubernetes will automatically create a replica for you
- Describe a desired state the Kubernetes needs to achieve
- Deployment Controller master component converges actual state to the desired state
- Services seamlessly support scaling. Scaling is best with stateless Pods because they support horizontal scaling. This is possible becaues the state of the app is stored in the Data tier.

To get deployments- ```kubectl get -n <namsepace> deployments```

If we want to scale up certain tiers, we can use the ```scale``` command. Here is how we can do this -
```bash
kubectl scale -n <namespace> deployments <tier-name> --replicas=<number-of-replicas>
```

## Autoscaling

- Scale automatically based on CPU utilization (or custom metrics)
- Set target CPU along with minimum and max replicas
- Target CPU is expressed as a percentage of the Pod's CPU request.

#### Metrics

- Autoscaling depends on metrics being collected
- Metrics Server is one solution for collecting metrics
- Several manifest files are used to deploy Metrics server

Autoscaling will need the metrics server to run and collect the metrics. And then autoscaler will increaes or decrease replicas based on the metircs and communicate with the Metrics API.
