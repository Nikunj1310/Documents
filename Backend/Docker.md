
> **Master Index:** Everything you need to learn, practice, and deploy with Docker. Start here for quick reference or dive deep into specific topics.

---

## What is Docker?

Docker packages applications + dependencies into portable containers that run identically everywhere. Solves "works on my machine" problem.

**Core Concept:** Container = lightweight, isolated process sharing the host OS kernel (faster than VMs).

---

## Quick Start (5 Minutes)

```bash
# Install (Pop!_OS)
sudo apt update && sudo apt install docker.io docker-compose-plugin
sudo usermod -aG docker $USER
newgrp docker

# Verify
docker run hello-world

# Run your first container
docker run -p 3000:3000 node:18-alpine
```

---

## Learning Path

###  Core Concepts
**Start here if Docker is new to you.**

**[[Introduction to Docker]]** — What are containers? Images? Layers? How does Docker work vs VMs?

**Topics covered:**
- Why Docker exists (the "works on my machine" problem)
- Images vs Containers vs Layers
- Docker Hub and official images
- Essential commands (`run`, `ps`, `logs`, `exec`)
- First hands-on: `docker run hello-world`

**Time:** 20 minutes read

---

### Building — Create Your Own Images
**Learn to containerize your applications.**

**[[Writing Your First Dockerfile]]** — Step-by-step guide to building custom images

**Topics covered:**
- Dockerfile syntax (`FROM`, `COPY`, `RUN`, `CMD`, `EXPOSE`)
- Layer caching (why order matters)
- `.dockerignore` (what to exclude)
- Building and running (`docker build -t`, `docker run -p`)
- Port mapping (host ↔ container)
- Hands-on: Containerize a Node.js Express app

**Time:** 30 minutes read + 15 minutes practice

---

### Multi-Container — Real Applications
**Run app + database + cache together.**

**[[Docker Compose for Multi-Service Apps]]** — Orchestrate multiple containers with one command

**Topics covered:**
- Why Compose exists (managing multiple containers)
- `docker-compose.yml` structure (services, networks, volumes)
- Service networking (how containers talk: `db:5432`)
- Volumes (persisting database data)
- Environment variables (`.env` files)
- Dependency management (`depends_on`, health checks)
- All commands (`up`, `down`, `logs`, `exec`)
- Hands-on: Node.js + PostgreSQL + Redis stack

**Time:** 45 minutes read + 30 minutes practice

---

### Production — Security & Optimization
**Make your containers production-ready.**

**[[Docker Best Practices]]** — Security, performance, and reliability patterns

**Topics covered:**
- **Security:** Non-root users, no secrets in images, vulnerability scanning
- **Size optimization:** Alpine images, multi-stage builds, layer cleanup
- **Speed optimization:** Layer caching strategies, `npm ci`
- **Production patterns:** Health checks, restart policies, resource limits
- **Debugging:** Logs, exec, inspect, stats
- Production-ready Dockerfile template
- Complete checklist before deployment

**Time:** 40 minutes read

---

## Quick Reference

### Essential Commands

#### Images
```bash
docker pull <image>              # Download image
docker images                    # List images
docker build -t <name> .         # Build from Dockerfile
docker rmi <image>               # Delete image
docker scout cves <image>        # Scan vulnerabilities
```

#### Containers
```bash
docker run <image>               # Create and start
docker run -d <image>            # Run in background
docker run -p 8080:3000 <image>  # Map ports
docker run --name <name> <image> # Custom name
docker ps                        # List running
docker ps -a                     # List all
docker stop <container>          # Stop
docker start <container>         # Start stopped
docker rm <container>            # Delete
docker logs -f <container>       # Follow logs
docker exec -it <container> sh   # Shell inside
```

#### Docker Compose
```bash
docker compose up                # Start all services
docker compose up -d             # Background mode
docker compose up -d --build     # Rebuild images
docker compose down              # Stop and remove
docker compose down -v           # Also remove volumes
docker compose logs -f           # Follow logs
docker compose logs -f <service> # Specific service
docker compose exec <service> sh # Shell into service
docker compose ps                # List services
docker compose restart <service> # Restart one service
```

#### Cleanup
```bash
docker system prune              # Remove stopped containers + unused images
docker system prune -a           # Remove ALL unused images
docker volume prune              # Remove unused volumes
```

