# Deploy Jenkins on K8s Cluster using Helm

This repository demonstrates how to deploy Jenkins on a Kubernetes cluster using Helm and configure it to use Persistent Volumes on a specific worker node.

## Prerequisites
- A running Kubernetes cluster.
- Helm installed on your machine. Follow the official [Helm installation guide](https://helm.sh/docs/intro/install/).

---

## Deployment Steps

### Step 1: Install Helm
- For Debian-based systems:
  ``` bash
  sudo apt-get install curl gpg apt-transport-https --yes
  curl -fsSL https://packages.buildkite.com/helm-linux/helm-debian/gpgkey | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
  echo "deb [signed-by=/usr/share/keyrings/helm.gpg] https://packages.buildkite.com/helm-linux/helm-debian/any/ any main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
  sudo apt-get update
  sudo apt-get install helm
  helm version
  ```

---

### Step 2: Add the Jenkins Helm Repository
``` bash
helm repo add jenkins https://charts.jenkins.io
helm repo update
```

---

### Step 3: Create Namespace and Label Worker Node

- Create a dedicated namespace for Jenkins: 
  ```bash
  kubectl create namespace jenkins
  ```
  
- Label the desired worker node (e.g., `worker1`) so that Jenkins controller and agent pods can be scheduled there:
  ```bash
  kubectl label nodes worker1 jenkins-node=true
  ```
- This label will be referenced in the PV `nodeAffinity` and the Helm `values.yaml` file.

---

### Step 4: Prepare Persistent Volumes

#### Jenkins Controller Volume
- The Jenkins controller requires a persistent volume to store configuration and Jenkins home data.
- On `worker1`, create the directory and assign proper permissions:
  ```bash
  sudo mkdir -p /var/lib/jenkins-controller-data
  sudo chown -R 1000:1000 /var/lib/jenkins-controller-data
  sudo chmod -R 755 /var/lib/jenkins-controller-data
  ```
  
#### Jenkins Agent Volume
- The Jenkins agent requires a separate persistent volume to store job workspaces.
- On `worker1`, create the directory and set proper permissions:
  ```bash
  sudo mkdir -p /var/lib/jenkins-agent-workspaces
  sudo chown -R 1000:1000 /var/lib/jenkins-agent-workspaces
  sudo chmod -R 755 /var/lib/jenkins-agent-workspaces
  ```
- Apply both PersistentVolume definitions:
  ```bash
  kubectl apply -f jenkins-pv.yaml
  kubectl apply -f jenkins-agent-pv.yaml
  ```
- Verify they are created:
  ```bash
  kubectl get pv
  ```
---

### Step 5: Create PersistentVolumeClaims (PVC)
- Create PVCs for both controller and agent:
  ```bash
  kubectl apply -f jenkins-pvc.yaml
  kubectl apply -f jenkins-agent-pvc.yaml
  ```
- Check their binding status:
  ```bash
  kubectl get pvc -n jenkins
  ```
- You should see both PVCs in `Bound` state.

---

### Step 6: Deploy Jenkins with Helm
- Install Jenkins using Helm and the customized `values.yaml` file that references the created PVCs:
  ```bash
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
- Access Jenkins UI using username "admin" and the retrieved password from the secret.

  <img width="3821" height="1017" alt="Screenshot 2025-10-12 001513" src="https://github.com/user-attachments/assets/1824010f-81ff-4cac-a170-96c37e7d08e2" />

## References
- [Helm Documentation](https://helm.sh/docs/)
- [Jenkins on Kubernetes](https://www.jenkins.io/doc/book/installing/kubernetes/)
- [Jenkins Helm Chart Documentation](https://artifacthub.io/packages/helm/jenkinsci/jenkins)
