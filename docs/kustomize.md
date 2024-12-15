# Kustomize

## Working with Kustomize

#### Deploy Base/Development
You can have two directories - **base** and **overlay**. The base directory should contain the deployment, service, configMap, ingress and kustomization file for the **baseline** or **development** environment. They overlay directory can contain two other directories which are - **staging** and **production**.

The ```kustomization.yaml``` file can look like this - 
```yaml
commonLabels:
 app: webapp
 env: base
 version: "1.02"
 org: cloudacademy.com
 team: devops.labs
 developer: jeremy.cook

resources:
- configmap.yaml
- deployment.yaml
- service.yaml
- ingress.yaml 
```
The ```commonLabels``` section defines common metadata labels which get copied into each of the 4 manifest files declared in the ```resources``` section when Kustomize is executed.

Confirm that you are in the ```kustomize``` directory. Use ```Kustomize``` to generate and output the set of API resources as declared within the ```kustomization.yaml``` file within the ```base``` directory. In the terminal execute the following command:
```bash
kubectl kustomize base
```

Now use Kustomize to deploy the same baseline set of resources into the provided Kubernetes cluster. In the terminal execute the following command:
```bash
kubectl apply -k base
```

Confirm that all cluster resources were created successfully. In the terminal execute the following command:
```bash
kubectl get all
```

The sample web application should now be ready to serve Internet based traffic via its assigned FQDN host declared within the base/ingress.yaml file. 

#### Deploy Staging 
The ``staging`` directory might contain a ```kustomization.yaml``` file like this - 
```yaml
namePrefix: stg-

commonLabels:
  env: staging

commonAnnotations:
  note: staging deployment of cloudacademy lab webapp

bases:
- ../../base

patchesStrategicMerge:
- configmap.yaml
- ingress.yaml
```

- ```namePrefix``` defines a string that is added to the start of all resource names
- ```commonLabels``` defines metadata labels that are added to all resources
- ```commonAnnotations``` defines metadata annotations that are added to all resources
- ```bases``` defines the base directory which is to be merge patched with the resources declared in the ```patchesStrategicMerge``` section

Use Kustomize to generate and output the set of API resources as declared within the kustomization.yaml file within the staging (current) directory. In the terminal execute the following command:
```bash
kubectl kustomize .
```
Now use Kustomize to deploy the staging generated set of resources into the Kubernetes cluster. In the terminal execute the following command:
```bash
kubectl apply -k .
```
Confirm that all staging cluster resources were created successfully. In the terminal execute the following command:
```bash
kubectl get all -l env=staging
```

#### Deploy Production
You can deploy production the same way you have deployed Staging.