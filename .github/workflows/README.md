# CI/CD Workflows

This directory contains GitHub Actions workflows for Docker-based deployment.

## Workflows

### 1. `docker-build.yml` - Docker Build & Deploy
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
- Builds TypeScript
- Type checks the codebase
- Lints code

### 3. `frontend-ci.yml` - Frontend CI
Runs only when frontend files change.
- Tests on Node.js 18.x and 20.x
- Lints Next.js application
- Builds production bundle

## Setup

### Required GitHub Secrets

Add these in: **Settings → Secrets and variables → Actions**

```
DOCKER_USERNAME       - Your Docker Hub username
DOCKER_PASSWORD       - Docker Hub access token
SERVER_HOST           - Your server IP/domain
SERVER_USER           - SSH username
SSH_KEY               - Private SSH key
MONGODB_URI           - MongoDB connection string
JWT_SECRET            - JWT secret key
NEXT_PUBLIC_API_URL   - Backend API URL
```

### Server Requirements

- Docker installed
- Ports 8000 (backend) and 3000 (frontend) open
- SSH access configured

## How It Works

1. **CI runs** (backend-ci.yml, frontend-ci.yml) - Tests and validates code
2. **Docker build** (docker-build.yml) - Creates Docker images
3. **Push to Hub** - Images stored in Docker Hub
4. **Deploy** - Server pulls and runs containers automatically

## Deployment

Every push to `main` branch automatically:
- Builds new Docker images
- Pushes to Docker Hub
- Deploys to your server
- Restarts containers with new version

**Zero manual work needed!** 🚀

