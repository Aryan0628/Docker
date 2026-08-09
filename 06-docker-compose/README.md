# 06 — Docker Compose — The YML File

## What You'll Learn
- What Docker Compose is and why you need it
- How to write a `docker-compose.yml` file from scratch
- Every important field explained line by line
- Essential `docker compose` commands
- Hands-on: Full MERN stack in a single compose file

---

## The Problem: Too Many `docker run` Commands

By Chapter 05, running a MERN stack requires all of this:

```bash
# 1. Create a network
docker network create mern-network

# 2. Create volumes
docker volume create mongo-data

# 3. Run MongoDB
docker run -d \
  --name mongo \
  --network mern-network \
  -v mongo-data:/data/db \
  -p 27017:27017 \
  mongo:7

# 4. Build Express image
docker build -t my-express-api ./backend

# 5. Run Express
docker run -d \
  --name express-api \
  --network mern-network \
  -p 5000:5000 \
  -e MONGO_URL=mongodb://mongo:27017/myapp \
  my-express-api

# 6. Build React image
docker build -t my-react-app ./frontend

# 7. Run React
docker run -d \
  --name react-app \
  --network mern-network \
  -p 3000:3000 \
  my-react-app
```

That's **7 commands** just to start the app. And you have to remember the exact flags, network names, volume names, and environment variables every time.

**Docker Compose** lets you define all of this in a **single YAML file** and start everything with **one command**: `docker compose up`.

---

## What is Docker Compose?

Docker Compose is a tool for defining and running **multi-container applications**. You describe your entire application stack in a `docker-compose.yml` file, and Docker Compose handles:

- Building images
- Creating networks
- Creating volumes
- Starting containers in the right order
- Connecting everything together

```
7 separate docker commands    →    1 YAML file + 1 command
                                   docker compose up
```

---

## Anatomy of `docker-compose.yml`

Here's the complete structure with every field you need to know:

```yaml
# Optional: specify the compose file format version
# (modern Docker Compose doesn't require this)

services:
  # Each service = one container
  service-name:
    image: image-name:tag           # Use a pre-built image
    # OR
    build: ./path-to-dockerfile     # Build from a Dockerfile
    
    container_name: my-container    # Custom container name
    ports:
      - "HOST:CONTAINER"           # Port mapping
    volumes:
      - volume-name:/path          # Named volume
      - ./local/path:/path         # Bind mount
    environment:
      - KEY=value                  # Environment variables
    env_file:
      - .env                      # Load vars from file
    depends_on:
      - other-service              # Start order dependency
    networks:
      - network-name               # Connect to network
    restart: unless-stopped         # Restart policy
    command: npm start              # Override CMD

volumes:
  volume-name:                     # Declare named volumes

networks:
  network-name:                    # Declare custom networks
```

Now let's break down each field.

---

## Field-by-Field Explanation

### `services`

The top-level key. Each child under `services` defines a **container** (called a "service" in Compose terminology).

```yaml
services:
  backend:      # ← This becomes a container named "backend"
    ...
  frontend:     # ← This becomes a container named "frontend"
    ...
  database:     # ← This becomes a container named "database"
    ...
```

### `image` vs `build`

**Use `image`** when pulling from Docker Hub (no Dockerfile needed):
```yaml
services:
  mongo:
    image: mongo:7    # Pull from Docker Hub
```

**Use `build`** when you have a Dockerfile:
```yaml
services:
  backend:
    build: ./backend    # Build from ./backend/Dockerfile
```

**Advanced `build`** options:
```yaml
services:
  backend:
    build:
      context: ./backend        # Build context directory
      dockerfile: Dockerfile    # Name of the Dockerfile (if not default)
```

### `ports`

Maps host ports to container ports. Same as `-p` in `docker run`.

```yaml
ports:
  - "3000:3000"     # host:container
  - "8080:80"       # access container's port 80 via localhost:8080
```

> ⚠️ Always quote port mappings (`"3000:3000"` not `3000:3000`). YAML can misinterpret unquoted numbers with colons.

### `volumes`

Mount persistent storage or local directories.

```yaml
services:
  mongo:
    volumes:
      - mongo-data:/data/db          # Named volume (for databases)
  
  backend:
    volumes:
      - ./backend/src:/app/src       # Bind mount (for development)

# Declare named volumes at the bottom
volumes:
  mongo-data:
```

