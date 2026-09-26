# Docker Compose for Multi-Service Apps

> **Prerequisites:** Read [[Introduction to Docker]] and [[Writing Your First Dockerfile]] first.

---

## The Problem: Managing Multiple Containers

Real applications rarely run in isolation. A typical web app needs:
- **API server** (Node.js/Express)
- **Database** (PostgreSQL/MySQL)
- **Cache** (Redis)
- **Message Queue** (optional: RabbitMQ, BullMQ)

**Without Docker Compose:**
```bash
# Terminal 1: Start PostgreSQL
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=secret postgres

# Terminal 2: Start Redis
docker run -d -p 6379:6379 redis

# Terminal 3: Start your app
docker run -d -p 3000:3000 \
  -e DATABASE_URL=postgres://user:secret@localhost:5432/db \
  -e REDIS_URL=redis://localhost:6379 \
  myapp
```

**Problems:**
- Three separate commands to remember
- Manual port management
- Environment variables scattered everywhere
- Containers don't know about each other
- Hard to share with teammates

---

## The Solution: Docker Compose

**Docker Compose** lets you define and run multi-container applications using a single YAML file.

**With Docker Compose:**
```bash
docker compose up
```

That's it. One command starts your entire stack.

---

## What is Docker Compose?

Docker Compose is a tool for defining and running multi-container Docker applications. You use a YAML file (`docker-compose.yml`) to configure your application's services, networks, and volumes.

**Key Concepts:**

| Concept | Explanation | Example |
|---------|-------------|---------|
| **Service** | A container definition (what image to use, ports, environment variables) | `api`, `db`, `cache` |
| **Network** | Allows containers to communicate using service names as hostnames | Your app connects to `db:5432` instead of `localhost:5432` |
| **Volume** | Persistent storage that survives container restarts | Database data stored in `postgres_data` volume |

---

## Project Structure

```
my-project/
├── docker-compose.yml     ← We'll create this
├── .env                   ← Environment variables
├── .dockerignore
├── Dockerfile
├── package.json
├── package-lock.json
└── index.js
```

---

## Step 1: Create `.env` File (Keep Secrets Safe)

Never hardcode passwords in `docker-compose.yml`. Use environment variables instead.

**`.env`:**
```env
# Database credentials
DB_USER=myuser
DB_PASSWORD=strongpassword123
DB_NAME=myappdb

# Node environment
NODE_ENV=development
```

**Important:** Add `.env` to `.gitignore` and `.dockerignore` so secrets don't leak into version control or images.

---

## Step 2: Write `docker-compose.yml` (Explained)

Create a file named `docker-compose.yml` in your project root:

```yaml
services:
  # ------------------
  # Node.js Express API
  # ------------------
  api:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: express_api
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=${NODE_ENV}
      - DATABASE_URL=postgres://${DB_USER}:${DB_PASSWORD}@db:5432/${DB_NAME}
      - REDIS_URL=redis://cache:6379
    depends_on:
      - db
      - cache
    networks:
      - app_network
    restart: unless-stopped

  # ------------------
  # PostgreSQL Database
  # ------------------
  db:
    image: postgres:15-alpine
    container_name: postgres_db
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app_network
    restart: unless-stopped

  # ------------------
  # Redis Cache
  # ------------------
  cache:
    image: redis:7-alpine
    container_name: redis_cache
    networks:
      - app_network
    restart: unless-stopped

# ------------------
# Named Volumes
# ------------------
volumes:
  postgres_data:
    driver: local

# ------------------
# Networks
# ------------------
networks:
  app_network:
    driver: bridge
```

---

## Breaking Down the YAML (Line by Line)

### Service: `api` (Your Application)

```yaml
api:
  build:
    context: .
    dockerfile: Dockerfile
```
- **`build`**: Instead of using a pre-built image, Docker builds one from your `Dockerfile`.
- **`context: .`**: The build context (where Docker looks for files). `.` means current directory.
- **`dockerfile: Dockerfile`**: The name of the Dockerfile to use.

---

```yaml
  container_name: express_api
```
- **`container_name`**: Give the container a readable name instead of a random generated one.
- **Without this:** Docker generates names like `my-project-api-1`.
- **With this:** The container is named `express_api`.

---

```yaml
  ports:
    - "3000:3000"
```
- **`ports`**: Map host port to container port.
- **Format:** `"<host_port>:<container_port>"`
- **Effect:** You can access your app at `http://localhost:3000`.

---

```yaml
  environment:
    - NODE_ENV=${NODE_ENV}
    - DATABASE_URL=postgres://${DB_USER}:${DB_PASSWORD}@db:5432/${DB_NAME}
    - REDIS_URL=redis://cache:6379
```
- **`environment`**: Set environment variables inside the container.
- **`${DB_USER}`**: Pulls values from `.env` file.
- **`@db:5432`**: Notice we use the **service name** `db`, not `localhost`. Docker's internal DNS resolves `db` to the PostgreSQL container's IP.

