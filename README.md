# 🚀 Three-Tier MERN App on Kubernetes (Kind Cluster) with HPA & VPA

> **Simple, Student-Friendly Guide** | Deploy a full MERN stack on Kubernetes with auto-scaling — perfect for learning DevOps!

---

## 📖 What is this Project?

This project teaches you how to deploy a **3-tier MERN application** (MongoDB + Express + React + Node.js) on a **Kubernetes Kind cluster** running on **AWS EC2**.

You will learn:
- ✅ How to set up a local Kubernetes cluster using **Kind**
- ✅ How to deploy **frontend**, **backend**, and **database** separately
- ✅ How to use **HPA** (Horizontal Pod Autoscaler) to handle traffic spikes
- ✅ How to use **VPA** (Vertical Pod Autoscaler) to right-size resources
- ✅ How to keep **MongoDB data safe** using Persistent Storage

---

## 🎯 Real-World Example

Imagine a **fintech startup** (like early PayPal or Stripe) that needs:
- 🟢 **99.9% uptime** — customers can't afford downtime
- 🟢 **Handle traffic spikes** — salary day = 10x more users
- 🟢 **Save money** — startup budget is tight
- 🟢 **Keep financial data safe** — data must never be lost

This project shows exactly how DevOps engineers solve these problems!

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                        AWS EC2 Instance                     │
│  ┌─────────────────────────────────────────────────────┐  │
│  │              Kubernetes (Kind Cluster)                │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │  │
│  │  │  Frontend   │  │   Backend   │  │  Database   │  │  │
│  │  │  (React)    │  │  (Node.js)  │  │  (MongoDB)  │  │  │
│  │  │  Stateless  │  │  Stateless  │  │   Stateful  │  │  │
│  │  │  HPA ✅     │  │  HPA+VPA ✅ │  │   VPA ✅    │  │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  │  │
│  │                                                      │  │
│  │  ┌──────────────────────────────────────────────┐   │  │
│  │  │     Persistent Volume (MongoDB Data)        │   │  │
│  │  │     /mnt/kind-storage/mongo                  │   │  │
│  │  └──────────────────────────────────────────────┘   │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Quick Rule of Thumb:
| Layer | Type | Autoscaler | Why? |
|-------|------|-----------|------|
| **Frontend** | Stateless | **HPA** | More users = more pods |
| **Backend** | Stateless | **HPA + VPA** | Scale pods + right-size CPU/RAM |
| **Database** | Stateful | **VPA only** | Can't just add random pods |

---

## 📁 Project Structure

```
three-tier-k8s/
├── k8s-manifests/
│   ├── namespaces/           # Separate spaces for each layer
│   │   ├── frontend.yaml
│   │   ├── backend.yaml
│   │   └── database.yaml
│   │
│   ├── storage/              # Where MongoDB data lives forever
│   │   ├── storageclass.yaml
│   │   ├── mongo-pv.yaml
│   │   └── mongo-pvc.yaml
│   │
│   ├── database/             # MongoDB (StatefulSet)
│   │   ├── statefulset.yaml
│   │   ├── service.yaml
│   │   └── mongo-init-job.yaml
│   │
│   ├── backend/              # Node.js API (Deployment)
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   │
│   ├── frontend/             # React App (Deployment)
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   │
│   ├── autoscaling/          # HPA & VPA configs
│   │   ├── frontend-hpa.yaml
│   │   ├── frontend-vpa.yaml
│   │   ├── backend-hpa.yaml
│   │   ├── backend-vpa.yaml
│   │   └── database-vpa.yaml
│   │
│   ├── resource-quotas/      # Prevent one app from eating all resources
│   │   ├── frontend-quota.yaml
│   │   ├── backend-quota.yaml
│   │   └── database-quota.yaml
│   │
│   └── limit-ranges/         # Default CPU/RAM limits per pod
│       ├── frontend-limits.yaml
│       ├── backend-limits.yaml
│       └── database-limits.yaml
│
└── README.md                 # You are here! 📍
```

---

## 🛠️ Prerequisites

Before starting, you need:

1. **AWS EC2 Instance** (Ubuntu 22.04, t3.medium or bigger)
2. **Docker** installed
3. **Kind** (Kubernetes in Docker) installed
4. **kubectl** installed

---

## ⚙️ Step 1: Setup EC2 + Install Docker

SSH into your EC2 instance and run:

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
newgrp docker

# Check Docker
docker --version
```

---

## ⚙️ Step 2: Install kubectl & Kind

```bash
# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl
kubectl version --client