---

## Concept Map

### Image → Container → Service

```
Dockerfile (recipe)
    ↓ docker build
Image (blueprint)
    ↓ docker run
Container (running instance)
    ↓ docker-compose.yml
Service (container + config)
    ↓ docker compose up
Multi-container application
```

---

## Key Concepts (Quick Definitions)

| Concept               | What It Is                                            | Example                                  |
| --------------------- | ----------------------------------------------------- | ---------------------------------------- |
| **Image**             | Read-only template (app + dependencies)               | `node:18-alpine`                         |
| **Container**         | Running instance of an image                          | `docker run node:18-alpine`              |
| **Layer**             | Each Dockerfile instruction creates a layer (cached)  | `RUN npm install` = 1 layer              |
| **Dockerfile**        | Recipe for building an image                          | Instructions: `FROM`, `COPY`, `RUN`      |
| **Docker Hub**        | Public registry for images (like npm)                 | [hub.docker.com](https://hub.docker.com) |
| **Volume**            | Persistent storage (survives container deletion)      | `postgres_data:/var/lib/postgresql/data` |
| **Network**           | Allows containers to communicate                      | Service name = hostname (`db:5432`)      |
| **Service**           | Container definition in Compose                       | `api`, `db`, `cache`                     |
| **Multi-stage build** | Build in stages, only ship final stage (small images) | Stage 1: build, Stage 2: production      |

---

## Common Patterns (Copy-Paste Ready)

### Production Dockerfile (Node.js)
```dockerfile
FROM node:18-alpine AS dependencies
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine AS production
USER node
WORKDIR /app
COPY --chown=node:node --from=dependencies /app/node_modules ./node_modules
COPY --chown=node:node . .
EXPOSE 3000
ENV NODE_ENV=production
CMD ["node", "index.js"]
```

### Docker Compose (App + DB + Cache)
```yaml
services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgres://${DB_USER}:${DB_PASSWORD}@db:5432/${DB_NAME}
      - REDIS_URL=redis://cache:6379
    depends_on:
      - db
      - cache
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

  cache:
    image: redis:7-alpine
    restart: unless-stopped

volumes:
  postgres_data:
```

### Essential `.dockerignore`
```
node_modules
npm-debug.log
.git
.env
*.log
.DS_Store
```

---

## Troubleshooting Guide

### "Cannot connect to Docker daemon"
```bash
sudo systemctl start docker
sudo systemctl enable docker
```

### "Permission denied"
```bash
sudo usermod -aG docker $USER
newgrp docker
```

### "Port already in use"
```bash
# Find what's using port 3000
sudo lsof -i :3000
# Change host port in docker-compose.yml
ports:
  - "8080:3000"
```

### "Cannot connect to database"
- Check service name matches (`db:5432` in connection string)
- Database takes time to start (add retry logic or health checks)
- Verify both on same network

### "Changes not showing up"
```bash
docker compose up -d --build
```

### "Image too large"
- Use Alpine base images (`node:18-alpine`)
- Multi-stage builds
- Add `.dockerignore`
- Run `docker scout cves` to check

---

## Project Workflows

### New Project Setup
```bash
# 1. Create files
touch Dockerfile docker-compose.yml .dockerignore .env

# 2. Write Dockerfile (see template above)

# 3. Create .env
echo "DB_USER=myuser
DB_PASSWORD=secret123
DB_NAME=mydb" > .env

# 4. Add .env to .gitignore
echo ".env" >> .gitignore

# 5. Build and run
docker compose up -d --build

# 6. Check logs
docker compose logs -f

# 7. Test
curl http://localhost:3000
```

### Daily Development
```bash
# Start stack
docker compose up -d

# View logs
docker compose logs -f api

# Code changes → rebuild
docker compose up -d --build

# Shell into container
docker compose exec api sh

# Stop everything
docker compose down
```

### Before Deployment
```bash
# Security scan
docker scout cves myapp

# Check image size
docker images myapp

# Test production build
docker build -t myapp:prod .
docker run -p 3000:3000 myapp:prod

# Verify non-root
docker run myapp:prod whoami  # Should NOT be root
```

---

## Cheat Sheet (Print This)

| Task | Command |
|------|---------|
| Build image | `docker build -t myapp .` |
| Run container | `docker run -d -p 3000:3000 myapp` |
| View logs | `docker logs -f <container>` |
| Shell inside | `docker exec -it <container> sh` |
| Stop container | `docker stop <container>` |
| Remove container | `docker rm <container>` |
| Start Compose | `docker compose up -d` |
| Rebuild Compose | `docker compose up -d --build` |
| Stop Compose | `docker compose down` |
| View Compose logs | `docker compose logs -f` |
| Clean everything | `docker system prune -a` |

---

## Related Topics

- **[[Caching and Redis]]** — In-memory data store (commonly used with Docker)
- **[[BCrypt]]** — Password hashing (runs in containerized apps)
- **[[JWT Auth]]** — Stateless authentication (perfect for Docker/microservices)
- **[[BullMQ, Background Jobs & Message Queues]]** — Job processing in containers
- **[[Database and Structures/Introduction to Databases and Structures]]** — PostgreSQL, MySQL in Docker
- **[[Database and Structures/MySQL/Introduction to PostgreSQL]]** — Running Postgres in containers

---

## External Resources

- **Official Docs:** [docs.docker.com](https://docs.docker.com)
- **Docker Hub:** [hub.docker.com](https://hub.docker.com) (search for images)
- **Node.js Best Practices:** [github.com/nodejs/docker-node](https://github.com/nodejs/docker-node/blob/main/docs/BestPractices.md)
- **Awesome Docker:** [github.com/veggiemonk/awesome-docker](https://github.com/veggiemonk/awesome-docker)

---

## Practice Exercises

### Exercise 1: Basic Containerization
**Task:** Containerize a simple Express app that responds "Hello Docker" on `/`.

**Steps:**
1. Create `index.js`, `package.json`, `Dockerfile`, `.dockerignore`
2. Build: `docker build -t hello-docker .`
3. Run: `docker run -p 3000:3000 hello-docker`
4. Test: `curl http://localhost:3000`

**Time:** 15 minutes

---

### Exercise 2: Multi-Container App
**Task:** Run Express + PostgreSQL + Redis using Compose.

**Steps:**
1. Create `docker-compose.yml` with 3 services
2. Update app to connect to `db:5432` and `cache:6379`
3. Start: `docker compose up -d`
4. Verify all services: `docker compose ps`
5. Test database connection

**Time:** 30 minutes

---

### Exercise 3: Optimize Image Size
**Task:** Reduce a Node.js image from 900 MB to under 100 MB.

**Steps:**
1. Start with `FROM node:18` (measure size)
2. Switch to `FROM node:18-alpine`
3. Add multi-stage build
4. Compare: `docker images`

**Time:** 20 minutes

---

## Production Checklist

Before deploying to production:

- [ ] Base image pinned to specific version (`node:18.17.1-alpine3.18`)
- [ ] Multi-stage build implemented
- [ ] Running as non-root user (`USER node`)
- [ ] No secrets in Dockerfile or image layers
- [ ] `.dockerignore` excludes `node_modules`, `.git`, `.env`
- [ ] Layer caching optimized (`COPY package*.json` before `COPY . .`)
- [ ] Health checks defined
- [ ] Restart policy set (`restart: unless-stopped`)
- [ ] Named volumes for persistence (not bind mounts)
- [ ] Resource limits configured (CPU, memory)
- [ ] Security scan passed (`docker scout cves`)
- [ ] Image size under 200 MB
- [ ] Logs verified (`docker logs`)
- [ ] Tested in staging environment

---

## When to Use What

| Scenario | Tool |
|----------|------|
| Single container | `docker run` |
| Multiple containers | `docker compose` |
| Production orchestration | Kubernetes, Docker Swarm |
| CI/CD | GitHub Actions + Docker |
| Local development | `docker compose` + volumes for live reload |
| Testing | `docker run --rm` (auto-remove after test) |

---

## Next Steps

After mastering Docker:
1. **Kubernetes** — Container orchestration at scale
2. **CI/CD Pipelines** — Automate Docker builds (GitHub Actions, GitLab CI)
3. **Monitoring** — Prometheus, Grafana for container metrics
4. **Service Mesh** — Istio, Linkerd for microservices
5. **Cloud Deployment** — AWS ECS, Google Cloud Run, Azure Container Instances

---

