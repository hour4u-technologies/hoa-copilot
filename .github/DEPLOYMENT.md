# AWS EC2 Deployment Guide

This guide explains how to set up GitHub Actions to automatically deploy HOA Copilot to your AWS EC2 instance.

## Prerequisites

1. **AWS EC2 Instances** (separate for dev and prod):
   - Docker installed and running
   - SSH access configured
   - Port 80 open in security groups (for both dev and prod)

2. **GitHub Repository** with Actions enabled

3. **SSH Key Pairs** for both EC2 instances (dev and prod)

## Setup Instructions

### Step 1: Prepare EC2 Instance

1. **Install Docker on EC2** (if not already installed):
   ```bash
   sudo yum update -y
   sudo yum install docker -y
   sudo service docker start
   sudo usermod -a -G docker ec2-user
   ```

2. **Create Docker volume** (if not exists):
   ```bash
   docker volume create open-webui
   ```

3. **Test Docker access**:
   ```bash
   docker ps
   ```

### Step 2: Generate SSH Key Pair (if needed)

If you don't have an SSH key pair:

```bash
ssh-keygen -t rsa -b 4096 -C "github-actions" -f ~/.ssh/github_actions_deploy
```

Copy the public key to your EC2 instance:
```bash
ssh-copy-id -i ~/.ssh/github_actions_deploy.pub ec2-user@YOUR_EC2_IP
```

### Step 3: Configure GitHub Secrets and Variables

Go to your GitHub repository → Settings → Secrets and variables → Actions

#### Required Secrets (6 total):

**Development Environment:**
1. **EC2_DEV_HOST**: Your DEV EC2 instance public IP or domain name
2. **EC2_DEV_USER**: SSH username for DEV EC2 (usually `ec2-user` or `ubuntu`)
3. **EC2_DEV_SSH_PRIVATE_KEY**: Complete DEV SSH private key content

**Production Environment:**
4. **EC2_PROD_HOST**: Your PROD EC2 instance public IP or domain name
5. **EC2_PROD_USER**: SSH username for PROD EC2 (usually `ec2-user` or `ubuntu`)
6. **EC2_PROD_SSH_PRIVATE_KEY**: Complete PROD SSH private key content

#### Optional Variables:
- **EC2_DEV_PORT**: Port for DEV (default: 80)
- **EC2_PROD_PORT**: Port for PROD (default: 80)

**See `GITHUB_VARIABLES_SETUP.md` for detailed instructions and examples.**

### Step 4: Choose Deployment Workflow

You have two workflow options (both support dev/prod):

#### Option A: Deploy Existing Image (Recommended for Quick Start)
- **File**: `.github/workflows/deploy-ec2.yml`
- **Description**: Pulls and deploys the existing `ghcr.io/open-webui/open-webui:main` image
- **Use when**: You want to deploy quickly without building custom images
- **Environments**: Automatically deploys to PROD on `main` branch, DEV on `develop` branch

#### Option B: Build and Deploy Custom Image
- **File**: `.github/workflows/deploy-ec2-custom-build.yml`
- **Description**: Builds a custom Docker image from your code and deploys it
- **Use when**: You want to deploy your customized version of HOA Copilot
- **Environments**: Automatically deploys to PROD on `main` branch, DEV on `develop` branch

### Step 5: Configure Workflow (Optional)

The workflows are pre-configured, but you can customize:

1. **Update branch names** (if different):
   ```yaml
   branches:
     - main      # Production branch
     - develop   # Development branch
   ```

2. **Update Docker image** (Option A only):
   ```yaml
   DOCKER_IMAGE: ghcr.io/open-webui/open-webui:main
   ```

3. **Ports are configured via GitHub Variables** (both default to 80):
   - Set `EC2_DEV_PORT` variable for dev (default: 80)
   - Set `EC2_PROD_PORT` variable for prod (default: 80)

## Deployment Process

### Automatic Deployments:
- **Push to `main` branch** → Automatically deploys to **PRODUCTION**
- **Push to `develop` branch** → Automatically deploys to **DEVELOPMENT**

### Workflow Steps:
1. **Determine Environment** - Detects dev/prod based on branch
2. **Checkout** your code
3. **Build Image** (if using custom-build workflow)
4. **Configure SSH** connection to appropriate EC2 instance
5. **Stop** existing container (if running)
6. **Pull/Build** latest Docker image
7. **Start** new container with your configuration
8. **Verify** deployment was successful
9. **Health Check** - Verifies service is responding

## Manual Deployment

You can also trigger deployments manually:

1. Go to **Actions** tab in GitHub
2. Select **Deploy to AWS EC2** (or **Build and Deploy to AWS EC2**) workflow
3. Click **Run workflow**
4. Select:
   - **Environment**: `dev` or `prod`
   - **Branch**: (optional, defaults to current branch)
5. Click **Run workflow**

## Troubleshooting

### SSH Connection Issues

- Verify `EC2_HOST` is correct
- Check security group allows SSH (port 22) from GitHub Actions IPs
- Ensure SSH key is correctly formatted in secrets

### Docker Issues

- Verify Docker is running on EC2: `sudo service docker status`
- Check Docker permissions: `sudo usermod -a -G docker $USER`
- Verify volume exists: `docker volume ls`

### Container Not Starting

- Check logs: `docker logs open-webui`
- Verify port 80 is available: `sudo netstat -tulpn | grep 80`
- Check container status: `docker ps -a`

### Permission Issues

- Ensure EC2 user has Docker permissions
- Check volume permissions: `docker volume inspect open-webui`

## Security Best Practices

1. **Use IAM Roles**: Instead of storing credentials, use IAM roles for EC2
2. **Restrict SSH Access**: Limit SSH access to specific IPs in security group
3. **Use Secrets Manager**: For production, consider AWS Secrets Manager
4. **Enable Logging**: Monitor CloudWatch logs for deployment issues
5. **Regular Updates**: Keep Docker and system packages updated

## Additional Configuration

### Environment Variables

To add environment variables to your container, modify the `docker run` command in the workflow:

```yaml
docker run -d -p 80:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  -e OLLAMA_BASE_URL=http://your-ollama-url:11434 \
  -e OPENAI_API_KEY=your-key \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main
```

### Health Checks

The container includes a health check endpoint. You can verify it's working:

```bash
curl http://YOUR_EC2_IP/health
```

## Support

For issues or questions:
- Check GitHub Actions logs in the Actions tab
- Review Docker logs on EC2: `docker logs open-webui`
- Verify EC2 instance status and security groups
