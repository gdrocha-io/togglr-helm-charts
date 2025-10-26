# Togglr Frontend Helm Chart

This Helm chart deploys the Togglr Feature Toggle Management System Frontend on Kubernetes.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- Togglr Backend API (deployed separately)

## Installing the Chart

```bash
# Add the chart repository
helm repo add togglr https://gdrocha-io.github.io/togglr-charts
helm repo update

# Install with default values
helm install togglr-frontend togglr/togglr-frontend

# Install with custom values
helm install togglr-frontend togglr/togglr-frontend -f values.yaml
```

## Configuration

The following table lists the configurable parameters and their default values.

### Application Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `app.apiUrl` | Backend API URL | `http://togglr-backend:8080/api/v1` |
| `app.domain` | Application domain | `http://localhost:3000` |

### Deployment Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `1` |
| `image.repository` | Image repository | `gdrocha/togglr-frontend` |
| `image.tag` | Image tag | `latest` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `resources.limits.cpu` | CPU limit | `500m` |
| `resources.limits.memory` | Memory limit | `512Mi` |
| `resources.requests.cpu` | CPU request | `250m` |
| `resources.requests.memory` | Memory request | `256Mi` |

### Service Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `service.type` | Service type | `ClusterIP` |
| `service.port` | Service port | `80` |

### Ingress Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ingress.enabled` | Enable ingress | `false` |
| `ingress.className` | Ingress class name | `""` |
| `ingress.hosts[0].host` | Hostname | `togglr.local` |
| `ingress.tls` | TLS configuration | `[]` |

### Autoscaling Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `autoscaling.enabled` | Enable HPA | `false` |
| `autoscaling.minReplicas` | Minimum replicas | `1` |
| `autoscaling.maxReplicas` | Maximum replicas | `10` |
| `autoscaling.targetCPUUtilizationPercentage` | CPU target | `80` |

## Examples

### Basic Installation

```yaml
# values-basic.yaml
app:
  apiUrl: "https://api.togglr.example.com/api/v1"

ingress:
  enabled: true
  className: "nginx"
  hosts:
    - host: togglr.example.com
      paths:
        - path: /
          pathType: Prefix
```

### Production Configuration

```yaml
# values-production.yaml
replicaCount: 3

app:
  apiUrl: "https://api.togglr.example.com/api/v1"
  domain: "https://togglr.example.com"

ingress:
  enabled: true
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
  hosts:
    - host: togglr.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: togglr-frontend-tls
      hosts:
        - togglr.example.com

resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
```

### Development Configuration

```yaml
# values-dev.yaml
app:
  apiUrl: "http://togglr-backend.togglr-dev:8080/api/v1"

ingress:
  enabled: true
  className: "nginx"
  hosts:
    - host: togglr-dev.local
      paths:
        - path: /
          pathType: Prefix

resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

## Installation Examples

### Complete Stack Installation

```bash
# Install backend first
helm install togglr-backend togglr/togglr-backend \
  --set postgresql.enabled=true \
  --set app.jwt.secret="$(openssl rand -base64 32)"

# Wait for backend to be ready
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=togglr-backend --timeout=300s

# Install frontend
helm install togglr-frontend togglr/togglr-frontend \
  --set app.apiUrl=http://togglr-backend:8080/api/v1 \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host=togglr.local
```

### With External Backend

```bash
helm install togglr-frontend togglr/togglr-frontend \
  --set app.apiUrl=https://external-api.example.com/api/v1 \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host=togglr.example.com
```

## Health Checks

The frontend container includes health checks:

- **Liveness Probe**: HTTP GET on port 80
- **Readiness Probe**: HTTP GET on port 80
- **Startup Probe**: HTTP GET on port 80 with extended timeout

## Security Considerations

1. **HTTPS**: Always use HTTPS in production
2. **CSP Headers**: The application includes Content Security Policy headers
3. **Resource Limits**: Set appropriate resource limits
4. **Network Policies**: Consider implementing network policies

## Monitoring

The frontend serves static files and doesn't expose metrics endpoints, but you can monitor:

- HTTP response codes via ingress controller metrics
- Resource usage via Kubernetes metrics
- Application logs for errors

## Troubleshooting

### Common Issues

1. **Backend Connection Issues**
   ```bash
   # Check if backend is accessible
   kubectl exec -it deployment/togglr-frontend -- wget -qO- http://togglr-backend:8080/actuator/health
   ```

2. **Ingress Not Working**
   ```bash
   # Check ingress controller logs
   kubectl logs -n ingress-nginx deployment/ingress-nginx-controller
   ```

3. **Pod Not Starting**
   ```bash
   # Check pod logs
   kubectl logs deployment/togglr-frontend
   
   # Check pod events
   kubectl describe pod -l app.kubernetes.io/name=togglr-frontend
   ```

## Uninstalling

```bash
helm uninstall togglr-frontend
```

## Contributing

1. Make changes to the chart
2. Update version in `Chart.yaml`
3. Test with `helm lint ./charts/togglr-frontend`
4. Package with `helm package ./charts/togglr-frontend`