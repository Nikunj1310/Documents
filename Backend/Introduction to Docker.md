# Introduction to Docker

> **What You'll Learn:** This guide explains Docker from absolute scratch — what containers are, why they exist, and how to use them in real projects. Designed for beginners with zero Docker experience.

---

## What Problem Does Docker Solve?

Imagine you build an app on your laptop. It works perfectly. You send it to your friend, and suddenly it crashes with "module not found" or "version mismatch" errors.

**Why?** Your laptop has:
- Node.js version 18
- PostgreSQL installed
- Specific system libraries

Your friend's machine has:
- Node.js version 16
- No PostgreSQL
- Different operating system (Windows vs your Linux)

This is the **"It works on my machine"** problem. Docker solves it.

---

## What is Docker? (The Simple Explanation)

**Docker** packages your application + all its dependencies into a single, portable unit called a **container**.

Think of it like this:
- **Without Docker:** You ship only your code. The receiver must install Node.js, databases, libraries, etc.
- **With Docker:** You ship a complete "box" containing your code, Node.js, databases, libraries — everything. The receiver just opens the box and runs it.

### Real-World Analogy

| Concept | Real World | Docker World |
|---------|------------|--------------|
| **Shipping Container** | A standardized metal box that fits on any ship, train, or truck. | A Docker container runs on any machine with Docker installed. |
| **Blueprint** | Architectural plans for building a house. | A `Dockerfile` — instructions for building a container. |
| **House Built from Blueprint** | The actual constructed house. | A **Docker Image** — the built artifact from the Dockerfile. |
| **People Living in the House** | Multiple families can live in identical houses built from the same blueprint. | Multiple **Containers** running from the same image. |

---

## Core Concepts (Building Blocks)

### 1. **Image**
A **read-only template** that contains:
- Your application code
- Runtime (e.g., Node.js, Python)
- System libraries
- Dependencies (npm packages, pip packages)

**Programming Analogy:** An image is like a **Class** in object-oriented programming.

**Example:** `node:18-alpine` is an official image containing Node.js version 18 on a minimal Linux distribution.

---

### 2. **Container**
A **running instance** of an image. It's isolated from your host machine but shares the OS kernel (making it lightweight).

**Programming Analogy:** A container is like an **Object** (instance) created from a Class.

**Key Point:** You can run multiple containers from the same image, just like you can create multiple objects from one class.

**Example:** 
```bash
docker run node:18-alpine
```
This creates and starts a container from the `node:18-alpine` image.

---

### 3. **Layers (How Images Are Built)**

Images are built in **layers**, stacked on top of each other. Each instruction in a `Dockerfile` creates a new layer.

**Why This Matters:** Docker caches layers. If a layer hasn't changed, Docker reuses it instead of rebuilding, making builds extremely fast.

**Analogy:** Think of layers like **Git commits**. Each commit builds on the previous one. If you only change the last commit, Git doesn't recompute the entire history.

**Example:**
```dockerfile
FROM node:18-alpine          # Layer 1: Base operating system + Node.js
WORKDIR /app                 # Layer 2: Set working directory
COPY package*.json ./        # Layer 3: Copy dependency files
RUN npm install              # Layer 4: Install dependencies
COPY . .                     # Layer 5: Copy application code
CMD ["node", "index.js"]     # Layer 6: Define startup command
```

If you change only `index.js`, Docker rebuilds **only Layer 5 and 6**. Layers 1-4 are pulled from cache.

---

## How Docker Works (Under the Hood)

### Traditional Virtual Machines vs. Docker Containers

| Feature | Virtual Machine (VM) | Docker Container |
|---------|----------------------|------------------|
| **What it virtualizes** | Entire operating system | Only the application and dependencies |
| **Size** | GBs (includes full OS) | MBs (shares host OS kernel) |
| **Startup time** | Minutes | Seconds |
| **Resource usage** | Heavy (each VM runs its own OS) | Lightweight (shares host kernel) |
| **Isolation** | Complete (separate OS) | Process-level (shared kernel) |

**Visual Comparison:**

**Virtual Machine:**
```
┌─────────────────────────────────────┐
│        Host Operating System        │
├─────────────────────────────────────┤
│         Hypervisor (VMware)         │
├──────────┬──────────┬───────────────┤
│ Guest OS │ Guest OS │   Guest OS    │
│ App A    │ App B    │   App C       │
└──────────┴──────────┴───────────────┘
```

