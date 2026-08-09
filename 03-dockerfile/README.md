# 03 — Writing Your First Dockerfile

## What You'll Learn
- What a Dockerfile is and how it works
- Every essential Dockerfile instruction (`FROM`, `WORKDIR`, `COPY`, `RUN`, `EXPOSE`, `CMD`)
- How to build a Docker image from a Dockerfile
- Hands-on: Dockerize a simple Express.js API
- What `.dockerignore` does and why you need it

---

## What is a Dockerfile?

A **Dockerfile** is a text file with a set of instructions that tells Docker how to build an image. Think of it as a **recipe** — step by step, it describes what goes into the image.

```
Dockerfile          →    docker build    →    Docker Image    →    Container
(Recipe/Blueprint)       (Build Process)      (Ready to ship)     (Running app)
```

There's no file extension — the file is literally named `Dockerfile` (capital D, no extension).

---

## Dockerfile Instructions — Line by Line

Here's a Dockerfile for a simple Express.js API. We'll go through each line:

```dockerfile
# 1. Start from a base image
FROM node:18-alpine

# 2. Set the working directory inside the container
WORKDIR /app

# 3. Copy package files first (for caching)
COPY package.json package-lock.json ./

# 4. Install dependencies
RUN npm install

# 5. Copy the rest of the application code
COPY . .

# 6. Document which port the app uses
EXPOSE 3000

# 7. Define the command to run the app
CMD ["node", "app.js"]
```

Now let's understand each instruction:

### `FROM` — The Base Image

```dockerfile
FROM node:18-alpine
```

Every Dockerfile **must start** with `FROM`. It sets the base image — the foundation your image is built on.

- `node:18-alpine` = Alpine Linux + Node.js 18 pre-installed
- Alpine is a tiny Linux distro (~5MB), making your image small and fast
- You're essentially saying: "Start with a machine that already has Node.js 18"

> 🎯 **Interview point**: "I use `alpine` variants because they're significantly smaller — a `node:18` image is ~350MB, while `node:18-alpine` is ~178MB."

### `WORKDIR` — Set the Working Directory

```dockerfile
WORKDIR /app
```

Sets the working directory **inside the container** for all subsequent instructions. If the directory doesn't exist, Docker creates it.

Think of it as doing `mkdir /app && cd /app` inside the container.

All future `COPY`, `RUN`, and `CMD` commands will execute relative to this directory.

### `COPY` — Copy Files into the Image

```dockerfile
COPY package.json package-lock.json ./
COPY . .
```

Copies files from your **local machine** (build context) into the **image**.

Format: `COPY <source on host> <destination in image>`

- `COPY package.json package-lock.json ./` → Copy just the package files to `/app/`
- `COPY . .` → Copy everything from current directory to `/app/`

> 💡 **Why do we COPY package files separately?** For caching! See the "Layer Caching" section below.

### `RUN` — Execute Commands During Build

```dockerfile
RUN npm install
```

Runs a command **during the image build process**. The result becomes a new layer in the image.

Common uses:
- `RUN npm install` → Install Node.js dependencies
- `RUN apt-get update && apt-get install -y curl` → Install system packages
- `RUN mkdir -p /app/logs` → Create directories

> ⚠️ `RUN` executes at **build time** (when you do `docker build`), NOT at runtime (when you do `docker run`).

### `EXPOSE` — Document the Port

```dockerfile
EXPOSE 3000
```

This is **documentation only** — it tells anyone reading the Dockerfile that the app listens on port 3000. It does NOT actually publish the port.

You still need `-p 3000:3000` when running the container. `EXPOSE` is just a hint.

> 🎯 **Interview gotcha**: "`EXPOSE` doesn't publish the port. It's metadata. You still need `-p` in `docker run` to actually make the port accessible."

### `CMD` — The Startup Command

```dockerfile
CMD ["node", "app.js"]
```

Defines the **default command** that runs when a container starts. There can be only ONE `CMD` in a Dockerfile (if you have multiple, only the last one takes effect).

Two formats:
- **Exec form (preferred)**: `CMD ["node", "app.js"]` — runs directly
- **Shell form**: `CMD node app.js` — runs via `/bin/sh -c`

