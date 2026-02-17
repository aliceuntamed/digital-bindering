# Deployment Pipeline Configuration

This project includes a complete deployment pipeline with multiple deployment strategies and comprehensive guides.

## Quick Start

1. **Configure GitHub Secrets** → See [GITHUB_SECRETS_SETUP.md](GITHUB_SECRETS_SETUP.md)
2. **Test Locally** → See [DEPLOYMENT_VALIDATION.md](DEPLOYMENT_VALIDATION.md)
3. **Deploy to Staging** → Push to `develop` branch
4. **Deploy to Production** → Create a version tag `v1.0.0`

## Documentation

- **[GITHUB_SECRETS_SETUP.md](GITHUB_SECRETS_SETUP.md)** - Complete guide to setting up GitHub repository secrets
- **[DEPLOYMENT_VALIDATION.md](DEPLOYMENT_VALIDATION.md)** - Step-by-step validation and testing guide
- **[DEPLOYMENT.md](DEPLOYMENT.md)** - Detailed architecture and reference documentation

## Pipeline Overview

### 1. **CI/CD with GitHub Actions**

#### `.github/workflows/ci.yml` - Build & Test
- Runs on every push and PR
- Builds the Node.js application
- Uploads build artifacts for reference

#### `.github/workflows/cd.yml` - Build, Push & Deploy
- Builds and pushes Docker images to GitHub Container Registry (GHCR)
- Auto-deploys to staging on `develop` branch
- Auto-deploys to production on version tags (e.g., `v1.0.0`)

**Required Secrets in GitHub:**
- `STAGING_DEPLOY_HOST`, `STAGING_DEPLOY_USER`, `STAGING_DEPLOY_KEY`
- `PROD_DEPLOY_HOST`, `PROD_DEPLOY_USER`, `PROD_DEPLOY_KEY`

### 2. **Docker Deployment**

#### `docker-compose.prod.yml`
Production-ready Docker Compose configuration with:
- Environment variable injection for all Firebase config
- Health checks
- Logging configuration
- Port mapping to port 80

**Usage:**
```bash
docker compose -f docker-compose.prod.yml up -d
```

#### `deploy.sh`
Bash script for SSH-based deployment to staging or production.

**Usage:**
```bash
./deploy.sh staging latest
./deploy.sh production v1.2.3
```

**Requirements:**
- SSH key at `~/.ssh/deploy_key` or via `$DEPLOY_KEY` env var
- Environment variables: `STAGING_DEPLOY_HOST`, `STAGING_DEPLOY_USER`, etc.
- Alternatively, create `.env.staging` and `.env.production` files

### 3. **Kubernetes Deployment**

#### `k8s/deployment.yaml`
- 2 replicas with rolling update strategy
- Resource requests/limits
- Liveness and readiness probes
- Security context (non-root user, read-only filesystem)
- Environment variables from ConfigMaps and Secrets

#### `k8s/service-and-config.yaml`
- LoadBalancer service on port 80
- ConfigMap for non-sensitive configuration
- Secret template for sensitive Firebase credentials

#### `k8s/hpa.yaml`
Horizontal Pod Autoscaler:
- Scales between 2-5 replicas
- Based on CPU (70%) and memory (80%) utilization
- Scaleup: immediate
- Scaledown: 5 min stabilization

#### `k8s-deploy.sh`
Deploy all Kubernetes manifests with a single command.

**Usage:**
```bash
./k8s-deploy.sh my-cluster default
```

**Prerequisites:**
- Kubernetes cluster running
- `kubectl` configured
- Image push secret configured (GitHub GHCR credentials)

## Setup Instructions

### 1. Configure GitHub Actions

1. Go to repository **Settings** → **Secrets and variables** → **Actions**
2. Add deployment secrets:
   ```
   STAGING_DEPLOY_HOST=staging.example.com
   STAGING_DEPLOY_USER=deploy
   STAGING_DEPLOY_KEY=(your private SSH key)
   
   PROD_DEPLOY_HOST=prod.example.com
   PROD_DEPLOY_USER=deploy
   PROD_DEPLOY_KEY=(your private SSH key)
   ```

### 2. Prepare Deployment Servers

On your staging and production servers:

```bash
# Create app directory
mkdir -p /app
cd /app

# Copy docker-compose.prod.yml and .env files
# Create deploy user with SSH access
# Configure Docker daemon

# Pull latest image
docker compose -f docker-compose.prod.yml pull
```

### 3. Create Environment Files (Optional)

Create `.env.staging` and `.env.production` in the project root:

```bash
# .env.staging
VITE_API_URL=https://api-staging.example.com
VITE_FIREBASE_API_KEY=your_staging_key
VITE_FIREBASE_PROJECT_ID=your_staging_project
# ... more config

# .env.production
VITE_API_URL=https://api.example.com
VITE_FIREBASE_API_KEY=your_prod_key
VITE_FIREBASE_PROJECT_ID=your_prod_project
# ... more config
```

### 4. Deploy to Kubernetes (Optional)

Update image references in `k8s/deployment.yaml`:

```yaml
image: ghcr.io/your-username/digi-b1nd3r-web-app:latest
```

Create Kubernetes secrets and ConfigMap values, then deploy:

```bash
chmod +x k8s-deploy.sh
./k8s-deploy.sh my-cluster default
```

## Deployment Workflows

### Development
- Push to `develop` branch → Auto-deploys to staging
- Test in staging environment

### Production Release
- Create git tag: `git tag v1.0.0 && git push origin v1.0.0`
- GitHub Actions automatically builds and deploys to production
- Monitoring and logs from deployed environment

### Manual Deployment (Docker)

```bash
chmod +x deploy.sh
./deploy.sh production v1.0.0
```

## Monitoring & Rollback

### Docker Compose
```bash
# Check status
ssh user@host "docker compose -f docker-compose.prod.yml ps"

# View logs
ssh user@host "docker compose -f docker-compose.prod.yml logs -f web"

# Rollback to previous version
ssh user@host "docker compose -f docker-compose.prod.yml pull && docker compose -f docker-compose.prod.yml up -d"
```

### Kubernetes
```bash
# Check deployment status
kubectl get deployment digi-binder-web
kubectl describe pod <pod-name>

# View logs
kubectl logs -f deployment/digi-binder-web

# Rollback to previous revision
kubectl rollout undo deployment/digi-binder-web
kubectl rollout history deployment/digi-binder-web
```

## Best Practices

1. **Always test in staging first** before promoting to production
2. **Use semantic versioning** for release tags (v1.0.0, v1.1.0, etc.)
3. **Monitor health checks** in both Docker and Kubernetes
4. **Keep secrets out of version control** - use GitHub Secrets or Kubernetes Secrets
5. **Regular image cleanup** - consider registry retention policies
6. **Document environment-specific configs** in `.env` files (template only, not in git)
