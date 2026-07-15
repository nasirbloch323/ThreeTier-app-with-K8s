# 🚀 Secure Deployment of Three-Tier MERN Architecture on Kubernetes

A production-style deployment of a **MERN (MongoDB, Express, React, Node.js)** application on a **Kubernetes cluster (KIND)**, using isolated namespaces, Kubernetes Secrets, and separate Deployments/Services for each tier — built as a real-world DevOps use case for a fintech-style SaaS platform.

---

## 📌 Project Overview

This project demonstrates how a DevOps Engineer sets up, secures, and deploys a three-tier application on Kubernetes:

- **Frontend** → React app (Stateless)
- **Backend** → Node.js/Express API (Stateless)
- **Database** → MongoDB Atlas (Stateful, external managed DB)

Each tier runs in its **own Kubernetes namespace** for isolation, organization, and security — following industry best practices used by real startups scaling from proof-of-concept to production.

---

## 🏗 Architecture

```
                 ┌─────────────────────┐
                 │      Frontend        │
                 │  (Namespace: frontend)│
                 │  React App - NodePort │
                 └──────────┬───────────┘
                             │
                 ┌───────────▼───────────┐
                 │       Backend          │
                 │ (Namespace: backend)   │
                 │ Node.js API - ClusterIP│
                 └──────────┬───────────┘
                             │
                 ┌───────────▼───────────┐
                 │      MongoDB Atlas     │
                 │ (External Managed DB)  │
                 └────────────────────────┘
```

---

## 📁 Project Structure

```
three-tier-app/
├── k8s-manifests/
│   ├── namespaces/
│   │   ├── frontend.yaml
│   │   ├── backend.yaml
│   │   └── database.yaml
│   │
│   ├── frontend/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   │
│   ├── backend/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   │
│   └── database/
│       └── mongo-secret.yaml
│
├── frontend/                 # React source code
├── backend/                  # Node.js source code
│
└── kind-cluster/
    └── kind-config.yaml       # KIND cluster config
```

---

## 🛠 Tech Stack

- **Frontend:** React.js
- **Backend:** Node.js, Express.js
- **Database:** MongoDB Atlas
- **Container Orchestration:** Kubernetes (KIND)
- **Containerization:** Docker
- **Cloud:** AWS EC2 (Ubuntu 22.04)
- **Security:** Kubernetes Secrets, Namespace isolation

---

## ⚙️ Step-by-Step Setup

### 1️⃣ Launch EC2 Instance
- AMI: Ubuntu 22.04 LTS
- Instance type: `t2.medium` (2 vCPU, 4GB RAM)
- Security Group: allow ports `22`, `80`, `443`, `3000`, `30007`

```bash
ssh -i your-key.pem ubuntu@your-ec2-public-ip
```

### 2️⃣ Install Docker

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
newgrp docker
docker --version
```

### 3️⃣ Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl
kubectl version --client
```

### 4️⃣ Install KIND

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
kind version
```

### 5️⃣ Create KIND Cluster

`kind-config.yaml`:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
    image: kindest/node:v1.28.0
    extraPortMappings:
      - containerPort: 80
        hostPort: 80
        protocol: TCP
      - containerPort: 443
        hostPort: 443
        protocol: TCP
      - containerPort: 30007
        hostPort: 30007
        protocol: TCP
  - role: worker
    image: kindest/node:v1.28.0
  - role: worker
    image: kindest/node:v1.28.0
```

```bash
kind create cluster --config kind-config.yaml --name three-tier-cluster
kubectl cluster-info --context kind-three-tier-cluster
kubectl get nodes
```


```

### 7️⃣ Create Namespaces

```bash
kubectl create namespace frontend
kubectl create namespace backend
kubectl create namespace database
```

### 8️⃣ Create MongoDB Secret

```bash
kubectl -n backend create secret generic mongo-secret --from-env-file=.env
kubectl -n backend get secret mongo-secret -o yaml
```

### 9️⃣ Apply Kubernetes Manifests

```bash
kubectl apply -f k8s-manifests --recursive
```

### 🔎 Verify Deployment

```bash
kubectl get pods -n frontend
kubectl get pods -n backend
kubectl get svc -n frontend
kubectl get svc -n backend
```

### 🌐 Access the Application

```bash
kubectl port-forward svc/frontend-service 3000:80 -n frontend --address 0.0.0.0
```

Open in browser:
```
http://<EC2_PUBLIC_IP>:3000
```

---

## 🔐 Security Practices Used

- Database credentials stored in **Kubernetes Secrets** (not hardcoded)
- **Namespace isolation** between frontend, backend, and database
- **ClusterIP** used for backend (internal-only access)
- **NodePort** used only where external access is needed (frontend)

---

## 📊 Stateless vs Stateful

| Component | Type | Reason |
|-----------|------|--------|
| Frontend | Stateless | No local data, horizontally scalable |
| Backend | Stateless | No local data, horizontally scalable |
| Database | Stateful | Persistent storage, externalized to MongoDB Atlas |

---

## ✅ Key Learnings

- Deploying a multi-tier app with proper namespace isolation
- Managing sensitive data using Kubernetes Secrets
- Difference between ClusterIP and NodePort services
- Setting up a local Kubernetes cluster using KIND for PoC before scaling to EKS
- Real-world DevOps workflow for a fintech-style SaaS platform

---

## 👤 Author

**nasir** — DevOps Engineer  
📎 Project inspired by real-world startup infrastructure practices

---

## 📄 License

This project is for educational purposes as part of a DevOps learning path (Miseacademy).
