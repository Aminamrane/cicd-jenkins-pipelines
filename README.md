# CI/CD & Helm Charts Repository

Multi-repository CI/CD architecture with Helm charts for Kubernetes deployments.

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Multi-Repository CI/CD Architecture                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌──────────────┐     ┌──────────────┐     ┌──────────────────────────┐   │
│   │   Backend    │     │   Frontend   │     │      Terraform           │   │
│   │   Repository │     │   Repository │     │      Repository          │   │
│   │              │     │              │     │                          │   │
│   │  FastAPI +   │     │  React +     │     │  Infrastructure as Code  │   │
│   │  Python      │     │  TypeScript  │     │  (AWS/GCP/Azure)         │   │
│   └──────┬───────┘     └──────┬───────┘     └────────────┬─────────────┘   │
│          │                    │                          │                  │
│          ▼                    ▼                          ▼                  │
│   ┌──────────────┐     ┌──────────────┐     ┌──────────────────────────┐   │
│   │  Jenkins     │     │  Jenkins     │     │  Jenkins                 │   │
│   │  Backend     │     │  Frontend    │     │  Terraform               │   │
│   │  Pipeline    │     │  Pipeline    │     │  Pipeline                │   │
│   │              │     │              │     │                          │   │
│   │ • Unit Tests │     │ • Unit Tests │     │ • Terraform Plan         │   │
│   │ • SonarQube  │     │ • SonarQube  │     │ • Terraform Apply        │   │
│   │ • Docker     │     │ • Docker     │     │ • Auto-trigger on        │   │
│   │   Build/Push │     │   Build/Push │     │   .tf changes            │   │
│   └──────┬───────┘     └──────┬───────┘     └────────────┬─────────────┘   │
│          │                    │                          │                  │
│          │   ┌────────────────┴─────┐                    │                  │
│          │   │                      │                    │                  │
│          ▼   ▼                      ▼                    ▼                  │
│   ┌──────────────────────────────────────────────────────────────────────┐ │
│   │                     THIS REPOSITORY                                   │ │
│   │                  cicd-jenkins-pipelines                               │ │
│   │                                                                       │ │
│   │  ┌─────────────────┐  ┌─────────────────┐  ┌───────────────────┐    │ │
│   │  │  Helm Charts    │  │  Jenkins        │  │  Docker           │    │ │
│   │  │                 │  │  Pipelines      │  │  Compose          │    │ │
│   │  │  charts/        │  │                 │  │                   │    │ │
│   │  │  ├── backend/   │  │  jenkins/       │  │  docker/          │    │ │
│   │  │  ├── frontend/  │  │  ├── Backend    │  │  └── test.yml     │    │ │
│   │  │  ├── values-dev │  │  ├── Frontend   │  │                   │    │ │
│   │  │  └── values-prod│  │  ├── Helm       │  │                   │    │ │
│   │  │                 │  │  └── Integration│  │                   │    │ │
│   │  └─────────────────┘  └─────────────────┘  └───────────────────┘    │ │
│   └──────────────────────────────────────────────────────────────────────┘ │
│                                      │                                      │
│                                      ▼                                      │
│                        ┌──────────────────────────┐                        │
│                        │    Kubernetes Cluster    │                        │
│                        │                          │                        │
│                        │  ┌─────────┐ ┌─────────┐ │                        │
│                        │  │ Backend │ │Frontend │ │                        │
│                        │  │  Pods   │ │  Pods   │ │                        │
│                        │  └─────────┘ └─────────┘ │                        │
│                        │                          │                        │
│                        │       Traefik Ingress    │                        │
│                        └──────────────────────────┘                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 📁 Repository Structure

```
cicd-jenkins-pipelines/
├── charts/                         # Helm charts for Kubernetes
│   ├── backend/                    # Backend FastAPI chart
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       ├── ingress.yaml
│   │       ├── hpa.yaml
│   │       └── _helpers.tpl
│   ├── frontend/                   # Frontend React chart
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   │       └── ...
│   ├── values-dev.yaml             # Dev environment overrides
│   └── values-prod.yaml            # Prod environment overrides
├── jenkins/                        # Jenkins pipeline definitions
│   ├── Jenkinsfile.backend         # Backend CI/CD pipeline
│   ├── Jenkinsfile.frontend        # Frontend CI/CD pipeline
│   ├── Jenkinsfile.helm            # Helm deployment pipeline
│   └── Jenkinsfile.integration     # Integration test pipeline
├── docker/                         # Docker configurations
│   └── docker-compose.test.yml     # Integration testing
└── README.md
```

