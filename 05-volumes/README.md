# 05 — Docker Volumes & Data Persistence

## What You'll Learn
- Why data disappears when containers are removed
- What Docker volumes are and why they exist
- Named volumes vs Bind mounts
- How to persist MongoDB data across container restarts
- When to use volumes vs bind mounts

---

## The Problem: Containers Are Ephemeral

By default, any data written inside a container is **stored in the container's writable layer**. When you `docker rm` the container, that data is gone forever.

```bash
# Run MongoDB
docker run -d --name my-mongo -p 27017:27017 mongo:7

# Insert some data
docker exec -it my-mongo mongosh
> use mydb
> db.users.insertOne({ name: "Aryan", role: "SDE Intern" })
> db.users.find()    # ← Data is there!
> exit

# Remove the container
docker stop my-mongo
docker rm my-mongo

# Run MongoDB again
docker run -d --name my-mongo -p 27017:27017 mongo:7

# Check for data
docker exec -it my-mongo mongosh
> use mydb
> db.users.find()    # ← Empty! Data is GONE! 😱
> exit

docker rm -f my-mongo
```

This is a **huge problem** for databases. You can't lose your MongoDB data every time a container restarts or gets redeployed.

**Solution: Docker Volumes** — persistent storage that lives outside the container.

---

## What is a Volume?

A volume is a **directory managed by Docker** that exists on the host filesystem but is **mounted** into a container. The container reads and writes to it like a normal directory, but the data persists even after the container is deleted.

```
Without Volumes:
┌────────────────────────┐
│     Container          │
│  ┌──────────────────┐  │
│  │  /data/db         │  │  ← Data stored HERE
│  │  (container layer)│  │     Dies with container!
│  └──────────────────┘  │
└────────────────────────┘

With Volumes:
┌────────────────────────┐         ┌─────────────────────┐
│     Container          │         │   Host Filesystem    │
│  ┌──────────────────┐  │         │                     │
│  │  /data/db         │──────────►│  Docker Volume       │
│  │  (mounted volume) │  │         │  /var/lib/docker/    │
│  └──────────────────┘  │         │  volumes/mongo-data/ │
└────────────────────────┘         └─────────────────────┘
                                     ↑ Persists after
                                       container is deleted!
```

---

## Two Types of Persistent Storage

### 1. Named Volumes (Docker-managed)

Docker manages the storage location on the host. You don't need to know or care where the data lives on disk.

```bash
# Create a named volume
docker volume create mongo-data

# Use it in a container
docker run -d \
  --name my-mongo \
  -v mongo-data:/data/db \
  -p 27017:27017 \
  mongo:7
```

The `-v mongo-data:/data/db` means:
- `mongo-data` → Name of the volume on the host
- `/data/db` → Path inside the container where MongoDB stores its data
- Docker maps one to the other

### 2. Bind Mounts (You choose the path)

You explicitly specify a path on your host machine to mount into the container.

```bash
# Mount your local project directory into the container
docker run -d \
  --name my-api \
  -v $(pwd)/src:/app/src \
  -p 3000:3000 \
  my-express-app
```

The `-v $(pwd)/src:/app/src` means:
- `$(pwd)/src` → Your local machine's `src` folder
- `/app/src` → Path inside the container
- Changes on your machine appear instantly in the container (and vice versa)

### Comparison

| Feature                  | Named Volume              | Bind Mount                     |
|--------------------------|---------------------------|--------------------------------|
| Managed by               | Docker                    | You (specify host path)        |
| Location on host         | Docker decides            | You decide                     |
| Syntax                   | `-v name:/path`           | `-v /host/path:/container/path`|
| Use case                 | Database data, persistence| Development (live code reload) |
| Portable                 | Yes (Docker handles it)   | No (path is machine-specific)  |
| Works on all OS          | Yes                       | Path issues on Windows         |
| Listed in `docker volume` | Yes                      | No                             |

> 🎯 **Rule of thumb**: 
> - **Named volumes** → for data you want to persist (databases)
> - **Bind mounts** → for code you're actively developing (live reload)

---

## Hands-On: Persist MongoDB Data

Let's prove that volumes actually work.

### Step 1: Create a Volume and Run MongoDB

