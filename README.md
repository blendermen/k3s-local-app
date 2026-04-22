
![Build Status](https://img.shields.io/github/actions/workflow/status/blendermen/azure-aks-devops/main.yml?branch=main)
![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
# 🚀 K3S DevOps Project: React, Flask & PostgreSQL

A comprehensive full-stack application ecosystem deployed on **local K3S/K3D**


<img width="547" alt="Screenshot_20260423_004248" src="https://github.com/user-attachments/assets/f10635fa-8c22-49aa-b929-70cb9ada23d8" />

## 🏗️ System Architecture

The application consists of three tiers running within the Kubernetes cluster:

* **Frontend:** React (Vite) application served via Nginx.
* **Backend:** REST API built with Flask (Python), handling the visit counter logic.
* **Database:** PostgreSQL 15 instance with persistent storage 
* **Ingress:** **NGINX Ingress Controller**

---


## 🔄 CI/CD Pipeline (GitHub Actions)

Deployment is fully automated upon every `push` to the `main` branch:

1.  **Build:** Docker images are built for both frontend and backend.
2.  **Push:** Images are pushed to **Dockerhub** using the commit SHA as a tag (ensuring immutability).
3.  **Deploy:** Deployment is managed by ArgoCD using another GitHub repository k3s-local-manifest

---

## 🛠️ Deployment Guide

### 1. Local Development (Docker Compose)
Quickly start the environment for local testing:

```bash
docker-compose up --build
```
Frontend: http://localhost:8080
Backend: http://localhost:5000

