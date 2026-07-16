# 🚀 Three-Tier MERN App on Kubernetes (Kind Cluster) with StatefulSets & Persistent Storage

A beginner-friendly, step-by-step guide to deploying a **Frontend + Backend + MongoDB** application on Kubernetes — simulating how a real DevOps engineer sets up a production-style app in a startup.

> 📌 Even if you're new to Kubernetes, follow this README top-to-bottom and you'll have the full app running.

---

## 🎯 What This Project Does

We deploy a **3-tier application**:

| Layer | Technology | Kubernetes Object |
|---|---|---|
| Frontend | React | Deployment |
| Backend | Node.js + Express | Deployment |
| Database | MongoDB | **StatefulSet** |

We also set up:
- **Namespaces** — to keep frontend, backend, and database organized separately.
- **Persistent Storage** — so MongoDB data is NOT lost when a pod restarts.
- **Resource Quotas & Limit Ranges** — so no single app eats up all the server resources.
- **Health Checks (Probes)** — so Kubernetes automatically restarts broken pods.

---

## 🧠 First, Understand These Concepts (in Simple Words)

| Term | Simple Meaning |
|---|---|
| **StorageClass** | A "template" that tells Kubernetes what type of storage to use (disk, cloud, etc.) |
| **PersistentVolume (PV)** | The actual storage space on the machine |
| **PersistentVolumeClaim (PVC)** | A "request" made by an app asking for storage |
| **Deployment** | Used for apps that don't store data themselves (frontend, backend) |
| **StatefulSet** | Used for apps that need to remember data & keep the same identity (like MongoDB) |
| **Headless Service** | Gives each database pod its own fixed address (like `mongo-0`, `mongo-1`) |
| **ReplicaSet** | A group of database servers holding the same data, so if one dies, another takes over |

### 🔑 Why does MongoDB need extra steps (StatefulSet + Init Job)?

Think of it like starting a car:

1. **StatefulSet** = installing the engine (MongoDB server starts with storage, ready to be part of a team).
2. **Init Job** = turning the key (it actually creates/activates the replica set using `rs.initiate()`).

Without step 2, MongoDB is running but NOT properly configured — some features like transactions won't work.

---

## 📁 Project Structure

```
three-tier-k8s/
├── namespaces/
│   ├── frontend.yaml
│   ├── backend.yaml
│   └── database.yaml
├── resource-quotas/
├── limit-ranges/
├── frontend/
│   ├── deployment.yaml
│   └── service.yaml
├── backend/
│   ├── deployment.yaml
│   └── service.yaml
├── database/
│   ├── statefulset.yaml
│   ├── service.yaml
│   └── mongo-init-job.yaml
├── storage/
│   ├── storageclass.yaml
│   ├── mongo-pv.yaml
│   └── mongo-pvc.yaml      # optional, only needed if using Deployment (not StatefulSet)
└── kind-config.yaml
```

---

## 🛠️ Step 1 — Set Up Your Server (AWS EC2)

We'll use an EC2 instance as our base machine.

1. Launch an EC2 instance (Ubuntu recommended).
2. SSH into it.
3. Install Docker:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
newgrp docker
docker --version
```

---

## 🛠️ Step 2 — Install `kubectl` and `kind`

```bash
# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl
kubectl version --client

# Install kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
kind version
```

---

## 🛠️ Step 3 — Prepare Storage Folder on EC2

Before creating the cluster, make a folder on your EC2 machine where MongoDB data will actually live:

```bash
sudo mkdir -p /mnt/kind-storage/mongo
sudo chmod 777 /mnt/kind-storage/mongo
```

---

## 🛠️ Step 4 — Create the Kind Cluster

Save this as `kind-config.yaml`:

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
      - containerPort: 30007   # Frontend NodePort
        hostPort: 30007
        protocol: TCP
  - role: worker
    image: kindest/node:v1.28.0
  - role: worker
    image: kindest/node:v1.28.0
```

Create the cluster:

```bash
kind create cluster --name three-tier-cluster --config kind-config.yaml
```

---

## 🛠️ Step 5 — Create Namespaces

We keep frontend, backend, and database separate so things stay organized (like separate rooms in a house).