**Docker:**
```
┌─────────────────────────────────────┐
│        Host Operating System        │
├─────────────────────────────────────┤
│           Docker Engine             │
├──────────┬──────────┬───────────────┤
│ Container│ Container│  Container    │
│ App A    │ App B    │   App C       │
└──────────┴──────────┴───────────────┘
```

**Key Takeaway:** Containers share the host OS kernel, making them much lighter and faster than VMs.

---

## Your First Docker Command (Hands-On)

Let's run a container to see Docker in action.

```bash
docker run hello-world
```

**What happens:**
1. Docker checks if the `hello-world` image exists on your machine.
2. It doesn't, so Docker **pulls** (downloads) it from Docker Hub (a public image registry).
3. Docker creates a container from the image.
4. The container runs, prints a message, and exits.

**Output:**
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

---

## Understanding Docker Hub

**Docker Hub** is like **npm** for Docker images. It's a public registry where you can:
- Download official images (Node.js, PostgreSQL, Redis, etc.)
- Upload your own images

**Example:** Search for Node.js images:
```bash
docker search node
```

**Official images** are maintained by Docker and trusted. They're marked with `[OK]` in the search results.

---

## Common Docker Commands (Essential Reference)

### Working with Images

| Command | What It Does | Example |
|---------|--------------|---------|
| `docker pull <image>` | Download an image from Docker Hub | `docker pull node:18-alpine` |
| `docker images` | List all images on your machine | `docker images` |
| `docker rmi <image>` | Delete an image | `docker rmi node:18-alpine` |
| `docker build -t <name> .` | Build an image from a Dockerfile | `docker build -t myapp .` |

### Working with Containers

| Command | What It Does | Example |
|---------|--------------|---------|
| `docker run <image>` | Create and start a container | `docker run node:18-alpine` |
| `docker run -d <image>` | Run in detached mode (background) | `docker run -d myapp` |
| `docker run -p <host>:<container> <image>` | Map ports (expose container port to host) | `docker run -p 3000:3000 myapp` |
| `docker run --name <name> <image>` | Give the container a custom name | `docker run --name api myapp` |
| `docker ps` | List running containers | `docker ps` |
| `docker ps -a` | List all containers (including stopped) | `docker ps -a` |
| `docker stop <container>` | Stop a running container | `docker stop api` |
| `docker start <container>` | Start a stopped container | `docker start api` |
| `docker rm <container>` | Delete a container | `docker rm api` |
| `docker logs <container>` | View container logs | `docker logs api` |
| `docker logs -f <container>` | Follow logs in real-time | `docker logs -f api` |
| `docker exec -it <container> <command>` | Run a command inside a running container | `docker exec -it api sh` |

### Cleanup

| Command | What It Does |
|---------|--------------|
| `docker system prune` | Remove all stopped containers, unused networks, dangling images |
| `docker system prune -a` | Remove ALL unused images (not just dangling) |

---

## What's Next?

Now that you understand the basics, the next sections will cover:
1. **[[Writing Your First Dockerfile]]** — Create a custom image for a Node.js app
2. **[[Docker Compose for Multi-Service Apps]]** — Run your app + database + cache together
3. **[[Docker Best Practices]]** — Security, optimization, and production patterns

---

## Quick Troubleshooting

### "Cannot connect to Docker daemon"
Docker service isn't running.
```bash
sudo systemctl start docker
sudo systemctl enable docker  # Start on boot
```

### "Permission denied while trying to connect"
Your user isn't in the `docker` group.
```bash
sudo usermod -aG docker $USER
newgrp docker
```

### Port already in use
Another container (or process) is using the port.
```bash
# Find what's using port 3000
sudo lsof -i :3000

# Stop the conflicting container
docker ps
docker stop <container_name>
```

---

## Key Takeaways

- **Docker solves "it works on my machine"** by packaging apps + dependencies together.
- **Images** are blueprints (like classes), **containers** are running instances (like objects).
- **Layers make builds fast** through caching.
- **Containers are lighter than VMs** because they share the host OS kernel.
- **Docker Hub** is the public registry for images (like npm for Node.js).

Continue to **[[Writing Your First Dockerfile]]** to build your own Docker image.
