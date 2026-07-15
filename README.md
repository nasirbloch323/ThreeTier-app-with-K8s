# 🚀 Secure Deployment of Three-Tier Architecture on Kubernetes
### With Resource Requests & Limits + Liveness & Readiness Probes

A complete hands-on DevOps project where a **MERN stack application** (Frontend + Backend + Database) is deployed on a **Kubernetes (KIND) cluster** running on an **AWS EC2 Ubuntu server**, with proper **namespace isolation**, **resource governance**, and **health checks**.

---

## 📖 Table of Contents

- [🎯 Objectives](#-objectives)
- [❗ Problem Statement](#-problem-statement)
- [🏢 Industry Scenario](#-industry-scenario)
- [📌 Application Stack](#-application-stack)
- [🧠 Core Concepts Explained](#-core-concepts-explained)
- [🏗 Project Structure](#-project-structure)
- [⚙️ Step-by-Step Setup](#️-step-by-step-setup)
  - [Step 1: Install Docker](#step-1-install-docker)
  - [Step 2: Install kubectl & KIND](#step-2-install-kubectl--kind)
  - [Step 3: Create KIND Cluster](#step-3-create-kind-cluster)
  - [Step 4: Create Namespaces](#step-4-create-namespaces)
  - [Step 5: Apply ResourceQuotas](#step-5-apply-resourcequotas)
  - [Step 6: Apply LimitRanges](#step-6-apply-limitranges)
  - [Step 7: Create Secrets & ConfigMaps](#step-7-create-secrets--configmaps)
  - [Step 8: Deploy Frontend](#step-8-deploy-frontend)
  - [Step 9: Deploy Backend](#step-9-deploy-backend)
  - [Step 10: Access the Application](#step-10-access-the-application)
- [✅ Verification Commands](#-verification-commands)
- [✅ Conclusion](#-conclusion)

---

## 🎯 Objectives

As a DevOps Engineer in a startup, our goals are:

- Set up an **EC2 Ubuntu server** for a Kubernetes cluster.
- Install **Docker + KIND** for a lightweight local cluster.
- Deploy a **three-tier MERN app** (Frontend + Backend + Database).
- Use **separate namespaces** (`frontend`, `backend`, `database`).
- Apply **ResourceQuotas & LimitRanges** at namespace level.
- Define **resources + probes** in Deployments.
- Understand **Stateless vs Stateful** apps.

---

## ❗ Problem Statement

- Application layers (frontend, backend, database) are tightly coupled and lack isolation.
- No resource controls → one service can consume all cluster resources.
- Lack of health monitoring → unresponsive pods cause downtime.
- Sensitive data (DB credentials) not securely managed.

---

## 🏢 Industry Scenario

A fintech startup is building a cloud-native web application that must meet enterprise-grade standards:

| Requirement | Meaning |
|---|---|
| **Scalable** | Handle sudden traffic spikes with zero downtime |
| **Cost-Effective** | Use minimal infrastructure while optimizing resources |
| **Reliable** | Ensure high availability for financial transactions |
| **Secure & Organized** | Isolate layers, protect sensitive data, enforce resource controls |
| **Resource Management** | Define requests, limits, quotas, and limit ranges |

---

## 📌 Application Stack

| Layer | Tech | Purpose |
|---|---|---|
| **Frontend** | React (MERN UI) | User interface for customers |
| **Backend** | Node.js + Express | Business logic and DB connectivity |
| **Database** | MongoDB Atlas | Managed, persistent, highly available storage |

### 🔹 Key Responsibilities as DevOps Engineer

1. **Cluster Setup** — Provision AWS EC2, install KIND for dev/testing, plan EKS for production.
2. **Application Deployment** — Write manifests for Deployments, Services, ConfigMaps, Secrets, PVCs.
3. **Security & Compliance** — Store DB credentials & API keys in Kubernetes Secrets.
4. **Resource Management** — Define requests/limits + enforce ResourceQuotas & LimitRanges.
5. **Scalability & Reliability** — Configure HPA, liveness/readiness probes, and PVCs.

---

## 🧠 Core Concepts Explained

### 1️⃣ Requests & Limits

- **Requests** → Minimum CPU/Memory guaranteed to a pod. The scheduler uses this to decide where to place the pod.
- **Limits** → Maximum CPU/Memory a pod can use. If exceeded → **throttling (CPU)** or **OOMKill (Memory)**.

> 💡 **Analogy:** Like booking a hotel room — you request a *minimum room size*, but there's a *maximum limit* on how much space you can occupy.

#### 🖥️ CPU Throttling
- CPU is a **compressible** resource.
- If a pod tries to use more CPU than its limit, Kubernetes **slows it down** instead of killing it.
- Effect: increased latency, slower requests — but the pod keeps running.
- Example: Limit set to `0.5 CPU`. App tries to use `1 CPU` → throttled to half speed.

#### 💾 OOMKill (Out of Memory Kill)
- Memory is a **non-compressible** resource.
- If a pod exceeds its memory limit, the **Linux OOMKiller** terminates the container immediately.
- Kubernetes then restarts it (based on `restartPolicy`).
- Example: Limit set to `256Mi`. App tries to use `300Mi` → container is killed & restarted.

| Resource | If Exceeds Limit | What Happens |
|---|---|---|
| **CPU** | Over limit | Throttled (slowed down, still runs) |
| **Memory** | Over limit | OOMKill (container killed & restarted) |

### 2️⃣ ResourceQuotas
- Applied **per namespace**.
- Restricts total CPU, memory, pod count, PVCs, etc. for that namespace.
- Example: The `frontend` namespace can't consume more than `2 CPUs & 4Gi memory` in total.

> 💡 **Analogy:** Like a company giving a fixed budget to each department so no single team can overspend.

### 3️⃣ LimitRange
- Sets **default & max/min** resource values **per container** inside a namespace.
- If a Deployment doesn't specify requests/limits, LimitRange auto-applies defaults.

### 4️⃣ Liveness Probe
- Kubernetes periodically checks if the container is **still alive**.
- If it fails → Kubernetes **restarts the pod**.
- Prevents "hung"/frozen apps from staying stuck forever.

> 💡 **Analogy:** Like a doctor checking your pulse — if there's no pulse, you get revived.

### 5️⃣ Readiness Probe
- Checks if the container is **ready to serve traffic**.
- If it fails → Pod is **removed from the Service load balancer** (but not restarted).
- Prevents sending traffic to an app that's still starting up.

> 💡 **Analogy:** Like a restaurant waiter — you don't seat customers until the table is actually ready.

### How They All Work Together

```
                         Kubernetes Cluster
                        ┌────────────────────┐
                        │  Namespace: Backend │
                        └─────────▲──────────┘
                 ┌────────────────┴─────────────────┐
                 │                                   │
     ┌───────────────────────┐          ┌────────────────────────┐
     │     ResourceQuota      │          │       LimitRange        │
     │  (Namespace Budget)    │          │    (Default per Pod)    │
     │  CPU: max 2             │          │  CPU: 200m - 500m       │
     │  Memory: max 2Gi         │          │  Memory: 256Mi - 512Mi  │
     └───────────▲────────────┘          └────────────▲────────────┘
                 │                                     │
                 └───────────────┬─────────────────────┘
                                  │
                   ┌──────────────────────────┐
                   │  Deployment (Pod Spec)     │
                   │  requests: 200m / 256Mi    │
                   │  limits:   500m / 512Mi    │
                   └──────────────────────────┘
```

---

## 🏗 Project Structure

```
three-tier-app/
├── k8s-manifests/
│
├── namespaces/
│   ├── frontend.yaml
│   ├── backend.yaml
│   └── database.yaml
│
├── resource-quotas/
│   ├── frontend-quota.yaml
│   ├── backend-quota.yaml
│   └── database-quota.yaml
│
├── limit-ranges/
│   ├── frontend-limits.yaml
│   ├── backend-limits.yaml
│   └── database-limits.yaml
│
├── frontend/
│   ├── deployment.yaml
│   └── service.yaml
│
├── backend/
│   ├── deployment.yaml
│   └── service.yaml
│
└── kind-config.yaml
```

---

## ⚙️ Step-by-Step Setup

> ⚠️ Run all commands on your **AWS EC2 Ubuntu server** (SSH into it first).

### Step 1: Install Docker

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
newgrp docker
docker --version
```

✅ This installs Docker, starts the service, enables it on boot, and adds your user to the `docker` group so you don't need `sudo` every time.

---

### Step 2: Install kubectl & KIND

**Install kubectl** (the CLI tool to talk to your Kubernetes cluster):

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/kubectl
kubectl version --client
```

**Install KIND** (Kubernetes IN Docker — lets you run a full cluster locally using Docker containers as nodes):

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
kind version
```

---

### Step 3: Create KIND Cluster

Create a file named `kind-config.yaml`:

```yaml
# kind-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  # Control plane node
  - role: control-plane
    image: kindest/node:v1.28.0
    extraPortMappings:
      - containerPort: 80
        hostPort: 80
        protocol: TCP
      - containerPort: 443
        hostPort: 443
        protocol: TCP
      - containerPort: 30007   # Node.js NodePort
        hostPort: 30007
        protocol: TCP
  # Worker node 1
  - role: worker
    image: kindest/node:v1.28.0
  # Worker node 2
  - role: worker
    image: kindest/node:v1.28.0
```

This config creates:
- **1 control-plane node** (manages the cluster)
- **2 worker nodes** (run your actual application pods)
- **Port mappings** so traffic on ports 80, 443, and 30007 can reach the cluster from outside

Now create the cluster:

```bash
kind create cluster --name three-tier-cluster --config kind-config.yaml
```

Verify it's running:

```bash
kubectl get nodes
```

---

### Step 4: Create Namespaces

Namespaces isolate frontend, backend, and database from each other — like separate rooms in a house.

**`namespaces/frontend.yaml`**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: frontend
```

**`namespaces/backend.yaml`**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: backend
```

**`namespaces/database.yaml`**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: database
```

Apply them:

```bash
kubectl apply -f namespaces/frontend.yaml
kubectl apply -f namespaces/backend.yaml
kubectl apply -f namespaces/database.yaml
```

Verify:
```bash
kubectl get namespaces
```

---

### Step 5: Apply ResourceQuotas

These set the **total budget** of CPU/memory each namespace can consume.

**`resource-quotas/frontend-quota.yaml`**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: frontend-quota
  namespace: frontend
spec:
  hard:
    requests.cpu: "1"
    requests.memory: "1Gi"
    limits.cpu: "2"
    limits.memory: "2Gi"
```

**`resource-quotas/backend-quota.yaml`**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: backend-quota
  namespace: backend
spec:
  hard:
    requests.cpu: "1"
    requests.memory: "1Gi"
    limits.cpu: "2"
    limits.memory: "2Gi"
```

**`resource-quotas/database-quota.yaml`**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: database-quota
  namespace: database
spec:
  hard:
    requests.cpu: "2"
    requests.memory: "2Gi"
    limits.cpu: "4"
    limits.memory: "4Gi"
```

Apply them:

```bash
kubectl apply -f resource-quotas/frontend-quota.yaml
kubectl apply -f resource-quotas/backend-quota.yaml
kubectl apply -f resource-quotas/database-quota.yaml
```

Verify:
```bash
kubectl describe quota -n frontend
```

---

### Step 6: Apply LimitRanges

These set **default per-container** resource values inside each namespace.

**`limit-ranges/frontend-limits.yaml`**
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: frontend-limits
  namespace: frontend
spec:
  limits:
    - default:
        cpu: "500m"
        memory: "512Mi"
      defaultRequest:
        cpu: "200m"
        memory: "256Mi"
      type: Container
```

**`limit-ranges/backend-limits.yaml`**
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: backend-limits
  namespace: backend
spec:
  limits:
    - default:
        cpu: "500m"
        memory: "512Mi"
      defaultRequest:
        cpu: "200m"
        memory: "256Mi"
      type: Container
```

**`limit-ranges/database-limits.yaml`**
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: database-limits
  namespace: database
spec:
  limits:
    - default:
        cpu: "1"
        memory: "1Gi"
      defaultRequest:
        cpu: "500m"
        memory: "512Mi"
      type: Container
```

Apply them:

```bash
kubectl apply -f limit-ranges/frontend-limits.yaml
kubectl apply -f limit-ranges/backend-limits.yaml
kubectl apply -f limit-ranges/database-limits.yaml
```

---

### Step 7: Create Secrets & ConfigMaps

Create a `.env` file with your MongoDB URI and other secrets, then:

```bash
# Backend secret (DB credentials, kept secure)
kubectl -n backend create secret generic mongo-secret \
  --from-env-file=.env

# Frontend configmap (non-sensitive config)
kubectl -n frontend create configmap frontend-config \
  --from-env-file=.env
```

> 🔒 Never commit your real `.env` file to GitHub. Add it to `.gitignore`.

---

### Step 8: Deploy Frontend

**`frontend/deployment.yaml`**
```yaml
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
          image: umair1012/frontend:latest
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
    - port: 80
      targetPort: 3000
      nodePort: 30007
```

Apply it:

```bash
kubectl apply -f frontend/deployment.yaml
```

---

### Step 9: Deploy Backend

**`backend/deployment.yaml`**
```yaml
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
          image: umair1012/backend:latest
          ports:
            - containerPort: 5000
          env:
            - name: MONGO_URI
              valueFrom:
                secretKeyRef:
                  name: mongo-secret
                  key: MONGO_URI
          resources:
            requests:
              cpu: "200m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
          livenessProbe:
            httpGet:
              path: /health
              port: 5000
            initialDelaySeconds: 15
            periodSeconds: 20
          readinessProbe:
            httpGet:
              path: /ready
              port: 5000
            initialDelaySeconds: 10
            periodSeconds: 15
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: backend
spec:
  type: NodePort
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
      nodePort: 30008
```

Apply it:

```bash
kubectl apply -f backend/deployment.yaml
```

---

### Step 10: Access the Application

Forward the frontend service port to your EC2 instance:

```bash
kubectl port-forward svc/frontend-service 3000:3000 -n frontend --address 0.0.0.0
```

Then open in your browser:

```
http://<EC2_PUBLIC_IP>:3000
```

> ✅ Make sure your EC2 **Security Group** allows inbound traffic on port `3000` (and `30007`/`30008` if using NodePort directly).

---

## ✅ Verification Commands

```bash
# Check all pods across namespaces
kubectl get pods -A

# Check pods in a specific namespace
kubectl get pods -n frontend
kubectl get pods -n backend

# Check resource usage vs quota
kubectl describe quota -n frontend
kubectl describe quota -n backend

# Check limit ranges
kubectl describe limitrange -n frontend

# Check pod health/probe status
kubectl describe pod <pod-name> -n frontend

# Check services
kubectl get svc -A

# Check logs of a pod
kubectl logs <pod-name> -n backend
```

---

## ✅ Conclusion

- **Namespaces** separate frontend, backend, and database for isolation.
- **ResourceQuotas + LimitRanges** ensure fair cluster resource usage and prevent one service from hogging resources.
- **Deployments** define per-app resources + liveness/readiness probes for self-healing and zero-downtime.
- **Secrets** securely store MongoDB credentials.
- The overall architecture is **scalable, secure, and production-ready** 🚀

---

### 🙌 Author
Deployed & documented as part of a hands-on DevOps learning project — Miseacademy.