---

```yaml
  depends_on:
    - db
    - cache
```
- **`depends_on`**: Controls startup order. Docker starts `db` and `cache` before `api`.
- **Important Caveat:** This only waits for the container to start, **not** for the database to be ready to accept connections. See [[Handling Database Readiness]] for production patterns.

---

```yaml
  networks:
    - app_network
```
- **`networks`**: Attach this service to a custom network.
- **Why?** All services on the same network can communicate using service names as hostnames.

---

```yaml
  restart: unless-stopped
```
- **`restart`**: Restart policy.
- **`unless-stopped`**: Restart the container automatically if it crashes, unless you manually stop it.
- **Options:**
  - `no` — Never restart (default)
  - `always` — Always restart
  - `on-failure` — Restart only if the container exits with an error
  - `unless-stopped` — Restart unless you explicitly stopped it

---

### Service: `db` (PostgreSQL)

```yaml
db:
  image: postgres:15-alpine
```
- **`image`**: Use a pre-built image from Docker Hub instead of building from a Dockerfile.
- **`postgres:15-alpine`**: PostgreSQL version 15 on Alpine Linux (small size).

---

```yaml
  environment:
    POSTGRES_USER: ${DB_USER}
    POSTGRES_PASSWORD: ${DB_PASSWORD}
    POSTGRES_DB: ${DB_NAME}
```
- **PostgreSQL environment variables**: These are specific to the official PostgreSQL image.
- **`POSTGRES_USER`**: Creates a user with this username.
- **`POSTGRES_PASSWORD`**: Sets the password for that user.
- **`POSTGRES_DB`**: Creates a database with this name.

---

```yaml
  volumes:
    - postgres_data:/var/lib/postgresql/data
```
- **`volumes`**: Mount a named volume to persist data.
- **Format:** `<volume_name>:<path_in_container>`
- **`/var/lib/postgresql/data`**: Where PostgreSQL stores its data files inside the container.
- **Why?** Without volumes, all data is lost when the container stops. With volumes, data persists across container restarts.

---

### Service: `cache` (Redis)

```yaml
cache:
  image: redis:7-alpine
  container_name: redis_cache
  networks:
    - app_network
  restart: unless-stopped
```

- **No `ports` section**: Redis is only accessible from within the Docker network, not from your host machine.
- **Why?** Your app connects to Redis using `cache:6379` from inside the network. You don't need to expose it to your laptop unless you want to inspect it with a GUI tool like RedisInsight.

---

### Volumes Section

```yaml
volumes:
  postgres_data:
    driver: local
```

- **`volumes`**: Defines named volumes used by services.
- **`postgres_data`**: The name of the volume.
- **`driver: local`**: Stores data on the host machine's disk (default).
- **Where is the data?** Docker manages it. On Linux: `/var/lib/docker/volumes/`.

---

### Networks Section

```yaml
networks:
  app_network:
    driver: bridge
```

- **`networks`**: Defines custom networks.
- **`app_network`**: The name of the network.
- **`driver: bridge`**: Default network type. Containers on the same bridge network can communicate.

**Docker's Magic DNS:** Inside `app_network`, Docker provides DNS resolution:
- `db` resolves to the PostgreSQL container's IP
- `cache` resolves to the Redis container's IP
- `api` resolves to your app's IP

---

## How Service Communication Works

### Without Docker Compose (Manual Setup)

```javascript
// Your app tries to connect to localhost
const client = new Pool({
  host: 'localhost',  // ❌ Doesn't work! Postgres is in another container
  port: 5432,
  user: 'myuser',
  password: 'secret',
  database: 'myappdb'
});
```

**Problem:** `localhost` inside the `api` container refers to the container itself, not your host machine or the `db` container.

---

### With Docker Compose (Service Names)

```javascript
// Your app uses the service name as hostname
const client = new Pool({
  host: 'db',  // ✅ Works! Docker resolves 'db' to the PostgreSQL container
  port: 5432,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME
});
```

**Or use the environment variable from compose:**
```javascript
const client = new Pool({
  connectionString: process.env.DATABASE_URL
  // postgres://myuser:secret@db:5432/myappdb
});
```

---

## Step 3: Update Your Application Code

**Example `index.js` with PostgreSQL and Redis:**