> 💡 Use exec form. It handles signals properly (important for graceful shutdown).

### `CMD` vs `RUN` — The Common Confusion

| Aspect  | `RUN`                              | `CMD`                              |
|---------|------------------------------------|------------------------------------|
| When    | During **image build**             | When **container starts**          |
| Purpose | Install deps, create dirs, config  | Start the application              |
| Count   | Can have many                      | Only ONE (last one wins)           |
| Example | `RUN npm install`                  | `CMD ["node", "app.js"]`          |

---

## Hands-On: Dockerize an Express.js API

Let's build a real example. Create these files in the `03-dockerfile` folder:

### File 1: `app.js`

```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.json({
    message: 'Hello from Dockerized Express! 🐳',
    environment: process.env.NODE_ENV || 'development',
    timestamp: new Date().toISOString()
  });
});

app.get('/health', (req, res) => {
  res.json({ status: 'healthy', uptime: process.uptime() });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### File 2: `package.json`

```json
{
  "name": "docker-express-demo",
  "version": "1.0.0",
  "description": "A simple Express app to learn Docker",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

### File 3: `Dockerfile`

```dockerfile
# Use Node.js 18 on Alpine Linux as base
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files first (layer caching optimization)
COPY package.json package-lock.json* ./

# Install dependencies
RUN npm install

# Copy application code
COPY . .

# Document the port
EXPOSE 3000

# Start the application
CMD ["node", "app.js"]
```

### File 4: `.dockerignore`

```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
```

---

## Building the Image

```bash
# Navigate to the 03-dockerfile directory
cd /path/to/03-dockerfile

# Build the image
docker build -t my-express-app .
```

Let's break down this command:
- `docker build` → Build an image from a Dockerfile
- `-t my-express-app` → Tag (name) the image as "my-express-app"
- `.` → The build context (current directory). Docker sends this directory to the daemon

**You'll see output like:**
```
[+] Building 15.2s (10/10) FINISHED
 => [1/5] FROM node:18-alpine
 => [2/5] WORKDIR /app
 => [3/5] COPY package.json package-lock.json* ./
 => [4/5] RUN npm install
 => [5/5] COPY . .
 => exporting to image
 => naming to docker.io/library/my-express-app
```

### Running Your Image

```bash
# Run the container
docker run -d --name my-api -p 3000:3000 my-express-app

# Test it!
curl http://localhost:3000
# Output: {"message":"Hello from Dockerized Express! 🐳", ...}

curl http://localhost:3000/health
# Output: {"status":"healthy","uptime":12.345}

# View logs
docker logs my-api

# Clean up
docker stop my-api
docker rm my-api
```

---

## Layer Caching — Why Order Matters

Remember the Dockerfile instructions create **layers**. Docker **caches** layers and only rebuilds from the point where something changed.

### Bad Order (No caching benefit):

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .              # ← Code changes invalidate this layer
RUN npm install       # ← This re-runs EVERY time code changes!
CMD ["node", "app.js"]
```

Every time you change even one line of `app.js`, Docker has to re-run `npm install` (which can take minutes).

### Good Order (Optimized caching):

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json package-lock.json* ./   # ← Package files change rarely
RUN npm install                            # ← Cached unless packages change!
COPY . .                                   # ← Only this layer rebuilds
CMD ["node", "app.js"]
```

Now when you change `app.js`, Docker reuses the cached `npm install` layer. Builds go from minutes to seconds.

```
Change app.js → only Layer 5 rebuilds (COPY . .)
Change package.json → Layers 3-5 rebuild (npm install runs again)

┌───────────────────────────────────┐
│ Layer 5: COPY . .                 │ ← Rebuilds on code change
├───────────────────────────────────┤
│ Layer 4: RUN npm install          │ ← CACHED (packages didn't change)
├───────────────────────────────────┤
│ Layer 3: COPY package*.json ./    │ ← CACHED
├───────────────────────────────────┤
│ Layer 2: WORKDIR /app             │ ← CACHED
├───────────────────────────────────┤
│ Layer 1: FROM node:18-alpine      │ ← CACHED
└───────────────────────────────────┘
```

> 🎯 **Interview answer**: "I copy `package.json` before `COPY . .` so that Docker can cache the `npm install` layer. Dependencies only get reinstalled when `package.json` changes, not when application code changes."

---

## `.dockerignore` — What to Exclude

Just like `.gitignore` prevents files from being tracked by Git, `.dockerignore` prevents files from being copied into the build context.

### Why does it matter?
When you run `docker build .`, Docker sends the **entire current directory** to the Docker daemon. If you have a huge `node_modules` folder (500MB+), it gets sent too — even though you're going to `npm install` fresh inside the container.

### Common `.dockerignore` for Node.js:

```
node_modules
npm-debug.log*
.git
.gitignore
.env
README.md
docker-compose*.yml
.dockerignore
Dockerfile
```

> 🎯 **Always exclude `node_modules`**. You install fresh dependencies inside the container with `RUN npm install`. The host's `node_modules` might have platform-specific binaries (e.g., macOS binaries won't work in a Linux container).

