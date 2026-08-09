# 02 — Docker Images & Essential Commands

## What You'll Learn
- How to pull, list, and remove images
- How to run containers with important flags
- How to manage container lifecycle (start, stop, remove)
- How to inspect logs and exec into running containers
- Hands-on: Run Nginx and Node.js containers

---

## Part 1: Working with Images

### Pulling an Image

When you run `docker pull`, you're downloading an image from Docker Hub to your local machine.

```bash
# Pull the official Nginx web server image
docker pull nginx

# Pull a specific version of Node.js (Alpine = lightweight Linux)
docker pull node:18-alpine

# Pull MongoDB
docker pull mongo:7
```

> 💡 You don't always need to `pull` first. If you do `docker run nginx` and the image isn't on your machine, Docker will automatically pull it for you.

### Listing Images

```bash
# List all images on your machine
docker images
```

**Output:**
```
REPOSITORY   TAG          IMAGE ID       CREATED        SIZE
nginx        latest       a8758716bb6a   2 days ago     187MB
node         18-alpine    7e0503a39d78   5 days ago     178MB
mongo        7            1234abcd5678   1 week ago     750MB
```

Key columns:
- **REPOSITORY**: Name of the image
- **TAG**: Version/variant (e.g., `18-alpine`, `latest`)
- **IMAGE ID**: Unique hash identifier (first 12 chars shown)
- **SIZE**: How much disk space it takes

### Removing Images

```bash
# Remove an image by name:tag
docker rmi nginx

# Remove an image by ID
docker rmi a8758716bb6a

# Force remove (if a container is using it)
docker rmi -f nginx

# Remove ALL unused images (careful with this!)
docker image prune -a
```

> ⚠️ You can't remove an image if a container (even a stopped one) is using it. Either remove the container first, or use `-f` to force.

---

## Part 2: Running Containers — `docker run` Deep Dive

`docker run` is the most important Docker command. It creates and starts a container from an image.

### Basic Syntax

```bash
docker run [OPTIONS] IMAGE [COMMAND]
```

### The Flags You MUST Know

Let's break down each flag with a real example:

#### `-d` (Detached Mode)

Runs the container **in the background** so you get your terminal back.

```bash
# Without -d: terminal is "stuck" showing container logs
docker run nginx

# With -d: runs in background, returns container ID
docker run -d nginx
```

#### `-p` (Port Mapping)

Maps a port on **your machine** to a port **inside the container**.

```bash
# Map your machine's port 8080 to container's port 80
docker run -d -p 8080:80 nginx
```

Now open `http://localhost:8080` in your browser — you'll see the Nginx welcome page!

```
Format: -p HOST_PORT:CONTAINER_PORT

Your Machine (Host)          Container
┌──────────────────┐        ┌──────────────────┐
│                  │        │                  │
│   localhost:8080 ├───────►│  nginx:80        │
│                  │        │                  │
└──────────────────┘        └──────────────────┘
```

> 🎯 **Common interview question**: "What does `-p 3000:5000` mean?"  
> **Answer**: It maps port 3000 on the host machine to port 5000 inside the container. So accessing `localhost:3000` will reach the app running on port 5000 inside the container.

#### `--name` (Name Your Container)

Gives your container a human-readable name instead of a random one like `cranky_einstein`.

```bash
docker run -d --name my-nginx -p 8080:80 nginx
```

#### `-it` (Interactive + TTY)

Opens an **interactive terminal session** inside the container. Used for debugging or exploring.

```bash
# Open a bash shell inside an Ubuntu container
docker run -it ubuntu bash

# You're now INSIDE the container!
# Try: ls, cat /etc/os-release, exit
```

- `-i` = Keep STDIN open (interactive)
- `-t` = Allocate a terminal (so you get a proper prompt)
- They're almost always used together as `-it`

#### `--rm` (Auto-Remove)

Automatically removes the container when it stops. Useful for quick tests.

```bash
# This container will be deleted as soon as it stops
docker run --rm -d --name temp-nginx -p 8080:80 nginx
```

#### `-e` (Environment Variables)

Pass environment variables into the container.

```bash
# Start MongoDB with a root username and password
docker run -d --name my-mongo \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=secret123 \
  -p 27017:27017 \
  mongo:7
```

### All Flags Together — A Real Example

```bash
docker run -d \
  --name my-express-api \
  -p 3000:3000 \
  -e NODE_ENV=production \
  -e DB_HOST=mongodb://localhost:27017 \
  --rm \
  node:18-alpine
```

Translation:
- `-d` → Run in background
- `--name my-express-api` → Call it "my-express-api"
- `-p 3000:3000` → Map port 3000
- `-e NODE_ENV=production` → Set environment variable
- `--rm` → Delete container when it stops
- `node:18-alpine` → Use this image

---

## Part 3: Managing Containers

### List Containers

```bash
# Show RUNNING containers
docker ps

# Show ALL containers (including stopped)
docker ps -a
```

**Output:**
```
CONTAINER ID   IMAGE   COMMAND                  STATUS          PORTS                  NAMES
abc123def456   nginx   "/docker-entrypoint.…"   Up 5 minutes    0.0.0.0:8080->80/tcp   my-nginx
```

### Stop a Container

```bash
# Stop by name
docker stop my-nginx

# Stop by container ID
docker stop abc123def456

# Stop ALL running containers
docker stop $(docker ps -q)
```

> `docker stop` sends a SIGTERM signal (graceful shutdown), waits 10 seconds, then sends SIGKILL.

### Start a Stopped Container

