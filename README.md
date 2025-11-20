## 📁 Repository Structure
```
cicd-jenkins-pipelines/
├── jenkins/
│   ├── Jenkinsfile.backend       # Backend CI pipeline
│   ├── Jenkinsfile.frontend      # Frontend CI pipeline
│   └── Jenkinsfile.integration   # Integration test pipeline
├── docker/
│   └── docker-compose.test.yml   # Docker Compose for integration tests
├── scripts/
│   └── (automation scripts - TBD)
└── config/
    └── (configuration files - TBD)
```

## 🔄 Pipeline Flow

### 1. Backend Pipeline (`Jenkinsfile.backend`)
**Triggered by:** Push to `fastapi-backend` repo

**Stages:**
- ✅ Checkout backend code
- ✅ Install Python dependencies
- ✅ Code quality (Ruff, Black)
- ✅ Security scan (Bandit, Safety)
- ✅ Unit tests (Pytest + coverage)
- ✅ Build Docker image
- ✅ Push to Docker registry

**Output:** `aminamrane/fastapi-backend:${GIT_SHA}`

---

### 2. Frontend Pipeline (`Jenkinsfile.frontend`)
**Triggered by:** Push to `react-frontend` repo

**Stages:**
- ✅ Checkout frontend code
- ✅ Install Node dependencies
- ✅ ESLint + Prettier
- ✅ TypeScript check
- ✅ Unit tests (Vitest)
- ✅ Build React app (Vite)
- ✅ Build Docker image
- ✅ Push to Docker registry

**Output:** `aminamrane/react-frontend:${GIT_SHA}`

---

### 3. Integration Pipeline (`Jenkinsfile.integration`)
**Triggered by:** Successful backend + frontend builds

**Stages:**
- ✅ Pull latest Docker images
- ✅ Start all services (Docker Compose)
- ✅ Health checks
- ✅ API integration tests
- ✅ E2E tests (Playwright)
- ✅ Performance tests
- ✅ Tear down environment

**Output:** ✅ Ready for deployment or ❌ Failed integration

---

## 🚀 Jenkins Setup

### Prerequisites
- Jenkins installed with Docker support
- Docker and Docker Compose installed on Jenkins agent
- GitHub webhooks configured

### Jenkins Plugins Required
- Docker Pipeline
- Git
- Pipeline
- Pipeline: Stage View
- Credentials Binding

### Configure Multibranch Pipelines

#### 1. Backend Pipeline
```
Name: fastapi-backend-pipeline
Branch Source: GitHub
Repository: https://github.com/Aminamrane/fastapi-backend
Script Path: Load from another repo
  Repository: https://github.com/Aminamrane/cicd-jenkins-pipelines
  Script Path: jenkins/Jenkinsfile.backend
```

#### 2. Frontend Pipeline
```
Name: react-frontend-pipeline
Branch Source: GitHub
Repository: https://github.com/Aminamrane/react-frontend
Script Path: Load from another repo
  Repository: https://github.com/Aminamrane/cicd-jenkins-pipelines
  Script Path: jenkins/Jenkinsfile.frontend
```

#### 3. Integration Pipeline
```
Name: integration-test-pipeline
Type: Pipeline (not multibranch)
Definition: Pipeline script from SCM
  Repository: https://github.com/Aminamrane/cicd-jenkins-pipelines
  Script Path: jenkins/Jenkinsfile.integration
```

---

## 🔐 Required Jenkins Credentials

| ID | Type | Description |
|----|------|-------------|
| `docker-hub-credentials` | Username/Password | Docker Hub login |
| `github-token` | Secret text | GitHub personal access token |

---

## 🐳 Docker Images

| Service | Image | Registry |
|---------|-------|----------|
| Backend | `aminamrane/fastapi-backend` | Docker Hub |
| Frontend | `aminamrane/react-frontend` | Docker Hub |

---

## 🔗 Related Repositories

- [Backend Application](https://github.com/Aminamrane/fastapi-backend)
- [Frontend Application](https://github.com/Aminamrane/react-frontend)
- Infrastructure (managed by DevOps team)


- [ ] Add Slack notifications
- [ ] Implement deployment stages
