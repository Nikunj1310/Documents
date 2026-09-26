# Docker Best Practices

> **Prerequisites:** Read [[Introduction to Docker]], [[Writing Your First Dockerfile]], and [[Docker Compose for Multi-Service Apps]] first.

---

## Why Best Practices Matter

A beginner's Dockerfile that "just works" often has hidden problems:
- **Security vulnerabilities** (running as root, exposed secrets)
- **Massive image sizes** (900 MB for a simple Node.js app)
- **Slow builds** (re-installing dependencies every time)
- **Production failures** (works locally, breaks in deployment)

This guide teaches you production-grade patterns from day one.

---

## 1. Security Best Practices

### Never Run as Root

**The Problem:**
By default, processes inside containers run as **root** (user ID 0). If an attacker exploits your app, they have root access inside the container — and potentially the host machine.

**Bad Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "index.js"]  # ❌ Runs as root!
```

**Check who's running:**
```bash
docker run myapp whoami
# Output: root
```

---

**Good Dockerfile (Non-Root User):**
```dockerfile
FROM node:18-alpine

# Create a non-root user (or use the built-in 'node' user)
USER node

WORKDIR /app

# Ensure the 'node' user owns the /app directory
COPY --chown=node:node package*.json ./
RUN npm ci --only=production

COPY --chown=node:node . .

CMD ["node", "index.js"]  # ✅ Runs as 'node' user
```

**Verify:**
```bash
docker run myapp whoami
# Output: node
```

**Key Points:**
- Official Node.js images include a `node` user by default.
- `--chown=node:node` ensures copied files are owned by the `node` user.
- `USER node` must come **before** `WORKDIR` or file operations.

---

### Never Bake Secrets Into Images

**Bad Practice:**
```dockerfile
ENV DATABASE_PASSWORD=secret123  # ❌ Visible in image layers!
```

**Why it's bad:**
Anyone who has your image can inspect its layers and extract secrets:
```bash
docker history myapp
docker inspect myapp
```

---

**Good Practice: Inject Secrets at Runtime**

**Option 1: Environment Variables (Docker Compose)**
```yaml
services:
  api:
    environment:
      - DATABASE_PASSWORD=${DB_PASSWORD}  # From .env file
```

**Option 2: Docker Secrets (Swarm/Kubernetes)**
```yaml
services:
  api:
    secrets:
      - db_password

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

**Option 3: Environment Variables (CLI)**
```bash
docker run -e DATABASE_PASSWORD=$DB_PASSWORD myapp
```

---

### Use Official Base Images Only

**Bad:**
```dockerfile
FROM random-user/node:18  # ❌ Unknown source
```

**Good:**
```dockerfile
FROM node:18-alpine  # ✅ Official Docker image
```

