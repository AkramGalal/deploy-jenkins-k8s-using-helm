# Deploy Jenkins on K8s Cluster using Helm
This repository shows the deployment of Jenkins on a Kubernetes cluster using Helm.

## Prerequisites
-   A running Kubernetes cluster.
-   Helm installed on your machine. Follow the official [Helm installation guide](https://helm.sh/docs/intro/install/).

## Deployment Steps
### Step 1: Install Helm
- For Debian-based systems.
  
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

### Step 4: Install Jenkins
- Create a namespace for Jenkins.
  ``` bash
  kubectl create namespace jenkins
  ```

- Choose worker1 node to be the worker node of Jenkins Pods.
  kubectl label nodes worker1 jenkins-node=true
  
- Prepare Persistent Volume for Jenkins controller on the worker node. Jenkins requires a persistent volume to store its workspace and configuration.
- Create the directory and set proper permissions (on worker1):
  ``` bash
  sudo mkdir -p /var/lib/jenkins-controller-data
  sudo chown -R 1000:1000 /var/lib/jenkins-controller-data
  sudo chmod -R 755 /var/lib/jenkins-controller-data
  ```

- Prepare Persistent Volume for Jenkins agent on the worker node. Jenkins agent requires a persistent volume to store its workspace of jobs.
- Create the directory and set proper permissions (on worker1):
  ``` bash
  sudo mkdir -p /var/lib/jenkins-agent-workspaces
  sudo chown -R 1000:1000 /var/lib/jenkins-agent-workspaces
  sudo chmod -R 755 /var/lib/jenkins-agent-workspaces
  ```
  
- Apply the PersistentVolume (PV) of Jenkins controller.
  ``` bash
  kubectl apply -f jenkins-controller-pv.yaml
  ```
  
- Apply the PersistentVolume (PV) of Jenkins agent.
  ``` bash
  kubectl apply -f jenkins-agent-pv.yaml
  ```
  
- Install Jenkins with Helm using updated 'values.yaml' file to apply the created PVs.
  ``` bash
  helm install jenkins jenkins/jenkins -n jenkins -f values.yaml
  ```
  
### Step 5: Access Jenkins UI
- Edit service file to change Jenkins service type from ClusterIP to NodePort.
  ``` bash
  kubectl -n jenkins edit svc jenkins
  ```
- Verify Jenkins deployment status.
  
  <img width="1442" height="311" alt="Screenshot 2025-10-11 233407" src="https://github.com/user-attachments/assets/e25a27f6-5173-4622-a1b6-ce7c47092a46" />

- Locate the Jenkins service and the port number. Access Jenkins in web browser using the node IP address and the port number.
- Retrieve Jenkins admin password generated during the installation to access UI.
  ``` bash
  kubectl get secret jenkins -n jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode; echo
  ```
- Sign in to Jenkins UI using username "admin" and the retrieved password from the secret.

  <img width="3821" height="1017" alt="Screenshot 2025-10-12 001513" src="https://github.com/user-attachments/assets/1824010f-81ff-4cac-a170-96c37e7d08e2" />

## References
- [Official Helm Documentation](https://helm.sh/docs/)
- [Official Jenkins-Kubernetes Documentation](https://www.jenkins.io/doc/book/installing/kubernetes/)
