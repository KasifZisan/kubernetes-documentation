# Minikube and Kubectl Install

## Kubectl
- First check if ```kubectl``` is installed - ```kubectl version --client```. If it is not installed, then we have to install ```kubectl```.
- Download kubectl - ```curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"```
- Validate the binary
    - Download the kubectl checksum file - ```curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"```
    - Validate the kubectl binary against the checksum file - ``` echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check```
    - ``` 
        if [ $? -eq 0 ]; then
              echo "Step 4: Checksum is valid."
         else
              echo "Step 4: Checksum validation failed. Exiting..."
              exit 1
         fi
      ```
- Install kubectl - ```sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl```

## Minikube
- First check if ```minikube``` is installed - ```minikube version```
- Download Minikube - ```curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64```
- Install Minikube - ```sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64```
- Check if minikube is running - ```minikube status --format '{{.Host}}' | grep "Running"```

#### Starting a Minikube Cluster
If you want to start a new cluster named ```test-cluster``` with 2 nodes (one control-plane and one worker node) -
```bash
minikube start --nodes 2 -p test-cluster --driver=docker
```

To verify if ```kubectl``` has ```kubernetes``` access - ```kubectl get nodes```

Get minikube cluster info - ```kubectl cluster-info```