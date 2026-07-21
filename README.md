# 🚀 Three-Tier MERN Application on Kubernetes (Kind Cluster) with Ingress Controller

A production-ready, scalable, and cost-efficient three-tier MERN stack application deployed on a **Kubernetes Kind cluster** running on **AWS EC2**.

---

## 📋 Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         🌐 Internet                              │
└──────────────────────┬────────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────────┐
│              🛡️ Ingress Controller (NGINX)                       │
│         Routes: / → Frontend  |  /api → Backend                 │
└──────────────────────┬────────────────────────────────────────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│  🎨 Frontend │ │  ⚙️ Backend   │ │  🗄️ Database  │
│   (React)    │ │ (Node.js API) │ │   (MongoDB)   │
│  Namespace   │ │  Namespace    │ │   Namespace   │
│  2 Replicas  │ │  2 Replicas   │ │  StatefulSet  │
│     HPA      │ │     HPA       │ │     PVC       │
└──────────────┘ └──────────────┘ └──────────────┘
```

---

## 🎯 Features

| Feature | Description |
|---------|-------------|
| **🏗️ Three-Tier Architecture** | Frontend (React), Backend (Node.js + Express), Database (MongoDB) |
| **📦 Namespaces** | Isolated namespaces for frontend, backend, and database |
| **💾 Persistent Storage** | StatefulSet with PVC for MongoDB data durability |
| **📈 Auto Scaling** | HPA (Horizontal) + VPA (Vertical) for all tiers |
| **⚖️ Resource Management** | ResourceQuotas and LimitRanges per namespace |
| **🔒 Security** | Secrets, ConfigMaps, and RBAC policies |
| **🌐 Ingress Routing** | Single entry point with NGINX Ingress Controller |
| **🔍 Health Checks** | Liveness and Readiness probes on all pods |
| **☁️ Cloud Ready** | Easily migratable from Kind to EKS/GKE/AKS |

---

## 📁 Project Structure

```
three-tier-k8s/
├── README.md
├── kind-config.yaml
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── public/
│   │   └── index.html
│   └── src/
│       ├── App.js
│       ├── index.js
│       └── App.css
└── k8s-manifests/
    ├── namespaces/
    │   ├── frontend.yaml
    │   ├── backend.yaml
    │   └── database.yaml
    ├── storage/
    │   ├── storageclass.yaml
    │   ├── mongo-pv.yaml
    │   └── mongo-pvc.yaml
    ├── database/
    │   ├── secret.yaml
    │   ├── statefulset.yaml
    │   └── service.yaml
    ├── backend/
    │   ├── configmap.yaml
    │   ├── deployment.yaml
    │   └── service.yaml
    ├── frontend/
    │   ├── configmap.yaml
    │   ├── deployment.yaml
    │   └── service.yaml
    ├── ingress/
    │   └── ingress.yaml
    ├── autoscaling/
    │   ├── frontend-hpa.yaml
    │   ├── frontend-vpa.yaml
    │   ├── backend-hpa.yaml
    │   ├── backend-vpa.yaml
    │   ├── database-hpa.yaml
    │   └── database-vpa.yaml
    ├── resource-quotas/
    │   ├── frontend-quota.yaml
    │   ├── backend-quota.yaml
    │   └── database-quota.yaml
    └── limit-ranges/
        ├── frontend-limits.yaml
        ├── backend-limits.yaml
        └── database-limits.yaml
```

---

## 🚀 Quick Start

### Prerequisites

- AWS EC2 Instance (Ubuntu 22.04+, t3.large or higher recommended)
- Docker installed
- Kind installed
- kubectl installed

### 1️⃣ Install Docker & Kind on EC2

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
newgrp docker
docker --version

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client

# Install Kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/
kind version
```

### 2️⃣ Prepare Storage Directory

```bash
sudo mkdir -p /mnt/kind-storage/mongo
sudo chmod 777 /mnt/kind-storage/mongo
```

### 3️⃣ Create Kind Cluster

```bash
kind create cluster --name three-tier-cluster --config kind-config.yaml
```

### 4️⃣ Install NGINX Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

# Wait for controller
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```

### 5️⃣ Deploy Application (Follow Order!)

```bash
# 1. Namespaces
kubectl apply -f k8s-manifests/namespaces/

# 2. Storage
kubectl apply -f k8s-manifests/storage/

# 3. Database (StatefulSet)
kubectl apply -f k8s-manifests/database/

# 4. Backend
kubectl apply -f k8s-manifests/backend/

# 5. Frontend
kubectl apply -f k8s-manifests/frontend/

# 6. Ingress
kubectl apply -f k8s-manifests/ingress/

