# CI/CD Setup Guide - Docker Deployment

This document provides a comprehensive guide to the Docker-based CI/CD pipeline for the chat application.

## Overview

The CI/CD pipeline uses **Docker for deployment** and includes:
- **Continuous Integration**: Automated testing and building on every push/PR
- **Continuous Deployment**: Automated Docker-based deployment on main branch
- **Docker Support**: Containerized builds and deployments

## Workflow Files

### 1. `docker-build.yml` - Docker Build & Deploy ⭐
**Main deployment workflow** - Runs on push to `main`/`develop` branches.

**What it does:**
- ✅ Builds Docker images for backend and frontend
- ✅ Pushes images to Docker Hub
- ✅ Automatically deploys to your server
- ✅ Updates running containers with zero downtime

**Flow:**
```
Push to main → Build images → Push to Docker Hub → Deploy to server → ✅ Live!
```

### 2. `backend-ci.yml` - Backend CI
Runs only when backend files change.
- Tests on Node.js 18.x and 20.x
- TypeScript compilation
- Type checking
- Linting

### 3. `frontend-ci.yml` - Frontend CI
Runs only when frontend files change.
- Tests on Node.js 18.x and 20.x
- Next.js linting
- Production build verification

## Docker Setup

### Backend Dockerfile
- Multi-stage build for optimized image size
- Production dependencies only
- Exposes port 8000

### Frontend Dockerfile
- Multi-stage build for Next.js
- Production-optimized build
- Exposes port 3000

### Docker Compose
A `docker-compose.yml` file is provided for local development and testing.

## Setup Instructions

### 1. GitHub Secrets Configuration

Navigate to your repository: **Settings → Secrets and variables → Actions**

Add the following **required** secrets:

#### Docker Hub:
```
DOCKER_USERNAME=your-dockerhub-username
DOCKER_PASSWORD=your-dockerhub-access-token
```

#### Server Deployment:
```
SERVER_HOST=your-server-ip-or-domain
SERVER_USER=your-ssh-username
SSH_KEY=your-private-ssh-key
```

#### Application Environment Variables:
```
MONGODB_URI=your-mongodb-connection-string
JWT_SECRET=your-jwt-secret-key
NEXT_PUBLIC_API_URL=http://your-backend-url:8000
```

### 2. Docker Hub Setup

1. Sign up at https://hub.docker.com
2. Go to **Account Settings → Security → New Access Token**
3. Create token with **Read & Write** permissions
4. Use this token as `DOCKER_PASSWORD` in GitHub secrets

### 3. Server Preparation

#### Install Docker on Server:
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

#### Open Required Ports:
```bash
sudo ufw allow 8000/tcp  # Backend
sudo ufw allow 3000/tcp  # Frontend
sudo ufw allow 22/tcp   # SSH
```

### 4. SSH Key Setup

Generate SSH key for server access:
```bash
ssh-keygen -t ed25519 -C "github-actions"
ssh-copy-id your-username@your-server-ip
cat ~/.ssh/id_ed25519  # Copy this to GitHub secrets as SSH_KEY
```

## How It Works

### Complete Flow:

1. **You push code** to GitHub
2. **CI runs** (backend-ci.yml, frontend-ci.yml) - Tests and validates
3. **Docker build** (docker-build.yml) triggers:
   - Builds Docker images using Dockerfiles
   - Pushes images to Docker Hub
   - Deploys to your server via SSH
   - Pulls latest images on server
   - Stops old containers
   - Starts new containers
4. **✅ Your app is live!**

## Deployment

Every push to `main` branch automatically:
- Builds new Docker images
- Pushes to Docker Hub
- Deploys to your server
- Restarts containers with new version

**Zero manual work needed!** 🚀

## Local Testing

### Test Docker Builds Locally:
```bash
# Build backend
cd chat-backend
docker build -t chat-backend .

# Build frontend
cd chat-frontend
docker build -t chat-frontend .

# Run with docker-compose
cd ..
docker-compose up
```

## Monitoring

### Check Workflow Status:
- Go to **Actions** tab in GitHub
- See workflow runs and status
- View detailed logs for each step

### Check Server Status:
```bash
# SSH into server
ssh your-user@your-server-ip

# Check running containers
docker ps

# View logs
docker logs chat-backend
docker logs chat-frontend
```

## Troubleshooting

### Docker Build Failures
1. Check Dockerfile syntax
2. Verify all required files are present
3. Ensure `.dockerignore` is properly configured
4. Check GitHub Actions logs

### Deployment Failures
1. Verify all secrets are correctly configured
2. Test SSH connection manually: `ssh -i key user@host`
3. Check server has Docker installed
4. Verify ports are open
5. Review deployment logs in Actions tab

### Container Issues
1. Check container logs: `docker logs chat-backend`
2. Verify environment variables are set
3. Check port conflicts: `docker ps` and `netstat -tulpn`
4. Ensure images exist: `docker pull yourname/chat-backend:latest`

## Best Practices

1. **Branch Protection**: Enable branch protection for `main` branch
   - Require status checks to pass
   - Require pull request reviews

2. **Secrets Management**: Never commit sensitive data
   - Use GitHub Secrets for all sensitive values
   - Use `.env.example` files as templates

3. **Docker Images**: Keep images optimized
   - Use multi-stage builds
   - Clean up old images regularly

4. **Monitoring**: Set up alerts
   - Email notifications for failed builds
   - Monitor container health

## Next Steps

1. ✅ CI/CD workflows are set up
2. ⬜ Configure GitHub Secrets
3. ⬜ Set up Docker Hub account
4. ⬜ Prepare server with Docker
5. ⬜ Test deployment
6. ⬜ Set up monitoring and alerts

## Support

For issues or questions:
- Check GitHub Actions documentation: https://docs.github.com/en/actions
- Review workflow logs in the Actions tab
- Consult Docker documentation: https://docs.docker.com/
- See `DOCKER_DEPLOYMENT.md` for detailed Docker flow
