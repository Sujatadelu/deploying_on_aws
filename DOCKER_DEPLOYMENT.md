# Docker Deployment - Complete Flow

## 🐳 How Docker Deployment Works

### The Complete Journey with Docker:

```
┌─────────────────────────────────────────────────────────────┐
│ STEP 1: You Push Code to GitHub                             │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 2: CI Runs (backend-ci.yml, frontend-ci.yml)           │
│    ✅ Tests code                                             │
│    ✅ Builds locally                                         │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 3: docker-build.yml Workflow Triggers                  │
│    (Runs when you push to main/develop)                      │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 4: Docker Build Process                                │
│                                                              │
│    For Backend:                                              │
│    1. Reads chat-backend/Dockerfile                         │
│    2. Starts with node:20-alpine (lightweight Linux)        │
│    3. Copies package.json                                    │
│    4. Runs: npm ci (install dependencies)                   │
│    5. Copies your source code                                │
│    6. Runs: npm run build (TypeScript → JavaScript)          │
│    7. Creates final image with only production files         │
│                                                              │
│    For Frontend:                                             │
│    1. Reads chat-frontend/Dockerfile                         │
│    2. Starts with node:20-alpine                             │
│    3. Copies package.json                                    │
│    4. Runs: npm ci                                           │
│    5. Copies your source code                                │
│    6. Runs: npm run build (Next.js production build)        │
│    7. Creates final image with built .next folder            │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 5: Push to Docker Hub                                  │
│                                                              │
│    Backend Image:                                            │
│    → yourname/chat-backend:latest                           │
│    → yourname/chat-backend:abc123 (with commit SHA)         │
│                                                              │
│    Frontend Image:                                           │
│    → yourname/chat-frontend:latest                          │
│    → yourname/chat-frontend:abc123                          │
│                                                              │
│    (Stored in Docker Hub - like GitHub for Docker images)    │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 6: Your Server Pulls & Runs                            │
│                                                              │
│    On your server (AWS, DigitalOcean, your own server):     │
│                                                              │
│    1. docker pull yourname/chat-backend:latest              │
│       (Downloads the image from Docker Hub)                 │
│                                                              │
│    2. docker run -p 8000:8000 yourname/chat-backend:latest │
│       (Runs the container - your app is live!)              │
│                                                              │
│    3. docker pull yourname/chat-frontend:latest            │
│    4. docker run -p 3000:3000 yourname/chat-frontend:latest│
└─────────────────────────────────────────────────────────────┘
                          ↓
                    🎉 APP IS LIVE! 🎉
```

## 📋 Detailed Step-by-Step

### Step 1: Dockerfile (The Recipe)

**Backend Dockerfile** (`chat-backend/Dockerfile`):

```dockerfile
# Stage 1: Build your app
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./          # Copy package files
RUN npm ci                      # Install ALL dependencies (dev + prod)
COPY . .                        # Copy your source code
RUN npm run build              # Build TypeScript → JavaScript

# Stage 2: Create production image (smaller)
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production   # Install ONLY production dependencies
COPY --from=builder /app/dist ./dist  # Copy built files from stage 1
EXPOSE 8000                     # Port your app uses
CMD ["node", "dist/index.js"]   # Command to start your app
```

**What this does:**
- Creates a **lightweight Linux container** with Node.js
- Installs your dependencies
- Builds your TypeScript code
- Creates a **smaller final image** (only production files)

### Step 2: docker-build.yml (The Builder)

**What happens in `docker-build.yml`:**

```yaml
1. Checkout code
   ↓
2. Set up Docker Buildx (tool to build images)
   ↓
3. Login to Docker Hub (so we can push images)
   ↓
4. Build Docker image using Dockerfile
   - Reads: chat-backend/Dockerfile
   - Builds: Creates the image
   - Tags: yourname/chat-backend:latest
   ↓
5. Push to Docker Hub
   - Uploads image to cloud storage
   - Now available for anyone to pull
```

### Step 3: Docker Hub (The Storage)

**What gets stored:**
```
Docker Hub (like GitHub but for Docker images):
├── yourname/chat-backend
│   ├── latest (most recent version)
│   ├── abc123 (version with commit SHA)
│   └── xyz789 (another version)
│
└── yourname/chat-frontend
    ├── latest
    └── abc123
```

### Step 4: Server Deployment

**On your server, you run:**

```bash
# Pull the latest backend image
docker pull yourname/chat-backend:latest

# Stop old container (if running)
docker stop chat-backend || true
docker rm chat-backend || true

# Run new container
docker run -d \
  --name chat-backend \
  -p 8000:8000 \
  -e MONGODB_URI=your-mongodb-url \
  -e JWT_SECRET=your-secret \
  yourname/chat-backend:latest
```

