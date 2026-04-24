# Docker Setup & Docker Hub Deployment

Pipeline CI/CD sudah di-setup untuk otomatis build Docker image dan push ke Docker Hub!

## 📦 Docker Hub Account Setup

### 1. Create Docker Hub Access Token

1. Buka [hub.docker.com](https://hub.docker.com)
2. Login ke account Anda (misbahalkhafadh)
3. Go to **Account Settings** → **Security** → **New Access Token**
4. Configure token:
   - Token name: `github-ci-token` (atau nama lain)
   - Permissions: ✓ Read & Write (untuk push image)
5. Copy token yang di-generate

### 2. Setup GitHub Secrets

1. Buka GitHub repository Anda
2. Go to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret** dan tambahkan:

```
Name: DOCKER_USERNAME
Value: misbahalkhafadh

Name: DOCKER_PASSWORD
Value: [paste Docker Hub access token dari step 1]
```

## 🏗️ Dockerfile Explanation

File `Dockerfile` sudah di-setup dengan multi-stage build:

```dockerfile
# Stage 1: Build dengan Node.js
FROM node:18-alpine AS builder
# Install dependencies & build React app

# Stage 2: Serve dengan Node.js lightweight
FROM node:18-alpine
# Copy built app ke production image
# Run dengan `serve` command untuk static files
```

### Keuntungan:
- ✅ Minimal image size (hanya production files)
- ✅ Fast builds dengan caching
- ✅ Health check built-in
- ✅ Port 3000 exposed

## 🚀 How It Works

### Workflow Trigger

| Branch | Action | Docker Image Tag |
|--------|--------|------------------|
| `main` | push | `latest`, `vX.Y.Z`, `COMMIT_SHA` |
| `develop` | push | `develop`, `vX.Y.Z`, `COMMIT_SHA` |
| `feature/*` | push | ❌ No docker build |
| PR | N/A | ❌ No docker build |

### Example - Push ke main

```bash
# Push to main branch
git push origin main

# CI/CD Pipeline akan:
# 1. Build & Test
# 2. Security Scan
# 3. Build Docker Image
# 4. Push ke Docker Hub dengan tag:
#    - misbahalkhafadh/my-react-app:latest
#    - misbahalkhafadh/my-react-app:v0.0.0 (dari package.json)
#    - misbahalkhafadh/my-react-app:abc1234 (commit SHA)
```

## 🐳 Gunakan Docker Image

### Pull & Run Image

```bash
# Pull latest image
docker pull misbahalkhafadh/my-react-app:latest

# Run container
docker run -d -p 3000:8080 misbahalkhafadh/my-react-app:latest

# Access app
curl http://localhost:3000
```

### Docker Compose (untuk testing)

Create `docker-compose.yml`:
```yaml
version: '3.8'

services:
  app:
    image: misbahalkhafadh/my-react-app:latest
    ports:
      - "3000:3000"
    environment:
      - VITE_ENV=production
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000"]
      interval: 30s
      timeout: 3s
      retries: 3
```

Run:
```bash
docker-compose up -d
docker-compose logs -f
docker-compose down
```

### Deploy ke Production

Setelah image siap di Docker Hub, bisa deploy ke:

1. **Docker Host** (VPS/Server dengan Docker)
   ```bash
   docker pull misbahalkhafadh/my-react-app:latest
   docker run -d -p 80:3000 misbahalkhafadh/my-react-app:latest
   ```

2. **Kubernetes** (K8s cluster)
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: my-react-app
   spec:
     containers:
     - name: app
       image: misbahalkhafadh/my-react-app:latest
       ports:
       - containerPort: 3000
   ```

3. **Docker Swarm**
   ```bash
   docker service create \
     --name my-app \
     --publish 80:3000 \
     misbahalkhafadh/my-react-app:latest
   ```

## 🔍 Verify Image

### Check Docker Hub

1. Buka [hub.docker.com/r/misbahalkhafadh/my-react-app](https://hub.docker.com/r/misbahalkhafadh/my-react-app)
2. Lihat Tags yang tersedia:
   - `latest` 
   - `develop`
   - `v0.0.0`
   - `abc1234def`

### Check Locally

```bash
# List images
docker images

# Inspect image
docker inspect misbahalkhafadh/my-react-app:latest

# Run image
docker run misbahalkhafadh/my-react-app:latest

# Check logs
docker logs <container_id>

# Execute command
docker exec <container_id> sh -c "ls -la dist/"
```

## 📋 Docker Image Specs

- **Base OS**: Alpine Linux (minimal, ~150MB)
- **Node Version**: 18-alpine
- **Port**: 3000
- **Health Check**: Enabled (curl to /health)
- **Start Command**: `serve -s dist -l 3000`
- **Build Time**: ~2-3 minutes (first run), ~30s (cached)

## 🔗 Image URLs

**Docker Hub**: 
- https://hub.docker.com/r/misbahalkhafadh/my-react-app

**Pull Commands**:
```bash
# Latest (dari main)
docker pull misbahalkhafadh/my-react-app:latest

# Develop branch
docker pull misbahalkhafadh/my-react-app:develop

# Specific version
docker pull misbahalkhafadh/my-react-app:v0.0.0

# Specific commit
docker pull misbahalkhafadh/my-react-app:abc1234
```

## ⚠️ Troubleshooting

### Docker build gagal

**Check logs di GitHub Actions:**
1. Repo → Actions tab
2. Click workflow run
3. Click `docker-build` job
4. Lihat error detail

**Common issues:**
- Build context error → Pastikan `docker build .` berjalan locally
- Push failed → Check DOCKER_USERNAME & DOCKER_PASSWORD secrets
- Auth error → Generate new Docker Hub token

### Test locally

```bash
# Build image locally
docker build -t misbahalkhafadh/my-react-app:test .

# Run test
docker run -p 3000:3000 misbahalkhafadh/my-react-app:test

# Check
curl http://localhost:3000
```

### Image too large

Jika image lebih dari 500MB:
- Use `alpine` base image (sudah dilakukan)
- Remove unnecessary dependencies dari `package.json`
- Use `.dockerignore` untuk skip files

### Container tidak jalan

```bash
# Run dengan debug
docker run -it misbahalkhafadh/my-react-app:latest sh

# Inside container
ls -la dist/
npm ls serve
```

## 📚 Next Steps

1. **Setup secrets**: Add DOCKER_USERNAME & DOCKER_PASSWORD ke GitHub
2. **Trigger build**: Push ke `main` atau `develop` branch
3. **Verify**: Check Docker Hub untuk image yang ter-push
4. **Test**: Pull & run image locally
5. **Deploy**: Gunakan image untuk deploy ke production

---

**Questions?** Check GitHub Actions logs atau Docker Hub repository untuk debugging.