---

## Bonus: Other Useful Dockerfile Instructions

### `ENV` — Set Environment Variables

```dockerfile
ENV NODE_ENV=production
ENV PORT=3000
```

These are available both during build (`RUN`) and at runtime (`CMD`).

### `ARG` — Build-time Variables

```dockerfile
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}-alpine
```

Only available during build, NOT at runtime. Useful for parameterizing builds.

### `ENTRYPOINT` vs `CMD`

| Instruction   | Can be overridden at runtime? | Use case                     |
|---------------|-------------------------------|------------------------------|
| `CMD`         | Yes, easily                   | Default command              |
| `ENTRYPOINT`  | Harder to override            | Fixed executable             |

```dockerfile
# CMD: can be overridden
CMD ["node", "app.js"]
# docker run my-app node other-script.js  ← overrides CMD

# ENTRYPOINT: always runs
ENTRYPOINT ["node"]
CMD ["app.js"]
# docker run my-app other-script.js  ← runs: node other-script.js
```

For beginners, stick with `CMD`. You'll rarely need `ENTRYPOINT`.

---

## 🎤 Interview Q&A

### Q1: What is a Dockerfile?
**A**: A Dockerfile is a text file containing a series of instructions that Docker uses to build an image. Each instruction creates a layer in the image. It starts with a `FROM` instruction specifying the base image, followed by commands to copy code, install dependencies, and define the startup command.

### Q2: What is the difference between `CMD` and `RUN`?
**A**: `RUN` executes commands during the **image build process** and the result becomes part of the image layer (e.g., `RUN npm install`). `CMD` specifies the **default command to execute when a container starts** (e.g., `CMD ["node", "app.js"]`). You can have multiple `RUN` instructions, but only one `CMD` — the last one wins.

### Q3: What is `.dockerignore` and why is it important?
**A**: `.dockerignore` is a file that specifies which files and directories should be excluded from the Docker build context. It's important because it prevents unnecessary files like `node_modules`, `.git`, and `.env` from being sent to the Docker daemon, which speeds up builds and reduces image size. It also prevents accidentally baking secrets (`.env` files) into the image.

### Q4: Why do we copy `package.json` before copying the rest of the code?
**A**: For **layer caching optimization**. Docker caches each layer and only rebuilds from the point where a change is detected. By copying `package.json` first and running `npm install`, Docker can cache the dependency installation layer. When only application code changes, Docker reuses the cached dependencies layer, making builds much faster.

### Q5: What is the difference between `EXPOSE` and `-p` in `docker run`?
**A**: `EXPOSE` is a **documentation instruction** in the Dockerfile — it declares which port the application listens on, but doesn't actually publish it. The `-p` flag in `docker run` **actually maps** the host port to the container port, making the service accessible. You need `-p` to access the service; `EXPOSE` alone does nothing at runtime.

---

## What's Next?
Your containers are running, but they can't talk to each other yet. Let's learn how Docker networking works.

→ [Next: 04 — Docker Networking Basics](../04-networking/README.md)
