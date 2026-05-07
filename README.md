# EKS Flask MySQL Project – Architecture & GitHub Update Guide

## Project Overview

This project is a two-tier application deployed on Amazon EKS using Kubernetes.

* Frontend/Application Layer: Flask Python application
* Database Layer: MySQL database
* Containerization: Docker
* Orchestration: Kubernetes on AWS EKS

---

# High-Level Architecture

```text
                ┌───────────────────────┐
                │       End User        │
                └──────────┬────────────┘
                           │
                           ▼
                ┌───────────────────────┐
                │ Kubernetes Service    │
                │ (LoadBalancer/NodePort)
                └──────────┬────────────┘
                           │
                           ▼
                ┌───────────────────────┐
                │ Flask Application Pod │
                │   app.py              │
                └──────────┬────────────┘
                           │
                           ▼
                ┌───────────────────────┐
                │ MySQL Service         │
                └──────────┬────────────┘
                           │
                           ▼
                ┌───────────────────────┐
                │ MySQL Pod             │
                │ Persistent Database   │
                └───────────────────────┘
```

---

# Repository Structure

```text
eks-flask-mysql-project-master/
│
├── app.py
├── Dockerfile
├── requirements.txt
├── message.sql
│
├── templates/
│   └── index.html
│
├── static/
│   └── sir.jpg
│
└── eks-manifests/
    ├── mysql-configmap.yml
    ├── mysql-deployment.yml
    ├── mysql-secrets.yml
    ├── mysql-svc.yml
    ├── two-tier-app-deployment.yml
    └── two-tier-app-svc.yml
```

---

# Component Explanation

## 1. Flask Application

File: `app.py`

Responsibilities:

* Handles web requests
* Inserts messages into MySQL
* Reads messages from MySQL
* Renders HTML pages using Flask templates

Endpoints:

* `/` → Displays all messages
* `/submit` → Inserts new messages

---

## 2. Docker Container

File: `Dockerfile`

Purpose:

* Packages Flask application with dependencies
* Creates a portable image
* Used for Kubernetes deployment

Build Example:

```bash
docker build -t flask-mysql-app .
```

---

## 3. MySQL Database

Kubernetes Resources:

* Deployment
* Service
* ConfigMap
* Secrets

Purpose:

* Stores application messages
* Provides persistent backend storage

Files:

* `mysql-deployment.yml`
* `mysql-svc.yml`
* `mysql-configmap.yml`
* `mysql-secrets.yml`

---

## 4. Kubernetes Services

### Flask App Service

File:

* `two-tier-app-svc.yml`

Purpose:

* Exposes Flask app externally
* Routes traffic to Flask pods

### MySQL Service

File:

* `mysql-svc.yml`

Purpose:

* Internal communication between Flask app and MySQL

---

# Deployment Flow

```text
Developer Pushes Code → GitHub Repository
            ↓
Docker Image Build
            ↓
Push Image to DockerHub/ECR
            ↓
Apply Kubernetes Manifests
            ↓
EKS Creates Pods & Services
            ↓
Users Access Application
```

---

# Recommended Architecture Improvements

## Current Limitations

* No CI/CD pipeline
* No persistent storage volume
* No ingress controller
* No monitoring/logging
* Secrets stored in plain base64
* Single replica deployment

---

# Recommended Production Architecture

