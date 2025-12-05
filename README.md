# CI/CD Jenkins Pipelines

Multi-repository CI/CD architecture with Jenkins pipelines and Helm charts.

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

### Backend Pipeline (Jenkinsfile.backend)
- Checkout code from fastapi-backend repository
- Build Docker image with versioned tags
- Push image to Docker Hub
- Trigger Helm deployment for backend only

### Frontend Pipeline (Jenkinsfile.frontend)
- Checkout code from react-frontend repository
- Build Docker image with versioned tags
- Push image to Docker Hub
- Trigger Helm deployment for frontend only

### Helm Pipeline (Jenkinsfile.helm)
- Deploy applications using Helm charts
- Supports selective deployment (backend, frontend, or all)
- Environment-specific configurations (dev, staging, prod)
- Dry run mode for previewing changes

### Terraform Pipeline (Jenkinsfile.terraform)
- Manage infrastructure with Terraform
- Plan, apply, or destroy actions
- Manual approval for production
- Auto-triggers Helm after apply

## Docker Images

- Backend: `aminamr/fastapi-backend`
- Frontend: `aminamr/react-frontend`

Image tags: `v1.0.{build}-{commit}` and `latest`

## Jenkins Setup

### Required Credentials
1. `github-credentials`: GitHub username + token
2. `docker-hub-credentials`: Docker Hub username + password

### Jobs
1. `fastapi-backend-pipeline`: Points to `jenkins/Jenkinsfile.backend`
2. `react-frontend-pipeline`: Points to `jenkins/Jenkinsfile.frontend`
3. `helm-deploy`: Points to `jenkins/Jenkinsfile.helm`
4. `terraform-infrastructure`: Points to `jenkins/Jenkinsfile.terraform`

## Pipeline Communication

```
Backend Pipeline ──────> Helm Pipeline (SERVICE=backend)
                              │
Frontend Pipeline ─────> Helm Pipeline (SERVICE=frontend)
                              │
Terraform Pipeline ────> Helm Pipeline (SERVICE=all)
```

Each application pipeline triggers Helm to deploy only its own service.
