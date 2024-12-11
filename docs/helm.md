# Helm

## Introduction
Helm is the package manager for Kubernetes, used to simplify and enhance the deployment experience for deploying resources into a Kubernetes cluster. 

But one might be asking why do we need to use ```Helm``` if we already have ```kubectl```? The truth is for complex and intricate deployments we need Helm to simplify and enhance the deployment experience.

For example, while using ```kubectl``` we have to ```apply``` the deployment, service, and configMap seperately for each of the tiers. This is not only labor intensive but also prone to error. Depedencies can be forgotten or the installation sequence might be forgotten. Also working with manifests are problematic. There is no parameterization and there is no way to interact with the appilcation lifecycle. All of these creates manifest proliferation. Helm can solve this.

Helm has a concept of Charts which is essentially a package containing all the related parts of a specific cluster environment. Helm can be used to deploy applications that consist many resources with a single command. Helm takes care of deploying the chart and all its individual resources in a sequenced order. 

## Benefits
Helm can literally be deployed with a single command. You can also deploy in different environments with different input values that help you customize the overall behaviour of the application. Helm can help you upgrade existing relases, and also keeps track of the release history for you which allows you to rollback to previous release if required. 

Helm makes it easier to package and distribute charts. After creating the chart you can simply package it up with a chart name. The result is an archive file or chart package which you can host in a chart repository. 

## Termninology
**Chart**

Chart is the package that has a specific file structure, contains many files and templates which when rendered make up the resources that you deploy into your cluster.

**Repository**

Repository is an HTTP server that hosts and serves ```index.yaml``` file together with one or several packaged charts. 

**Template**

Templates are used within the chart package that make deployment more general and reusable. By taking a kubernetes manifest and abstracting it into a template. You can also parameterize it, such that while chart installation time, it can take in values that alter the behaviour of the deployed resources. 

This is useful for deployments that need to take place in multiple environments. Templating your resources helps you reduce and minimize manifest proliferation. 

**Releases**

When you deploy a chart into a cluster, Helm creates a release for it. A release represent an instance of a chart. The same chart can be used to deploy differently named releases. Releases can be upgraded, rollback and even deleted. 

A release maintains a history of change where every change being its own revision. 

**Architecture**

Helm 3 has a client-only architecture. It uses Kubernetes' RBAC - Roll-Based Access Control system to install resources into Pods. Helm directloy communicates with the kubernetes API server and render the templates. Helm 3 also stores release information using Kubernetes Secrets. Whenever a new release happens, a new Kubernetes Secret resource is being created within the same Kubernetes namespace in which the actual deployment took place. 

## Commands

```bash
helm search hub [KEYWORD] #uses fuzzy based searching for publicly registered charts
helm search repo [KEYWORD] #search for local repos, you can add additional repos for search when - help repo add
helm pull [CHART]
helm install [NAME] [CHART]
helm upgrade [RELEASE] [CHART]
helm rollback [RELEASE] [REVISION]
helm uinstall [RELEASE]
```

#### Helm Repository Management
```bash
helm repo add [NAME] [URL]
heml repo list
helm repo remove [NAME]
helm repo update # updates the latest information about charts from the respective chart repositories
helm repo index [DIR] # scan the current directory and generate an index file
```

#### Helm Release Management
```bash
helm status [RELEASE]
helm list
helm history [RELEASE]
helm get manifest [RELEASE]
```

#### Helm Chart Management
```bash
helm create [NAME]
helm template [NAME] [CHART]
helm package [CHART]
helm lint [CHART]
```
## Charts
By using the command ```helm create``` you can get a file structure of the chart. 

First of all we have the ```Charts.yaml``` file where we have top level information of the Chart. 
```yaml
apiVersion: 
name: 
description:
type:
version:
appVersion:
```
It must contain a ```name``` and a optional ```description``` about the chart. The optional ```type``` can be either ```application``` or ```library```. If the type is ```application``` then the chart becomes deployable meaning resources will actually be created in the cluster. A ```library``` type means the chart contains reusable functions that can be used in other deployed application charts. The mandatory ```version``` field contains the version of the chart itself. The optional ```appVersion``` field tracks the version of the application deployed.  

The ```charts``` folder stores other charts that the current chart is dependent on. During deployed, charts contains in the ```charts``` directory are deployed into the cluster. 

The ```values.yaml``` file contains a structured list of default values -
```yaml
replicaCount: 1

image:
    repository:
    pullPolicy:

serviceAccount:
    create: true
    annotations: {}
    name:

service:
    type:
    port:
```
These values go to the various templates that reference them like this - ```{{ .Values.service.type }}```

If you want to override the value stored in the ```values.yaml``` file then you can use the ```install``` or ```upgrade``` command. For example you can use this command - ```helm upgrade ... \ --set=services.port=9090```

The ```templates``` folder is used to hold all the templates together. Here each template files contains the defenition of a simple cluster resource. A ```NOTES.txt``` file can be put into the ```templates``` folder that contains end-user instructions on how the deployed application should be accessed once deployed and running within the cluster. 

The ```_helpers.tpl``` contains template partials. Sometimes while creating your own template you can use the same thing multiple times in the same template or differnt template. All of these partials can be stored in a file with an ```_``` infront so that it is understood that this is not directly a template. 

The ```tests``` file are written to contain the tests that you wrote to exercies once the templates are generated. Tests are implemented typically using a Kubernetes Job. If the tests exit with a zero exit code then it means that the test is successful and if it exists with a non-zero exit code then the test is not successful. Tests are run by the subocmmand - ```helm test [RELEASE]```. These tests can be used to tests traffic paths, network connections, credentials and/or other specific tasks.

After finishing the development of Chart, the next step is to package it using - ```helm package [CHART DIR]```. After packaging you can install the chart using - ```helm install```. If needed you can also do a dry run of a chart installation ```helm install ... --dry-run```.

To host a newly created chart into a chart repository, you simply need to create a index chart file using - ```helm repo index .``` And then store both the generated ```index.yaml``` file together with the chart archive in a directory which is then served up by a standard web server. 

#### Running a Hosted Chart
```bash
helm repo add local http://127.0.0.1:8000
helm repo update
helm search repo cloudcademy

helm install ca-demo1 local/cloudcademyapp
```
## Templates


