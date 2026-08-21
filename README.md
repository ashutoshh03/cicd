# Hello World CI/CD + Kubernetes

A simple Node.js Hello World application demonstrating a complete CI/CD pipeline with GitHub Actions and Kubernetes deployment.

## Prerequisites

- Git
- Node.js (v22+)
- npm
- Docker
- kubectl
- Minikube

## Project Structure

```
.
├── app.js                    # Express server
├── app.test.js              # Unit tests
├── package.json             # Node.js dependencies and scripts
├── Dockerfile               # Docker image definition
├── deployment.yaml          # Kubernetes deployment manifest
├── service.yaml             # Kubernetes service manifest
└── .github/workflows/
    └── ci-cd.yml            # GitHub Actions workflow
```

## Local Development

### 1. Install Dependencies

```bash
npm install
```

### 2. Run Tests

```bash
npm test
```

### 3. Start the Application

```bash
npm start
```

The application will run on `http://localhost:3000`

### 4. Test Endpoints

```bash
# Hello World endpoint
curl http://localhost:3000/

# Health check endpoint
curl http://localhost:3000/health
```

## Docker

### Build Docker Image

```bash
docker build -t hello-world:latest .
```

### Run Docker Container

```bash
docker run -p 3000:3000 hello-world:latest
```

## Kubernetes (Minikube)

### Start Minikube

```bash
minikube start
```

### Build and Load Docker Image

```bash
eval $(minikube docker-env)
docker build -t hello-world:latest .
```

### Deploy to Kubernetes

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### Check Deployment Status

```bash
# Check pods
kubectl get pods

# Expected output:
# NAME                            READY   STATUS
# hello-world-xxxxxx              1/1     Running
# hello-world-xxxxxx              1/1     Running
```

### Access the Application

```bash
# Get Minikube IP
MINIKUBE_IP=$(minikube ip)

# Access via NodePort service
curl http://$MINIKUBE_IP:30000/

# Or use kubectl port-forward
kubectl port-forward service/hello-world 3000:3000
curl http://localhost:3000/
```

### View Logs

```bash
kubectl logs -f deployment/hello-world
```

### Clean up

```bash
kubectl delete -f deployment.yaml
kubectl delete -f service.yaml
minikube stop
```

## GitHub Actions Pipeline

The pipeline (`.github/workflows/ci-cd.yml`) performs:

1. **Build**: Installs dependencies and runs build script
2. **Unit Tests**: Runs Jest tests (must pass to continue)
3. **Upload Artifact**: Stores the application as a ZIP file
4. **Docker Build**: Creates Docker image
5. **Kubernetes Deploy**: Deploys to Kubernetes cluster

### Trigger Pipeline

Push code to `main` or `master` branch:

```bash
git push origin main
```

## API Endpoints

### GET /

Returns a Hello World message:

```json
{
  "message": "Hello World!"
}
```

### GET /health

Health check endpoint:

```json
{
  "status": "ok"
}
```
# cicd
