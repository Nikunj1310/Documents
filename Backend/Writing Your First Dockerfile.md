# Writing Your First Dockerfile

> **Prerequisites:** Read [[Introduction to Docker]] first to understand images, containers, and layers.

---

## What is a Dockerfile?

A **Dockerfile** is a text file containing instructions to build a Docker image. Think of it as a recipe that tells Docker:
1. What base operating system to use
2. What software to install
3. Where to put your code
4. How to run your application

**Analogy:** If a Docker image is a house, the Dockerfile is the architectural blueprint that tells builders how to construct it.

---

## Example Project Structure

We'll containerize a simple Node.js Express app. Here's the project structure:

```
my-express-app/
├── package.json
├── package-lock.json
├── index.js
├── Dockerfile          ← We'll create this
└── .dockerignore       ← We'll create this too
```

**`index.js`** (simple Express server):
```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello from Docker!');
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

**`package.json`**:
```json
{
  "name": "my-express-app",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.18.2"
  },
  "scripts": {
    "start": "node index.js"
  }
}
```

---

## Step 1: Create a `.dockerignore` File

Before writing the Dockerfile, create a `.dockerignore` file. This works exactly like `.gitignore` — it tells Docker which files to **exclude** when copying your project into the image.

**Why?**
- Your local `node_modules` folder is huge (can be 100+ MB).
- You don't want to copy it into the image because Docker will install dependencies itself.
- Secrets like `.env` files should never be baked into images.

**`.dockerignore`:**
```
node_modules
npm-debug.log
.git
.env
.DS_Store
Dockerfile
.dockerignore
```

---

## Step 2: Write Your First Dockerfile (Simple Version)

Create a file named `Dockerfile` (no extension) in your project root:

```dockerfile
# Start from the official Node.js 18 image (Alpine variant = small size)
FROM node:18-alpine

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json first (see Layer Caching below)
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of your application code
COPY . .

# Tell Docker the container listens on port 3000 (documentation only)
EXPOSE 3000

# Define the command to run when the container starts
CMD ["node", "index.js"]
```

---

## Breaking Down Each Instruction

### `FROM node:18-alpine`
- **What:** Specifies the **base image** your image builds on top of.
- **Why `alpine`?** Alpine Linux is a minimal distribution (~5 MB). `node:18-alpine` is ~40 MB vs `node:18` at ~900 MB.
- **Format:** `FROM <image>:<tag>`

**Common base images:**
- `node:18-alpine` — Node.js 18 on Alpine Linux
- `python:3.11-slim` — Python 3.11 minimal
- `nginx:alpine` — Nginx web server

---

### `WORKDIR /app`
- **What:** Sets the working directory inside the container. All subsequent commands run from this directory.
- **Why:** Without this, files would scatter in the root `/` directory. `/app` is a convention.
- **Effect:** If `/app` doesn't exist, Docker creates it automatically.

**Analogy:** Like running `cd /app` in a terminal, except it persists for all future commands.

---

### `COPY package*.json ./`
- **What:** Copies `package.json` and `package-lock.json` from your **host machine** into the container's `/app` directory.
- **Why copy dependencies first?** Layer caching optimization (explained below).
- **Syntax:** `COPY <source_on_host> <destination_in_container>`

**The `*` wildcard:** Matches `package.json` and `package-lock.json` in one command.

---

### `RUN npm install`
- **What:** Executes a command **inside the container** during the build process.
- **Why:** Installs Node.js dependencies listed in `package.json`.
- **Creates a layer:** This layer contains your `node_modules` folder.

**Key Point:** `RUN` executes at **build time** (when you run `docker build`), not when the container starts.

---

### `COPY . .`
- **What:** Copies **all remaining files** from your project into `/app`.
- **Why after `npm install`?** Layer caching (see below).
- **Effect:** Your `index.js`, `README.md`, and other files land in `/app`.

---

### `EXPOSE 3000`
- **What:** Documents that the container listens on port 3000.
- **Important:** This does **NOT** actually publish the port. It's documentation for humans and tools.
- **To actually expose the port:** Use `-p` flag when running the container:
  ```bash
  docker run -p 3000:3000 myapp
  ```

---

### `CMD ["node", "index.js"]`
- **What:** Defines the **default command** to run when the container starts.
- **Format:** JSON array syntax: `["executable", "param1", "param2"]`
- **Runs at:** **Runtime** (when you run `docker run`), not build time.

**Alternative formats:**
```dockerfile
CMD ["npm", "start"]        # Runs npm start
CMD ["node", "index.js"]    # Runs node index.js
CMD node index.js           # Shell form (not recommended)
```

**Best Practice:** Use JSON array format for clarity and to avoid shell parsing issues.

---

## Layer Caching: Why Order Matters

Docker builds images **layer by layer**. Each instruction creates a new layer. Docker caches layers and reuses them if nothing changed.

### The Problem with Wrong Order

**Bad Dockerfile (dependencies last):**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .                    # Copies EVERYTHING (including code)
RUN npm install             # Installs dependencies
CMD ["node", "index.js"]
```

