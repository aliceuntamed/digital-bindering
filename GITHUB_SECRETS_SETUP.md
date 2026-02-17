# GitHub Secrets Setup Guide

This guide walks you through configuring GitHub repository secrets required for the CI/CD pipeline.

## Required Secrets

The deployment pipeline requires 6 repository secrets for Docker Compose deployment via SSH:

| Secret Name | Description | Example |
|---|---|---|
| `STAGING_DEPLOY_HOST` | Staging server hostname or IP | `staging.example.com` |
| `STAGING_DEPLOY_USER` | SSH user for staging server | `deploy` |
| `STAGING_DEPLOY_KEY` | SSH private key (PEM format) | (private key content) |
| `PROD_DEPLOY_HOST` | Production server hostname or IP | `prod.example.com` |
| `PROD_DEPLOY_USER` | SSH user for production server | `deploy` |
| `PROD_DEPLOY_KEY` | SSH private key (PEM format) | (private key content) |

## Setup Steps

### 1. Prepare SSH Keys

On your local machine or secure key management system:

```bash
# Generate SSH key pair (if you don't have one)
ssh-keygen -t ed25519 -f deploy_key -C "github-actions"
# DO NOT add a passphrase for automated deployments

# Display the private key (for GitHub secret)
cat deploy_key

# Display the public key (for server setup)
cat deploy_key.pub
```

### 2. Configure Deployment Servers

On both staging and production servers:

```bash
# Create deploy user (if it doesn't exist)
sudo useradd -m -s /bin/bash deploy

# Create .ssh directory
sudo mkdir -p /home/deploy/.ssh
sudo chmod 700 /home/deploy/.ssh

# Add public key to authorized_keys
echo "$(cat deploy_key.pub)" | sudo tee -a /home/deploy/.ssh/authorized_keys
sudo chmod 600 /home/deploy/.ssh/authorized_keys
sudo chown -R deploy:deploy /home/deploy/.ssh

# Create app directory
sudo mkdir -p /app
sudo chown -R deploy:deploy /app

# Allow deploy user to run docker without sudo
sudo usermod -aG docker deploy
sudo systemctl restart docker

# Verify Docker access
sudo su - deploy -c "docker ps"
```

### 3. Add Secrets to GitHub Repository

1. Go to your repository on GitHub
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Add each secret:

#### Secret 1: `STAGING_DEPLOY_HOST`
- **Name:** `STAGING_DEPLOY_HOST`
- **Secret:** `staging.example.com` (or your staging server's hostname/IP)

#### Secret 2: `STAGING_DEPLOY_USER`
- **Name:** `STAGING_DEPLOY_USER`
- **Secret:** `deploy`

#### Secret 3: `STAGING_DEPLOY_KEY`
- **Name:** `STAGING_DEPLOY_KEY`
- **Secret:** (contents of `deploy_key` file - the private key)

#### Secret 4: `PROD_DEPLOY_HOST`
- **Name:** `PROD_DEPLOY_HOST`
- **Secret:** `prod.example.com` (or your production server's hostname/IP)

#### Secret 5: `PROD_DEPLOY_USER`
- **Name:** `PROD_DEPLOY_USER`
- **Secret:** `deploy`

#### Secret 6: `PROD_DEPLOY_KEY`
- **Name:** `PROD_DEPLOY_KEY`
- **Secret:** (contents of `deploy_key` file - the private key, same as staging if using same key)

### 4. Test SSH Connection (Optional)

Before running the workflow, test the connection:

```bash
ssh -i deploy_key deploy@staging.example.com "docker ps"
ssh -i deploy_key deploy@prod.example.com "docker ps"
```

Both should return a list of containers (likely empty initially).

## Kubernetes Alternative

If deploying to **Kubernetes** instead of Docker Compose:

1. Skip the SSH key setup above
2. Instead, set up a `kubeconfig` file in GitHub Secrets
3. Modify the CD workflow to use `kubectl apply` instead of SSH deployment
4. Required secrets would be: `KUBECONFIG` (base64-encoded kubeconfig file)

See `DEPLOYMENT.md` for Kubernetes instructions.

## Troubleshooting

### Connection Refused
- Verify SSH port is open on the server (default: 22)
- Check that `sshd` is running: `sudo systemctl status ssh`
- Ensure the deploy user exists and has proper permissions

### Permission Denied (publickey)
- Verify public key was added to `~/.ssh/authorized_keys`
- Check file permissions: `ls -la ~/.ssh/` should show `700` for directory and `600` for files
- Ensure user owns the directory: `chown -R deploy:deploy /home/deploy/.ssh`

### Docker Permission Denied
- Add deploy user to docker group: `sudo usermod -aG docker deploy`
- Restart docker: `sudo systemctl restart docker`
- User must log out and back in for group changes to take effect

### Workflow Still Fails
- Check the workflow run logs in GitHub Actions for detailed error messages
- SSH into the server manually with the same key and commands to debug
- Verify `docker-compose.prod.yml` exists at `/app/docker-compose.prod.yml`

## Security Best Practices

1. **Use Ed25519 keys** instead of RSA (more secure, smaller)
2. **Rotate deploy keys** regularly (quarterly recommended)
3. **Limit deploy user permissions** - only allow docker access, not full sudo
4. **Use IP whitelisting** on servers if possible (allow only GitHub Actions IPs)
5. **Audit secret access** - GitHub logs who accessed secrets
6. **Never commit private keys** to the repository
7. **Consider using environment-specific keys** - separate keys for staging/prod