# 7. Autoscaling (HPA + VPA)
kubectl apply -f k8s-manifests/autoscaling/

# 8. Resource Quotas & Limits
kubectl apply -f k8s-manifests/resource-quotas/
kubectl apply -f k8s-manifests/limit-ranges/
```

### 6️⃣ Initialize MongoDB Replica Set

```bash
# Exec into MongoDB pod
kubectl exec -it mongo-0 -n database -- mongosh

# Inside mongosh, run:
rs.initiate({
  _id: "rs0",
  members: [{_id: 0, host: "mongo-0.mongo-service.database.svc.cluster.local:27017"}]
})

# Check status
rs.status()
# Look for: "stateStr": "PRIMARY"
```

### 7️⃣ Access Your Application

```bash
# Get EC2 Public IP
EC2_IP=$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)
echo "Access your app at: http://${EC2_IP}.nip.io/"
echo "API at: http://${EC2_IP}.nip.io/api/items"
```

---

## 🔧 Useful Commands

### Check All Resources
```bash
kubectl get all --all-namespaces
```

### Check Pods Status
```bash
kubectl get pods -n frontend
kubectl get pods -n backend
kubectl get pods -n database
```

### Check Logs
```bash
kubectl logs -f deployment/frontend-deployment -n frontend
kubectl logs -f deployment/backend-deployment -n backend
kubectl logs -f statefulset/mongo -n database
```

### Port Forward for Local Testing
```bash
# Frontend
kubectl port-forward svc/frontend-service 3000:80 -n frontend

# Backend
kubectl port-forward svc/backend-service 5000:5000 -n backend

# MongoDB
kubectl port-forward svc/mongo-service 27017:27017 -n database
```

### Scale Manually
```bash
kubectl scale deployment frontend-deployment --replicas=5 -n frontend
kubectl scale deployment backend-deployment --replicas=5 -n backend
```

### Check HPA Status
```bash
kubectl get hpa -n frontend
kubectl get hpa -n backend
kubectl get hpa -n database
```

### Patch Ingress Controller to NodePort (if needed)
```bash
kubectl patch svc ingress-nginx-controller -n ingress-nginx -p '{"spec":{"type":"NodePort"}}'
```

---

## 🌐 Ingress Routing

| Path | Service | Description |
|------|---------|-------------|
| `/` | frontend-service | React Frontend |
| `/api` | backend-service | Node.js API |

---

## 📊 Resource Configuration

### Resource Quotas per Namespace

| Namespace | CPU Request | Memory Request | CPU Limit | Memory Limit |
|-----------|-------------|----------------|-----------|--------------|
| frontend | 1 CPU | 1 Gi | 2 CPU | 2 Gi |
| backend | 1 CPU | 1 Gi | 2 CPU | 2 Gi |
| database | 500m CPU | 512 Mi | 1 CPU | 1 Gi |

### HPA Configuration

| Tier | Min Replicas | Max Replicas | Target CPU |
|------|--------------|--------------|------------|
| Frontend | 2 | 5 | 70% |
| Backend | 2 | 5 | 70% |
| Database | 1 | 3 | 60% |

---

## 🔒 Security Best Practices

- ✅ **Namespaces** isolate frontend, backend, and database
- ✅ **Secrets** store MongoDB credentials securely
- ✅ **ConfigMaps** manage non-sensitive configuration
- ✅ **ResourceQuotas** prevent resource hogging
- ✅ **LimitRanges** enforce default resource limits
- ✅ **Network Policies** can be added for inter-namespace communication control

---

## 🚀 Migration to Production (EKS)

This architecture is designed to be **migration-ready**:

1. **Replace Kind with EKS** → Use `eksctl` or AWS Console
2. **Replace hostPath PV** → Use EBS CSI Driver or EFS
3. **Replace NodePort** → Use AWS Load Balancer Controller
4. **Add TLS** → Use cert-manager with Let's Encrypt
5. **Add Monitoring** → Deploy Prometheus + Grafana
6. **Add Logging** → Deploy Fluentd + Elasticsearch + Kibana

---

## 🧹 Cleanup

```bash
# Delete application
kubectl delete namespace frontend backend database

# Delete cluster
kind delete cluster --name three-tier-cluster

# Clean storage
sudo rm -rf /mnt/kind-storage/mongo
```

---

## 📚 References

- [Kind Documentation](https://kind.sigs.k8s.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [MongoDB Kubernetes Operator](https://github.com/mongodb/mongodb-kubernetes-operator)

---

## 📄 License

MIT License - Free for personal and commercial use.

---

## 👨‍💻 Author

**DevOps Engineer** | Building scalable, cloud-native applications 🚀