**What happens when you change `index.js`?**
1. `COPY . .` changes → Layer invalidated
2. Docker rebuilds `RUN npm install` **even though dependencies didn't change**
3. Slow builds (npm install can take minutes)

---

### The Solution: Copy Dependencies First

**Good Dockerfile (dependencies first):**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./       # Copy ONLY dependency files
RUN npm install             # Install dependencies (cached if unchanged)
COPY . .                    # Copy code (changes frequently)
CMD ["node", "index.js"]
```

**What happens when you change `index.js`?**
1. `COPY package*.json` → No change → Layer pulled from cache
2. `RUN npm install` → No change → Layer pulled from cache ✅
3. `COPY . .` → Changed → Only this layer rebuilds
4. **Result:** Builds finish in seconds, not minutes

**Rule:** Put instructions that change rarely at the top, frequently-changing instructions at the bottom.

---

## Step 3: Build Your Image

Run this command in your project directory (where `Dockerfile` lives):

```bash
docker build -t myapp .
```

**Breaking it down:**
- `docker build` — Build an image from a Dockerfile
- `-t myapp` — Tag (name) the image as "myapp"
- `.` — Build context (current directory)

**What happens:**
1. Docker reads the Dockerfile
2. Executes each instruction
3. Creates a new layer for each instruction
4. Tags the final image as `myapp:latest`

**Output:**
```
[+] Building 12.3s (10/10) FINISHED
 => [1/5] FROM node:18-alpine
 => [2/5] WORKDIR /app
 => [3/5] COPY package*.json ./
 => [4/5] RUN npm install
 => [5/5] COPY . .
 => exporting to image
 => => naming to docker.io/library/myapp:latest
```

---

## Step 4: Run Your Container

```bash
docker run -p 3000:3000 myapp
```

**Breaking it down:**
- `docker run` — Create and start a container
- `-p 3000:3000` — Map host port 3000 to container port 3000
- `myapp` — Use the image we just built

**Port mapping explained:**
```
-p <host_port>:<container_port>
```

Your app runs on port 3000 **inside the container**. To access it from your laptop's browser, you map it to your laptop's port 3000.

**Visualization:**
```
Your Browser (localhost:3000)
        ↓
Host Machine Port 3000
        ↓
Docker Network
        ↓
Container Port 3000
        ↓
Express App listening on 3000
```

**Test it:** Open `http://localhost:3000` in your browser. You should see "Hello from Docker!"

---

## Step 5: Run in Background (Detached Mode)

Stop the container (Ctrl+C), then run in background:

```bash
docker run -d -p 3000:3000 --name myapp_container myapp
```

**New flags:**
- `-d` — Detached mode (runs in background)
- `--name myapp_container` — Give the container a custom name

**Check if it's running:**
```bash
docker ps
```