```javascript
const express = require('express');
const { Pool } = require('pg');
const redis = require('redis');

const app = express();

// PostgreSQL connection using environment variable
const pgClient = new Pool({
  connectionString: process.env.DATABASE_URL
});

// Redis connection using environment variable
const redisClient = redis.createClient({
  url: process.env.REDIS_URL
});

redisClient.connect();

app.get('/', async (req, res) => {
  try {
    // Test PostgreSQL
    const result = await pgClient.query('SELECT NOW()');
    
    // Test Redis
    await redisClient.set('test', 'Hello from Redis!');
    const redisValue = await redisClient.get('test');
    
    res.json({
      message: 'All services working!',
      postgres: result.rows[0],
      redis: redisValue
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

**Update `package.json`:**
```json
{
  "dependencies": {
    "express": "^4.18.2",
    "pg": "^8.11.0",
    "redis": "^4.6.7"
  }
}
```

---

## Step 4: Essential Docker Compose Commands

### Start All Services

```bash
docker compose up
```

**What happens:**
1. Docker reads `docker-compose.yml`
2. Builds the `api` image from your Dockerfile
3. Pulls `postgres:15-alpine` and `redis:7-alpine` from Docker Hub
4. Creates the `app_network` network
5. Creates the `postgres_data` volume
6. Starts containers in dependency order: `db` and `cache` first, then `api`
7. Attaches logs to your terminal (foreground mode)

**Output:**
```
[+] Running 4/4
 ✔ Network my-project_app_network    Created
 ✔ Container postgres_db             Started
 ✔ Container redis_cache             Started
 ✔ Container express_api             Started
```

---

### Start in Background (Detached Mode)

```bash
docker compose up -d
```

**`-d`**: Detached mode. Containers run in the background.

---

### Rebuild Images (After Code Changes)

```bash
docker compose up -d --build
```

**`--build`**: Forces Docker to rebuild images even if they exist. Use this after changing:
- `Dockerfile`
- Application code
- `package.json`

---

### View Logs

```bash
# All services
docker compose logs -f

# Specific service
docker compose logs -f api

# Last 50 lines
docker compose logs --tail=50 api
```

**`-f`**: Follow logs in real-time (like `tail -f`).

---

### Stop All Services

```bash
docker compose down
```

**What happens:**
1. Stops all containers
2. Removes containers
3. Removes the network
4. **Keeps volumes** (your database data is safe)

---

### Stop and Remove Volumes (WARNING)

```bash
docker compose down -v
```

**`-v`**: Removes volumes. **This deletes all database data!** Only use for a fresh start.

---

### Execute Commands Inside Containers

```bash
# Open a shell in the API container
docker compose exec api sh

# Run a command in the database container
docker compose exec db psql -U myuser -d myappdb

# Run npm commands
docker compose exec api npm install some-package
```

---

### Check Running Services

```bash
docker compose ps
```

**Output:**
```
NAME              IMAGE                  STATUS          PORTS
express_api       my-project-api         Up 2 minutes    0.0.0.0:3000->3000/tcp
postgres_db       postgres:15-alpine     Up 2 minutes    5432/tcp
redis_cache       redis:7-alpine         Up 2 minutes    6379/tcp
```

---

### Restart a Single Service

```bash
docker compose restart api
```

Useful when you change environment variables in `.env`.

---

## How Volumes Work (Data Persistence)

### Without Volumes

```yaml
db:
  image: postgres:15-alpine
  # No volumes defined
```

**What happens:**
1. You start the container and create tables/data
2. You stop the container: `docker compose down`
3. You start it again: `docker compose up`
4. **All data is gone!** You're back to an empty database

---

### With Volumes

```yaml
db:
  image: postgres:15-alpine
  volumes:
    - postgres_data:/var/lib/postgresql/data
```

**What happens:**
1. You start the container and create tables/data
2. PostgreSQL writes data to `/var/lib/postgresql/data` inside the container
3. Docker syncs this to the `postgres_data` volume on your host
4. You stop the container: `docker compose down`
5. You start it again: `docker compose up`
6. **Data is still there!** Docker remounts the volume, and PostgreSQL finds its data files

---

### Managing Volumes

```bash
# List all volumes
docker volume ls

# Inspect a volume
docker volume inspect my-project_postgres_data

# Remove unused volumes
docker volume prune

# Remove a specific volume (DELETES DATA!)
docker volume rm my-project_postgres_data
```

---

## Common Patterns and Tips

### 1. **Exposing Database Ports (For Development)**

If you want to connect to PostgreSQL from a GUI tool like pgAdmin or TablePlus:

```yaml
db:
  image: postgres:15-alpine
  ports:
    - "5432:5432"  # Add this line
  volumes:
    - postgres_data:/var/lib/postgresql/data
```

Now you can connect from your host using:
- Host: `localhost`
- Port: `5432`
- User/Password/Database: From `.env`

**Production:** Remove this. Databases shouldn't be exposed to the host in production.

---

### 2. **Using Different Port on Host**

If port 3000 is already in use:

```yaml
api:
  ports:
    - "8080:3000"  # Host port 8080, container port 3000
```

Access at `http://localhost:8080`.

---

### 3. **Environment-Specific Compose Files**

