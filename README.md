![Build Status](https://img.shields.io/github/actions/workflow/status/blendermen/azure-aks-devops/main.yml?branch=main)
![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
# 🚀 K3D DevOps Project: React, Flask & PostgreSQL

A comprehensive full-stack application ecosystem deployed on **local K3S **. 

<img width="547" alt="Screenshot_20260326_225941" src="https://github.com/user-attachments/assets/14f93e9b-9ab8-4ed3-ada7-f9c0d0bc52f6" />

## 🏗️ System Architecture

The application consists of three tiers running within the Kubernetes cluster:

* **Frontend:** React (Vite) application served via Nginx.
* **Backend:** REST API built with Flask (Python), handling the visit counter logic.
* **Database:** PostgreSQL 15 instance with persistent storage (**Persistent Volume Claim** on Azure Disk).
* **Ingress:** **NGINX Ingress Controller** managing external traffic routing (L7 Load Balancer).

---


## 🔄 CI/CD Pipeline (GitHub Actions)

Deployment is fully automated upon every `push` to the `main` branch:

1.  **Build:** Docker images are built for both frontend and backend.
2.  **Push:** Images are pushed to **Dockerhub** using the commit SHA as a tag (ensuring immutability).
3.  **Deploy:** Deployment is managed by ArgoCD using another github repository k3s-local-manifest

---

## 🛠️ Deployment Guide

### 1. Local Development (Docker Compose)
Quickly start the environment for local testing:

```bash
docker-compose up --build
```
Frontend: http://localhost:8080
Backend: http://localhost:5000

