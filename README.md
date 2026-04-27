# dennis-ai-infra

Infrastructure-as-code for the Dennis AI platform — provisioning, orchestration, and CI/CD pipelines powering a multi-repo personal AI website.

![IaC](https://img.shields.io/badge/IaC-Terraform-7B42BC?logo=terraform)
![Docker](https://img.shields.io/badge/Container-Docker-2496ED?logo=docker)
![K8s](https://img.shields.io/badge/Orchestration-Kubernetes-326CE5?logo=kubernetes)
![CI/CD](https://img.shields.io/badge/CI%2FCD-GitHub_Actions-2088FF?logo=githubactions)

---

## Purpose

This repository contains the infrastructure layer for the Dennis AI ecosystem. It defines cloud resources, container orchestration, networking, secrets management, and deployment pipelines used by:

| Repository | Role |
|---|---|
| [dennis-ai-site](https://github.com/dennismwebb/dennis-ai-site) | Next.js frontend — avatar, chat UI |
| [dennis-ai-backend](https://github.com/dennismwebb/dennis-ai-backend) | .NET 8 API — AI agent, services |
| [dennis-ai-data](https://github.com/dennismwebb/dennis-ai-data) | AI data assets — prompts, embeddings, knowledge base |

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│                    Cloud Provider (Azure)                │
│                                                         │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐  │
│  │  Networking   │  │   Compute    │  │   Storage &   │  │
│  │  & DNS        │  │   (AKS /     │  │   Secrets     │  │
│  │              │  │   App Svc)   │  │   (Key Vault) │  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬────────┘  │
│         │                 │                  │           │
│         └─────────────────┼──────────────────┘           │
│                           │                              │
│                  ┌────────▼────────┐                     │
│                  │  Kubernetes /   │                     │
│                  │  Container Apps │                     │
│                  │                 │                     │
│                  │  ┌───────────┐  │                     │
│                  │  │ Frontend  │  │                     │
│                  │  │ (Next.js) │  │                     │
│                  │  └───────────┘  │                     │
│                  │  ┌───────────┐  │                     │
│                  │  │ Backend   │  │                     │
│                  │  │ (.NET 8)  │  │                     │
│                  │  └───────────┘  │                     │
│                  └─────────────────┘                     │
│                                                         │
│  ┌──────────────────────────────────────────────────┐   │
│  │           CI/CD — GitHub Actions                  │   │
│  │  Build → Test → Containerize → Deploy             │   │
│  └──────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

### Key Components

- **Terraform modules** — Declarative cloud resource provisioning (resource groups, networking, compute, storage, DNS)
- **Kubernetes manifests / Helm charts** — Container orchestration, service mesh, ingress, scaling policies
- **Docker Compose** — Local development environment mirroring production topology
- **GitHub Actions workflows** — CI/CD pipelines for build, test, containerize, and deploy across all repos
- **Secrets management** — Azure Key Vault integration with environment-specific configurations

---

## Repository Structure

```
dennis-ai-infra/
├── terraform/
│   ├── environments/
│   │   ├── dev/
│   │   │   └── main.tf
│   │   ├── staging/
│   │   │   └── main.tf
│   │   └── prod/
│   │       └── main.tf
│   ├── modules/
│   │   ├── networking/
│   │   ├── compute/
│   │   ├── storage/
│   │   ├── keyvault/
│   │   └── dns/
│   ├── backend.tf
│   ├── variables.tf
│   └── outputs.tf
├── k8s/
│   ├── base/
│   │   ├── namespace.yaml
│   │   ├── frontend-deployment.yaml
│   │   ├── backend-deployment.yaml
│   │   ├── ingress.yaml
│   │   └── secrets.yaml
│   └── overlays/
│       ├── dev/
│       ├── staging/
│       └── prod/
├── docker/
│   ├── docker-compose.yaml
│   ├── docker-compose.override.yaml
│   └── .env.example
├── .github/
│   └── workflows/
│       ├── deploy-infra.yaml
│       ├── build-frontend.yaml
│       ├── build-backend.yaml
│       └── pr-checks.yaml
├── scripts/
│   ├── bootstrap.sh
│   ├── seed-secrets.sh
│   └── teardown.sh
├── docs/
│   └── runbook.md
└── README.md
```

---

## Prerequisites

| Tool | Version | Purpose |
|---|---|---|
| [Terraform](https://www.terraform.io/) | ≥ 1.6 | Infrastructure provisioning |
| [Docker](https://www.docker.com/) | ≥ 24.x | Container builds & local dev |
| [kubectl](https://kubernetes.io/docs/tasks/tools/) | ≥ 1.28 | Kubernetes cluster management |
| [Helm](https://helm.sh/) | ≥ 3.x | Chart-based K8s deployments |
| [Azure CLI](https://learn.microsoft.com/cli/azure/) | ≥ 2.x | Cloud authentication & management |
| [Node.js](https://nodejs.org/) | ≥ 20 LTS | Frontend build tooling |
| [.NET SDK](https://dotnet.microsoft.com/) | 8.0 | Backend build tooling |

---

## Setup Instructions

### 1. Clone the repository

```bash
git clone https://github.com/dennismwebb/dennis-ai-infra.git
cd dennis-ai-infra
```

### 2. Configure cloud credentials

```bash
az login
az account set --subscription <SUBSCRIPTION_ID>
```

### 3. Initialize Terraform

```bash
cd terraform/environments/dev
terraform init
terraform plan -out=tfplan
terraform apply tfplan
```

### 4. Launch local development stack

```bash
cd docker
cp .env.example .env
# Edit .env with your local configuration
docker compose up -d
```

This starts the full platform locally: Next.js frontend, .NET 8 backend, and any supporting services (database, cache, etc.).

### 5. Seed secrets (first-time setup)

```bash
./scripts/seed-secrets.sh --env dev
```

---

## Development Workflow

### Infrastructure Changes

1. Create a feature branch: `git checkout -b feat/add-cdn-module`
2. Modify Terraform modules or K8s manifests
3. Validate locally:
   ```bash
   terraform validate
   terraform plan
   ```
4. Open a pull request — the `pr-checks.yaml` workflow runs `terraform plan` and linting automatically
5. After approval and merge, the `deploy-infra.yaml` workflow applies changes to the target environment

### Adding a New Service

1. Create a Terraform module under `terraform/modules/` if new cloud resources are required
2. Add a Kubernetes deployment manifest in `k8s/base/`
3. Create Kustomize overlays for each environment in `k8s/overlays/`
4. Update `docker-compose.yaml` for local development parity
5. Add or update the corresponding CI/CD workflow in `.github/workflows/`

### Working with Docker Compose

```bash
# Start all services
docker compose up -d

# Rebuild after code changes
docker compose up -d --build

# View logs
docker compose logs -f <service-name>

# Tear down
docker compose down -v
```

---

## Deployment

### Environments

| Environment | Trigger | Approval |
|---|---|---|
| **dev** | Push to `main` | Automatic |
| **staging** | Manual dispatch / tag | Automatic |
| **prod** | Release tag (`v*.*.*`) | Manual approval required |

### CI/CD Pipeline Flow

```
Push / PR                     Merge to main               Release tag
    │                              │                          │
    ▼                              ▼                          ▼
┌──────────┐              ┌──────────────┐           ┌──────────────┐
│ PR Checks│              │  Deploy Dev  │           │ Deploy Prod  │
│ - lint   │              │  - terraform │           │ - approval   │
│ - plan   │              │  - k8s apply │           │ - terraform  │
│ - test   │              └──────────────┘           │ - k8s apply  │
└──────────┘                                         └──────────────┘
```

### Manual Deployment

```bash
# Apply infrastructure
cd terraform/environments/prod
terraform apply

# Deploy to Kubernetes
kubectl apply -k k8s/overlays/prod/
```

---

## Environment Variables

| Variable | Description | Required |
|---|---|---|
| `ARM_SUBSCRIPTION_ID` | Azure subscription ID | Yes |
| `ARM_TENANT_ID` | Azure AD tenant ID | Yes |
| `ARM_CLIENT_ID` | Service principal client ID | Yes (CI) |
| `ARM_CLIENT_SECRET` | Service principal secret | Yes (CI) |
| `TF_STATE_STORAGE_ACCOUNT` | Terraform remote state backend | Yes |
| `DOCKER_REGISTRY` | Container registry URL | Yes |
| `KUBECONFIG` | Path to Kubernetes config | Local only |

---

## Related Repositories

| Repository | Description |
|---|---|
| [dennis-ai-site](https://github.com/dennismwebb/dennis-ai-site) | Next.js frontend — avatar, chat UI, pages |
| [dennis-ai-backend](https://github.com/dennismwebb/dennis-ai-backend) | .NET 8 API — AI agent orchestration, services |
| [dennis-ai-data](https://github.com/dennismwebb/dennis-ai-data) | AI data assets — prompts, embeddings, knowledge base |

---

## License

This project is private and proprietary. All rights reserved.
