# Deploy Jenkins on k8s Cluster using Helm
This repository shows the deployment of Jenkins on a Kubernetes cluster using Helm.

## Deployment Steps
### Step 1: Install Helm
- For Debian-based systems
  ``` bash
  sudo apt-get install curl gpg apt-transport-https --yes
  curl -fsSL https://packages.buildkite.com/helm-linux/helm-debian/gpgkey | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
  echo "deb [signed-by=/usr/share/keyrings/helm.gpg] https://packages.buildkite.com/helm-linux/helm-debian/any/ any main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
  sudo apt-get update
  sudo apt-get install helm
  helm version
  ```

### Step 2: Add the Jenkins Helm Repository
``` bash
helm repo add jenkins https://charts.jenkins.io
```

### Step 3: Update the Helm Repositories
``` bash
helm repo update
```

### Step 4: Install Jenkins and Customize admin password
- Create namespace
  ``` bash
  kubectl create namespace jenkins
  ```
- Install Jenkins into that namespace
  ``` bash
  helm install jenkins jenkins/jenkins -n jenkins
  ```

### Step 5: Access Jenkins UI
- Locate the Jenkins service and note the port number. Use this port to access Jenkins in web browser using the cluster IP address or node IP address.
  ``` bash
  kubectl get svc --namespace default jenkins
  ```

