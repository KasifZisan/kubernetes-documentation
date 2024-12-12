# Kubernetes Dashboard Install and Configure
The Kubernetes Dashboard is a web-based Kubernetes user interface. You can use Kubernetes Dashboard to deploy containerized applications to a Kubernetes cluster, troubleshoot your containerized application, and/or manage other cluster resources.

- Create a new ```monitoring``` namespace within the cluster - 
```bash
kubectl create ns monitoring
```
- Using Helm, install the Kubernetes Dashboard using the publicly available Kubernetes Dashboard Helm Chart. Deploy the dashboard into the monitoring namespace within the lab provided cluster. Use the following command - 
```bash
{
helm repo add k8s-dashboard https://kubernetes.github.io/dashboard
helm repo update
helm install k8s-dashboard --namespace monitoring k8s-dashboard/kubernetes-dashboard --set=protocolHttp=true --set=serviceAccount.create=true --set=serviceAccount.name=k8sdash-serviceaccount --version 3.0.2
}
```
- Establish permissions within the cluster to allow the Kubernetes Dashboard to read and write all cluster resources -
```bash
kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=monitoring:k8sdash-serviceaccount
```
- The Kubernetes Dashboard web interface now needs to be exposed to the Internet so that you can browse to it. To do so, create a new NodePort based Service, and expose the web admin interface on port 30990. Execute the following command - 
```bash
{
kubectl expose deployment k8s-dashboard-kubernetes-dashboard --type=NodePort --name=k8s-dashboard --port=30990 --target-port=9090 -n monitoring
kubectl patch service k8s-dashboard -n monitoring -p '{"spec":{"ports":[{"nodePort": 30990, "port": 30990, "protocol": "TCP", "targetPort": 9090}]}}'
}
```
- Get the public IP address of the Kubernetes cluster that Prometheus has been deployed into (it is in the Flask code). In the terminal execute the following command:
```bash
export | grep K8S_CLUSTER_PUBLICIP
```