> 💡 Named volumes must be declared in the top-level `volumes:` section at the bottom of the file.

### `environment`

Set environment variables. Two syntax options:

```yaml
# List syntax (recommended)
environment:
  - NODE_ENV=production
  - MONGO_URL=mongodb://mongo:27017/myapp
  - PORT=5000

# Map syntax (also works)
environment:
  NODE_ENV: production
  MONGO_URL: mongodb://mongo:27017/myapp
  PORT: 5000
```

### `env_file`

Load environment variables from a file:

```yaml
env_file:
  - .env
```

Where `.env` contains:
```
NODE_ENV=production
MONGO_URL=mongodb://mongo:27017/myapp
SECRET_KEY=mysupersecretkey
```

> 🎯 **Best practice**: Use `env_file` for secrets. Don't put passwords directly in `docker-compose.yml` (it gets committed to Git).

### `depends_on`

Controls **startup order**. If your Express API needs MongoDB to start first:

```yaml
services:
  backend:
    depends_on:
      - mongo       # Start mongo before backend
  mongo:
    image: mongo:7
```

> ⚠️ `depends_on` only controls **start order**, not whether the dependency is "ready". MongoDB might start but not be accepting connections yet. For true readiness, you'd use health checks (out of scope for now).

### `networks`

Connect services to networks. If you don't specify any, Compose creates a **default network** for all services automatically.

```yaml
services:
  backend:
    networks:
      - app-network
  mongo:
    networks:
      - app-network

networks:
  app-network:
```

> 💡 In most cases, you don't need to define networks manually. Compose creates a default network and all services can reach each other by service name.

### `restart`

Restart policy if the container crashes or the system reboots.

```yaml
restart: "no"              # Never restart (default)
restart: always            # Always restart
restart: on-failure        # Only restart on non-zero exit code
restart: unless-stopped    # Restart unless manually stopped
```

### `container_name`

Override the auto-generated container name:

```yaml
services:
  mongo:
    image: mongo:7
    container_name: my-mongodb    # Instead of "projectname-mongo-1"
```

### `command`

Override the `CMD` from the Dockerfile:

```yaml
services:
  backend:
    build: ./backend
    command: npm run dev    # Overrides CMD in Dockerfile
```

---

## Hands-On: Full MERN Stack Compose File

Let's build the big one — a complete MERN stack running with Docker Compose.

### Project Structure

```
mern-docker/
├── docker-compose.yml
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   ├── app.js
│   └── .dockerignore
└── frontend/
    ├── Dockerfile
    ├── package.json
    ├── src/
    │   └── index.html
    └── .dockerignore
```

### The `docker-compose.yml` — Line by Line

```yaml
# docker-compose.yml
# Full MERN stack: React frontend + Express backend + MongoDB

services:

  # ============================================
  # MongoDB Database
  # ============================================
  mongo:
    image: mongo:7                          # Use official MongoDB 7 image
    container_name: mern-mongo              # Name the container
    ports:
      - "27017:27017"                       # Expose for MongoDB Compass access
    volumes:
      - mongo-data:/data/db                 # Persist data in a named volume
    environment:
      - MONGO_INITDB_DATABASE=mernapp       # Create default database
    restart: unless-stopped                 # Auto-restart on crash

  # ============================================
  # Express.js Backend API
  # ============================================
  backend:
    build: ./backend                        # Build from ./backend/Dockerfile
    container_name: mern-backend            # Name the container
    ports:
      - "5000:5000"                         # API accessible on localhost:5000
    environment:
      - NODE_ENV=development
      - PORT=5000
      - MONGO_URL=mongodb://mongo:27017/mernapp   # 'mongo' = service name (DNS)
    depends_on:
      - mongo                               # Start MongoDB first
    restart: unless-stopped
    volumes:
      - ./backend:/app                      # Bind mount for live reload (dev)
      - /app/node_modules                   # Don't overwrite container's node_modules

  # ============================================
  # React Frontend
  # ============================================
  frontend:
    build: ./frontend                       # Build from ./frontend/Dockerfile
    container_name: mern-frontend           # Name the container
    ports:
      - "3000:3000"                         # React dev server on localhost:3000
    depends_on:
      - backend                             # Start backend first
    restart: unless-stopped

# ============================================
# Named Volumes (for data persistence)
# ============================================
volumes:
  mongo-data:                               # Docker manages this volume
```