**Base file:** `docker-compose.yml`
**Development overrides:** `docker-compose.dev.yml`

```yaml
# docker-compose.dev.yml
services:
  api:
    volumes:
      - .:/app  # Mount source code for live reloading
      - /app/node_modules  # Don't overwrite node_modules
    environment:
      - NODE_ENV=development
```

**Run:**
```bash
docker compose -f docker-compose.yml -f docker-compose.dev.yml up
```

---

### 4. **Health Checks**

```yaml
db:
  image: postgres:15-alpine
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
    interval: 10s
    timeout: 5s
    retries: 5
```

**What it does:** Docker periodically checks if PostgreSQL is ready. You can use this with `depends_on`:

```yaml
api:
  depends_on:
    db:
      condition: service_healthy  # Wait until health check passes
```

---

## Troubleshooting

### "Cannot connect to database"

**Symptoms:** Your app logs show `ECONNREFUSED` or `could not connect to server`.

**Common causes:**
1. **Service name typo:** Check `host: 'db'` matches the service name in `docker-compose.yml`.
2. **Wrong network:** Make sure both services are on the same network.
3. **Database not ready:** PostgreSQL takes a few seconds to start. Add retry logic or health checks.

**Quick fix:** Add a delay in your app:
```javascript
setTimeout(async () => {
  await pgClient.connect();
  console.log('Connected to PostgreSQL');
}, 3000);  // Wait 3 seconds
```

---

### "Port already in use"

**Error:** `Bind for 0.0.0.0:3000 failed: port is already allocated`

**Solution:**
```bash
# Find what's using the port
sudo lsof -i :3000

# Stop the conflicting container
docker compose down

# Or change the host port
ports:
  - "8080:3000"
```

---

### "Volume mount failed"

**Error:** Volume paths must be absolute on Windows.

**Solution:** Use named volumes instead of bind mounts:
```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data  # ✅ Named volume
  # - ./data:/var/lib/postgresql/data      # ❌ Bind mount (avoid)
```

---

### Changes not showing up

**After editing code, rebuild:**
```bash
docker compose up -d --build
```

**For live reloading (development):**
```yaml
api:
  volumes:
    - .:/app  # Mount source code
    - /app/node_modules
  command: npx nodemon index.js  # Use nodemon for auto-restart
```

---

## Pop!_OS Specific Notes

### Installing Docker Compose

Modern Docker includes Compose as a plugin. Check if it's installed:

```bash
docker compose version
```

If not installed:
```bash
sudo apt update
sudo apt install docker-compose-plugin
```

### Command Format

**Modern (plugin):** `docker compose up` (space, no hyphen)
**Legacy (standalone):** `docker-compose up` (hyphen)

Both work identically. The modern plugin is recommended.

---

## Complete Working Example

**Project structure:**
```
my-fullstack-app/
├── docker-compose.yml
├── .env
├── .dockerignore
├── Dockerfile
├── package.json
└── index.js
```

**All files ready to copy-paste:**

**`.env`:**
```env
DB_USER=appuser
DB_PASSWORD=secure_password_123
DB_NAME=appdb
NODE_ENV=development
```

**`docker-compose.yml`:**
```yaml
services:
  api:
    build: .
    container_name: api
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgres://${DB_USER}:${DB_PASSWORD}@db:5432/${DB_NAME}
      - REDIS_URL=redis://cache:6379
    depends_on:
      - db
      - cache
    networks:
      - app_network
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    container_name: db
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app_network
    restart: unless-stopped

  cache:
    image: redis:7-alpine
    container_name: cache
    networks:
      - app_network
    restart: unless-stopped

volumes:
  postgres_data:

networks:
  app_network:
```

**Run:**
```bash
docker compose up -d --build
```

**Test:**
```bash
curl http://localhost:3000
```

---

## Key Takeaways

- **Docker Compose manages multi-container apps** with a single YAML file.
- **Service names become hostnames** (your app connects to `db:5432`, not `localhost:5432`).
- **Volumes persist data** across container restarts.
- **`depends_on` controls startup order** but doesn't wait for services to be ready.
- **`.env` files keep secrets out of YAML** files.
- **`docker compose up -d --build`** rebuilds and restarts everything.
- **`docker compose down`** stops containers but keeps volumes (data safe).
- **`docker compose down -v`** deletes volumes (⚠️ data loss).

---

## What's Next?

You now know how to run multi-container applications. To make this production-ready, learn:
- **[[Docker Best Practices]]** — Security, optimization, multi-stage builds
- **[[Docker Networking Deep Dive]]** — Bridge vs host vs overlay networks
- **[[Docker Volumes and Data Management]]** — Bind mounts vs named volumes vs tmpfs

Related: [[Introduction to Docker]] | [[Writing Your First Dockerfile]] | [[Caching and Redis]]