```text
                 ┌─────────────────────┐
                 │      GitHub         │
                 └─────────┬───────────┘
                           │
                           ▼
                ┌──────────────────────┐
                │ GitHub Actions CI/CD │
                └─────────┬────────────┘
                          │
         ┌────────────────┴────────────────┐
         ▼                                 ▼
 ┌───────────────┐                ┌────────────────┐
 │ Docker Build  │                │ Security Scan  │
 └──────┬────────┘                └────────────────┘
        │
        ▼
 ┌───────────────┐
 │ DockerHub/ECR │
 └──────┬────────┘
        │
        ▼
 ┌──────────────────────────────┐
 │ Amazon EKS Cluster           │
 │                              │
 │  ┌────────────────────────┐  │
 │  │ Flask App Deployment   │  │
 │  │ Multiple Replicas      │  │
 │  └──────────┬─────────────┘  │
 │             │                │
 │      ┌──────▼──────┐         │
 │      │ Ingress/ALB │         │
 │      └─────────────┘         │
 │                              │
 │  ┌────────────────────────┐  │
 │  │ MySQL StatefulSet      │  │
 │  │ Persistent Volumes     │  │
 │  └────────────────────────┘  │
 └──────────────────────────────┘
```

---

# Steps to Update in GitHub

## Step 1: Create New GitHub Repository

Example:

```bash
git init
git remote add origin https://github.com/yourusername/eks-flask-mysql.git
```

---

## Step 2: Add Project Files

```bash
git add .
git commit -m "Initial EKS Flask MySQL project"
```

---

## Step 3: Push to GitHub

```bash
git branch -M main
git push -u origin main
```

---

# Add CI/CD Using GitHub Actions

Create:

```text
.github/workflows/deploy.yml
```

Example Workflow:

```yaml
name: Deploy to EKS

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Login to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build Docker Image
        run: |
          docker build -t yourdockerhub/flask-app:${{ github.sha }} .

      - name: Push Docker Image
        run: |
          docker push yourdockerhub/flask-app:${{ github.sha }}
```

---

# Kubernetes Deployment Steps

## Step 1: Create EKS Cluster

```bash
eksctl create cluster \
--name flask-cluster \
--region ap-south-1 \
--nodegroup-name workers \
--node-type t3.medium \
--nodes 2
```

---

## Step 2: Deploy MySQL

```bash
kubectl apply -f eks-manifests/mysql-configmap.yml
kubectl apply -f eks-manifests/mysql-secrets.yml
kubectl apply -f eks-manifests/mysql-deployment.yml
kubectl apply -f eks-manifests/mysql-svc.yml
```

---

## Step 3: Deploy Flask App

```bash
kubectl apply -f eks-manifests/two-tier-app-deployment.yml
kubectl apply -f eks-manifests/two-tier-app-svc.yml
```

---

## Step 4: Verify Pods

```bash
kubectl get pods
kubectl get svc
```

---

# Best Practices

## Security

* Use Kubernetes Secrets
* Enable IAM Roles for Service Accounts (IRSA)
* Store secrets in AWS Secrets Manager

## Scalability

* Add Horizontal Pod Autoscaler
* Use LoadBalancer or ALB Ingress
* Increase Flask replicas

## Reliability

* Use MySQL StatefulSet
* Add Persistent Volume Claims
* Configure readiness/liveness probes

## Monitoring

* Prometheus
* Grafana
* CloudWatch Container Insights

---

# Suggested Future Enhancements

* Add Terraform for Infrastructure as Code
* Add Helm Charts
* Add ArgoCD for GitOps
* Add HTTPS using cert-manager
* Add Redis caching layer
* Add API versioning
* Add unit/integration tests

---

# Final Recommended GitHub Structure

```text
.
├── .github/
│   └── workflows/
│       └── deploy.yml
│
├── app/
│   ├── app.py
│   ├── requirements.txt
│   └── templates/
│
├── docker/
│   └── Dockerfile
│
├── kubernetes/
│   ├── mysql/
│   └── flask-app/
│
├── terraform/
│   └── eks/
│
└── README.md
```

---

# Conclusion

This project demonstrates:

* Docker containerization
* Flask + MySQL integration
* Kubernetes deployments
* Amazon EKS orchestration
* Two-tier cloud-native architecture

The next production-ready step is adding:

1. GitHub Actions CI/CD
2. Persistent storage
3. Monitoring
4. Auto scaling
5. Secure secret management
