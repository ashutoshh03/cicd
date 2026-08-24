# CI/CD Quick Reference Guide

## 🚀 Quick Start

### GitHub Actions Deployment (Task 1)

```bash
# 1. Configure repository
git config user.name "Your Name"
git config user.email "your.email@example.com"

# 2. Push to main branch to trigger workflow
git add .
git commit -m "Deploy application"
git push origin main

# 3. Monitor workflow
# https://github.com/your-org/your-repo/actions
```

### Jenkins Deployment (Task 2)

```bash
# 1. Access Jenkins
# http://localhost:8080

# 2. Create new Pipeline job from GitHub repository
# Configure Pipeline script from SCM: Jenkinsfile

# 3. Trigger build
# Click "Build Now" or commit to trigger automatically

# 4. Monitor execution
# View Console Output for detailed logs
```

### Windows VM Deployment (Task 3)

```bash
# 1. Add GitHub Secrets (in GitHub Settings → Secrets)
WINDOWS_VM_HOST=<VM IP>
WINDOWS_USERNAME=<username>
WINDOWS_PASSWORD=<password>
WINRM_CERT=<certificate>
WINRM_KEY=<key>

# 2. Push to main branch
git push origin main

# 3. Workflow automatically triggers
# OR manually trigger from Actions tab
```

### Linux VM Deployment (Phase 2.2)

```bash
# 1. Add GitHub Secrets
LINUX_VM_HOST=<VM IP>
LINUX_VM_USER=<username>
LINUX_VM_SSH_KEY=<private key>

# 2. Push to main branch
git push origin main

# 3. Monitor deployment
# https://github.com/your-org/your-repo/actions
```

### Helm Deployment (Phase 2.1)

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

## 📋 Essential Commands

### Kubernetes Operations

```bash
# Check deployment status
kubectl get deployment hello-world -o wide

# View all pods
kubectl get pods -l app=hello-world

# View pod logs
kubectl logs -f deployment/hello-world

# Describe deployment
kubectl describe deployment hello-world

# Port forward for testing
kubectl port-forward svc/hello-world 3000:80

# Test application
curl http://localhost:3000/
curl http://localhost:3000/health

# Scale deployment
kubectl scale deployment hello-world --replicas=5

# Restart deployment
kubectl rollout restart deployment/hello-world
```

### Helm Operations

```bash
# List releases
helm list -n dev

# Get values
helm get values hello-world -n dev

# Upgrade release
helm upgrade hello-world ./helm/hello-world-chart \
  -f ./helm/values-dev.yaml \
  -n dev

# Rollback
helm rollback hello-world 1 -n dev

# Uninstall
helm uninstall hello-world -n dev

# Validate chart
helm lint ./helm/hello-world-chart

# Dry run
helm install hello-world ./helm/hello-world-chart \
  -f ./helm/values-dev.yaml \
  --dry-run --debug
```

### Windows VM Operations

```powershell
# Check WinRM
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

### Linux VM Operations

```bash
# SSH into VM
ssh user@linux-vm-host

# Check process
ps aux | grep node

# View logs
tail -f /opt/hello-world/application.log

# Test endpoints
curl http://localhost:3000/
curl http://localhost:3000/health

# Check running port
lsof -i :3000
netstat -tlnp | grep 3000
```

---

## 🔍 Troubleshooting

### GitHub Actions Failures

```bash
# 1. Check workflow logs
# Actions → Workflow → Run → Job → Logs

# 2. Verify secrets
# Settings → Secrets → Review all secrets

# 3. Check runner logs
# Actions → Workflow → Run → Logs

# 4. Re-run workflow
# Click "Re-run jobs" or "Re-run all jobs"
```

### Kubernetes Pod Issues

```bash
# Pod not starting
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs <pod-name> --previous

# Pod CrashLoopBackOff
kubectl get events -n default
kubectl logs <pod-name> --tail=50

# Port not accessible
kubectl port-forward svc/hello-world 3000:80
# Test from another terminal
curl http://localhost:3000/
```

### Windows VM Connection Issues

```powershell
# Test WinRM
Test-WSMan <vm-ip> -Port 5986

# Check firewall
Get-NetFirewallRule -DisplayName "Windows Remote*"

# Enable WinRM
Enable-PSRemoting -Force

# Test connectivity with credentials
$cred = Get-Credential
Test-WSMan <vm-ip> -Port 5986 -Authentication Basic -Credential $cred
```

### Linux VM Connection Issues

```bash
# Test SSH
ssh -v user@linux-vm-host

# Check SSH key permissions
ls -la ~/.ssh/id_rsa
chmod 600 ~/.ssh/id_rsa

# Check SSH service
sudo systemctl status ssh

# Test port connectivity
telnet linux-vm-host 22
ssh -p 22 user@linux-vm-host
```

---

## 📊 Monitoring & Logs

### GitHub Actions Logs

```bash
# Via CLI
gh run list
gh run view <run-id>
gh run view <run-id> --log

# Via Web
# https://github.com/your-org/your-repo/actions
```

### Kubernetes Logs

```bash
# All pod logs
kubectl logs -f -l app=hello-world

# Specific pod logs
kubectl logs <pod-name> -f