```bash
kubectl apply -f namespaces/namespaces.yaml
```

---

## 🛠️ Step 6 — Set Up Storage (StorageClass + PV)

```bash
kubectl apply -f storage/storageclass.yaml
kubectl apply -f storage/mongo-pv.yaml
```

> ℹ️ You do **not** need `mongo-pvc.yaml` if you're using a StatefulSet — the StatefulSet creates its own PVC automatically via `volumeClaimTemplates`.

---

## 🛠️ Step 7 — Create MongoDB Secret

Kubernetes Secrets keep your DB username/password safe (not written in plain YAML):

```bash
kubectl create secret generic mongo-secret \
  --namespace=database \
  --from-literal=MONGO_INITDB_ROOT_USERNAME=admin \
  --from-literal=MONGO_INITDB_ROOT_PASSWORD=securepass
```

---

## 🛠️ Step 8 — Deploy MongoDB (Headless Service + StatefulSet)

```bash
kubectl apply -f database/service.yaml
kubectl apply -f database/statefulset.yaml
```

Wait until the pod is running:

```bash
kubectl get pods -n database -w
```

You should see a pod named `mongo-0` (not a random name — that's the "stable identity" a StatefulSet gives you).

---

## 🛠️ Step 9 — Run the Init Job (Turn the Key 🔑)

This actually activates the MongoDB replica set:

```bash
kubectl apply -f database/mongo-init-job.yaml
kubectl logs job/mongo-init -n database
```

You should see: `✅ Replica set initiated!`

> 🔧 **If you ever need to change the Init Job:** Jobs are meant to run once, so Kubernetes won't let you edit them directly. Instead:
> ```bash
> kubectl delete job mongo-init -n database
> kubectl apply -f database/mongo-init-job.yaml
> ```
> or in one command:
> ```bash
> kubectl replace --force -f database/mongo-init-job.yaml
> ```

---

## 🛠️ Step 10 — Deploy Backend

```bash
kubectl apply -f backend/deployment.yaml
kubectl apply -f backend/service.yaml
```

The backend connects to MongoDB using this connection string (stored in `.env` / Secret):

```
MONGO_URI=mongodb://admin:securepass@mongo-service.database.svc.cluster.local:27017/?authSource=admin&replicaSet=rs0
```

---

## 🛠️ Step 11 — Deploy Frontend

```bash
kubectl apply -f frontend/deployment.yaml
kubectl apply -f frontend/service.yaml
```

---

## ✅ Step 12 — Access the App

Since we mapped port `30007` in our Kind config, open your browser at:

```
http://<EC2-Public-IP>:30007
```

---

## 🔍 Useful Commands (Debugging & Maintenance)

**Check what's inside MongoDB's storage folder:**
```bash
kubectl exec -it mongo-0 -n database -- bash
ls -la /data/db
exit
```
- Empty folder (`.` and `..` only) → Mongo will initialize fresh.
- Files like `WiredTiger`, `mongod.lock`, `journal/` present → Mongo already has data, and it will skip creating a new root user.

**Completely reset the database (careful — deletes everything!):**
```bash
kubectl delete statefulset mongo -n database
kubectl delete pvc -n database --all
```
Then on EC2:
```bash
sudo rm -rf /mnt/kind-storage/mongo/*
```

---

## 📌 Key Takeaways

- ✅ Frontend & Backend → **Deployments** (stateless, easily scalable).
- ✅ MongoDB → **StatefulSet** (needs identity + persistent storage).
- ✅ Headless Service gives MongoDB a stable, predictable DNS name.
- ✅ PVC + StatefulSet together make sure your data survives pod restarts.
- ✅ Init Job is what actually activates the MongoDB replica set — without it, Mongo runs but isn't fully configured.

---

## 🏁 Conclusion

This project simulates a real startup DevOps workflow: provisioning infrastructure, deploying a multi-tier app, organizing it with namespaces, and making sure the database survives crashes and restarts using Kubernetes' native storage tools. It's a solid foundation you can later extend with CI/CD, monitoring, and moving from Kind → EKS for real production use.

---

**Best regards,**
**nasirbloch323**