**What `docker run` does:**
- Downloads image if not present
- Creates a **container** (running instance)
- Maps port 8000 (container) → 8000 (server)
- Sets environment variables
- Starts your app

## 🔄 Automatic Deployment with Docker

### Option 1: Manual (You run commands on server)

```bash
# Every time you deploy:
docker pull yourname/chat-backend:latest
docker stop chat-backend
docker rm chat-backend
docker run -d --name chat-backend -p 8000:8000 yourname/chat-backend:latest
```

### Option 2: Automatic (Server watches Docker Hub)

Add to `docker-build.yml` after pushing:

```yaml
- name: Deploy to Server
  uses: appleboy/ssh-action@master
  with:
    host: ${{ secrets.SERVER_HOST }}
    username: ${{ secrets.SERVER_USER }}
    key: ${{ secrets.SSH_KEY }}
    script: |
      docker pull ${{ secrets.DOCKER_USERNAME }}/chat-backend:latest
      docker stop chat-backend || true
      docker rm chat-backend || true
      docker run -d \
        --name chat-backend \
        -p 8000:8000 \
        -e MONGODB_URI="${{ secrets.MONGODB_URI }}" \
        -e JWT_SECRET="${{ secrets.JWT_SECRET }}" \
        ${{ secrets.DOCKER_USERNAME }}/chat-backend:latest
```

### Option 3: Using Docker Compose (Easier Management)

Create `docker-compose.prod.yml` on your server:

```yaml
version: '3.8'
services:
  backend:
    image: yourname/chat-backend:latest
    ports:
      - "8000:8000"
    environment:
      - MONGODB_URI=${MONGODB_URI}
      - JWT_SECRET=${JWT_SECRET}
    restart: always

  frontend:
    image: yourname/chat-frontend:latest
    ports:
      - "3000:3000"
    depends_on:
      - backend
    restart: always
```

Then on server:
```bash
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d
```

## 🎯 Complete Example Flow

### Scenario: You fix a bug and push to GitHub

```
1. You: git push origin main
   ↓
2. GitHub Actions: CI runs → ✅ Passes
   ↓
3. GitHub Actions: docker-build.yml runs
   - Builds new Docker image
   - Tags: yourname/chat-backend:abc123
   - Pushes to Docker Hub
   ↓
4. Docker Hub: Stores new image
   ↓
5. Your Server (automatic or manual):
   - Pulls: docker pull yourname/chat-backend:latest
   - Stops old: docker stop chat-backend
   - Starts new: docker run yourname/chat-backend:latest
   ↓
6. ✅ Your app is updated and running!
```

## 🔑 Key Concepts

### Docker Image vs Container

- **Image** = Template/Blueprint (like a class in programming)
  - Example: `yourname/chat-backend:latest`
  - Stored in Docker Hub
  - Contains your app + Node.js + dependencies

- **Container** = Running instance (like an object)
  - Example: `chat-backend` (running container)
  - Created from an image
  - Your actual running app

### Multi-Stage Build (Why it's used)

**Without multi-stage:**
```
Image size: ~500MB (includes dev dependencies, source code, etc.)
```

**With multi-stage (what we use):**
```
Image size: ~150MB (only production files)
```

**How it works:**
1. **Stage 1 (builder)**: Builds your app (can be large)
2. **Stage 2 (production)**: Only copies built files (small)

## 📝 What You Need to Set Up

### 1. Docker Hub Account
- Sign up at https://hub.docker.com
- Get your username

### 2. GitHub Secrets
Add to GitHub: Settings → Secrets → Actions
```
DOCKER_USERNAME = your-dockerhub-username
DOCKER_PASSWORD = your-dockerhub-password-or-token
```

### 3. Server Setup
Your server needs:
- Docker installed
- Access to pull from Docker Hub
- Ports 8000 (backend) and 3000 (frontend) open

## ✅ Summary

**Docker Deployment Flow:**
```
Code → CI → Docker Build → Docker Hub → Server Pulls → Container Runs → ✅ Live!
```

**Files Involved:**
- `Dockerfile` = Recipe for building image
- `docker-build.yml` = Builds and pushes image
- Docker Hub = Stores images
- Server = Pulls and runs containers

**Benefits:**
- ✅ Works the same everywhere
- ✅ Easy to update (just pull new image)
- ✅ Isolated (doesn't affect server)
- ✅ Can run multiple versions
- ✅ Easy to rollback

