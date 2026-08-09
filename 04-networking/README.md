# 04 — Docker Networking Basics

## What You'll Learn
- Why containers need their own networking
- Docker's default bridge network
- How to create custom networks
- How containers discover each other by name (DNS)
- Port mapping revisited
- Hands-on: Connect Express to MongoDB over a Docker network

---

## Why Do Containers Need Networking?

In a MERN app, your Express API needs to talk to MongoDB. On your local machine (without Docker), you'd just connect to `mongodb://localhost:27017`. But with Docker, each container is an **isolated environment** — they can't see each other by default.

Docker networking solves this. It lets containers talk to each other in a controlled way.

```
Without Docker:
  Express → localhost:27017 → MongoDB     (same machine, easy)

With Docker (no networking):
  [Express Container] → ??? → [MongoDB Container]     (isolated, can't connect!)

With Docker (proper networking):
  [Express Container] → mongodb://mongo:27017 → [MongoDB Container]     (connected!)
```

---

## Docker Network Types

Docker provides several network drivers. You only need to know three:

| Network Type | What it does                                       | Use case                     |
|-------------|--------------------------------------------------|------------------------------|
| **bridge**  | Default network. Containers can talk via IP/name   | Single-host, most common     |
| **host**    | Container shares the host's network directly       | Performance-critical apps    |
| **none**    | No networking at all                               | Security-isolated containers |

For interviews and daily use, you'll almost exclusively use **bridge** networks.

---

## The Default Bridge Network

When you install Docker, it creates a default bridge network. Any container you run without specifying a network goes here.

```bash
# See all networks
docker network ls
```

**Output:**
```
NETWORK ID     NAME      DRIVER    SCOPE
abc123def456   bridge    bridge    local
def789ghi012   host      host      local
ghi345jkl678   none      null      local
```

### The Problem with the Default Bridge

Containers on the default bridge network **cannot discover each other by name**. They can only communicate by IP address, which changes every time a container restarts. This is fragile and not practical.

```bash
# Run two containers on the default bridge
docker run -d --name container-a nginx
docker run -d --name container-b nginx

# From container-a, try to reach container-b by name → FAILS
docker exec container-a ping container-b
# ping: bad address 'container-b'

# Clean up
docker rm -f container-a container-b
```

**Solution**: Create a **custom bridge network**. Containers on a custom network CAN discover each other by name.

---

## Custom Bridge Networks — The Right Way

### Creating a Network

```bash
# Create a custom network
docker network create my-app-network

# Verify it was created
docker network ls
```

### Running Containers on the Network

```bash
# Run MongoDB on the custom network
docker run -d \
  --name mongo \
  --network my-app-network \
  -p 27017:27017 \
  mongo:7

# Run another container on the same network
docker run -it --rm \
  --network my-app-network \
  node:18-alpine sh
```

Now from inside the Node.js container, you can reach MongoDB by its **container name**:

```bash
# Inside the Node.js container:
ping mongo
# PING mongo (172.18.0.2): 56 data bytes
# 64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.123 ms
```

It works! 🎉 The container name `mongo` resolves to the container's IP address automatically.

---

## Container DNS — How Name Resolution Works

When you put containers on a **custom bridge network**, Docker runs a built-in DNS server. Containers can reach each other by name instead of IP.

```
Custom Bridge Network: "my-app-network"
┌─────────────────────────────────────────────────────┐
│                                                     │
│   ┌──────────────┐         ┌──────────────────┐     │
│   │   express     │────────►│     mongo         │     │
│   │  (container)  │  DNS:   │   (container)     │     │
│   │              │ "mongo" │                    │     │
│   │ connects to: │ resolves│  Listening on      │     │
│   │ mongo:27017  │ to IP   │  port 27017        │     │
│   └──────────────┘         └──────────────────┘     │
│                                                     │
│          Docker DNS Server (built-in)               │
│          Resolves container names → IPs              │
└─────────────────────────────────────────────────────┘
```

### Key Rules:
1. **Custom networks**: Containers CAN reach each other by name ✅
2. **Default bridge**: Containers CANNOT reach each other by name ❌ (IP only)
3. The DNS name is the `--name` you give the container
4. Containers on **different** networks cannot communicate at all

> 🎯 **Interview one-liner**: "I use custom bridge networks because they provide automatic DNS resolution — containers can find each other by name instead of hardcoding IP addresses."

---

## Port Mapping Revisited

Remember `-p HOST:CONTAINER`? Let's understand when you need it and when you don't.

### Rule of Thumb:
- **Container-to-container** communication (within the same network): You do NOT need `-p`. Containers talk directly on internal ports.
- **Host-to-container** communication (accessing from your browser or `curl`): You NEED `-p`.

