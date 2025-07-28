# 🚀 Full DevOps Workflow: Code Push → Jenkins Build → Ansible Deploy → Docker Container → Kubernetes Service

This project demonstrates a complete **CI/CD pipeline** from code commit to deployment on a Kubernetes cluster using **manually installed tools** — Jenkins, Ansible, Docker, and Kubernetes — across AWS EC2 instances.

---

## 🧰 Tech Stack

- 🐳 Docker (Manually installed)
- ☸️ Kubernetes (Kubeadm-based cluster)
- 🛠️ Jenkins (Installed on EC2 without container)
- 📦 Ansible (Installed on same host as Jenkins)
- 🐍 Flask (Sample Python web application)
- 🌐 GitHub (Source code repository)
- ☁️ AWS EC2 (Infrastructure platform)

---

## 🗺️ Project Architecture

```text
GitHub Repo (Code Push)
        |
        v
Jenkins CI Server (Builds Docker Image)
        |
        v
Ansible Playbook (Deploys to Kubernetes)
        |
        v
Docker Image Deployed in K8s Cluster
        |
        v
Kubernetes Service (Exposes the Flask App)
```

---

## ⚙️ Project Components

### 📁 Application Directory Structure

```
flask-app/
├── app.py
├── Dockerfile
├── requirements.txt
├── deployment.yaml
├── deploy.yml          # Ansible Playbook
```

---

## 🔨 Step-by-Step Setup Guide

### 1️⃣ Infrastructure Setup

Provision 3 EC2 Instances:
- `jenkins-master`: Jenkins + Ansible
- `k8s-master`: K8s Control Plane + Docker
- `k8s-worker`: K8s Node + Docker

Enable ports: `22`, `80`, `443`, `8080`, `6443`

---

### 2️⃣ Jenkins Setup (Manual)

```bash
# Java
sudo apt update && sudo apt install -y openjdk-11-jdk

# Jenkins
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update && sudo apt install -y jenkins
sudo systemctl start jenkins && sudo systemctl enable jenkins
```

---

### 3️⃣ Ansible Setup

```bash
sudo apt install -y ansible
```

**Inventory File (`/etc/ansible/hosts`):**
```ini
[k8s-nodes]
k8s-worker ansible_host=<WORKER_IP> ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/key.pem
```

---

### 4️⃣ Docker Installation on K8s Worker

```bash
sudo apt update && sudo apt install -y docker.io
sudo usermod -aG docker $USER
sudo systemctl enable docker && sudo systemctl start docker
```

---

### 5️⃣ Kubernetes Setup

Install `kubeadm`, `kubectl`, and `kubelet` on both master and worker.  
Initialize master and join the worker.

```bash
# Master node
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
# Configure kubeconfig
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
# Apply CNI
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

---

## 🔧 CI/CD Pipeline Using Jenkins

### 🧪 Jenkinsfile

```groovy
pipeline {
    agent any
    stages {
        stage('Clone') {
            steps {
                git 'https://github.com/yourusername/flask-app.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t flask-app .'
                sh 'docker tag flask-app himanshu8090/flask-app:v1'
                sh 'docker push himanshu8090/flask-app:v1'
            }
        }
        stage('Deploy to K8s') {
            steps {
                sh 'ansible-playbook deploy.yml'
            }
        }
    }
}
```

---

## 📜 Ansible Playbook: `deploy.yml`

```yaml
- name: Deploy Flask App on Kubernetes
  hosts: k8s-nodes
  become: yes
  tasks:
    - name: Copy deployment YAML
      copy:
        src: deployment.yaml
        dest: /tmp/deployment.yaml

    - name: Apply Deployment
      shell: "kubectl apply -f /tmp/deployment.yaml"
```

---

## 📦 Dockerfile

```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
EXPOSE 5000
CMD ["python", "app.py"]
```

---

## ☸️ Kubernetes Manifest: `deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask
  template:
    metadata:
      labels:
        app: flask
    spec:
      containers:
        - name: flask-container
          image: himanshu8090/flask-app:v1
          ports:
            - containerPort: 5000
---
apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  type: NodePort
  selector:
    app: flask
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
```

---

## 🌐 Accessing the Application

Once deployed, run:
```bash
kubectl get svc flask-service
```

Open in browser:  
`http://<NodePublicIP>:<NodePort>`

---

## 👨‍💻 Author

**Himanshu Kumar Singh**  
B.Tech CSE | DevOps & Cloud Enthusiast  
🔗 [LinkedIn](https://www.linkedin.com/in/himanshukrsingh0)  
📦 [DockerHub](https://hub.docker.com/u/himanshu8090)

---

## 🧠 What You’ll Learn

- Manual tool installation for CI/CD pipelines
- Jenkins job automation
- Ansible deployment scripting
- Docker + Kubernetes integration
- Real-world DevOps pipeline from scratch

---

## 📌 License

This project is licensed under the MIT License.
