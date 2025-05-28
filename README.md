# 🔧 Multi-Tier DevOps Project Setup - Java App on EKS


## 📦 Instance Setup

### 🖥️ EKS Management Node

* **Instance Count:** 1
* **OS:** Ubuntu
* **Type:** t2.medium
* **Storage:** 20 GB
* **Ports Allowed:** 22, 80, 443, 500–1000, 1000–11000

#### ✅ Steps:

```bash
sudo apt update
```

* **Install AWS CLI:**

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws configure
```

* **Install Terraform:**

```bash
wget https://releases.hashicorp.com/terraform/<version>/terraform_<version>_linux_amd64.zip
unzip terraform_<version>_linux_amd64.zip
sudo mv terraform /usr/local/bin/
terraform init
terraform plan
terraform apply --auto-approve
```

* **Install kubectl:**

```bash
sudo apt install -y apt-transport-https ca-certificates curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt-add-repository "deb https://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt update
sudo apt install -y kubectl
```

* **Configure kubeconfig:**

```bash
aws eks --region <region> update-kubeconfig --name <cluster-name>
kubectl get nodes
```

* **Install ArgoCD:**

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl get svc -n argocd
kubectl edit svc argocd-server -n argocd # Change type to LoadBalancer
kubectl get svc -n argocd
```

* **Retrieve Initial Admin Password:**

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

---

### 🧰 Jenkins, SonarQube, Nexus (3 Instances)

* **Instance Count:** 3 (1 per tool)
* **OS:** Ubuntu
* **Type:** t2.medium
* **Storage:** 25 GB
* **Security Group:** Same SG across all

---

## 🧪 Jenkins Setup

#### Install:

```bash
sudo apt update
sudo apt install -y docker.io
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

#### Install Trivy:

```bash
sudo apt install wget
wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_0.40.0_Linux-64bit.deb
sudo dpkg -i trivy_0.40.0_Linux-64bit.deb
```

#### Jenkins Plugins:

* SonarQube Scanner
* Stage View
* Pipeline Maven Integration
* Maven Integration
* Config File Provider
* Docker Pipeline

#### Jenkins Configurations:

* **Tools**:

  * Maven (`maven3`) → Automatically install
  * Sonar Scanner (`sonar-scanner`) → Automatically install

* **Credentials (Global)**:

  * GitHub
  * DockerHub
  * SonarQube (type: Secret Text) → `squ_95aa6fa3cf091e972dc2dff23cf50c6ec7999261`

* **System Configuration**:

  * SonarQube Server
  * Managed Files → Global Maven Settings

---

## 🧹 SonarQube Setup

#### Install:

```bash
sudo apt update
sudo apt install -y docker.io
sudo usermod -aG docker $USER
newgrp docker
docker run -d -p 9000:9000 sonarqube:lts-community
```

* Access UI: `http://<instance-ip>:9000`
* Generate Token:
  `Administration → Security → Users`

---

## 📦 Nexus Setup

#### Install:

```bash
sudo apt update
sudo apt install -y docker.io
sudo usermod -aG docker $USER
newgrp docker
docker run -d -p 8081:8081 sonatype/nexus3
```

* Access UI: `http://<instance-ip>:8081`
* Retrieve Password:

```bash
docker exec -it <container-id> /bin/bash
cat /opt/sonatype/sonatype-work/nexus3/admin.password
```

* Username: `admin`

#### Configuration:

* Repositories → Deployment Policy → Set to **Allow Redeploy**

---

## ⚙️ Application Configuration

### Maven `pom.xml`:

* Add distribution section for both **releases** and **snapshots** pointing to Nexus repository URLs.

---

## 📁 Repositories

* CI: [Multi-Tier-BankApp-CI](https://github.com/shani877/Multi-Tier-BankApp-CI.git)
* CD: [Multi-Tier-BankApp-CD](https://github.com/shani877/Multi-Tier-BankApp-CD.git)
* Infra: [eks-cluster](https://github.com/shani877/eks-cluster.git)
* Tools Install: [scripts](https://github.com/shani877/scripts.git)

---

## 🎥 Demo

Watch full deployment demo on YouTube:
[![Demo Video](https://img.youtube.com/vi/xAjledixxTE/0.jpg)](https://youtu.be/xAjledixxTE?si=wBgBK-NJ6MjQELMQ)

---