# Install Kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
kind version
```

---

## ⚙️ Step 3: Create Storage Directory on EC2

MongoDB needs a folder on the host machine to save data permanently:

```bash
sudo mkdir -p /mnt/kind-storage/mongo
sudo chmod 777 /mnt/kind-storage/mongo
```

---

## ⚙️ Step 4: Create Kind Cluster

Create a file called `config.yml`:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  # Control Plane (brain of the cluster)
  - role: control-plane
    image: kindest/node:v1.28.0
    extraPortMappings:
      - containerPort: 80      # HTTP
        hostPort: 80
        protocol: TCP
      - containerPort: 443     # HTTPS
        hostPort: 443
        protocol: TCP
      - containerPort: 30007   # NodePort for frontend
        hostPort: 30007
        protocol: TCP
    extraMounts:
      - hostPath: /mnt/kind-storage/mongo
        containerPath: /data/mongo

  # Worker Node 1
  - role: worker
    image: kindest/node:v1.28.0
    extraMounts:
      - hostPath: /mnt/kind-storage/mongo
        containerPath: /data/mongo

  # Worker Node 2
  - role: worker
    image: kindest/node:v1.28.0
    extraMounts:
      - hostPath: /mnt/kind-storage/mongo
        containerPath: /data/mongo
```

Now create the cluster:

```bash
kind create cluster --name three-tier-cluster --config config.yml
```

---

## ⚙️ Step 5: Deploy Everything (Follow This Order!)

> ⚠️ **Important:** Apply files in this exact order!

### 1️⃣ Namespaces (Create separate rooms for each app)

```yaml
# namespaces/frontend.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: frontend

---
# namespaces/backend.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: backend

---
# namespaces/database.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: database
```

```bash
kubectl apply -f k8s-manifests/namespaces/
```

---

### 2️⃣ Storage (So MongoDB data survives pod restarts)

```yaml
# storage/storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer

---
# storage/mongo-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongo-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/data/mongo"
  storageClassName: standard-storage

---
# storage/mongo-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-pvc
  namespace: database
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: standard-storage
```

```bash
kubectl apply -f k8s-manifests/storage/
```

---

### 3️⃣ Secrets (Hide passwords safely)

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
  namespace: database
type: Opaque
stringData:
  MONGO_INITDB_ROOT_USERNAME: admin
  MONGO_INITDB_ROOT_PASSWORD: securepass
```

```bash
kubectl apply -f secret.yaml
```

---

### 4️⃣ MongoDB (StatefulSet for persistent identity)

```yaml
# database/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo-service
  namespace: database
spec:
  clusterIP: None  # Headless service for stable DNS
  selector:
    app: mongo
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017

---
# database/statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongo
  namespace: database
spec:
  serviceName: mongo-service
  replicas: 1
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
        - name: mongo
          image: mongo:6
          command: ["mongod"]
          args: ["--replSet", "rs0", "--bind_ip_all"]
          ports:
            - containerPort: 27017
          env:
            - name: MONGO_INITDB_ROOT_USERNAME
              valueFrom:
                secretKeyRef:
                  name: mongo-secret
                  key: MONGO_INITDB_ROOT_USERNAME
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mongo-secret
                  key: MONGO_INITDB_ROOT_PASSWORD
          volumeMounts:
            - name: mongo-storage
              mountPath: /data/db
          resources:
            requests:
              cpu: "500m"
              memory: "512Mi"
            limits:
              cpu: "1"
              memory: "1Gi"
      volumes:
        - name: mongo-storage
          persistentVolumeClaim:
            claimName: mongo-pvc
```

```bash
kubectl apply -f k8s-manifests/database/
```

---

### 5️⃣ MongoDB Init Job (Initialize replica set)

```yaml
# database/mongo-init-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: mongo-init
  namespace: database
spec:
  backoffLimit: 5
  template:
    spec:
      restartPolicy: OnFailure
      containers:
        - name: mongo-init
          image: mongo:6
          command:
            - sh
            - -c
            - |
              echo "⏳ Waiting for MongoDB to be ready..."
              until mongosh --host mongo-0.mongo-service.database.svc.cluster.local:27017 --eval "db.adminCommand('ping')" > /dev/null 2>&1; do
                echo "MongoDB not ready, retrying in 3s..."
                sleep 3
              done

              echo "🚀 Initiating replica set..."
              mongosh --host mongo-0.mongo-service.database.svc.cluster.local:27017                 -u "$MONGO_INITDB_ROOT_USERNAME"                 -p "$MONGO_INITDB_ROOT_PASSWORD"                 --authenticationDatabase admin                 --eval 'rs.initiate({
                  _id: "rs0",
                  members: [{ _id: 0, host: "mongo-0.mongo-service.database.svc.cluster.local:27017" }]
                })'

              echo "✅ MongoDB initialization complete!"
          env:
            - name: MONGO_INITDB_ROOT_USERNAME
              valueFrom:
                secretKeyRef:
                  name: mongo-secret
                  key: MONGO_INITDB_ROOT_USERNAME
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mongo-secret
                  key: MONGO_INITDB_ROOT_PASSWORD
