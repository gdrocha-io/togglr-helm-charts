# Togglr Helm Charts

<div align="center">

**Kubernetes Helm Charts for Togglr Feature Toggle Management System**

[![Artifact Hub](https://img.shields.io/endpoint?url=https://artifacthub.io/badge/repository/togglr)](https://artifacthub.io/packages/search?repo=togglr)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Helm](https://img.shields.io/badge/helm-v3-blue.svg)](https://helm.sh/)

</div>

## Overview

This repository contains Helm charts for deploying the Togglr Feature Toggle Management System on Kubernetes. The charts provide production-ready deployments with configurable options for scaling, security, and monitoring.

## Available Charts

| Chart | Description | Version |
|-------|-------------|---------|
| [togglr-backend](./charts/togglr-backend) | Backend API service | 1.0.0 |
| [togglr-frontend](./charts/togglr-frontend) | Frontend web application | 1.0.0 |

## Quick Start

### Add Helm Repository

```bash
helm repo add togglr https://gdrocha-io.github.io/togglr-helm-charts/
helm repo update
```

### Install Complete Stack

```bash
# Install backend with PostgreSQL
helm install togglr-backend togglr/togglr-backend \
  --set postgresql.enabled=true \
  --set postgresql.auth.database=togglr \
  --set postgresql.auth.username=togglr \
  --set postgresql.auth.password=your-password \
  --set app.jwt.secret=your-jwt-secret

# Install frontend
helm install togglr-frontend togglr/togglr-frontend
```

### Install with Custom Values
```
helm install togglr-backend togglr/togglr-backend -f {path}/values.yaml
helm install togglr-frontend togglr/togglr-frontend -f {path}/values.yaml
```

## Chart Documentation

### Backend Chart
- **Repository**: [togglr-backend](https://github.com/gdrocha-io/togglr-backend)
- **Documentation**: [charts/togglr-backend/README.md](./charts/togglr-backend/README.md)
- **Default Port**: 8080
- **Database**: PostgreSQL

### Frontend Chart
- **Repository**: [togglr-frontend](https://github.com/gdrocha-io/togglr-frontend)
- **Documentation**: [charts/togglr-frontend/README.md](./charts/togglr-frontend/README.md)
- **Default Port**: 80
- **Dependencies**: Requires backend API endpoint

## Production Deployment

### Prerequisites
- Kubernetes 1.19+
- Helm 3.0+
- PostgreSQL database (internal or external)

### Recommended Setup

```bash
# 1. Create namespace
kubectl create namespace togglr

# 2. Install backend with database
helm install togglr-backend togglr/togglr-backend \
  --namespace togglr \
  --set postgresql.enabled=true \
  --set postgresql.primary.persistence.size=20Gi \
  --set app.jwt.secret="$(openssl rand -base64 32)" \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host=api.togglr.example.com

# 3. Install frontend
helm install togglr-frontend togglr/togglr-frontend \
  --namespace togglr \
  --set app.apiUrl=https://api.togglr.example.com/api/v1 \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host=togglr.example.com
```

### Security Considerations

- **JWT Secret**: Always use a strong, randomly generated JWT secret
- **Database Credentials**: Use Kubernetes secrets for database passwords
- **TLS**: Enable TLS/SSL for production deployments
- **Network Policies**: Consider implementing network policies for pod-to-pod communication

## Development

### Local Development

```bash
# Clone repository
git clone https://github.com/gdrocha-io/togglr-helm-charts.git
cd togglr-helm-charts

# Install charts locally
helm install togglr-backend ./charts/togglr-backend
helm install togglr-frontend ./charts/togglr-frontend
```

### Testing Charts

```bash
# Lint charts
helm lint ./charts/togglr-backend
helm lint ./charts/togglr-frontend

# Template and validate
helm template togglr-backend ./charts/togglr-backend
helm template togglr-frontend ./charts/togglr-frontend

# Test installation
helm install --dry-run --debug togglr-backend ./charts/togglr-backend
```

## Contributing

We welcome contributions to improve the Helm charts!

### Development Setup

```bash
# Fork and clone the repository
git clone https://github.com/YOUR_USERNAME/togglr-helm-charts.git
cd togglr-helm-charts

# Create feature branch
git checkout -b feature/chart-improvement

# Make changes and test
helm lint ./charts/*
helm template test ./charts/togglr-backend

# Submit pull request
```

### Chart Guidelines

- Follow [Helm best practices](https://helm.sh/docs/chart_best_practices/)
- Include comprehensive documentation
- Test charts with different configurations
- Maintain backward compatibility when possible

## Support

- 🐛 [Issue Tracker](https://github.com/gdrocha-io/togglr-helm-charts/issues)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

<div align="center">

**[⭐ Star this project](https://github.com/gdrocha-io/togglr-helm-charts) if you find it useful!**

Made with ❤️ by [Gabriel da Rocha](https://github.com/gdrocha)

</div>