**How to verify official images on Docker Hub:**
1. Go to [hub.docker.com](https://hub.docker.com)
2. Search for "node"
3. Look for the **"Docker Official Image"** badge

---

### Scan Images for Vulnerabilities

Use **Docker Scout** (built into Docker) to scan for known CVEs (Common Vulnerabilities and Exposures).

```bash
docker scout cves myapp
```

**Output example:**
```
✗ HIGH: CVE-2023-12345 in openssl@1.1.1
  Upgrade to openssl@1.1.1w to fix
```

**Fix vulnerabilities:**
1. Update base image: `FROM node:18-alpine` → `FROM node:20-alpine`
2. Rebuild: `docker build -t myapp .`
3. Rescan: `docker scout cves myapp`

**Alternative tools:**
- **Snyk:** `snyk container test myapp`
- **Trivy:** `trivy image myapp`

---

### Use Specific Image Tags (Never `latest`)

**Bad:**
```dockerfile
FROM node:latest  # ❌ Unpredictable
```

**Why it's bad:**
- `latest` changes over time (Node 18 today, Node 22 next month).
- Your app breaks when dependencies update unexpectedly.
- Builds aren't reproducible.

---

**Good:**
```dockerfile
FROM node:18-alpine  # ✅ Pinned to Node.js 18
```

**Best:**
```dockerfile
FROM node:18.17.1-alpine3.18  # ✅ Pinned to exact version
```

**How to find exact tags:**
Visit [hub.docker.com/r/library/node/tags](https://hub.docker.com/_/node/tags)

---

## 2. Image Size Optimization

### Use Alpine or Distroless Images

**Size comparison:**
- `node:18` — **~900 MB**
- `node:18-slim` — **~200 MB**
- `node:18-alpine` — **~40 MB**
- `gcr.io/distroless/nodejs18-debian11` — **~120 MB** (most secure)

**Alpine Linux:**
- Minimal Linux distribution (~5 MB base)
- Uses `apk` package manager
- Good for most use cases

**Distroless:**
- Contains only your app + runtime (no shell, no package manager)
- Most secure (no tools for attackers to exploit)
- Harder to debug (no `sh` to exec into)

**Example:**
```dockerfile
FROM node:18-alpine  # Small and practical
```

---

### Multi-Stage Builds (Critical for Production)

**The Problem:**
Your app needs build tools (`npm`, `gcc`, `make`) to install dependencies, but the production image doesn't need them. Including build tools in the final image wastes space and increases attack surface.

**Bad (Single-Stage Build):**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install  # Installs devDependencies too
COPY . .
CMD ["node", "index.js"]
```

**Image size:** ~200 MB (includes devDependencies, npm cache, build tools)

---

**Good (Multi-Stage Build):**
```dockerfile
# ------------------
# STAGE 1: Dependencies
# ------------------
FROM node:18-alpine AS dependencies

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production  # Only production dependencies

# ------------------
# STAGE 2: Production
# ------------------
FROM node:18-alpine AS production

USER node

WORKDIR /app

# Copy ONLY production node_modules from stage 1
COPY --chown=node:node --from=dependencies /app/node_modules ./node_modules

# Copy application code
COPY --chown=node:node . .

EXPOSE 3000

CMD ["node", "index.js"]
```

**Image size:** ~50 MB (no devDependencies, no npm cache)

---

**How it works:**
1. **Stage 1** (`dependencies`): Installs only production dependencies.
2. **Stage 2** (`production`): Copies `node_modules` from stage 1, ignoring everything else from that stage.
3. Docker discards stage 1 from the final image.

**Benefits:**
- Final image only contains runtime dependencies.
- No `npm`, no build tools, no devDependencies.
- Smaller, faster, more secure.

---

### Clean Up in the Same Layer

**Bad:**
```dockerfile
RUN apk add --no-cache python3 build-base
RUN npm install
RUN apk del python3 build-base  # ❌ Doesn't reduce image size!
```

**Why?** Each `RUN` creates a new layer. The cleanup command removes files from the filesystem **view**, but the layer with the installed packages still exists in the image.

---

**Good:**
```dockerfile
RUN apk add --no-cache python3 build-base && \
    npm install && \
    apk del python3 build-base
```

**Better (Multi-Stage):**
```dockerfile
# Install build tools in a separate stage, don't carry them to production
FROM node:18-alpine AS builder
RUN apk add --no-cache python3 build-base
RUN npm install

FROM node:18-alpine AS production
COPY --from=builder /app/node_modules ./node_modules
```

---

### Use `.dockerignore`

**The Problem:**
Without `.dockerignore`, Docker copies **everything** into the build context, including:
- `node_modules` (100+ MB)
- `.git` (entire Git history)
- Log files, temporary files, secrets

**Impact:**
- Slow builds (uploading gigabytes to Docker daemon)
- Larger images (if you accidentally `COPY . .` before filtering)

---

**Essential `.dockerignore`:**
```
node_modules
npm-debug.log
.git
.gitignore
.env
.env.local
*.log
coverage
.DS_Store
Dockerfile
.dockerignore
README.md
.vscode
.idea
```

**Test it:**
```bash
# Without .dockerignore
docker build -t myapp .
# Sending build context to Docker daemon: 512 MB

# With .dockerignore
docker build -t myapp .
# Sending build context to Docker daemon: 2 MB
```

---

## 3. Build Speed Optimization

### Layer Caching Strategy

Docker caches layers. If a layer hasn't changed, Docker reuses it. **Order matters.**

**Bad (Breaks Cache on Every Code Change):**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .                    # ❌ Copies everything (including code)
RUN npm install             # Cache invalidated every time code changes
CMD ["node", "index.js"]
```

---

**Good (Optimized for Cache):**
```dockerfile
FROM node:18-alpine
WORKDIR /app

# 1. Copy dependency files first (change rarely)
COPY package*.json ./

# 2. Install dependencies (cached unless package.json changes)
RUN npm ci --only=production

# 3. Copy source code (changes frequently)
COPY . .

CMD ["node", "index.js"]
```

**Rebuild times:**
- Code change: **3 seconds** (only rebuilds `COPY . .` and `CMD`)
- Dependency change: **30 seconds** (rebuilds from `RUN npm ci` onward)

---

### Use `npm ci` Instead of `npm install`

| Command | Use Case | Speed | Lock File |
|---------|----------|-------|-----------|
| `npm install` | Development | Slower | May update `package-lock.json` |
| `npm ci` | CI/CD & Docker | Faster | Strictly follows `package-lock.json` |

**Why `npm ci` is better for Docker:**
- Faster (skips dependency resolution)
- Deletes `node_modules` before installing (clean slate)
- Fails if `package.json` and `package-lock.json` are out of sync

```dockerfile
RUN npm ci --only=production
```

---

### BuildKit Features (Modern Docker)

Enable **BuildKit** for faster builds and advanced features:

```bash
export DOCKER_BUILDKIT=1
docker build -t myapp .
```

**Or set permanently in `/etc/docker/daemon.json`:**
```json
{
  "features": {
    "buildkit": true
  }
}
```

**BuildKit benefits:**
- Parallel layer builds
- Better caching
- Build secrets (inject secrets without baking them into layers)

---

## 4. Production-Ready Dockerfile Template

This template combines all best practices:

```dockerfile
# syntax=docker/dockerfile:1

# ------------------
# STAGE 1: Dependencies
# ------------------
FROM node:18-alpine AS dependencies

WORKDIR /app

# Copy dependency manifests
COPY package*.json ./

# Install production dependencies only
RUN npm ci --only=production && \
    npm cache clean --force

# ------------------
# STAGE 2: Production
# ------------------
FROM node:18-alpine AS production

# Security: Run as non-root user
USER node

# Set working directory
WORKDIR /app

# Copy dependencies from stage 1
COPY --chown=node:node --from=dependencies /app/node_modules ./node_modules

# Copy application code
COPY --chown=node:node . .

# Document exposed port
EXPOSE 3000

# Set environment
ENV NODE_ENV=production

# Health check (optional but recommended)
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

# Run the app
CMD ["node", "index.js"]
```

**Features:**
✅ Multi-stage build (small image)
✅ Non-root user (security)
✅ Layer caching optimized
✅ Production dependencies only
✅ Health check included

---

## 5. Docker Compose Best Practices

### Use `.env` Files for Configuration

**Bad:**
```yaml
services:
  api:
    environment:
      - DB_PASSWORD=secret123  # ❌ Hardcoded secret
```

**Good:**
```yaml
services:
  api:
    environment:
      - DB_PASSWORD=${DB_PASSWORD}  # ✅ From .env
```

**`.env`:**
```env
DB_PASSWORD=secret123
```

**Don't forget:**
```bash
echo ".env" >> .gitignore
```

---

### Use Named Volumes (Not Bind Mounts)

**Bad (Bind Mount):**
```yaml
volumes:
  - ./data:/var/lib/postgresql/data  # ❌ Host path dependency
```

**Problems:**
- Path must exist on host
- Doesn't work on Windows (path format issues)
- Hard to manage across environments

---

**Good (Named Volume):**
```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data  # ✅ Docker-managed

volumes:
  postgres_data:
```

**Benefits:**
- Works on all platforms
- Docker manages storage location
- Easy to back up: `docker run --rm -v postgres_data:/data -v $(pwd):/backup alpine tar czf /backup/backup.tar.gz /data`

---

### Restart Policies

```yaml
services:
  api:
    restart: unless-stopped  # ✅ Restart on crash, unless manually stopped
```

**Options:**
- `no` — Never restart (default, bad for production)
- `always` — Always restart (even after reboot)
- `on-failure` — Restart only if container exits with error code
- `unless-stopped` — Restart unless you explicitly stopped it (recommended)

---

### Health Checks in Compose

```yaml
services:
  db:
    image: postgres:15-alpine
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s

  api:
    depends_on:
      db:
        condition: service_healthy  # ✅ Wait for DB to be ready
```

**Without health checks:** Your app starts before the database is ready and crashes.

**With health checks:** Docker waits until the database passes the health check.

---

## 6. Debugging and Troubleshooting

### View Container Logs

```bash
# Follow logs in real-time
docker logs -f <container_name>

# Last 50 lines
docker logs --tail=50 <container_name>

# With timestamps
docker logs -t <container_name>
```

---

### Execute Commands Inside Running Container

```bash
# Open a shell
docker exec -it <container_name> sh

# Run a one-off command
docker exec <container_name> ls -la /app

# Check environment variables
docker exec <container_name> env
```

---

### Inspect Container Details

```bash
# Full container config
docker inspect <container_name>

# Just the IP address
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container_name>

# Environment variables
docker inspect -f '{{.Config.Env}}' <container_name>
```

---

### Check Resource Usage

```bash
docker stats
```

**Output:**
```
CONTAINER     CPU %     MEM USAGE / LIMIT     NET I/O
myapp_api     0.50%     45MB / 2GB            1.2kB / 850B
myapp_db      1.20%     120MB / 2GB           5kB / 3kB
```

---

### Debug Build Issues

```bash
# Build with verbose output
docker build --progress=plain -t myapp .

# Build a specific stage (multi-stage)
docker build --target dependencies -t myapp-deps .

# Don't use cache (force rebuild everything)
docker build --no-cache -t myapp .
```

---

## 7. Production Checklist

Before deploying to production, verify:

- [ ] **Base image is official and pinned** (`node:18-alpine`, not `node:latest`)
- [ ] **Multi-stage build** to minimize image size
- [ ] **Running as non-root user** (`USER node`)
- [ ] **No secrets in image** (use environment variables or Docker secrets)
- [ ] **`.dockerignore` excludes** `node_modules`, `.git`, `.env`
- [ ] **Layer caching optimized** (`COPY package*.json` before `COPY . .`)
- [ ] **Health checks defined** in Dockerfile or Compose
- [ ] **Restart policy set** (`restart: unless-stopped`)
- [ ] **Named volumes for data persistence** (not bind mounts)
- [ ] **Security scan passed** (`docker scout cves`)
- [ ] **Resource limits set** (see below)

---

## 8. Resource Limits (Preventing Container from Hogging Resources)

### In Docker Compose

```yaml
services:
  api:
    image: myapp
    deploy:
      resources:
        limits:
          cpus: '0.5'      # 50% of one CPU core
          memory: 512M     # 512 MB RAM
        reservations:
          cpus: '0.25'     # Guaranteed 25% CPU
          memory: 256M     # Guaranteed 256 MB RAM
```

### In Docker Run

```bash
docker run \
  --cpus="0.5" \
  --memory="512m" \
  --memory-swap="1g" \
  myapp
```

---

## 9. Common Pitfalls and How to Avoid Them

### Pitfall 1: Image Grows Uncontrollably

**Cause:** Not using multi-stage builds, including devDependencies.

**Fix:** Use multi-stage builds + `npm ci --only=production`.

---

### Pitfall 2: Builds Take Forever

**Cause:** Poor layer caching (copying code before dependencies).

**Fix:** Copy `package*.json` first, then `COPY . .`.

---

### Pitfall 3: Container Crashes in Production

**Cause:** Missing environment variables, database connection failures.

**Fix:** Add health checks, retry logic, and proper error handling.

---

### Pitfall 4: Data Lost After `docker compose down`

**Cause:** No volumes defined.

**Fix:** Use named volumes for databases.

---

### Pitfall 5: Security Vulnerabilities

**Cause:** Using outdated base images, running as root.

**Fix:** Pin specific image versions, scan with `docker scout`, use non-root user.

---

## Quick Command Reference

```bash
# Build and tag
docker build -t myapp:v1.0 .

# Run with all flags
docker run -d --name myapp -p 3000:3000 \
  -e NODE_ENV=production \
  --restart unless-stopped \
  --memory="512m" \
  myapp:v1.0

# Compose shortcuts
docker compose up -d --build        # Build and start
docker compose logs -f api          # Follow API logs
docker compose exec api sh          # Shell into API
docker compose down -v              # Stop and remove volumes

# Cleanup
docker system prune -a              # Remove all unused images
docker volume prune                 # Remove unused volumes
```

---

## Key Takeaways

- **Security:** Never run as root, never bake secrets, always scan images.
- **Size:** Use Alpine/distroless + multi-stage builds.
- **Speed:** Optimize layer caching, use `npm ci`.
- **Reliability:** Add health checks, restart policies, resource limits.
- **Production:** Use named volumes, `.env` files, and specific image tags.

---

## What's Next?

Advanced topics:
- **[[Docker Networking Deep Dive]]** — Bridge, host, overlay, macvlan networks
- **[[Docker Volumes and Data Management]]** — Bind mounts, tmpfs, volume drivers
- **[[Docker in CI/CD Pipelines]]** — GitHub Actions, GitLab CI integration
- **[[Kubernetes Basics]]** — Orchestrating containers at scale

Related: [[Introduction to Docker]] | [[Writing Your First Dockerfile]] | [[Docker Compose for Multi-Service Apps]]
