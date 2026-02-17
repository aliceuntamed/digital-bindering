# Deployment Validation Guide

This guide helps you test and validate all deployment paths before running the live pipeline.

## Step 1: Local Docker Build Test

Test that the Docker image builds and runs correctly locally.

### Build the image
```bash
docker build -t digi-binder:test .
```

### Check image size
```bash
docker images digi-binder:test
# Should be ~40-50MB (nginx-based, not Node.js)
```

### Run the container
```bash
docker run -d --name test-app -p 8080:8080 digi-binder:test
```

### Test the app
```bash
# Visit in browser: http://localhost:8080
# Or test via curl:
curl http://localhost:8080

# Test health endpoint
curl http://localhost:8080/health
# Should return: "healthy"
```

### View logs
```bash
docker logs test-app
```

### Cleanup
```bash
docker stop test-app
docker rm test-app
```

## Step 2: Docker Compose Production Test

Test the production docker-compose.yml locally.

### Create a test environment file
```bash
cp .env.production.example .env.prod.test

# Edit with test values (Firebase config, API URLs)
nano .env.prod.test
```

### Run docker-compose
```bash
# Don't source the env file - docker-compose reads it automatically
docker compose -f docker-compose.prod.yml up -d
```

### Verify it's running
```bash
docker compose -f docker-compose.prod.yml ps

# Should show: digi-binder-web is running
```

### Check logs
```bash
docker compose -f docker-compose.prod.yml logs -f web
```

### Test the service
```bash
curl http://localhost:80
curl http://localhost:80/health
```

### Cleanup
```bash
docker compose -f docker-compose.prod.yml down
```

## Step 3: SSH Deployment Test (Staging)

Test SSH access to your staging server and Docker deployment.

### Prerequisites
- Staging server IP/hostname
- Deploy SSH key at `~/.ssh/deploy_key`
- Deploy user created on server

### Test SSH connection
```bash
ssh -i ~/.ssh/deploy_key deploy@staging.example.com "echo 'SSH works!'"
```

### Copy docker-compose.prod.yml to server
```bash
scp -i ~/.ssh/deploy_key docker-compose.prod.yml deploy@staging.example.com:/app/

# Also copy nginx.conf
scp -i ~/.ssh/deploy_key nginx.conf deploy@staging.example.com:/app/
```

### Copy environment file
```bash
# Create the env file with your staging credentials
nano .env.staging
# Fill in VITE_FIREBASE_* and VITE_API_URL values

# Copy to server
scp -i ~/.ssh/deploy_key .env.staging deploy@staging.example.com:/app/.env
```

### Test deployment via SSH
```bash
ssh -i ~/.ssh/deploy_key deploy@staging.example.com << 'EOF'
  set -e
  cd /app
  
  echo "Testing docker-compose..."
  docker compose -f docker-compose.prod.yml config
  
  echo "Pulling latest image..."
  docker compose -f docker-compose.prod.yml pull
  
  echo "Starting services..."
  docker compose -f docker-compose.prod.yml up -d
  
  echo "Checking status..."
  docker compose -f docker-compose.prod.yml ps
  
  echo "Deployment complete!"
EOF
```

### Test the running app
```bash
curl http://staging.example.com
curl http://staging.example.com/health

# Or check in browser: http://staging.example.com
```

### View logs on server
```bash
ssh -i ~/.ssh/deploy_key deploy@staging.example.com \
  "docker compose -f docker-compose.prod.yml logs -f"
```

### Cleanup
```bash
ssh -i ~/.ssh/deploy_key deploy@staging.example.com \
  "cd /app && docker compose -f docker-compose.prod.yml down"
```

## Step 4: Kubernetes Test (Optional)

Test Kubernetes deployment locally with minikube.

### Prerequisites
```bash
# Install minikube
brew install minikube  # macOS
# or appropriate package manager for your OS

# Start minikube
minikube start --driver=docker --cpus=4 --memory=4096

# Enable ingress addon (optional)
minikube addons enable ingress
```