**Output:**
```
CONTAINER ID   IMAGE   COMMAND              STATUS         PORTS                   NAMES
abc123def456   myapp   "node index.js"      Up 10 seconds  0.0.0.0:3000->3000/tcp  myapp_container
```

---

## Step 6: View Logs

```bash
docker logs myapp_container
```

**Output:**
```
Server running on port 3000
```

**Follow logs in real-time:**
```bash
docker logs -f myapp_container
```

Press Ctrl+C to stop following (container keeps running).

---

## Step 7: Stop and Remove Container

**Stop the container:**
```bash
docker stop myapp_container
```

**Remove the container:**
```bash
docker rm myapp_container
```

**Shortcut (force remove running container):**
```bash
docker rm -f myapp_container
```

---

## Dockerfile Instructions Reference

| Instruction | Purpose | Example |
|-------------|---------|---------|
| `FROM` | Base image | `FROM node:18-alpine` |
| `WORKDIR` | Set working directory | `WORKDIR /app` |
| `COPY` | Copy files from host to container | `COPY . .` |
| `ADD` | Like COPY but can extract archives & download URLs | `ADD archive.tar.gz /app` |
| `RUN` | Execute command during build | `RUN npm install` |
| `CMD` | Default command when container starts | `CMD ["node", "index.js"]` |
| `ENTRYPOINT` | Configurable executable (advanced) | `ENTRYPOINT ["node"]` |
| `EXPOSE` | Document which port the app uses | `EXPOSE 3000` |
| `ENV` | Set environment variables | `ENV NODE_ENV=production` |
| `ARG` | Build-time variables | `ARG VERSION=1.0` |
| `VOLUME` | Create mount point for persistent data | `VOLUME /data` |
| `USER` | Run commands as specific user | `USER node` |

---

## Common Beginner Mistakes

### 1. **Forgetting `.dockerignore`**
**Problem:** Copies `node_modules` from host, making image huge and builds slow.

**Solution:** Always create `.dockerignore` and exclude `node_modules`.

---

### 2. **Using `COPY . .` Before `RUN npm install`**
**Problem:** Every code change forces npm install to re-run.

**Solution:** Copy `package*.json` first, then `COPY . .` after.

---

### 3. **Forgetting `-p` Flag**
**Problem:** Container runs but you can't access it from browser.

**Error:** "Connection refused" when visiting `localhost:3000`.

**Solution:** Always map ports with `-p <host>:<container>`.

---

### 4. **Using `node:latest`**
**Problem:** "Latest" changes over time. Your image might break when Node.js updates.

**Solution:** Use specific versions: `node:18-alpine`, `node:20-slim`.

---

## Hands-On Exercise

**Challenge:** Modify the Dockerfile to:
1. Set an environment variable `PORT=4000`
2. Make the app listen on that port
3. Rebuild and run

**Hint for `index.js`:**
```javascript
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

**Hint for `Dockerfile`:**
```dockerfile
ENV PORT=4000
EXPOSE 4000
```

**Run command:**
```bash
docker build -t myapp .
docker run -p 4000:4000 myapp
```

Visit `http://localhost:4000`.

---

## What's Next?

You've learned how to containerize a single application. In real projects, you'll need multiple services (app + database + cache).

Continue to **[[Docker Compose for Multi-Service Apps]]** to learn how to orchestrate multiple containers.

---

## Key Takeaways

- **Dockerfile** is a recipe for building images.
- **Layer caching** speeds up builds — put rarely-changing instructions first.
- **`COPY package*.json` before `COPY . .`** is critical for fast rebuilds.
- **`-p` flag maps ports** so you can access the container from your host.
- **Use specific image tags** (`node:18-alpine`) instead of `latest`.
- **`.dockerignore` excludes files** from being copied into the image.

Related: [[Introduction to Docker]] | [[Docker Compose for Multi-Service Apps]] | [[Docker Best Practices]]
