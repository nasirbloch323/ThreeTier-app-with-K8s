# 🚀 Multi-Branch Jenkins CI/CD Pipeline with Jenkins Agents

## 📌 Project Overview

This project demonstrates a complete **Multi-Branch CI/CD Pipeline** using **Jenkins**, **Jenkins Agents**, **Docker**, **Docker Hub**, and **AWS EC2**.

The pipeline automatically detects Git branches (**dev**, **stg**, and **prod**), builds Docker images, pushes them to Docker Hub, and deploys the application to the correct environment using Docker Compose.

---

# 🏗️ Project Architecture

```
                GitHub
                   │
             Git Push/Webhook
                   │
                   ▼
        Jenkins Controller (EC2)
                   │
        ┌──────────┼──────────┐
        │          │          │
        ▼          ▼          ▼
   Agent-Dev   Agent-Stg   Agent-Prod
        │          │          │
        ▼          ▼          ▼
 Docker Build  Docker Build Docker Build
        │          │          │
        └──────Push Images────┘
                 │
                 ▼
            Docker Hub
                 │
                 ▼
      Deploy using Docker Compose
```

---

# 📂 Project Structure

```
three-tier-app/

│
├── backend/
├── frontend/
├── docker-compose.yml
├── Jenkinsfile
├── README.md
```

---

# 🚀 Technologies Used

- Jenkins
- Jenkins Agents
- Docker
- Docker Compose
- Docker Hub
- Git
- GitHub
- AWS EC2
- Ubuntu 22.04
- Node.js
- React
- MongoDB

---

# 🌿 Branch Strategy

| Branch | Environment |
|----------|-------------|
| dev | Development |
| stg | Staging |
| prod | Production |

---

# ☁ AWS Infrastructure

## EC2 Instances

- Jenkins Controller
- Dev Agent
- Staging Agent
- Production Agent

---

# 📦 Installation Guide

# Step 1 Install Java

```
sudo apt update
sudo apt install openjdk-17-jdk -y
java --version
```

---

# Step 2 Install Jenkins

```
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
/usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update

sudo apt install jenkins -y

sudo systemctl enable jenkins

sudo systemctl start jenkins
```

Check Status

```
sudo systemctl status jenkins
```

Unlock Jenkins

```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

# Step 3 Install Docker

```
sudo apt update

sudo apt install docker.io -y

sudo systemctl enable docker

sudo systemctl start docker

sudo usermod -aG docker ubuntu

newgrp docker
```

Verify

```
docker --version
```

---

# Step 4 Install Docker Compose

```
sudo apt install docker-compose -y
```

Verify

```
docker-compose --version
```

---

# Step 5 Install Git

```
sudo apt install git -y
```

Verify

```
git --version
```

---

# Step 6 Install Jenkins Plugins

Manage Jenkins

↓

Plugins

↓

Install

- Docker Pipeline
- Pipeline Stage View
- Git
- GitHub
- SSH Agent
- Credentials Binding
- Pipeline
- Blue Ocean (Optional)

---

# Step 7 Configure Docker Permission

```
sudo usermod -aG docker jenkins

sudo systemctl restart jenkins
```

---

# Step 8 Docker Hub Credentials

Manage Jenkins

↓

Credentials

↓

Global

↓

Username with Password

```
ID

dockerhub-credentials
```

---

# Step 9 Configure SSH Credentials

Create SSH credentials for

- Dev Agent
- Staging Agent
- Production Agent

---

# Step 10 Create Jenkins Agents

Go to

```
Manage Jenkins

↓

Nodes

↓

New Node
```

Create

```
agent-dev

agent-stg

agent-prod
```

---

# Step 11 Install Required Packages on Every Agent

```
sudo apt update

sudo apt install openjdk-17-jdk docker.io docker-compose git -y

sudo usermod -aG docker ubuntu

newgrp docker
```

---

# Step 12 Configure GitHub Webhook

Repository

↓

Settings

↓

Webhooks

↓

Payload URL

```
http://JENKINS-IP/github-webhook/
```

Content Type

```
application/json
```

---

# Step 13 Create Multibranch Pipeline

Dashboard

↓

New Item

↓

Multibranch Pipeline

↓

GitHub Repository

↓

Save

---

# Step 14 Pipeline Workflow

```
Developer Push Code

↓

GitHub

↓

Webhook

↓

Jenkins Controller

↓

Detect Branch

↓

Run Agent

↓

Checkout Code

↓

Docker Login

↓

Build Images

↓

Tag Images

↓

Push Images

↓

Deploy

↓

Docker Compose

↓

Application Running
```

---

# 🏷 Docker Image Tags

```
dev-1

dev-2

stg-5

prod-12
```

---

# 🔐 Jenkins Credentials Used

| Credential | Purpose |
|------------|----------|
| Docker Hub Credentials | Push Images |
| SSH Key | Connect Agent |
| GitHub Credentials | Clone Repository |

---

# 📁 Docker Images

Backend

```
username/three-tier-app-backend:dev-1
```

Frontend

```
username/three-tier-app-frontend:dev-1
```

---

# 📊 Jenkins Pipeline Stages

- Checkout Source
- Docker Login
- Build Images
- Push Images
- Generate .env
- Approval (Staging & Production)
- Deploy
- Cleanup

---

# 🚀 Deployment

Development

```
docker-compose up -d
```

Staging

```
docker-compose up -d
```

Production

```
docker-compose up -d
```

---

# 📸 Project Output

✔ Jenkins Multibranch Pipeline

✔ Jenkins Agents

✔ Docker Build

✔ Docker Hub Push

✔ Docker Compose Deployment

✔ Development Deployment

✔ Staging Deployment

✔ Production Deployment

✔ Automatic Branch Detection

✔ Stage View

---

# 🎯 Learning Outcomes

- Jenkins Controller & Agents
- Multi-Branch Pipelines
- Docker Image Build
- Docker Hub Integration
- Docker Compose Deployment
- Jenkins Credentials
- GitHub Webhooks
- AWS EC2 Management
- Branch-Based Deployments
- CI/CD Automation

---

# 👨‍💻 Author

**Nasir Mehmood**

DevOps Engineer

---

# ⭐ If you found this project helpful, don't forget to Star the repository.
