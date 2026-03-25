# AzureCI-MultiStage-Pipeline

![Azure DevOps](https://img.shields.io/badge/Azure%20DevOps-0078D7?style=for-the-badge&logo=azure-devops&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)

> Production-grade multi-stage CI/CD pipeline deploying containerized applications from Dev → QA → Prod on Azure Kubernetes Service (AKS), with full infrastructure provisioned via Terraform.

---

## 📌 Project Overview

This project demonstrates a real-world CI/CD pipeline setup for a containerized application on Azure. It covers the complete DevOps lifecycle — from code commit to production deployment — using industry-standard tools.

**Key Achievements:**
- ⚡ Reduced deployment time by ~70% using parallelized pipeline jobs
- 🔒 Zero-downtime deployments with automated rollback on failed health checks
- 🏗️ Fully reproducible infrastructure using Terraform with remote state backend
- ✅ Gated deployments with manual approval steps before Production

---

## 🏗️ Architecture

```
Developer Push
     │
     ▼
┌─────────────────┐
│  GitHub / Azure │
│   Repo (main)   │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────────────────────────────┐
│              Azure DevOps Pipeline                  │
│                                                     │
│  Stage 1: BUILD          Stage 2: DEV               │
│  ┌──────────────┐        ┌──────────────┐           │
│  │ Docker Build │──────► │ Deploy to    │           │
│  │ Push to ACR  │        │ Dev AKS NS   │           │
│  └──────────────┘        └──────┬───────┘           │
│                                 │                   │
│  Stage 3: QA             Stage 4: PROD              │
│  ┌──────────────┐        ┌──────────────┐           │
│  │ Deploy to QA │──────► │ Manual Gate  │           │
│  │ Run Tests    │        │ Deploy Prod  │           │
│  └──────────────┘        └──────────────┘           │
└─────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────┐
│   AKS Cluster   │  ← Provisioned by Terraform
│  (Dev/QA/Prod   │
│  Namespaces)    │
└─────────────────┘
```

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| Azure DevOps | Pipeline orchestration (YAML-based) |
| Terraform | AKS cluster + ACR provisioning, remote state |
| Docker | Application containerization |
| Azure Kubernetes Service (AKS) | Container orchestration |
| Azure Container Registry (ACR) | Private Docker image registry |
| GitHub Actions | PR validation and Terraform plan |

---

## 📁 Project Structure

```
AzureCI-MultiStage-Pipeline/
├── .azure-pipelines/
│   └── azure-pipelines.yml        # Main multi-stage pipeline definition
├── terraform/
│   ├── main.tf                    # AKS cluster + ACR resources
│   ├── variables.tf               # Input variables
│   ├── outputs.tf                 # Output values
│   ├── backend.tf                 # Remote state config (Azure Blob)
│   └── modules/
│       ├── aks/                   # AKS module
│       └── acr/                   # ACR module
├── kubernetes/
│   ├── deployment.yaml            # App deployment manifest
│   ├── service.yaml               # LoadBalancer service
│   └── hpa.yaml                   # Horizontal Pod Autoscaler
├── app/
│   ├── Dockerfile                 # Multi-stage Docker build
│   └── src/                       # Application source code
├── scripts/
│   └── rollback.sh                # Automated rollback script
└── README.md
```

---

## ⚙️ Pipeline Stages

### Stage 1: Build
- Docker image built using multi-stage Dockerfile
- Image tagged with build number and pushed to Azure Container Registry (ACR)
- Runs in parallel with linting and unit tests

### Stage 2: Dev Deployment
- Deploys to `dev` namespace in AKS
- Health check probe validates deployment
- Automatic rollback triggered if health check fails

### Stage 3: QA Deployment
- Deploys to `qa` namespace in AKS
- Runs integration tests against QA environment
- Deployment gate — blocks Stage 4 if tests fail

### Stage 4: Production Deployment
- **Manual approval gate** — requires sign-off before proceeding
- Blue-green style deployment strategy
- Post-deployment smoke tests
- Rollback available via pipeline re-run on previous artifact

---

## 🚀 Getting Started

### Prerequisites
- Azure subscription
- Azure DevOps organization
- Terraform >= 1.3.0
- Azure CLI installed and logged in

### Step 1: Provision Infrastructure

```bash
cd terraform/
terraform init
terraform plan -out=tfplan
terraform apply tfplan
```

### Step 2: Configure Pipeline Variables

In Azure DevOps → Pipelines → Library, create a variable group `pipeline-vars` with:

```
ACR_NAME         = <your-acr-name>
AKS_CLUSTER_NAME = <your-aks-name>
RESOURCE_GROUP   = <your-rg-name>
```

### Step 3: Run the Pipeline

- Push to `main` branch — triggers the full multi-stage pipeline
- Open a PR — triggers build + Terraform plan only (no deployment)

---

## 📊 Pipeline Performance

| Metric | Before | After |
|--------|--------|-------|
| Deployment Time | ~35 min | ~10 min |
| Manual Steps | 8 | 1 (prod approval) |
| Rollback Time | ~20 min | ~3 min (automated) |
| Environment Parity | Low | High (same Terraform modules) |

---

## 🔒 Security Highlights

- ACR integrated with AKS using Managed Identity (no credentials stored)
- Terraform state stored in Azure Blob with state locking (no conflicts)
- Secrets managed via Azure Key Vault + pipeline variable groups
- RBAC applied on AKS namespaces — Dev/QA/Prod are isolated

---

## 📚 Learnings

- Setting up multi-stage YAML pipelines in Azure DevOps
- Terraform remote backend with Azure Blob Storage
- Kubernetes namespace-based environment isolation
- Implementing deployment gates and manual approvals
- Automated rollback using `kubectl rollout undo`

---

## 👤 Author

**Deepak T R**
- 📧 deepuraj0527@gmail.com
- 💼 [LinkedIn](https://linkedin.com/in/deepak-tr)
- 🐙 [GitHub](https://github.com/deepak-tr)
- 🏅 AZ-104 Certified | AZ-400 In Progress