```

```bash
kubectl apply -f k8s-manifests/database/mongo-init-job.yaml
```

---

### 6️⃣ Backend (Node.js API)

```yaml
# backend/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
  namespace: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: misacademy/backend:v3
          ports:
            - containerPort: 5000
          resources:
            requests:
              cpu: "200m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
          livenessProbe:
            httpGet:
              path: /api/items
              port: 5000
            initialDelaySeconds: 15
            periodSeconds: 20
            timeoutSeconds: 5
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /api/items
              port: 5000
            initialDelaySeconds: 10
            periodSeconds: 15
            timeoutSeconds: 5
            failureThreshold: 3
          envFrom:
            - secretRef:
                name: mongo-secret

---
# backend/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: backend
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
```

```bash
kubectl apply -f k8s-manifests/backend/
```

---

### 7️⃣ Frontend (React App)

```yaml
# frontend/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  namespace: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: misacademy/frontend:v3
          ports:
            - containerPort: 3000
          resources:
            requests:
              cpu: "200m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
          livenessProbe:
            httpGet:
              path: /
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 15
          readinessProbe:
            httpGet:
              path: /
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 10

---
# frontend/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  namespace: frontend
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
      nodePort: 30007
```

```bash
kubectl apply -f k8s-manifests/frontend/
```

---

## 📈 Step 6: Setup Autoscaling (HPA & VPA)

### What is HPA? 🤔
**HPA** = **Horizontal Pod Autoscaler**
- Adds more pods when CPU is high
- Removes pods when CPU is low
- Think: "More workers in a restaurant when it's busy"

### What is VPA? 🤔
**VPA** = **Vertical Pod Autoscaler**
- Gives each pod more CPU/RAM if needed
- Takes away resources if pod is idle
- Think: "Giving a worker a bigger desk when they need it"

---

### 🟢 Frontend HPA

```yaml
# autoscaling/frontend-hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-hpa
  namespace: frontend
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: frontend-deployment
  minReplicas: 2
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 30
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 25
          periodSeconds: 60
```

---

### 🟢 Backend HPA

```yaml
# autoscaling/backend-hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
  namespace: backend
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend-deployment
  minReplicas: 2
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 30
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 25
          periodSeconds: 60
```

---

### 🟢 Backend VPA

```yaml
# autoscaling/backend-vpa.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: backend-vpa
  namespace: backend
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: backend-deployment
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
      - containerName: backend
        minAllowed:
          cpu: "200m"
          memory: "256Mi"
        maxAllowed:
          cpu: "2"
          memory: "2Gi"
        controlledResources: ["cpu", "memory"]
        controlledValues: "RequestsAndLimits"
```

---

### 🟢 Database VPA (Stateful workloads need VPA, NOT HPA!)

```yaml
# autoscaling/database-vpa.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: database-vpa
  namespace: database
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: StatefulSet
    name: mongo
  updatePolicy:
    updateMode: "Initial"  # Safer for databases
  resourcePolicy:
    containerPolicies:
      - containerName: mongo
        minAllowed:
          cpu: "500m"
          memory: "512Mi"
        maxAllowed:
          cpu: "4"
          memory: "4Gi"
        controlledResources: ["cpu", "memory"]
        controlledValues: "RequestsAndLimits"
