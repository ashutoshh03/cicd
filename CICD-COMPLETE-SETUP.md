# Complete CI/CD Infrastructure - Documentation

This project contains a comprehensive CI/CD infrastructure implementing all tasks and phases for automated deployment across multiple environments and platforms.

## 📋 Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Project Structure](#project-structure)
3. [Tasks Overview](#tasks-overview)
4. [Phase 2 Overview](#phase-2-overview)
5. [Setup and Configuration](#setup-and-configuration)
6. [Deployment Instructions](#deployment-instructions)
7. [Monitoring and Troubleshooting](#monitoring-and-troubleshooting)
8. [Security Considerations](#security-considerations)

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    GitHub Repository                             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ├─────────────────────────┐
                              │                         │
                    ┌─────────▼──────────┐    ┌────────▼─────────┐
                    │  GitHub Actions    │    │   Jenkins        │
                    │  (ci-cd.yml)       │    │  (Jenkinsfile)   │
                    └─────────┬──────────┘    └────────┬─────────┘
                              │                        │
          ┌───────────────────┼───────────────────────┬┘
          │                   │                       │
    ┌─────▼──────┐     ┌─────▼──────┐       ┌────────▼──────┐
    │  Build &   │     │  Docker    │       │  Kubernetes   │
    │  Test      │     │  Image     │       │  Deployment   │
    │            │     │            │       │               │
    └─────┬──────┘     └─────┬──────┘       └────────┬───────┘
          │                  │                       │
          └──────────────────┼───────────────────────┘
                             │
                    ┌────────▼─────────┐
                    │ Kubernetes       │
                    │ Minikube         │
                    └────────┬────────┘
                             │
                    ┌────────▼─────────┐
                    │ Windows VM       │
                    │ Ansible + WinRM  │
                    └──────────────────┘
```

## 📁 Project Structure

```
cicd/
├── .github/
│   └── workflows/
│       ├── ci-cd.yml                    # Task 1: GitHub Actions pipeline
│       ├── deploy-windows.yml           # Task 3: Windows VM deployment
│   ├── inventory.yml                    # Windows VM inventory
│   └── playbook.yml                     # Windows deployment playbook
├── helm/
│   ├── hello-world-chart/               # Phase 2.1: Helm chart
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   └── templates/
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       ├── configmap.yaml
│   │       ├── _helpers.tpl
│   │       └── NOTES.txt
│   ├── values-dev.yaml                  # Dev environment values
│   ├── values-qa.yaml                   # QA environment values
│   ├── values-prod.yaml                 # Production environment values
│   └── README.md                        # Helm chart documentation
├── k8s/
│   ├── deployment.yaml                  # Kubernetes deployment
│   └── service.yaml                     # Kubernetes service
├── app.js                               # Node.js application
├── package.json                         # Dependencies
├── Dockerfile                           # Docker configuration
├── Jenkinsfile                          # Task 2: Jenkins pipeline
└── README.md                            # This file
```

## 📚 Tasks Overview

### Task 1: GitHub Actions CI/CD Pipeline ✅

**Pipeline Flow:** Build → Test → Artifact → Docker → Kubernetes

**File:** [.github/workflows/ci-cd.yml](.github/workflows/ci-cd.yml)

**Features:**
- Manual, push, and pull-request triggers for main/master
- Node.js 22 setup and dependency installation
- Automated testing with npm test
- Application artifact creation (ZIP format)
- Docker image building and upload as a workflow artifact
- Image tagging with the commit SHA
- Minikube Kubernetes deployment
- Automatic rollout verification
- Health check verification
- Service endpoint testing

**Workflow Stages:**
1. **Build and Test** - Runs npm install, test, and build
2. **Build Docker** - Creates Docker image with versioning
3. **Deploy Kubernetes** - Deploys to Minikube with 2 replicas

**Secrets Required:** None (the workflow uses GitHub-hosted resources)

---

### Task 2: Jenkins Pipeline ✅

**Pipeline Flow:** Build → Docker → Kubernetes (Minikube) → Self-Healing Demo

**File:** [Jenkinsfile](Jenkinsfile)

**Features:**
- Environment verification (Node.js, npm, Docker, kubectl, Minikube)
- Automated Minikube startup if not running
- Dependency installation
- Comprehensive testing
- Application build
- Docker image creation inside Minikube
- Kubernetes deployment with image update
- Deployment verification
- **Self-healing demonstration:**
  - Pod deletion
  - Automatic pod recreation by Kubernetes
  - Verification of new pod creation
  - Deployment status check

**Pipeline Stages:**
1. Checkout
2. Verify Environment
3. Install Dependencies
4. Unit Test
5. Build Application
6. Archive Artifact
7. Build Docker Image
8. Deploy to Kubernetes
9. Verify Deployment
10. Verify Self-Healing

**Requirements:**
- Jenkins agent with Docker, Node.js, Minikube
- kubectl configured for Minikube

---

### Task 3: GitHub Actions Windows VM Deployment ✅

**Flow:** GitHub Actions → Ansible → WinRM + Certificate → Windows VM

**File:** [.github/workflows/deploy-windows.yml](.github/workflows/deploy-windows.yml)

**Features:**
- WinRM connection with certificate authentication
- Ansible playbook execution
- Certificate file management
- Secret validation
- Connectivity verification
- Comprehensive error handling
- Deployment summary and troubleshooting guide

**Ansible Playbook:** [ansible/playbook.yml](ansible/playbook.yml)

**Playbook Features:**
- Node.js prerequisite validation
- Application directory creation
- File transfer to Windows VM
- NPM dependency installation
- Process management (stop existing, start new)
- Application startup verification
- Health endpoint testing
- Process monitoring
- Comprehensive logging

**Secrets Required:**
- `WINDOWS_VM_HOST` - IP/hostname of Windows VM
- `WINDOWS_USERNAME` - Windows VM username
- `WINRM_CERT` - WinRM client certificate (PEM)
- `WINRM_KEY` - WinRM client private key (PEM)

**Setup Instructions:**
1. Enable WinRM on Windows VM:
   ```powershell
   Enable-PSRemoting -Force
   ```
2. Create and install certificates for certificate-based authentication
3. Add GitHub secrets with certificate and connection details

---

## 🚀 Phase 2 Overview

### Phase 2.1: Helm Integration for Kubernetes ✅

**Directory:** [helm/](helm/)

**Features:**
- Production-ready Helm chart structure
- Support for multiple environments (Dev, QA, Prod)
- Environment-specific values files
- Comprehensive resource management
- Health checks configuration
- Horizontal Pod Autoscaler (HPA) support
- Ingress configuration
- ConfigMap management
- Service account management
- Security contexts
- Pod Disruption Budget

**Files:**
- **Chart:** [helm/hello-world-chart/Chart.yaml](helm/hello-world-chart/Chart.yaml)
- **Base Values:** [helm/hello-world-chart/values.yaml](helm/hello-world-chart/values.yaml)
- **Dev Environment:** [helm/values-dev.yaml](helm/values-dev.yaml)
- **QA Environment:** [helm/values-qa.yaml](helm/values-qa.yaml)
- **Production Environment:** [helm/values-prod.yaml](helm/values-prod.yaml)

**Environment Comparison:**

| Feature | Dev | QA | Prod |
|---------|-----|----|----|
| Replicas | 1 | 2 | 3 |
| HPA Enabled | No | Yes (5 max) | Yes (10 max) |
| Resource CPU Limit | 200m | 300m | 1000m |
| Resource Memory Limit | 256Mi | 512Mi | 1Gi |
| Ingress Enabled | Yes | Yes | Yes |
| Security Level | Low | Medium | High |
| Startup Probe | No | No | Yes |
| PDB Enabled | No | Yes | Yes |

**Helm Chart Templates:**
- `deployment.yaml` - Kubernetes deployment with probes
- `service.yaml` - Service configuration
- `configmap.yaml` - Application configuration
- `_helpers.tpl` - Helper templates
- `NOTES.txt` - Post-deployment instructions

**Installation Examples:**

```bash
# Development
helm install hello-world ./helm/hello-world-chart \
  -f ./helm/values-dev.yaml \
  -n dev --create-namespace

# QA
helm install hello-world ./helm/hello-world-chart \
  -f ./helm/values-qa.yaml \
  -n qa --create-namespace

# Production
helm install hello-world ./helm/hello-world-chart \
  -f ./helm/values-prod.yaml \
  -n prod --create-namespace
```

---

### Phase 2.2: Linux VM Deployment

Linux VM deployment is not part of the current repository. The active remote VM deployment target is Windows via Ansible and WinRM.

---

## 🔧 Setup and Configuration

### Prerequisites

**Global Requirements:**
- Git and GitHub account
- Docker installed locally
- kubectl installed
- Minikube installed

**For GitHub Actions:**
- GitHub repository with Actions enabled
- GitHub secrets configured

**For Jenkins:**
- Jenkins instance running
- Jenkins agent with required tools

**For Windows VM (Task 3):**
- Windows Server 2016+ with PowerShell 5.1+
- WinRM enabled
- OpenSSL for certificate generation
- Network connectivity from CI/CD server

### GitHub Secrets Configuration

#### Task 3 - Windows Deployment

```bash
# Generate certificates (on your local machine)
# Then add to GitHub Secrets:
WINDOWS_VM_HOST=<Windows VM IP or hostname>
WINDOWS_USERNAME=<Windows username>
WINRM_CERT=<contents of client.pem>
WINRM_KEY=<contents of client-key.pem>
```

### Application Configuration

**Environment Variables:**

| Variable | Default | Description |
|----------|---------|-------------|
| PORT | 3000 | Application port |

---

## 📤 Deployment Instructions

### Task 1: GitHub Actions Pipeline

```bash
# 1. Commit and push to main branch
git add .
git commit -m "Trigger CI/CD pipeline"
git push origin main

# 2. Monitor in GitHub Actions tab
# Pipeline will automatically:
# - Build and test
# - Create Docker image
# - Deploy to Minikube
# - Verify deployment
```

### Task 2: Jenkins Pipeline

```bash
# 1. Ensure Jenkins agent is configured
# 2. Create new Jenkins job from Jenkinsfile
# 3. Configure job to poll GitHub repository
# 4. Trigger manually or on commit
# 5. Monitor job execution and logs

# View pipeline stages and self-healing demo in console output
```

### Task 3: Windows VM Deployment

```bash
# 1. Ensure GitHub secrets are configured
# 2. Commit and push changes to main branch
# 3. GitHub Actions workflow will:
#    - Trigger deployment-windows workflow
#    - Configure WinRM certificates
#    - Execute Ansible playbook
#    - Verify deployment

# Manual trigger:
# - Go to GitHub Actions tab
# - Select "Deploy to Windows VM" workflow
# - Click "Run workflow"
```

### Phase 2.1: Helm Deployment

```bash
# Development
helm install hello-world ./helm/hello-world-chart \
  -f ./helm/values-dev.yaml \
  -n dev --create-namespace

# Verify
kubectl get all -n dev

# Upgrade
helm upgrade hello-world ./helm/hello-world-chart \
  -f ./helm/values-dev.yaml \
  -n dev

# Uninstall
helm uninstall hello-world -n dev
```

---

## 📊 Monitoring and Troubleshooting

### GitHub Actions

```bash
# View workflow run
# https://github.com/your-repo/actions

# View detailed logs
# Click on workflow run → Click on job → View logs

# Troubleshoot failures
# 1. Check workflow logs
# 2. Verify secrets are configured
# 3. Check runner environment
```

### Jenkins

```bash
# View console output
# Jenkins Dashboard → Job Name → Build Number → Console Output

# Common issues:
# - Minikube not running: Check agent logs
# - Docker not available: Install Docker on agent
# - kubectl not configured: Configure kubeconfig on agent
```

### Kubernetes (Minikube)

```bash
# Check deployment status
kubectl get deployment hello-world -o wide
kubectl describe deployment hello-world

# Check pod status
kubectl get pods -l app=hello-world
kubectl describe pod <pod-name>

# View logs
kubectl logs -f deployment/hello-world

# Port forward and test
kubectl port-forward svc/hello-world 3000:80
curl http://localhost:3000/health

# Monitor self-healing
kubectl get pods -l app=hello-world --watch
# Delete a pod in another terminal to watch it recreate
```

### Windows VM

```powershell
# Check WinRM status
Get-WSManInstance winrm/config

# View running processes
Get-Process node

# Check application directory
Get-ChildItem C:\apps\hello-world

# View logs
Get-Content C:\apps\hello-world\app.log

# Test application
Invoke-WebRequest http://localhost:3000/health
```

### Helm Chart

```bash
# Validate chart
helm lint ./helm/hello-world-chart

# Dry run deployment
helm install hello-world ./helm/hello-world-chart \
  -f ./helm/values-dev.yaml \
  -n dev --create-namespace --dry-run --debug

# Get values
helm get values hello-world -n dev

# Get all resources
helm get all hello-world -n dev

# Check deployment status
kubectl rollout status deployment/hello-world -n dev

# View pod logs
kubectl logs -f -l app=hello-world -n dev
```

---

## 🔒 Security Considerations

### GitHub Secrets Management

- ✅ Store all credentials in GitHub Secrets
- ✅ Use fine-grained access tokens
- ✅ Rotate credentials regularly
- ✅ Audit secret access

### Windows VM Security

- ✅ Use certificate-based authentication (not just passwords)
- ✅ Restrict WinRM to specific hosts
- ✅ Use HTTPS for WinRM (port 5986)
- ✅ Keep Windows updates current

### Kubernetes Security

- ✅ Use RBAC for pod service accounts
- ✅ Set resource limits and requests
- ✅ Enable security contexts
- ✅ Use network policies
- ✅ Enable pod security policies

### Docker Security

- ✅ Use specific image tags (not latest)
- ✅ Scan images for vulnerabilities
- ✅ Use minimal base images
- ✅ Keep base images updated

---

## 📝 Common Operations

### View All Workflow Runs

```bash
# GitHub CLI
gh run list

# List with filtering
gh run list --workflow ci-cd.yml
gh run list --status failure
```

### Retry a Failed Workflow

```bash
# GitHub UI
# Actions → Workflow → Failed run → Re-run jobs

# GitHub CLI
gh run rerun <run-id>
```

### Update Application Version

```bash
# Update image tag
helm upgrade hello-world ./helm/hello-world-chart \
  -f ./helm/values-prod.yaml \
  --set image.tag=v1.0.1 \
  -n prod
```

### Scale Application

```bash
# Using kubectl
kubectl scale deployment hello-world --replicas=5 -n prod

# Using Helm
helm upgrade hello-world ./helm/hello-world-chart \
  -f ./helm/values-prod.yaml \
  --set replicaCount=5 \
  -n prod
```

### Rollback Deployment

```bash
# Helm rollback
helm rollback hello-world 1 -n prod

# Kubernetes rollback
kubectl rollout undo deployment/hello-world -n prod
```

---

## 📞 Support and Documentation

- **GitHub Actions:** https://docs.github.com/en/actions
- **Jenkins:** https://www.jenkins.io/doc/
- **Ansible:** https://docs.ansible.com/
- **Kubernetes:** https://kubernetes.io/docs/
- **Helm:** https://helm.sh/docs/
- **Docker:** https://docs.docker.com/

---

## ✅ Checklist for Complete Setup

- [ ] GitHub repository created and configured
- [ ] GitHub Secrets added (Windows credentials, if using Task 3)
- [ ] Local environment setup (Docker, kubectl, Minikube)
- [ ] Jenkins instance configured (if using Task 2)
- [ ] Windows VM configured with WinRM and certificates (Task 3)
- [ ] Test the GitHub Actions workflows (Task 1 and Task 3, if enabled)
- [ ] Test Jenkins pipeline (Task 2)
- [ ] Test Windows deployment (Task 3)
- [ ] Test Helm deployment (Phase 2.1)
- [ ] Verify monitoring and logging setup
- [ ] Document environment-specific configurations
- [ ] Setup alerts and notifications (optional)

---

## 📄 Version History

- **v1.0.0** - Initial implementation
  - Task 1: GitHub Actions CI/CD
  - Task 2: Jenkins Pipeline
  - Task 3: Windows VM Deployment
  - Phase 2.1: Helm Charts

---

This CI/CD infrastructure automates building, testing, and deploying the application to Kubernetes and, optionally, a Windows VM.
