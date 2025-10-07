# MLOps CI/CD Workflow Using Jenkins, Argo CD, and Minikube on GCP

## Overview

This is a complete CI/CD workflow for deploying an MLOps project using:

* **Jenkins** – Continuous Integration (CI) tool
* **Argo CD** – Continuous Deployment (CD) tool
* **GCP VM + Minikube** – Kubernetes-based deployment environment

---

## Workflow Summary

| Component             | Purpose                                                        |
| --------------------- | -------------------------------------------------------------- |
| **Jenkins**           | CI pipeline: code checkout, build, test, Docker image creation |
| **Argo CD**           | CD pipeline: continuous synchronization with Kubernetes        |
| **GCP VM + Minikube** | Kubernetes cluster environment for hosting application         |

---

## 1. GCP Setup

1. **Create a GCP account**
2. **Create a bucket** under your project
3. **Upload your CSV file** to this newly created bucket.

---

## 2. Virtual Environment Setup

1. Create a project folder locally and open a terminal inside it.

2. Run the following commands:

   ```bash
   python -m venv venv
   source venv/bin/activate
   ```

3. Create the following folder structure:

   ```
   project/
   ├── venv/
   ├── templates/
   ├── src/__init__.py
   ├── config/__init__.py
   ├── utils/__init__.py
   ├── pipeline/__init__.py
   ├── artifacts/
   ├── static/
   └── setup.py
   ```

4. Add project management code in `setup.py`.

5. Set your Google Cloud credentials:


---

## 3. Pipeline Development

Implement the following Python modules:

* **Data Ingestion**
* **Data Preprocessing**
* **Model Training**
* **Pipeline Stages**

---

## 4. Experiment Tracking with Comet-ML

1. Add `comet-ml` to your `requirements.txt`
2. Install packages:
3. Sign up on [Comet-ML](https://www.comet.com)
4. Create a project (set it to *Public*)
5. Copy your **API Key** from **Profile → API Key**
6. Update your model training script:

   ```python
   import comet_ml
   # Initialize experiment in your model class constructor
   experiment = comet_ml.Experiment(
       api_key="YOUR_API_KEY",
       workspace="YOUR_WORKSPACE",
       project_name="YOUR_PROJECT"
   )
   ```

---

## 5. Kubernetes Manifests

Create a folder called `manifests` with:

* `deployment.yaml`
* `service.yaml`

---

## 6. VM and Minikube Setup

### Create GCP VM

* Type: **e2-standard-4**
* OS: **Ubuntu**
* Disk: **256 GB**
* Allow all network access

### Install Docker

```bash
sudo apt update
sudo apt install git -y
sudo apt-get install ca-certificates curl -y
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

---

### Install Minikube

```bash
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
minikube start
```

### Install kubectl

```bash
curl -LO https://dl.k8s.io/release/v1.34.0/bin/linux/amd64/kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
kubectl get nodes
kubectl cluster-info
```

---

## 7. CI with Jenkins

### Run Jenkins on Docker

```bash
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 \
-v /var/run/docker.sock:/var/run/docker.sock \
-v $(which docker):/usr/bin/docker \
-u root -e DOCKER_GID=$(getent group docker | cut -d: -f3) \
--network minikube jenkins/jenkins:lts
```

* Access Jenkins at `http://<VM-EXTERNAL-IP>:8080`
* Install suggested plugins: **Docker**, **Docker Pipeline**, **Kubernetes**

### Configure Jenkins Environment

```bash
docker exec -it jenkins bash
apt update -y
apt install -y python3 python3-pip python3-venv
ln -s /usr/bin/python3 /usr/bin/python
exit
docker restart jenkins
```

---

## 8. GitHub Integration

1. Generate a **Personal Access Token (Classic)** with:

   * `repo`, `workflow`, `admin:repo_hook`, etc.
2. Add GitHub credentials in Jenkins (`Manage Jenkins → Credentials → Global`)
3. Create a **Pipeline** in Jenkins:

   * Definition: *Pipeline script from SCM*
   * SCM: *Git*
   * Provide Repo URL, Credentials, and Branch.

---

## 9. Docker Hub Integration

1. Create a repository on Docker Hub.

2. Generate a personal access token (read/write/delete permissions).

3. Add credentials in Jenkins.

4. Update environment variables in Jenkinsfile:

5. Match image name in `manifests/deployment.yaml` to:

   ```
   image: username/repo-name:latest
   ```

---

## 10. Continuous Deployment with Argo CD

### Install Argo CD on Minikube

```bash
kubectl create ns argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl get all -n argocd
```

### Expose Argo CD Server

```bash
kubectl edit svc argocd-server -n argocd
# Change type: ClusterIP → NodePort
kubectl get svc -n argocd
kubectl port-forward --address 0.0.0.0 service/argocd-server 31056:80 -n argocd
```

Access the Argo CD UI via:

```
http://<VM-EXTERNAL-IP>:31056
Username: admin
```

Retrieve password:

```bash
kubectl get secret -n argocd argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d
```

---

### Configure Kubeconfig for Jenkins

1. Copy `.kube/config` contents and replace:

   * `certificate-authority`, `client-certificate`, `client-key`
     with their base64-encoded `*-data` values.
2. Save the updated config as `kubeconfig` in your system.
3. Add it to Jenkins as a **Secret File Credential**.

---

### Connect Argo CD to GitHub

In the Argo CD dashboard:

* **Settings → Repositories → Connect Repo (HTTPS)**

  * Username: GitHub username
  * Password: GitHub token

Create a new application:

* Name: `gitopsapp`
* Project: `default`
* Sync Policy: `Automatic`
* Check: *PRUNE RESOURCES*, *SELF HEAL*
* Repo URL: GitHub repository
* Path: `manifests`
* Namespace: `argocd`

---

## 11. Expose the Application

```bash
kubectl get deploy -n argocd
kubectl get pods -n argocd
minikube tunnel
kubectl port-forward svc/mlops-service -n argocd --address 0.0.0.0 9090:80
```

Access your app via:

```
http://<VM-EXTERNAL-IP>:9090
```

---

## 12. Enable Webhooks (CI/CD Automation)

1. **GitHub:**

   * Settings → Webhooks → Add Webhook
   * Payload URL: `http://<VM-EXTERNAL-IP>:8080/github-webhook/`
   * Content type: `application/json`
2. **Jenkins:**

   * Pipeline → Configure → General → Check *GitHub hook trigger for GITScm polling*

---