```

> ⚠️ **Important:** We use `updateMode: "Initial"` for the database so it only sets resources when the pod starts — no mid-life restarts that could crash MongoDB!

Apply all autoscaling configs:

```bash
kubectl apply -f k8s-manifests/autoscaling/
```

---

## 🔍 Step 7: Check Everything is Working

### Check all pods are running:
```bash
kubectl get pods --all-namespaces
```

### Check HPA status:
```bash
kubectl get hpa --all-namespaces
kubectl describe hpa frontend-hpa -n frontend
```

### Check VPA status:
```bash
kubectl get vpa --all-namespaces
kubectl describe vpa backend-vpa -n backend
```

### Check MongoDB is ready:
```bash
kubectl exec -it mongo-0 -n database -- mongosh
```

Inside mongosh, run:
```javascript
rs.status()  // Should show "stateStr": "PRIMARY"
```

### Check services:
```bash
kubectl get svc --all-namespaces
```

### Access the frontend:
Open your browser and go to:
```
http://<YOUR_EC2_PUBLIC_IP>:30007
```

---

## 🧪 Step 8: Test Autoscaling (Load Test)

Install a load testing tool:
```bash
sudo apt install -y apache2-utils
```

Run a load test on the backend:
```bash
ab -n 10000 -c 100 http://<YOUR_EC2_PUBLIC_IP>:30007/api/items
```

Watch HPA scale up in real-time:
```bash
watch kubectl get hpa -n backend
```

You should see the backend pods increase from 2 to 5!

---

## 🧹 Cleanup (When You're Done)

```bash
# Delete the entire cluster
kind delete cluster --name three-tier-cluster

# Or delete everything manually
kubectl delete -f k8s-manifests/autoscaling/
kubectl delete -f k8s-manifests/frontend/
kubectl delete -f k8s-manifests/backend/
kubectl delete -f k8s-manifests/database/
kubectl delete -f k8s-manifests/storage/
kubectl delete -f k8s-manifests/namespaces/

# Clean host storage
sudo rm -rf /mnt/kind-storage/mongo/*
```

---

## 🐛 Common Errors & Fixes

### ❌ Error: "MongoDB not initializing"
**Fix:**
```bash
# Check if data already exists
kubectl exec -it mongo-0 -n database -- ls -la /data/db

# If files exist, delete PVC and recreate
kubectl delete pvc mongo-pvc -n database
kubectl delete pv mongo-pv
sudo rm -rf /mnt/kind-storage/mongo/*
kubectl apply -f k8s-manifests/storage/
kubectl apply -f k8s-manifests/database/
```

### ❌ Error: "HPA not scaling"
**Fix:**
```bash
# Check metrics server is running
kubectl get pods -n kube-system | grep metrics-server

# If not running, install it:
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

### ❌ Error: "VPA not working"
**Fix:**
```bash
# VPA needs to be installed separately on Kind
kubectl apply -f https://github.com/kubernetes/autoscaler/releases/latest/download/vertical-pod-autoscaler.yaml
```

### ❌ Error: "ImagePullBackOff"
**Fix:**
```bash
# Check if image exists
docker pull misacademy/backend:v3
docker pull misacademy/frontend:v3

# Or use your own images
```

---

## 📚 What You Learned

| Concept | Simple Explanation |
|---------|-------------------|
| **Namespace** | Separate rooms for different apps |
| **Deployment** | Blueprint for running stateless apps |
| **StatefulSet** | Blueprint for running stateful apps (like databases) |
| **Service** | How pods talk to each other |
| **PV/PVC** | Hard disk for your pods |
| **Secret** | Safe place for passwords |
| **HPA** | Add/remove pods based on load |
| **VPA** | Give/take CPU & RAM per pod |
| **Liveness Probe** | "Is the app alive?" check |
| **Readiness Probe** | "Is the app ready to receive traffic?" check |

---

## 🎓 For Students: Key Interview Questions

1. **Why use StatefulSet for MongoDB but Deployment for frontend/backend?**
   - StatefulSet gives each pod a stable name and persistent storage. MongoDB needs this because data must survive restarts. Frontend/backend don't care which pod handles the request.

2. **Why not use HPA on the database?**
   - Databases need careful coordination when adding replicas. You can't just add a random MongoDB pod — it needs to join the replica set properly. HPA is for stateless apps only.

3. **What's the difference between HPA and VPA?**
   - HPA = more pods (horizontal). VPA = bigger pods (vertical).

4. **Why use both HPA and VPA on the backend?**
   - HPA handles traffic spikes by adding pods. VPA ensures each pod has enough CPU/RAM to perform well.

5. **What happens if a MongoDB pod crashes?**
   - Kubernetes restarts it. Because we used PVC, the data is saved on the host disk and re-attached to the new pod.

---

## 🙏 Credits

- Built for students learning DevOps & Kubernetes
- Inspired by real-world fintech startup architecture
- Images: `misacademy/frontend:v3` and `misacademy/backend:v3`

---

## 📄 License

MIT License — Free for learning and projects!

---

> 💡 **Pro Tip:** Once you master this on Kind, you can easily move to **Amazon EKS** for production. The YAML files are 90% the same!

**Happy Learning! 🚀**