```bash
# Restart a stopped container (doesn't create a new one)
docker start my-nginx
```

### Remove a Container

```bash
# Remove a stopped container
docker rm my-nginx

# Force remove a running container
docker rm -f my-nginx

# Remove ALL stopped containers
docker container prune
```

### The Lifecycle of a Container

```
docker run          docker stop         docker rm
    │                   │                   │
    ▼                   ▼                   ▼
 CREATED ──► RUNNING ──► STOPPED ──► REMOVED
                ▲            │
                │            │
                └────────────┘
               docker start
```

---

## Part 4: Inspecting & Debugging Containers

### View Logs

```bash
# See all logs from a container
docker logs my-nginx

# Follow logs in real-time (like tail -f)
docker logs -f my-nginx

# Show last 50 lines
docker logs --tail 50 my-nginx

# Show logs with timestamps
docker logs -t my-nginx
```

### Execute Commands Inside a Running Container

```bash
# Open a shell inside a running container
docker exec -it my-nginx bash

# Run a single command without entering the container
docker exec my-nginx cat /etc/nginx/nginx.conf

# Open a shell in an Alpine-based container (use sh, not bash)
docker exec -it my-node sh
```

> 💡 `docker exec` is your go-to debugging tool. Can't figure out why your app isn't working? Exec into the container and poke around.

### Inspect Container Details

```bash
# Get EVERYTHING about a container (JSON output)
docker inspect my-nginx

# Get just the IP address
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' my-nginx
```

---

## Part 5: Hands-On Practice

### Exercise 1: Run Nginx

```bash
# 1. Run Nginx on port 8080
docker run -d --name web-server -p 8080:80 nginx

# 2. Visit http://localhost:8080 in your browser
#    → You should see "Welcome to nginx!"

# 3. Check it's running
docker ps

# 4. View the access logs
docker logs web-server

# 5. Look inside the container
docker exec -it web-server bash
ls /usr/share/nginx/html/    # This is where the HTML lives
cat /usr/share/nginx/html/index.html
exit

# 6. Clean up
docker stop web-server
docker rm web-server
```

### Exercise 2: Run a Node.js Container

```bash
# 1. Run Node.js interactively
docker run -it --rm node:18-alpine node

# You're now in the Node.js REPL (inside the container!)
# Try:
> console.log("Hello from Docker!")
> process.version    // Should show v18.x.x
> process.platform   // Should show 'linux' (even on your Mac!)
> .exit

# 2. Run a one-liner
docker run --rm node:18-alpine node -e "console.log('Node.js running in Docker!')"
```

> 🤯 Notice `process.platform` shows `linux` even though you're on macOS! That's because the container runs a Linux environment. This is the power of Docker.

### Exercise 3: Run MongoDB

```bash
# 1. Run MongoDB
docker run -d --name my-mongo -p 27017:27017 mongo:7

# 2. Connect to MongoDB shell inside the container
docker exec -it my-mongo mongosh

# 3. Try some commands
> show dbs
> use testdb
> db.users.insertOne({ name: "Aryan", role: "SDE Intern" })
> db.users.find()
> exit

# 4. Clean up
docker stop my-mongo
docker rm my-mongo
```

---

## Command Cheat Sheet

| Command                          | What it does                        |
|----------------------------------|-------------------------------------|
| `docker pull image:tag`          | Download an image                   |
| `docker images`                  | List local images                   |
| `docker rmi image`               | Remove an image                     |
| `docker run -d --name n -p H:C image` | Run a container                |
| `docker ps`                      | List running containers             |
| `docker ps -a`                   | List ALL containers                 |
| `docker stop name`               | Stop a container                    |
| `docker start name`              | Start a stopped container           |
| `docker rm name`                 | Remove a container                  |
| `docker logs name`               | View container logs                 |
| `docker exec -it name sh`       | Open shell in container             |
| `docker inspect name`            | Get container details               |
| `docker container prune`         | Remove all stopped containers       |
| `docker image prune -a`          | Remove all unused images            |

---

## 🎤 Interview Q&A

### Q1: What is the difference between `docker run` and `docker start`?
**A**: `docker run` creates a **new container** from an image and starts it. `docker start` restarts an **existing stopped container** without creating a new one. So `docker run` = `docker create` + `docker start`.

### Q2: What does the `-p 3000:5000` flag do?
**A**: It maps port 3000 on the host machine to port 5000 inside the container. Any traffic hitting `localhost:3000` will be forwarded to port 5000 inside the container where the application is listening.

### Q3: What is the difference between `docker stop` and `docker rm`?
**A**: `docker stop` gracefully stops a running container (sends SIGTERM, then SIGKILL after 10 seconds) but keeps the container on disk in a "stopped" state. `docker rm` removes a stopped container entirely from disk. A running container must be stopped before it can be removed (unless you use `docker rm -f`).

### Q4: How do you run a command inside a running container?
**A**: Using `docker exec`. For example, `docker exec -it my-container bash` opens an interactive bash shell inside the container. The `-i` flag keeps STDIN open and `-t` allocates a pseudo-TTY. You can also run one-off commands like `docker exec my-container ls /app`.

### Q5: What does the `-d` flag do in `docker run`?
**A**: The `-d` flag runs the container in **detached mode**, meaning it runs in the background and returns the container ID. Without `-d`, the container runs in the foreground and your terminal is occupied with the container's output logs.

---

## What's Next?
You've been running pre-built images from Docker Hub. Now let's build your own image by writing a **Dockerfile**.

→ [Next: 03 — Writing Your First Dockerfile](../03-dockerfile/README.md)
