![Build Status](https://img.shields.io/github/actions/workflow/status/blendermen/azure-aks-devops/main.yml?branch=main)
![Azure](https://img.shields.io/badge/azure-%230072C6.svg?style=for-the-badge&logo=microsoftazure&logoColor=white)
![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
# 🚀 Azure AKS DevOps Project: React, Flask & PostgreSQL

A comprehensive full-stack application ecosystem deployed on **Azure Kubernetes Service (AKS)**. This project demonstrates a secure Software Development Life Cycle (SDLC), cloud-native secret management, and container orchestration.

<img width="547" alt="Screenshot_20260326_225941" src="https://github.com/user-attachments/assets/14f93e9b-9ab8-4ed3-ada7-f9c0d0bc52f6" />

## 🏗️ System Architecture

The application consists of three tiers running within the Kubernetes cluster:

* **Frontend:** React (Vite) application served via Nginx.
* **Backend:** REST API built with Flask (Python), handling the visit counter logic.
* **Database:** PostgreSQL 15 instance with persistent storage (**Persistent Volume Claim** on Azure Disk).
* **Ingress:** **NGINX Ingress Controller** managing external traffic routing (L7 Load Balancer).

---

## 🔐 Security & Secret Management

A core focus of this project is the **total elimination of hardcoded credentials** from the source code:

* **Azure Key Vault:** All sensitive data (DB host, passwords) is stored in a secure Azure vault.
* **Secrets Store CSI Driver:** Used to dynamically mount and sync secrets from Key Vault directly into Pods.
* **Managed Identity:** The AKS cluster authenticates to Key Vault using an Azure-assigned Managed Identity (Azure RBAC), removing the need for manual service principal secret rotation.

---

## 🔄 CI/CD Pipeline (GitHub Actions)

Deployment is fully automated upon every `push` to the `main` branch:

1.  **Build:** Docker images are built for both frontend and backend.
2.  **Push:** Images are pushed to **Azure Container Registry (ACR)** using the commit SHA as a tag (ensuring immutability).
3.  **Deploy:** Kubernetes manifests are updated dynamically via `sed` and applied to the AKS cluster.

---

## 🛠️ Deployment Guide

### 1. Local Development (Docker Compose)
Quickly start the environment for local testing:

```bash
docker-compose up --build
```
Frontend: http://localhost:8080
Backend: http://localhost:5000

### 2. Cloud Deployment Setup (Azure CLI)
Create Resource Group(RG), Azure Container Registry(ACR), and Azure Kubernetes Service(AKS)
```bash
az group create --name MojaZupaRG --location westeurope
az acr create --resource-group MojaZupaRG --name mojazupaacr99 --sku Basic
az aks create --resource-group MojaZupaRG --name MojaZupaAKS --attach-acr mojazupaacr99 --enable-addons azure-keyvault-secrets-provider --enable-managed-identity
```
Create a KeyVault with 4 secrets.
```bash
# 1. Create a KeyVault
az keyvault create --name vault-mojazuparg --resource-group MojaZupaRG --location westeurope --enable-rbac-authorization

# 2. We assign the Key Vault Secrets Officer role to allow an identity(you) to securely read, create, and manage secrets in Azure Key Vault.
az role assignment create --role "Key Vault Secrets Officer" \
  --assignee "$(az ad signed-in-user show --query id -o tsv)" \
  --scope /subscriptions/<ID>/resourceGroups/MojaZupaRG/providers/Microsoft.KeyVault/vaults/vault-mojazuparg

# 3. Now, with Key Vault Secrets Officer role you can add 4 new secrets
az keyvault secret set --vault-name vault-mojazuparg --name "DB-HOST" --value "db"
az keyvault secret set --vault-name vault-mojazuparg --name "DB-NAME" --value "postgres"
az keyvault secret set --vault-name vault-mojazuparg --name "DB-USER" --value "postgres"
az keyvault secret set --vault-name vault-mojazuparg --name "DB-PASSWORD" --value "password"

# 4. We assign the Key Vault Secrets User role to allow an identity(your AKS cluser) to securely read (but not modify) secrets from Azure Key Vault.
export IDENTITY_ID=$(az aks show -g MojaZupaRG -n MojaZupaAKS --query identityProfile.kubeletidentity.clientId -o tsv)
az role assignment create --role "Key Vault Secrets User" --assignee $IDENTITY_ID --scope /subscriptions/<ID>/resourceGroups/MojaZupaRG/providers/Microsoft.KeyVault/vaults/vault-mojazuparg
```

Deploy the ingress controller with the following command: 
More info here: https://kubernetes.github.io/ingress-nginx/deploy/#quick-start
```bash
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

### 🚀 Production Deployment (Push to Main)
Thanks to the `.github/workflows/main.yml` configuration, every code push to the main branch triggers the following automated pipeline:

* **Build:** Generates new Docker images tagged with the unique `commit SHA`.
* **Push:** Uploads the images to **Azure Container Registry (ACR)**.
* **Update:** Automatically patches the `kubernetes-deployment.yaml` manifest with the latest image tags.
* **Deploy:** Executes `kubectl apply` to roll out changes to the **AKS cluster** instantly.

---

To ensure the CI/CD Pipeline functions correctly, you must add the following secrets in your repository settings (**Settings** -> **Secrets and variables** -> **Actions**):
| Secret Name | Value / Description |
| :--- | :--- |
| **AZURE_CREDENTIALS** | Output of the command: `az ad sp create-for-rbac --name "myApp" --role contributor --scopes /subscriptions/<ID> --json-auth` |
| **ACR_USERNAME** | Your ACR name (e.g., `mojazupaacr99`) |
| **ACR_LOGIN_SERVER** | Login server address (e.g., `mojazupaacr99.azurecr.io`) |


### 🔍 Troubleshooting & Key Concepts

* **Naming Convention:** Azure Key Vault typically uses dashes (e.g., `DB-PASSWORD`), whereas Python variables and environment lookups expect underscores (`DB_PASSWORD`). These are mapped correctly within the `SecretProviderClass` to ensure the application can read them seamlessly.
* **Ingress Routing:** Traffic is managed by the NGINX Ingress Controller. Requests starting with `/api` are routed to the **Flask backend**, while all other traffic (`/`) is directed to the **React frontend**. Ensure the Ingress Controller is installed (via Helm or AKS addon).
* **Database Persistence:** We utilize a `PersistentVolumeClaim` (PVC) backed by Azure Disk. This ensures that even if the database Pod is deleted or restarted, your data in `/var/lib/postgresql/data` remains safe and persistent.



### 🌐 Accessing the Application
The application is exposed to the internet via an **NGINX Ingress Controller** acting as a Load Balancer. 

#### 1. Retrieve the External IP
To find the entry point for the application, list the services in the `ingress-basic` namespace:

```bash
kubectl get services -n ingress-basic
