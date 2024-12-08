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

## Rolling Updates and Rollbacks

#### Rollouts

- Rollouts update Deployments
- Any changes to the Deployment template will trigger a rollout

#### Rolling Updates

- Deployments have different rollout strategies but Kubernetes uses Rolling Updates by default.
- Replicas are updated in groups, instead of all-at-once, until the rollout is complete. This allows service to continue **uninterrupted** until the updates rollout.
- But in Rolling Updates both new and old version are running for some time, so it should be handled gracefully.
- The alternative of this is the - **recreate strategy**, where all the running pods are shut down and then deployed again. But this will also increase down time.
- ```kubectl``` has commands to check, pause, resume and rollback (undo) rollouts.

```yaml
spec:
  strategy:
    rollingUpdate:

      # how many replicas over the desired total are allowed during a rollout
      # a higer surge allows new pods to be created than older pods to be deleted
      maxSurge: 25% 

      # max unavailable tells how many old pods can be deleted without new pods being ready
      maxUnavailable: 25%

      # the default is 25% for both

    type: RollingUpdate
```

You can perform rollouts with ```kubectl rollout``` command. For example you can use - ```kubectl rollout -n deployments status deployment app-tier```. Here, ```-n``` is used to mention the namespace, ```status``` is used to show the status of the rollout and ```app-tier``` is the tier name. 

To pause a rollout you can use - ```kubectl rollout -n deployments pause deployment app-tier```. Pausing a rollout won't delete the already created pods but it will stop the creation of any new pods.

To resume a rollout you can use - ```kubectl rollout -n deployments resume deployment app-tier```

#### Rollback
If you want to rollback to the previous version, you can use - ```kubectl rollout -n deployments undo deployment app-tier```. Here the ```undo``` commands is doing the rollback. If you want to rollback to a specific version then you can use the command - ```kubectl rollout history -n deployments deployment app-tier``` and then grab the specific version and pass it into that.

## Probes
Kubernets assumes that the Pods are ready as soon as the containers starts, but that is not always the case. For example, if the container needs some time to warm up, kubernetes should wait before sending any traffic to the pod. It is also possible that the Pod was functional but then it became unresponsive for some reason, maybe it went into a deadlock state, in this case Kubernetes should wait before sending any requests to that Pod.

Kubernetes has solutions to all these issues using **Probes**.

- Readiness Probes - Also referred to as Health Checks
    - Used to check when a Pod is ready to serve traffic/handle requests
    - Useful after startup to chek external dependencies
    - Readiness Probes set the Pod's ready condition. Services only send traffic to ready Pods. Probes integrate with Services to make sure this.
- Liveness Probes - Used to detect if a pod has entered into a broken state
    - Kubernetes will restart the Pod for you. This is the Key difference between Readiness and Liveness probes.
    - Declared in the same way as Readiness probes.

#### Declaring Probes

- Probes can be declared in a Pod's container
- All containers probes must pass for the Pod to pass
- Probe actions can be a command that runs in the container, an HTTP GET request, or opeaning a TCP socket
- By default, a probe checks the Pod every 10 seconds

## Init Containers
Sometimes you need to wait for a service, downloads, dynamic or decisions before starting a Pod's containers. The code that checks this could be grounded in the main application. But it is best practice to seperate initialization wait logic from the container Image. However, Initialization is tightly coupled to the main application (belongs to the Pod). 

So, Kubernetes provides us a way in the form of **Init Containers**. Init Containers allow you to run initialization tasks before starting the main container(s).

- Pods can declare any number of init containers
- Init containers run in order and run to completion
- They can use their own images and this can provide some benefits
- Easy way to block or delay starting an application until some pre-defined conditions are met
- Run every time a Pod is created

## Volumes
Containers in a Pod share their own network stack that each has their own file system. It can be sometimes useful to share data between containers in a  Pod. Lifetime of container file system is limited to the container's lifetime. So this can lead to unexpected consequences if a container restarts.

#### Pod Storage in Kubernetes

- Two high level storage options: Volumes and Persistent Volumes
- Used by mounting directory in one or more containers in a Pod
- Pods can use multiple Volumes and Persistent Volumes
- Difference between Volumes and Persistent Volumes is how their lifetime is managed. One has a lifetime same to the Pod's lifetime the other one is independent to the Pod's lifetime.

#### Volumes

- Volumes are tied to a pod and their lifecycle.
- Share data between containers and tolerate container restarts
- Use for non-durable storage that is deleted with the Pod
- Default Volume type is ```emptyDir```. Any data in the directory remains when the Pod is restarted but the data is deleted if the Pod is deleted
- If pod and the data is stored on a specific node but then the pod is restarted on a different node, then the data will be lost

#### Persistent Volumes

- Independent of Pod's lifetime
- Pods claim persistent volumes to use throughout their lifetime.
- Can be mounted by multiple Pods on different Nodes if underlying storage supports it
- Can be provisioned statically in advance or dynamically on-demand

#### Persistent Volume Claims (PVC)
Pods must send request to a persistent volume to claim it.

Persistent volume claims - 

- Describe a Pod's request for Persistent Volume storage
- Include how much storage, type of storage and access mode. The access mode describes the volume whether it is mounted in read-only, read-write or read-write many
- Access modes can be read-write once, read-only many or read-write many
- PVC stays pending if no PV can satisfy it and dyanmic provisioning is not enabled
- Connects to a Pod through a Volume of type PVC

**Supported Durable Storage:** Amazon EBS (Elastic Block Service)

## ConfigMaps and Secrets
Up until now, the Deployment template ```spec``` has contained all of the configurations required by the Pod. This is better than storing the configuration inside the binary or container image. But adding the configuration in the pod spec can make it a lot less portable. And if the configuration includes sensitive information such as API keys and passwords, it might present a security issue.

Kubernetes provides us with ConfigMaps and Secrets to store the Configurations and Secrets. This results in more portable manifests. But the cluster admin also need to ensure that all the proper encryption and safeguards are in place to make sure that the secrets are actually safe. Secrets have specialized types for storing credentials and TLS certs

#### Using ConfigMaps and Secrets

- ConfigMaps and Secrets store data in key-value pairs
- Pods must reference ConfigMaps and Secrets to use their data
- Pods can use the data by mounting them as files or through a volume or as environment variables

## Kubernetes Ecosystem

#### Helm

- Helm is Kubernetes Package Manager
- Packages are called **charts** and are installed on your cluster using the Helm CLI. We use the Helm CLI to install and udpate charts as it releases on the cluster
- Charts contain all the resources like deployments and services to run the application
- Helm charts make it easy to share complete application

#### Prometheus

- Open-source monitoring and alerting system
- A server for pulling time-series metric data and storing it
- Commonly paired with Grafana for visualizations and Dashboards
- Define alert rules and send notifications
- Easily installed via Helm charts