```bash
# MongoDB: ONLY other containers need to access it — NO -p needed!
docker run -d \
  --name mongo \
  --network my-app-network \
  mongo:7
# Express connects internally via mongo:27017 — no port published to host

# Express: we want to access it from our browser — NEED -p
docker run -d \
  --name express-api \
  --network my-app-network \
  -p 3000:3000 \
  my-express-app
```

The Express container talks to MongoDB on the **internal** network — no `-p` needed. You only use `-p` for services YOU need to access from your Mac (browser, curl, Postman).

```
Your Mac (Host)
│
├── localhost:3000 ──► [Express Container :3000]    ← -p 3000:3000 (you access this)
│                              │
│                              │ (internal network, no -p needed)
│                              │
│                              ▼
│                       [MongoDB Container :27017]  ← NO -p (only Express talks to it)
│
```

> 💡 **When WOULD you add `-p 27017:27017` to MongoDB?**  
> Only if you also want to access MongoDB from your Mac — e.g., using **MongoDB Compass** or **mongosh** on your host machine for debugging. In production, databases should NOT have their ports exposed to the host.

---

## Hands-On: Connect Express to MongoDB

Let's wire up Express and MongoDB using Docker networking.

### Step 1: Create the Network

```bash
docker network create mern-network
```

### Step 2: Run MongoDB

```bash
# No -p needed! Only Express (another container) will access MongoDB.
docker run -d \
  --name mongo \
  --network mern-network \
  mongo:7
```

> 💡 Notice there's no `-p 27017:27017` here. We don't need to access MongoDB from our Mac — only the Express container needs to reach it, and they're on the same network.

### Step 3: Run Express (from Chapter 03's image)

First, update the Express app to connect to MongoDB. Create a new version:

```javascript
// app-with-mongo.js
const express = require('express');
const app = express();
const PORT = 3000;

// Note: 'mongo' is the container name — Docker DNS resolves it!
const MONGO_URL = process.env.MONGO_URL || 'mongodb://mongo:27017/myapp';

app.get('/', (req, res) => {
  res.json({
    message: 'Express + MongoDB on Docker! 🐳',
    mongoUrl: MONGO_URL,
    note: 'The hostname "mongo" is resolved by Docker DNS'
  });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
  console.log(`MongoDB URL: ${MONGO_URL}`);
});
```

The key line is:
```javascript
const MONGO_URL = 'mongodb://mongo:27017/myapp';
//                          ^^^^^
//                    This is the container name, NOT localhost!
```

### Step 4: Run Express on the Same Network

```bash
docker run -d \
  --name express-api \
  --network mern-network \
  -p 3000:3000 \
  -e MONGO_URL=mongodb://mongo:27017/myapp \
  my-express-app
```

### Step 5: Verify

```bash
# Check both containers are running
docker ps

# Check they're on the same network
docker network inspect mern-network

# Test the API
curl http://localhost:3000
```

### Clean Up

```bash
docker rm -f express-api mongo
docker network rm mern-network
```

---

## Useful Network Commands

| Command                                    | What it does                              |
|--------------------------------------------|-------------------------------------------|
| `docker network ls`                        | List all networks                         |
| `docker network create <name>`             | Create a custom bridge network            |
| `docker network rm <name>`                 | Remove a network                          |
| `docker network inspect <name>`            | See details (connected containers, IPs)   |
| `docker network connect <net> <container>` | Attach a running container to a network   |
| `docker network disconnect <net> <container>` | Detach a container from a network      |

---

## 🎤 Interview Q&A

### Q1: How do Docker containers communicate with each other?
**A**: Containers communicate by being placed on the same Docker network. With a **custom bridge network**, containers can discover each other using container names as hostnames — Docker provides built-in DNS resolution. For example, an Express container can connect to MongoDB using `mongodb://mongo:27017` where `mongo` is the MongoDB container's name on the same network.

### Q2: What is the difference between the default bridge and a custom bridge network?
**A**: The default bridge network doesn't support automatic DNS resolution — containers can only communicate via IP addresses, which are dynamic. A custom bridge network provides built-in DNS, so containers can reach each other by name. Custom networks also provide better isolation and can be scoped to specific application stacks.

### Q3: When do you need port mapping (`-p`)?
**A**: Port mapping is needed when you want to access a container's service from the **host machine** (e.g., your browser or `curl`). For **container-to-container** communication on the same network, port mapping is not required — containers can communicate directly on their internal ports.

---

## What's Next?
Your containers can talk to each other now, but if you stop MongoDB, all your data is gone! Let's fix that with volumes.

→ [Next: 05 — Docker Volumes & Data Persistence](../05-volumes/README.md)