Let's call out the most important line:

```yaml
MONGO_URL=mongodb://mongo:27017/mernapp
#                    ^^^^^
# 'mongo' here is the SERVICE NAME from this compose file.
# Docker Compose creates a network and DNS resolves 'mongo' to the MongoDB container's IP.
# You do NOT use 'localhost' — that would refer to the backend container itself!
```

### The `anonymous volume` trick:

```yaml
volumes:
  - ./backend:/app              # Mount local code into container
  - /app/node_modules           # But DON'T overwrite node_modules!
```

Without that second line, the bind mount would overwrite the container's `/app/node_modules` with your Mac's `node_modules` (which might have macOS-specific binaries). The anonymous volume preserves the container's own `node_modules`.

---

### Backend Files

**`backend/Dockerfile`**:
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm install
COPY . .
EXPOSE 5000
CMD ["node", "app.js"]
```

**`backend/package.json`**:
```json
{
  "name": "mern-backend",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "start": "node app.js",
    "dev": "node --watch app.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.6.3",
    "cors": "^2.8.5"
  }
}
```

**`backend/app.js`**:
```javascript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');

const app = express();
const PORT = process.env.PORT || 5000;
const MONGO_URL = process.env.MONGO_URL || 'mongodb://localhost:27017/mernapp';

// Middleware
app.use(cors());
app.use(express.json());

// Connect to MongoDB
mongoose.connect(MONGO_URL)
  .then(() => console.log('✅ Connected to MongoDB'))
  .catch(err => console.error('❌ MongoDB connection error:', err));

// Simple schema
const ItemSchema = new mongoose.Schema({
  name: { type: String, required: true },
  createdAt: { type: Date, default: Date.now }
});
const Item = mongoose.model('Item', ItemSchema);

// Routes
app.get('/api/items', async (req, res) => {
  const items = await Item.find().sort({ createdAt: -1 });
  res.json(items);
});

app.post('/api/items', async (req, res) => {
  const item = new Item({ name: req.body.name });
  await item.save();
  res.status(201).json(item);
});

app.get('/api/health', (req, res) => {
  res.json({
    status: 'healthy',
    mongo: mongoose.connection.readyState === 1 ? 'connected' : 'disconnected'
  });
});

app.listen(PORT, () => {
  console.log(`🚀 Backend running on port ${PORT}`);
});
```

**`backend/.dockerignore`**:
```
node_modules
npm-debug.log*
.git
.env
```

---

### Frontend Files

**`frontend/Dockerfile`**:
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npx", "serve", "-s", "src", "-l", "3000"]
```

**`frontend/package.json`**:
```json
{
  "name": "mern-frontend",
  "version": "1.0.0",
  "dependencies": {
    "serve": "^14.2.0"
  }
}
```