# Previous pod logs
kubectl logs <pod-name> --previous

# With timestamps
kubectl logs <pod-name> -f --timestamps=true

# Stream from multiple pods
kubectl logs -f -l app=hello-world --all-containers=true
```

### Application Logs

**Windows VM:**
```
C:\apps\hello-world\app.log
```

**Linux VM:**
```
/opt/hello-world/application.log
```

**Kubernetes:**
```bash
kubectl logs deployment/hello-world -f
```

---

## 🔐 Environment Secrets Setup

### GitHub Secrets

**For Windows Deployment (Task 3):**
```
Settings → Secrets and variables → Actions → New repository secret

Name: WINDOWS_VM_HOST
Value: <Windows VM IP>

Name: WINDOWS_USERNAME
Value: <Windows user>

Name: WINDOWS_PASSWORD
Value: <Windows password>

Name: WINRM_CERT
Value: (contents of client.pem file)

Name: WINRM_KEY
Value: (contents of client-key.pem file)
```

**For Linux Deployment (Phase 2.2):**
```
Name: LINUX_VM_HOST
Value: <Linux VM IP>

Name: LINUX_VM_USER
Value: <SSH user>

Name: LINUX_VM_SSH_KEY
Value: (contents of ~/.ssh/id_rsa)
```

### Generate SSH Keys for Linux VM

```bash
# Generate key pair
ssh-keygen -t rsa -f ~/.ssh/id_rsa -N ""

# Copy public key to Linux VM
ssh-copy-id -i ~/.ssh/id_rsa.pub user@linux-vm-host

# Test connection
ssh user@linux-vm-host

# Get private key for GitHub Secret
cat ~/.ssh/id_rsa
```

---

## 📈 Performance Tuning

### Kubernetes HPA

```bash
# Check HPA status
kubectl get hpa hello-world -n prod

# Monitor HPA metrics
kubectl get hpa hello-world -n prod --watch

# Manual scaling (temporary)
kubectl autoscale deployment hello-world \
  --min=2 --max=10 --cpu-percent=70 -n prod
```

### Resource Limits

```bash
# Check current usage
kubectl top pods -l app=hello-world

# Check node resources
kubectl top nodes

# Set resource requests/limits
kubectl set resources deployment hello-world \
  --requests=cpu=250m,memory=256Mi \
  --limits=cpu=500m,memory=512Mi
```

---

## 🔄 Rollback Procedures

### Helm Rollback

```bash
# List release history
helm history hello-world -n prod

# Rollback to previous version
helm rollback hello-world -n prod

# Rollback to specific revision
helm rollback hello-world 2 -n prod
```

### Kubernetes Rollback

```bash
# Check rollout history
kubectl rollout history deployment/hello-world

# Rollback to previous version
kubectl rollout undo deployment/hello-world

# Rollback to specific revision
kubectl rollout undo deployment/hello-world --to-revision=2
```

---

## 📅 Scheduled Operations

### Daily Health Check

```bash
# Via cron job
0 9 * * * kubectl get deployment hello-world -o wide

# Manual check
kubectl get all -l app=hello-world
kubectl describe nodes
kubectl top pods,nodes
```

### Weekly Cleanup

```bash
# Delete old workflow runs (via GitHub CLI)
gh run list --limit 100 --jq '.[].databaseId' | \
  xargs -I {} gh api repos/OWNER/REPO/actions/runs/{} -X DELETE

# Clean up old backups on Linux VM
ssh user@linux-vm-host "find /opt/hello-world/backups -mtime +7 -delete"
```

---

## 📞 Support Resources

| Component | Documentation | Support |
|-----------|---------------|---------|
| GitHub Actions | https://docs.github.com/en/actions | GitHub Issues |
| Jenkins | https://www.jenkins.io/doc/ | Jenkins Community |
| Kubernetes | https://kubernetes.io/docs/ | K8s Community |
| Helm | https://helm.sh/docs/ | Helm Charts |
| Ansible | https://docs.ansible.com/ | Ansible Community |
| Docker | https://docs.docker.com/ | Docker Docs |

---

## ✅ Pre-Deployment Checklist

- [ ] All GitHub Secrets configured
- [ ] SSH keys generated (Linux VM)
- [ ] WinRM certificates created (Windows VM)
- [ ] Kubernetes cluster running
- [ ] Minikube started
- [ ] Docker daemon running
- [ ] Network connectivity verified
- [ ] Application dependencies installed
- [ ] Database/services configured (if needed)
- [ ] Monitoring alerts configured

---

## 🎯 Quick Status Check

```bash
# Complete infrastructure status
echo "=== GitHub Actions ===" && \
  gh run list --limit 1

echo -e "\n=== Kubernetes ===" && \
  kubectl get deployment,svc,pods -l app=hello-world

echo -e "\n=== Helm Releases ===" && \
  helm list --all-namespaces

echo -e "\n=== Windows VM ===" && \
  ssh user@windows-vm "Get-Process node | measure"

echo -e "\n=== Linux VM ===" && \
  ssh user@linux-vm "ps aux | grep node"
```

---

**Last Updated:** 2024  
**Version:** 1.0.0  
**Status:** Production Ready ✅