## 🔄 Pipeline Flow

### Pipeline Communication

```
Backend Repo Push  ──────►  Backend Pipeline  ──────►  Helm Pipeline (backend only)
                               │                              │
                               ├── Unit Tests                 ├── Deploy to K8s
                               ├── SonarQube                  └── Health Check
                               ├── Docker Build/Push
                               └── Trigger Helm ─────────────────►

Frontend Repo Push ──────►  Frontend Pipeline ──────►  Helm Pipeline (frontend only)
                               │                              │
                               ├── Unit Tests                 ├── Deploy to K8s
                               ├── SonarQube                  └── Health Check
                               ├── Docker Build/Push
                               └── Trigger Helm ─────────────────►

Terraform Changes  ──────►  Terraform Pipeline ─────►  Helm Pipeline (all services)
                               │                              │
                               ├── Terraform Plan             └── Redeploy All
                               ├── Approval (prod)
                               └── Terraform Apply
```

### 1. Backend Pipeline (`Jenkinsfile.backend`)

**Triggered by:** Push to `fastapi-backend` repository

| Stage | Description |
|-------|-------------|
| Checkout | Clone backend repository |
| Setup | Install Python dependencies (uv/pip) |
| Quality | Ruff linting, formatting, Bandit security scan |
| Test | Pytest with coverage (>80% target) |
| SonarQube | Code quality analysis |
| Build | Docker image with versioned tag |
| Push | Push to Docker Hub |
| Deploy | Trigger Helm pipeline (backend only) |

**Output:** `aminamrane/fastapi-backend:v1.0.{build}-{commit}`

---

### 2. Frontend Pipeline (`Jenkinsfile.frontend`)

**Triggered by:** Push to `react-frontend` repository

| Stage | Description |
|-------|-------------|
| Checkout | Clone frontend repository |
| Setup | npm ci |
| Quality | ESLint, TypeScript check, Prettier |
| Test | Vitest with coverage |
| SonarQube | Code quality analysis |
| Build | Vite production build |
| Docker | Build multi-stage Docker image |
| Push | Push to Docker Hub |
| Deploy | Trigger Helm pipeline (frontend only) |

**Output:** `aminamrane/react-frontend:v1.0.{build}-{commit}`

---

### 3. Helm Pipeline (`Jenkinsfile.helm`)

**Triggered by:** Backend/Frontend pipelines or manual trigger

| Parameter | Description |
|-----------|-------------|
| `SERVICE` | `backend`, `frontend`, or `all` |
| `IMAGE_TAG` | Docker image tag to deploy |
| `ENVIRONMENT` | `dev`, `staging`, or `prod` |
| `DRY_RUN` | Preview changes without applying |

| Stage | Description |
|-------|-------------|
| Checkout | Clone this repository |
| Validate | Check parameters, block latest→prod |
| Setup | Verify Kubernetes connectivity |
| Lint | Validate Helm charts |
| Deploy | `helm upgrade --install` |
| Verify | Check deployment status |
| Health | Rollout status check |

---

### 4. Integration Pipeline (`Jenkinsfile.integration`)

**Triggered by:** Successful backend + frontend builds

| Stage | Description |
|-------|-------------|
| Setup | Start all services (Docker Compose) |
| Health | Wait for services to be ready |
| API Tests | Test backend endpoints |
| E2E Tests | Playwright browser tests |
| Performance | Load testing |
| Cleanup | Tear down test environment |

---

## 🐳 Docker Image Tagging Strategy

```
Image Tags:
├── v1.0.{BUILD_NUMBER}-{GIT_SHA}   # Semantic version (preferred for prod)
├── {GIT_SHA}                        # Short commit hash (for traceability)
└── latest                           # Latest successful build (dev only)

Example:
├── aminamrane/fastapi-backend:v1.0.42-abc1234
├── aminamrane/fastapi-backend:abc1234
└── aminamrane/fastapi-backend:latest
```

## 📊 SonarQube Integration

Both backend and frontend pipelines integrate with SonarQube for:

- **Code Coverage** - Minimum 80% target
- **Code Smells** - Technical debt detection
- **Security Vulnerabilities** - OWASP detection
- **Duplications** - Copy-paste detection
- **Quality Gate** - Pass/fail threshold

### Required Jenkins Credentials

| ID | Type | Description |
|----|------|-------------|
| `sonarqube-url` | Secret text | SonarQube server URL |
| `sonarqube-token` | Secret text | SonarQube authentication token |

