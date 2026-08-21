# VISUAL DIAGRAMS: Explain With Pictures

## 1. THE COMPLETE FLOW

```
┌─────────────────────────────────────────────────────────────────┐
│                    DEVELOPER WORKFLOW                           │
└─────────────────────────────────────────────────────────────────┘

1. Developer Makes Changes
   ├── Edits: app.js (application code)
   ├── Edits: app.test.js (if needed)
   └── Commits to git
          ↓

2. Developer Pushes to GitHub
   $ git push origin main
          ↓

3. GitHub Detects Push
   └── Webhook triggers GitHub Actions
          ↓

4. GitHub Actions Workflow Starts
   ┌────────────────────────────────┐
   │ Runs on: ubuntu-latest (VM)    │
   │ GitHub provides the machine    │
   └────────────────────────────────┘
          ↓

5. Job 1: BUILD & TEST
   ├─ Checkout your code
   ├─ Setup Node.js 22
   ├─ npm install (355 packages)
   ├─ npm test (3 tests run)
   │  ├─ Test 1: GET / works? ✓
   │  ├─ Test 2: GET /health works? ✓
   │  └─ Test 3: 404 on unknown route? ✓
   │  
   │  ❌ If ANY test fails → STOP HERE
   │  ✅ If all pass → Continue
   │
   ├─ npm run build
   ├─ Create dist/hello-world.zip
   └─ Upload artifact to GitHub
          ↓

6. Job 2: BUILD DOCKER (Only if Job 1 passed)
   ├─ Checkout your code
   ├─ docker build -t hello-world:latest .
   │  └─ Creates Docker image
   ├─ docker save hello-world:latest -o hello-world-docker.tar
   │  └─ Saves image to file
   └─ Upload image artifact to GitHub
          ↓

7. Job 3: DEPLOY KUBERNETES (Only if Job 2 passed)
   ├─ Setup Minikube on GitHub runner
   ├─ Download Docker image artifact
   ├─ Load image to Minikube: docker load -i hello-world-docker.tar
   ├─ kubectl apply -f deployment.yaml
   │  └─ Creates 2 pod replicas
   ├─ kubectl apply -f service.yaml
   │  └─ Creates service to expose port 30000
   ├─ Wait for deployment (kubectl rollout status)
   ├─ kubectl get pods
   │  └─ Shows: 2/2 Running
   └─ kubectl get service
      └─ Shows: Service active on port 30000
          ↓

8. Result: Application is LIVE
   ├─ curl http://localhost:3000/
   │  └─ {"message":"Hello World!"}
   └─ curl http://localhost:3000/health
      └─ {"status":"ok"}

┌─────────────────────────────────────────────────────────────────┐
│ Total Time: ~10-15 minutes (fully automated, no human involved) │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. WHAT HAPPENS WHEN TESTS FAIL

```
Developer Code Has a Bug
         ↓
$ git push origin main
         ↓
GitHub Actions Triggered
         ↓
Job 1: BUILD & TEST
  - npm install ✓
  - npm test ✗ FAILS
         ↓
PIPELINE STOPS HERE ✗

Notification:
┌──────────────────────────────────────────┐
│ GitHub: "Workflow Failed"                │
│ Job: build-and-test                      │
│ Error: Tests failed (see logs)           │
│ Link: https://github.com/.../runs/12345  │
└──────────────────────────────────────────┘

Result:
✗ Docker NOT built
✗ Kubernetes NOT deployed
✗ Bad code does NOT reach users

