

![Build Status](https://img.shields.io/github/actions/workflow/status/blendermen/azure-aks-devops/main.yml?branch=main)
![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
# 🚀 K3S DevOps Project: React, Flask & PostgreSQL

A comprehensive full-stack application ecosystem deployed on **local K3S/K3D**


<img width="1329" height="910" alt="k3sdemo" src="https://github.com/user-attachments/assets/de1924ef-ac19-4a68-8692-47dbfcaaec51" />


<img width="1507" height="768" alt="argocd" src="https://github.com/user-attachments/assets/d927428b-7f96-4225-a054-2976edae6484" />



## 🏗️ System Architecture

The application consists of **three** tiers running within the Kubernetes cluster:

* **Frontend:** React (Vite) application served via Nginx.
* **Backend:** REST API built with Flask (Python), handling the visit counter logic.
* **Database:** PostgreSQL 15 instance with persistent storage 

This project uses **two** separate environments: **Development** and **Production**.

🔄 How it works

**1. Push and Build**  
When you push code to this repository:  
-An automated GitHub Action  [process](https://github.com/blendermen/k3s-local-app/blob/main/.github/workflows/main.yml) starts immediately.  
-A new Docker image is built.  
-The image is pushed to the DockerHub registry with a unique tag - SHA. 

 
**2. Automatic PR for Manifests**  
Once the image is ready, the system automatically goes to the [k3s-local-manifest](https://github.com/blendermen/k3s-local-manifest) repository and:  
-Switches to the k3s-local-manifest and creates a new branch.  
-Updates the image version to the new one (using kustomize overlays)  
-Creates a Pull Request (PR) automatically.  

 
**3. Verification and Production**  
-Development: You check the changes manually first in the Dev environment.  
-Approval: If everything works correctly in Dev, you manually approve and merge the PR.  
-Production: Only after your approval, the changes are applied to the Production environment.  

 
Summary: No code goes to production without being checked in Dev and manually approved via PR. 

To test it locally, add these entries to the /etc/hosts file  
_172.23.0.3  prod.mywebpage.pl  
172.23.0.3  dev.mywebpage.pl_  
  
172.23.0.3 = traefik IP  
kubectl get services -n=kube-system  
NAME             TYPE           CLUSTER-IP      EXTERNAL-IP             PORT(S)                      AGE  
traefik          LoadBalancer   10.43.47.5      **172.23.0.3**,172.23.0.4   80:32731/TCP,443:30847/TCP   5d23h   

 
 
## 🛠️ Deployment Guide

### 1. Local Development (Docker Compose)
Quickly start the environment for local testing:

```bash
docker-compose up --build
```
Frontend: http://localhost:8080
Backend: http://localhost:5000