## 🔐 Required Jenkins Credentials

| ID | Type | Description |
|----|------|-------------|
| `docker-hub-credentials` | Username/Password | Docker Hub login |
| `github-token` | Secret text | GitHub personal access token |
| `kubeconfig-credential` | Secret file | Kubernetes cluster config |
| `sonarqube-url` | Secret text | SonarQube server URL |
| `sonarqube-token` | Secret text | SonarQube auth token |

## 🚀 Jenkins Setup

### Prerequisites

- Jenkins 2.x with Pipeline plugin
- Docker and kubectl installed on agents
- Helm 3.x installed
- SonarQube server (optional)

### Required Jenkins Plugins

- Docker Pipeline
- Git
- Pipeline
- Pipeline: Stage View
- Credentials Binding
- SonarQube Scanner
- HTML Publisher
- JUnit

### Configure Jenkins Jobs

#### 1. Backend Pipeline
```
Name: fastapi-backend-pipeline
Type: Multibranch Pipeline
Branch Source: GitHub
Repository: https://github.com/Aminamrane/fastapi-backend
Script Path: jenkins/Jenkinsfile.backend (from this repo)
```

#### 2. Frontend Pipeline
```
Name: react-frontend-pipeline
Type: Multibranch Pipeline
Branch Source: GitHub
Repository: https://github.com/Aminamrane/react-frontend
Script Path: jenkins/Jenkinsfile.frontend (from this repo)
```

#### 3. Helm Deploy Pipeline
```
Name: helm-deploy
Type: Pipeline
Definition: Pipeline script from SCM
Repository: https://github.com/Aminamrane/cicd-jenkins-pipelines
Script Path: jenkins/Jenkinsfile.helm
```

#### 4. Integration Pipeline
```
Name: integration-tests
Type: Pipeline
Definition: Pipeline script from SCM
Repository: https://github.com/Aminamrane/cicd-jenkins-pipelines
Script Path: jenkins/Jenkinsfile.integration
```

## 📦 Helm Charts Usage

### Deploy to Development
```bash
# Backend only
helm upgrade --install app-backend charts/backend \
  --namespace app-dev \
  --values charts/values-dev.yaml \
  --set image.tag=v1.0.42-abc1234

# Frontend only
helm upgrade --install app-frontend charts/frontend \
  --namespace app-dev \
  --values charts/values-dev.yaml \
  --set image.tag=v1.0.42-abc1234
```

### Deploy to Production
```bash
# All services
helm upgrade --install app-backend charts/backend \
  --namespace app-prod \
  --values charts/values-prod.yaml \
  --set image.tag=v1.0.42-abc1234

helm upgrade --install app-frontend charts/frontend \
  --namespace app-prod \
  --values charts/values-prod.yaml \
  --set image.tag=v1.0.42-abc1234
```

## 🔗 Related Repositories

| Repository | Description | URL |
|------------|-------------|-----|
| Backend | FastAPI application | [fastapi-backend](https://github.com/Aminamrane/fastapi-backend) |
| Frontend | React application | [react-frontend](https://github.com/Aminamrane/react-frontend) |
| Terraform | Infrastructure as Code | [terraform-infrastructure](https://github.com/Aminamrane/terraform-infrastructure) |

## 📝 Environment Variables

### Backend Pipeline
| Variable | Description |
|----------|-------------|
| `DOCKER_IMAGE` | Docker image name |
| `SONAR_PROJECT_KEY` | SonarQube project identifier |

### Frontend Pipeline
| Variable | Description |
|----------|-------------|
| `VITE_API_URL` | Backend API URL for production |
| `DOCKER_IMAGE` | Docker image name |

### Helm Pipeline
| Variable | Description |
|----------|-------------|
| `HELM_NAMESPACE` | Target Kubernetes namespace |
| `KUBECONFIG` | Path to kubeconfig |

## 🛡️ Best Practices Implemented

- ✅ **Modular Pipelines** - Each repo has its own pipeline
- ✅ **Loose Coupling** - Pipelines communicate via triggers, not dependencies
- ✅ **Versioned Tags** - Semantic versioning for Docker images
- ✅ **Environment Variables** - Reusable configurations
- ✅ **Quality Gates** - SonarQube integration
- ✅ **Independent Deployments** - Each service deploys independently
- ✅ **Rollback Support** - Helm's atomic deployments
- ✅ **Parallel Stages** - Faster pipeline execution