```bash
# Create a named volume
docker volume create mongo-data

# Run MongoDB with the volume mounted
docker run -d \
  --name my-mongo \
  -v mongo-data:/data/db \
  -p 27017:27017 \
  mongo:7
```

### Step 2: Insert Data

```bash
docker exec -it my-mongo mongosh

# Inside mongosh:
use mydb
db.users.insertOne({ name: "Aryan", role: "SDE Intern" })
db.users.insertOne({ name: "Docker", role: "Container Platform" })
db.users.find()
# Shows both documents
exit
```

### Step 3: Destroy the Container

```bash
docker stop my-mongo
docker rm my-mongo

# Container is gone!
docker ps -a | grep mongo   # Nothing
```

### Step 4: Create a NEW Container with the SAME Volume

```bash
docker run -d \
  --name my-mongo-new \
  -v mongo-data:/data/db \
  -p 27017:27017 \
  mongo:7
```

### Step 5: Check — Data Survives! 🎉

```bash
docker exec -it my-mongo-new mongosh

use mydb
db.users.find()
# Output: BOTH documents are still there!
exit
```

The data survived because it's stored in the `mongo-data` volume, not in the container.

### Clean Up

```bash
docker rm -f my-mongo-new
# Volume still exists even after removing the container:
docker volume ls    # mongo-data is still there
```

---

## Bind Mounts for Development (Live Reload)

In development, you want code changes to reflect immediately without rebuilding the image. Bind mounts make this possible.

```bash
# Mount your local source code into the container
docker run -d \
  --name dev-api \
  -v $(pwd):/app \
  -p 3000:3000 \
  node:18-alpine \
  sh -c "cd /app && npm install && node app.js"
```

Now when you edit `app.js` on your Mac, the changes are immediately visible inside the container (though you'd need nodemon or a restart to pick them up).

> 💡 In real projects, you'd use bind mounts with `nodemon` for auto-restart, or pair it with Docker Compose (Chapter 06).

---

## Volume Management Commands

```bash
# Create a volume
docker volume create my-volume

# List all volumes
docker volume ls

# Inspect a volume (shows where it's stored on host)
docker volume inspect my-volume

# Remove a specific volume
docker volume rm my-volume

# Remove ALL unused volumes (⚠️ careful!)
docker volume prune
```

### Inspecting a Volume

```bash
docker volume inspect mongo-data
```

**Output:**
```json
[
  {
    "CreatedAt": "2024-01-15T10:30:00Z",
    "Driver": "local",
    "Labels": {},
    "Mountpoint": "/var/lib/docker/volumes/mongo-data/_data",
    "Name": "mongo-data",
    "Options": {},
    "Scope": "local"
  }
]
```

The `Mountpoint` shows where Docker stores the data on the host filesystem.

---

## Quick Reference

```
Volume Syntax in docker run:

Named Volume:     -v volume-name:/path/in/container
Bind Mount:       -v /absolute/host/path:/path/in/container
                  -v $(pwd)/relative/path:/path/in/container
```

---

## 🎤 Interview Q&A

### Q1: What are Docker volumes and why are they needed?
**A**: Docker volumes are a mechanism for persisting data generated by containers. They're needed because containers are ephemeral — any data written to a container's writable layer is lost when the container is removed. Volumes store data on the host filesystem outside the container's lifecycle, so data survives container restarts and removals. Common use case: persisting database data.

### Q2: What is the difference between a named volume and a bind mount?
**A**: A **named volume** is managed by Docker — Docker decides where to store it on the host filesystem and manages its lifecycle. You reference it by name (e.g., `-v my-data:/app/data`). A **bind mount** maps a specific host path to a container path (e.g., `-v /home/user/code:/app`). Named volumes are preferred for data persistence (databases), while bind mounts are useful in development for live code reloading.

### Q3: How do you persist MongoDB data in Docker?
**A**: By mounting a named volume to MongoDB's data directory (`/data/db`). For example: `docker run -v mongo-data:/data/db mongo:7`. The volume `mongo-data` stores all MongoDB data on the host. Even if the container is deleted, creating a new container with the same volume will restore all the data.

---

## What's Next?
You now know how to run containers, connect them via networks, and persist data with volumes. But managing multiple containers with individual `docker run` commands is tedious. Enter Docker Compose — define everything in a single YAML file.

→ [Next: 06 — Docker Compose — The YML File](../06-docker-compose/README.md)
