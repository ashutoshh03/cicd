# Hello World Helm Chart

A production-ready Helm chart for deploying the Hello World Node.js application to Kubernetes with support for multiple environments (Dev, QA, Prod).

## Features

- Multi-environment support (Dev, QA, Production)
- Automatic scaling with Horizontal Pod Autoscaler (HPA)
- Health checks (liveness, readiness, startup probes)
- Service configuration for different deployment types
- Ingress support for external access
- ConfigMap for application configuration
- Security best practices
- Resource limits and requests
- Pod Disruption Budget for high availability

## Prerequisites

- Kubernetes 1.20+
- Helm 3.0+
- Docker image available in repository

## Installation

### Basic Installation

```bash
# Add the repository (if applicable)
helm repo add hello-world https://example.com/charts
helm repo update

# Install the chart with default values (development)
helm install hello-world ./hello-world-chart

# Install with a custom release name
helm install my-app ./hello-world-chart

# Install in a specific namespace
helm install hello-world ./hello-world-chart -n default
```

### Environment-Specific Installation

#### Development Environment

```bash
helm install hello-world ./hello-world-chart \
  -f ./values-dev.yaml \
  -n dev \
  --create-namespace
```

#### QA Environment

```bash
helm install hello-world ./hello-world-chart \
  -f ./values-qa.yaml \
  -n qa \
  --create-namespace
```

#### Production Environment

```bash
helm install hello-world ./hello-world-chart \
  -f ./values-prod.yaml \
  -n prod \
  --create-namespace
```

### Upgrade Deployment

```bash
# Upgrade to a new version
helm upgrade hello-world ./hello-world-chart \
  -f ./values-prod.yaml

# Upgrade with new image version
helm upgrade hello-world ./hello-world-chart \
  -f ./values-prod.yaml \
  --set image.tag=1.0.1
```

## Configuration

### Key Configuration Values

| Parameter | Description | Default |
|-----------|-------------|---------|
| `global.environment` | Environment name | `dev` |
| `replicaCount` | Number of replicas | `2` |
| `image.repository` | Docker image repository | `hello-world` |
| `image.tag` | Docker image tag | `latest` |
| `service.type` | Kubernetes service type | `ClusterIP` |
| `service.port` | Service port | `80` |
| `service.targetPort` | Container port | `3000` |
| `ingress.enabled` | Enable ingress | `false` |
| `resources.limits.cpu` | CPU limit | `500m` |
| `resources.limits.memory` | Memory limit | `512Mi` |
| `autoscaling.enabled` | Enable HPA | `false` |

### Environment Comparison

| Feature | Dev | QA | Prod |
|---------|-----|----|----|
| Replicas | 1 | 2 | 3 |
| CPU Limit | 200m | 300m | 1000m |
| Memory Limit | 256Mi | 512Mi | 1Gi |
| HPA Enabled | No | Yes | Yes |
| Max Replicas | - | 5 | 10 |
| Ingress TLS | Staging | Prod | Prod |
| Security Level | Low | Medium | High |
| PDB Enabled | No | Yes | Yes |

## Accessing the Application

### Port Forward (Development)

```bash
kubectl port-forward svc/hello-world 3000:80
curl http://localhost:3000/
curl http://localhost:3000/health
```

### Using Ingress (QA/Prod)

```bash
# Get ingress details
kubectl get ingress

# Access via ingress hostname
curl https://hello-world.example.com/
curl https://hello-world.example.com/health
```

### Using LoadBalancer (Production)

```bash
# Get external IP
kubectl get svc hello-world

# Access via external IP
curl http://<EXTERNAL-IP>/
curl http://<EXTERNAL-IP>/health
```

## Monitoring and Troubleshooting

### Check Deployment Status

```bash
# Get deployment status
kubectl get deployment hello-world -o wide

# Get pod status
kubectl get pods -l app=hello-world

# Describe deployment
kubectl describe deployment hello-world
```

### View Logs

```bash
# View logs from all pods
kubectl logs -l app=hello-world -f

# View logs from specific pod
kubectl logs <pod-name> -f

# View previous logs (if pod crashed)
kubectl logs <pod-name> --previous
```

### Check Resource Usage

```bash
# Get pod resource metrics
kubectl top pods -l app=hello-world

# Get node resource metrics
kubectl top nodes
```

### Debugging

```bash
# Exec into a pod
kubectl exec -it <pod-name> -- /bin/sh

# Check events
kubectl describe pod <pod-name>

# Port forward to specific pod
kubectl port-forward <pod-name> 3000:3000
```

## Customization

### Override Specific Values

```bash
# Override replicas
helm install hello-world ./hello-world-chart \
  --set replicaCount=5

# Override image
helm install hello-world ./hello-world-chart \
  --set image.repository=my-registry/hello-world \
  --set image.tag=v2.0.0

# Override environment variables
helm install hello-world ./hello-world-chart \
  --set "env[0].name=CUSTOM_VAR" \
  --set "env[0].value=custom_value"
```

### Create Custom Values File

Create a `values-custom.yaml`:

```yaml
global:
  environment: custom
replicaCount: 4
resources:
  limits:
    cpu: 750m
    memory: 768Mi
```

Then install with:

```bash
helm install hello-world ./hello-world-chart -f values-custom.yaml
```

## Uninstallation

```bash
# Uninstall the release
helm uninstall hello-world

# Uninstall from specific namespace
helm uninstall hello-world -n prod
```

## CI/CD Integration

### GitHub Actions Deployment

```yaml
- name: Deploy with Helm
  run: |
    helm upgrade --install hello-world ./hello-world-chart \
      -f ./helm/values-${{ matrix.environment }}.yaml \
      -n ${{ matrix.environment }} \
      --create-namespace
```

### GitOps with ArgoCD

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: hello-world
spec:
  source:
    repoURL: https://github.com/example/hello-world
    path: helm/hello-world-chart
    helm:
      valueFiles:
      - values-prod.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: prod
```

## Best Practices

1. **Use specific image tags in production** - Never use `latest` in prod
2. **Set resource limits and requests** - Prevents node overload
3. **Enable pod disruption budgets** - Ensures availability during updates
4. **Use readiness probes** - Ensures traffic only goes to healthy pods
5. **Enable HPA for production** - Automatically scales based on demand
6. **Use ingress for external access** - Better than NodePort/LoadBalancer
7. **Implement pod anti-affinity** - Distributes pods across nodes
8. **Enable security contexts** - Restricts pod privileges
9. **Use ConfigMaps for configuration** - Separates config from code
10. **Monitor logs and metrics** - Essential for troubleshooting

## Support

For issues and feature requests, please refer to the main repository documentation.