### Create namespace
```bash
kubectl create namespace default
```

### Configure image pull secret (if using private registry)
```bash
kubectl create secret docker-registry ghcr-secret \
  --docker-server=ghcr.io \
  --docker-username=YOUR_GITHUB_USERNAME \
  --docker-password=YOUR_GITHUB_TOKEN \
  -n default
```

### Update image reference in k8s/deployment.yaml
```bash
# Replace with an image you can actually pull, or use a public image
sed -i 's|ghcr.io/your-username/.*|nginx:latest|g' k8s/deployment.yaml
```

### Deploy to Kubernetes
```bash
chmod +x k8s-deploy.sh
./k8s-deploy.sh default default
```

### Verify deployment
```bash
kubectl get deployments -n default
kubectl get pods -n default
kubectl logs -f deployment/digi-binder-web -n default
```

### Access the app
```bash
# Port-forward to localhost
kubectl port-forward service/digi-binder-web 8080:80 -n default

# Visit: http://localhost:8080
```

### Cleanup
```bash
kubectl delete namespace default --ignore-not-found=true
```

## Step 5: Verify GitHub Actions Secrets

Before pushing to trigger the pipeline, verify all secrets are set:

```bash
# List all environment variables expected by the workflows
echo "Required Secrets:"
echo "  STAGING_DEPLOY_HOST"
echo "  STAGING_DEPLOY_USER"
echo "  STAGING_DEPLOY_KEY"
echo "  PROD_DEPLOY_HOST"
echo "  PROD_DEPLOY_USER"
echo "  PROD_DEPLOY_KEY"

echo ""
echo "To verify in GitHub:"
echo "1. Go to Settings → Secrets and variables → Actions"
echo "2. Confirm all 6 secrets are listed"
echo "3. Check that secrets are NOT showing their values"
```

## Step 6: Test the CI/CD Pipeline

### Create a test branch
```bash
git checkout -b test/deployment
```

### Make a small change to trigger CI
```bash
# Edit a file (e.g., add a comment)
echo "# Test deployment" >> .gitignore
git add .
git commit -m "test: trigger CI/CD pipeline"
git push origin test/deployment
```

### Monitor GitHub Actions
1. Go to your repository on GitHub
2. Click **Actions** tab
3. Watch the workflow runs
4. Check:
   - ✅ CI workflow builds successfully
   - ✅ CD workflow builds Docker image
   - ❓ Deploy job runs (only if on develop/main branch with proper secrets)

### Check Docker image in registry
```bash
# List images in GitHub Container Registry
docker login ghcr.io -u YOUR_USERNAME -p YOUR_TOKEN
docker pull ghcr.io/your-username/digi-b1nd3r-web-app:test-deployment
docker run -d -p 8080:8080 ghcr.io/your-username/digi-b1nd3r-web-app:test-deployment
curl http://localhost:8080/health
```

### Cleanup test branch
```bash
git checkout main
git branch -D test/deployment
git push origin --delete test/deployment
```

## Troubleshooting Checklist

- [ ] Docker build completes without errors
- [ ] Docker image starts without crashing
- [ ] Health endpoint returns 200 status
- [ ] Nginx serves static files correctly
- [ ] SSH connection works to staging/prod
- [ ] Docker Compose config is valid on server
- [ ] Environment variables are passed correctly
- [ ] All GitHub Secrets are configured
- [ ] GitHub Actions workflow syntax is valid
- [ ] Image pulls work from GitHub Container Registry

## Next Steps After Validation

1. **Push to develop branch** to trigger staging deployment
2. **Monitor staging deployment** for 5 minutes
3. **Test the staging app** thoroughly
4. **Create a version tag** (v0.1.0) to trigger production deployment
5. **Monitor production** for any issues
6. **Set up monitoring/alerting** (optional but recommended)