Developer:
1. Sees the failure notification
2. Clicks link, reads test output
3. Fixes the code
4. git push (tries again)
5. Pipeline succeeds this time
```

---

## 3. KUBERNETES ARCHITECTURE

```
┌──────────────────────────────────────────────────────┐
│                KUBERNETES CLUSTER                     │
│             (In this case: Minikube)                  │
└──────────────────────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ↓                 ↓                 ↓
    ┌─────────┐       ┌─────────┐      ┌───────┐
    │   Pod 1 │       │   Pod 2 │      │ etcd  │
    │         │       │         │      │(data) │
    │ ┌─────┐ │       │ ┌─────┐ │      └───────┘
    │ │Node │ │       │ │Node │ │
    │ │app  │ │       │ │app  │ │
    │ │:3000│ │       │ │:3000│ │
    │ └─────┘ │       │ └─────┘ │
    │         │       │         │
    │ Health: │       │ Health: │
    │ ✓ OK    │       │ ✓ OK    │
    │         │       │         │
    └────┬────┘       └────┬────┘
         │                 │
         └────────┬────────┘
                  ↓
              ┌─────────────┐
              │   Service   │
              │ hello-world │
              │ ClusterIP   │
              │ NodePort:   │
              │   30000     │
              └────────┬────┘
                       │
                       ↓
             External Traffic
          (curl http://localhost:3000/)
                       │
          ┌────────────┴────────────┐
          ↓                         ↓
       Pod 1 (30% chance)     Pod 2 (70% chance)
       
Load Balancer automatically routes requests
to healthy pods
```

### What Makes This Production-Ready:

1. **2 Replicas**: If Pod 1 crashes:
   ```
   Pod 1: CRASHED ✗
   Service: Routes 100% to Pod 2 ✓
   User: No interruption
   ```

2. **Liveness Probe** (every 10 seconds):
   ```
   GET /health → Pod responds
   GET /health → Pod doesn't respond
   Action: Kill pod, start new one
   ```

3. **Readiness Probe** (every 5 seconds):
   ```
   GET /health → Pod not ready
   Action: Don't route traffic yet
   GET /health → Pod ready
   Action: Add to load balancer
   ```

---

## 4. DOCKER IMAGE LAYERS

```
                      Dockerfile
                          │
                          ↓
        ┌─────────────────────────────────────┐
        │ FROM node:22-alpine                 │
        │ Base image: Linux + Node.js          │
        │ Size: ~170MB                        │
        └─────────────────────────────────────┘
                          │
                          ↓
        ┌─────────────────────────────────────┐
        │ WORKDIR /app                        │
        │ Create and set working directory    │
        └─────────────────────────────────────┘
                          │
                          ↓
        ┌─────────────────────────────────────┐
        │ COPY package*.json ./               │
        │ Copy: package.json & package-lock   │
        │ (Only dependencies, not app code)   │
        └─────────────────────────────────────┘
                          │
                          ↓
        ┌─────────────────────────────────────┐
        │ RUN npm install                     │
        │ Downloads 355 npm packages          │
        │ Size: ~50MB                         │
        │                                     │
        │ Docker Cache: If package.json      │
        │ didn't change, reuse this layer    │
        └─────────────────────────────────────┘
                          │
                          ↓
        ┌─────────────────────────────────────┐
        │ COPY . .                            │
        │ Copy entire app code                │
        │ Size: ~30KB                         │
        └─────────────────────────────────────┘
                          │
                          ↓
        ┌─────────────────────────────────────┐
        │ EXPOSE 3000                         │
        │ Document that port 3000 is exposed  │
        └─────────────────────────────────────┘
                          │
                          ↓
        ┌─────────────────────────────────────┐
        │ CMD ["node", "app.js"]              │
        │ Default command when container runs │
        └─────────────────────────────────────┘
                          │
                          ↓
                ┌─────────────────────┐
                │  Final Docker Image  │
                │  hello-world:latest  │
                │  Size: ~200MB        │
                │  Ready to deploy     │
                └─────────────────────┘

Benefits:
✓ If only app.json changes → reuse npm install layer (fast rebuild)
✓ Small size → fast deployment
✓ No unnecessary files → secure
```

---

## 5. CI/CD PIPELINE STAGES

```
STAGE 1: CODE
┌────────────────────────────┐
│ git commit -m "Fix bug"    │
│ git push origin main       │
└────────────────────────────┘
           │
           ↓
STAGE 2: BUILD & TEST
┌────────────────────────────┐
│ npm install                │
│ npm test (3 tests)         │
│ npm run build              │
│ Create ZIP artifact        │
│                            │
│ Quality Gate:              │
│ ❌ Fail → Stop             │
│ ✅ Pass → Continue         │
└────────────────────────────┘
           │
           ↓
STAGE 3: BUILD DOCKER
┌────────────────────────────┐
│ docker build               │
│ docker save -o .tar        │
│ Upload artifact            │
│                            │
│ Only runs if tests passed  │
└────────────────────────────┘
           │
           ↓
STAGE 4: DEPLOY KUBERNETES
┌────────────────────────────┐
│ kubectl apply deployment   │
│ kubectl apply service      │
│ kubectl get pods           │
│                            │
│ Only runs if docker built  │
│ Result: App is LIVE        │
└────────────────────────────┘
           │
           ↓
✅ PIPELINE COMPLETE

Development to Production: ~10-15 minutes, fully automated
```

---

## 6. TESTING FLOW

```
npm test
   │
   ├─ Test Suite: Hello World API
   │
   ├─ Test 1: GET /
   │  │
   │  ├─ Send HTTP request to app
   │  │  GET http://app/
   │  │
   │  ├─ Check response
   │  │  status: 200 ✓
   │  │  body.message: "Hello World!" ✓
   │  │
   │  └─ Result: PASS ✓
   │
   ├─ Test 2: GET /health
   │  │
   │  ├─ Send HTTP request
   │  │  GET http://app/health
   │  │
   │  ├─ Check response
   │  │  status: 200 ✓
   │  │  body.status: "ok" ✓
   │  │
   │  └─ Result: PASS ✓
   │
   ├─ Test 3: GET /unknown
   │  │
   │  ├─ Send HTTP request
   │  │  GET http://app/unknown
   │  │
   │  ├─ Check response
   │  │  status: 404 ✓ (expected not found)
   │  │
   │  └─ Result: PASS ✓
   │
   └─ Summary
      Test Suites: 1 passed
      Tests: 3 passed, 0 failed
      Duration: < 1 second

Pipeline Decision:
✅ All tests passed → Continue to Docker build
❌ Any test failed → STOP, notify developer
```

---

## 7. GITHUB ACTIONS INFRASTRUCTURE

```
                       GitHub
                    (your code)
                         │
         ┌───────────────┴───────────────┐
         │                               │
         │         .github/              │
         │         workflows/            │
         │         ci-cd.yml             │
         │                               │
         └───────────────┬───────────────┘
                         │
                         │ Webhook triggered
                         │ on push to main
                         ↓
         ┌──────────────────────────────┐
         │   GitHub Actions Server      │
         │   (Runs workflows)           │
         └──────────────────────────────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
         ↓               ↓               ↓
    ┌────────┐      ┌────────┐      ┌────────┐
    │ Runner │      │ Runner │      │ Runner │
    │ Ubuntu │      │ Ubuntu │      │ Ubuntu │
    │ 22     │      │ 22     │      │ 22     │
    │        │      │        │      │        │
    │Job 1   │      │Job 2   │      │Job 3   │
    │        │      │        │      │        │
    │Build & │      │Build   │      │Deploy  │
    │Test    │      │Docker  │      │K8s     │
    └────────┘      └────────┘      └────────┘
         │               │               │
         └───────────────┼───────────────┘
                         │
         ┌───────────────┴───────────────┐
         │                               │
         ├─ Artifacts uploaded           │
         ├─ Logs available               │
         ├─ Status shown in GitHub       │
         └─ Email notification sent      │
```

---

## 8. ENVIRONMENT PROGRESSION (FUTURE)

```
Code Push to GitHub
         │
         ├─────────────────────────┐
         │                         │
         ↓                         ↓
    Pipeline (Automatic)    Manual Approval
         │                         │
         ↓                         ↓
    DEV Environment         STAGING Environment
    (auto-deploy)           (auto-deploy if approved)
    - Same tests            - Performance tests
    - Fast feedback         - Security scan
    - Latest code           - Integration tests
         │                         │
         └─────────────────────────┤
                                   │
                   Manual Approval │
                        Required   │
                                   ↓
                    PRODUCTION Environment
                    (auto-deploy if approved)
                    - Full test suite
                    - Canary deployment
                    - Monitoring active
                    - Real traffic

Risk Progression:
DEV (Highest Risk - Latest Code) 
  → STAGING (Medium Risk - Tested Code) 
    → PRODUCTION (Lowest Risk - Proven Code)
```

---

## 9. WHAT EACH COMMAND DOES

```
# Application
node app.js                → Runs the server on port 3000

# Testing
npm test                   → Runs Jest tests

# Docker
docker build -t hello-world:latest .
  └─ Creates container image from Dockerfile
  
docker run -p 3000:3000 hello-world:latest
  └─ Runs container, maps port 3000
  
docker save -> hello-world-docker.tar
  └─ Saves image to file (for transfer)
  
docker load -i hello-world-docker.tar
  └─ Loads image from file

# Kubernetes
kubectl apply -f deployment.yaml
  └─ Creates 2 pod instances
  
kubectl apply -f service.yaml
  └─ Exposes port 30000
  
kubectl get pods
  └─ Shows running pods
  
kubectl get service
  └─ Shows services and ports
  
kubectl logs deployment/hello-world
  └─ Shows application logs
  
kubectl describe pod <pod-name>
  └─ Shows pod details and events
  
kubectl port-forward service/hello-world 3000:3000
  └─ Forward local port 3000 to service

# Git
git init
  └─ Initialize repository
  
git add .
git commit -m "message"
  └─ Save changes
  
git push origin main
  └─ Push to GitHub → Triggers workflow
```

---

## 10. FAILURE SCENARIOS & RECOVERY

```
Scenario 1: TEST FAILS
Pipeline:  Build → [TEST FAILS] → STOP
Recovery:  Developer fixes code → git push → Retry

Scenario 2: DOCKER BUILD FAILS
Pipeline:  Build → Test ✓ → [DOCKER BUILD FAILS] → STOP
Recovery:  Fix Dockerfile → git push → Retry

Scenario 3: POD CRASHES (PRODUCTION)
             ┌─────┬────────────┬─────┐
             │ Pod │  Pod Alive │ Pod │
             │  1  │  Check Every 10s  │
             ├─────┼────────────┼─────┤
             │ ✓   │  ✓         │ ✓   │
             │ OK  │  OK        │ OK  │
             ├─────┼────────────┼─────┤
             │ ✗   │  NO        │ ✓   │
             │DEAD │  Response  │ OK  │
             └─────┴────────────┴─────┘
                   │                │
                   ↓                ↓
             Kubernetes       100% Traffic
             Kills it         to Pod 3
             and
             Restarts
Recovery:  Automatic (Kubernetes self-heals)

Scenario 4: NEED TO ROLLBACK
            kubectl rollout undo deployment/hello-world
            └─ Goes back to previous version instantly
Recovery:  Manual intervention if needed
```

---

Use these diagrams when explaining to the interviewer!
