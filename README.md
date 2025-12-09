# CI/CD Jenkins Pipelines - Production Ready

Multi-repository CI/CD architecture for AWS (EKS, ECR).

## Architecture

```
Backend/Frontend → Build → Security Scan → Push ECR → Trigger Helm
                                                          ↓
Terraform → Apply AWS infra ←────────────────── Helm Deploy → EKS
```

## Repository Structure

```
cicd-jenkins-pipelines/
├── charts/
│   ├── backend/          # Backend Helm chart
│   ├── frontend/         # Frontend Helm chart
│   ├── values-dev.yaml   # Development values
│   └── values-prod.yaml  # Production values
└── jenkins/
    ├── Jenkinsfile.backend    # Backend pipeline
    ├── Jenkinsfile.frontend   # Frontend pipeline
    ├── Jenkinsfile.helm       # Helm deployment pipeline
    └── Jenkinsfile.terraform  # Infrastructure pipeline
```

## Pipelines

### Backend/Frontend Pipeline
- Checkout code
- Run unit tests
- Build Docker image
- Security scan (Trivy)
- Push to ECR (prod) or Docker Hub (dev)
- Trigger Helm deployment

### Helm Pipeline
- Helm lint and validate
- Configure kubectl for EKS
- Deploy with helm upgrade --install
- Verify deployment

### Terraform Pipeline
- Terraform init/validate/plan
- Manual approval for production
- Terraform apply
- Trigger Helm after infra changes

## Parameters

### Backend/Frontend
| Parameter | Options | Description |
|-----------|---------|-------------|
| ENVIRONMENT | dev, staging, prod | Target environment |
| USE_ECR | true/false | Use AWS ECR instead of Docker Hub |
| SECURITY_SCAN | true/false | Run Trivy security scan |

### Helm
| Parameter | Options | Description |
|-----------|---------|-------------|
| SERVICE | all, backend, frontend | Service to deploy |
| IMAGE_TAG | string | Docker image tag |
| ENVIRONMENT | dev, staging, prod | Target environment |
| DRY_RUN | true/false | Preview only |

## AWS Integration

Environment variables to set in Jenkins:
- AWS_ACCOUNT_ID: AWS account number
- AWS_REGION: AWS region (default: eu-west-1)
- EKS_CLUSTER: EKS cluster name

## Pipeline Communication

```
Backend Pipeline ──> Helm (SERVICE=backend)
Frontend Pipeline ─> Helm (SERVICE=frontend)
Terraform Pipeline ─> Helm (SERVICE=all)
```

## Team Responsibilities

- CI/CD: Pipeline configuration (this repo)
- Infra: Terraform code (terraform-infrastructure repo)
- K8s: Kubernetes manifests / Helm values
# Test webhook