**`frontend/src/index.html`**:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>MERN Docker Demo</title>
  <style>
    body { font-family: 'Segoe UI', sans-serif; max-width: 600px; margin: 50px auto; padding: 20px; background: #1a1a2e; color: #eee; }
    h1 { color: #0f9; }
    button { background: #0f9; color: #1a1a2e; border: none; padding: 10px 20px; cursor: pointer; border-radius: 5px; font-weight: bold; }
    input { padding: 10px; border-radius: 5px; border: 1px solid #333; background: #16213e; color: #eee; width: 200px; }
    .item { background: #16213e; padding: 10px; margin: 5px 0; border-radius: 5px; }
    #status { color: #0f9; }
  </style>
</head>
<body>
  <h1>🐳 MERN Docker Demo</h1>
  <p>Status: <span id="status">Checking...</span></p>

  <div>
    <input type="text" id="itemName" placeholder="Enter item name">
    <button onclick="addItem()">Add Item</button>
  </div>
  <div id="items"></div>

  <script>
    const API = 'http://localhost:5000/api';

    async function checkHealth() {
      try {
        const res = await fetch(`${API}/health`);
        const data = await res.json();
        document.getElementById('status').textContent =
          `Backend: ${data.status} | MongoDB: ${data.mongo}`;
      } catch {
        document.getElementById('status').textContent = 'Backend unreachable';
      }
    }

    async function loadItems() {
      try {
        const res = await fetch(`${API}/items`);
        const items = await res.json();
        document.getElementById('items').innerHTML = items
          .map(i => `<div class="item">${i.name}</div>`).join('');
      } catch { /* ignore */ }
    }

    async function addItem() {
      const name = document.getElementById('itemName').value;
      if (!name) return;
      await fetch(`${API}/items`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name })
      });
      document.getElementById('itemName').value = '';
      loadItems();
    }

    checkHealth();
    loadItems();
  </script>
</body>
</html>
```

**`frontend/.dockerignore`**:
```
node_modules
npm-debug.log*
.git
```

---

## Docker Compose Commands

### Starting Your App

```bash
# Start all services (foreground — see all logs)
docker compose up

# Start all services (background)
docker compose up -d

# Build images AND start (use when you change Dockerfiles)
docker compose up --build

# Start only specific services
docker compose up -d mongo backend
```

### Stopping Your App

```bash
# Stop and remove all containers, networks
docker compose down

# Stop and also remove volumes (⚠️ deletes data!)
docker compose down -v

# Just stop (don't remove)
docker compose stop
```

### Viewing Status & Logs

```bash
# See running services
docker compose ps

# View logs for all services
docker compose logs

# Follow logs for a specific service
docker compose logs -f backend

# View last 50 lines of logs
docker compose logs --tail 50 backend
```

### Other Useful Commands

```bash
# Rebuild images without cache
docker compose build --no-cache

# Run a one-off command in a service
docker compose exec backend sh

# Scale a service (run multiple instances)
docker compose up -d --scale backend=3
```

---

## Command Cheat Sheet

| Command                           | What it does                              |
|-----------------------------------|-------------------------------------------|
| `docker compose up`               | Start all services (foreground)            |
| `docker compose up -d`            | Start all services (background)            |
| `docker compose up --build`       | Rebuild images and start                   |
| `docker compose down`             | Stop and remove everything                 |
| `docker compose down -v`          | Stop, remove, AND delete volumes           |
| `docker compose ps`               | List running services                      |
| `docker compose logs -f service`  | Follow logs for a service                  |
| `docker compose exec service sh`  | Shell into a running service               |
| `docker compose build`            | Build/rebuild images                       |
| `docker compose stop`             | Stop services (don't remove)               |
| `docker compose restart`          | Restart services                           |

---

## 🎤 Interview Q&A

### Q1: What is Docker Compose and why do you use it?
**A**: Docker Compose is a tool for defining and running multi-container applications using a YAML file. Instead of running multiple `docker run` commands with complex flags, you define all your services, networks, and volumes in a `docker-compose.yml` file and manage everything with a single `docker compose up` command. It's especially useful for development environments where you need multiple services like a web server, API, and database running together.

### Q2: What is the difference between `docker compose up` and `docker compose up --build`?
**A**: `docker compose up` starts services using existing images. If the image doesn't exist, it builds it. But if you've changed your Dockerfile or source code, it won't rebuild automatically. `docker compose up --build` forces a rebuild of all images before starting the containers, ensuring your latest code changes are included.

### Q3: What does `depends_on` do in Docker Compose?
**A**: `depends_on` controls the **startup order** of services. If service A depends on service B, Docker Compose will start B before A. However, it only waits for the container to start, not for the application inside to be ready. For example, MongoDB's container might start but not be accepting connections yet when Express tries to connect.

### Q4: How do services communicate in Docker Compose?
**A**: Docker Compose automatically creates a network for all services defined in the compose file. Services can reach each other using the **service name** as the hostname. For example, if you have a service called `mongo`, your Express app can connect to it using `mongodb://mongo:27017` — Docker's built-in DNS resolves the service name to the container's IP.

### Q5: What is the difference between `docker compose down` and `docker compose down -v`?
**A**: `docker compose down` stops and removes all containers and networks created by Compose, but **preserves named volumes** (so your data is safe). `docker compose down -v` also removes the named volumes, which means persistent data like database contents will be deleted. Use `-v` only when you want a clean slate.

---

## What's Next?
You now have all the Docker knowledge you need for an SDE intern interview. Let's put it all together with rapid-fire interview questions and a ready-to-use cheat sheet.

→ [Next: 07 — Interview Rapid Fire & Resume Talking Points](../07-interview-cheatsheet/README.md)
