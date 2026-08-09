# 00 — Setup & Installation

## What You'll Learn
- How to install Docker on your Mac
- Verify that Docker is working
- Understand the difference between Docker Engine, Docker Desktop, and Docker CLI

---

## Step 1: Install Docker Desktop

Docker Desktop is the easiest way to get Docker running on macOS. It bundles everything you need — the Docker Engine, CLI, and a GUI dashboard.

### Download & Install

1. Go to [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
2. Click **"Download for Mac"**
   - If you have an **Apple Silicon Mac** (M1/M2/M3/M4) → choose the **Apple Silicon** version
   - If you have an **Intel Mac** → choose the **Intel** version
3. Open the downloaded `.dmg` file
4. Drag the Docker icon to your Applications folder
5. Open Docker from your Applications (or Spotlight search "Docker")
6. You'll see the Docker whale icon 🐳 appear in your menu bar — that means it's running

> 💡 **Not sure which chip you have?**  
> Click the Apple logo () → "About This Mac" → look for "Chip" (Apple M1/M2/M3/M4) or "Processor" (Intel)

---

## Step 2: Verify the Installation

Open your terminal and run these commands:

```bash
# Check Docker version
docker --version
```
**Expected output** (version numbers may vary):
```
Docker version 27.x.x, build xxxxxxx
```

```bash
# Check Docker Compose version (comes bundled with Docker Desktop)
docker compose version
```
**Expected output**:
```
Docker Compose version v2.x.x
```

```bash
# The ultimate test — run a hello-world container
docker run hello-world
```
**Expected output**: A big friendly message that starts with:
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

If you see that message — congratulations, Docker is installed and working! 🎉

---

## Step 3: Understand What You Just Installed

Here's the thing — when people say "Docker", they could mean 3 different things:

### Docker Engine
The **core runtime** that actually builds and runs containers. It runs as a background service (daemon) on your machine. Think of it as the engine of a car — you don't interact with it directly, but it does all the work.

### Docker CLI
The **command-line tool** you type commands into (`docker run`, `docker build`, etc.). Think of it as the steering wheel — you use it to tell the engine what to do.

### Docker Desktop
A **GUI application** that wraps Docker Engine + Docker CLI + extras (like a visual dashboard to see your containers, images, volumes). Think of it as the entire car — it includes the engine, steering wheel, and a nice dashboard.

```
┌──────────────────────────────────────┐
│         DOCKER DESKTOP (GUI)         │
│                                      │
│   ┌──────────────────────────────┐   │
│   │        DOCKER CLI            │   │
│   │  (commands you type in       │   │
│   │   the terminal)              │   │
│   └──────────┬───────────────────┘   │
│              │                       │
│   ┌──────────▼───────────────────┐   │
│   │      DOCKER ENGINE           │   │
│   │  (daemon that runs           │   │
│   │   containers)                │   │
│   └──────────────────────────────┘   │
│                                      │
└──────────────────────────────────────┘
```

### Quick Summary

| Component       | What it is                     | You interact with it? |
|-----------------|--------------------------------|-----------------------|
| Docker Engine   | The runtime (daemon)           | No (runs in background) |
| Docker CLI      | Terminal commands               | Yes (`docker run ...`) |
| Docker Desktop  | GUI app bundling everything    | Yes (optional dashboard) |

---

## Step 4: Docker Desktop UI — Quick Walkthrough

When you open Docker Desktop, you'll see a few tabs:

- **Containers**: Shows all running and stopped containers. Right now this will be empty (or show the hello-world container).
- **Images**: Shows all downloaded images. You should see `hello-world` here.
- **Volumes**: Shows persistent data volumes (we'll learn this in Chapter 05).
- **Settings/Preferences**: Where you can allocate CPU, memory, and disk to Docker.

> 💡 You'll mostly use the **terminal** for everything. Docker Desktop GUI is just nice to have for a visual overview.

---

## Useful Commands to Remember

```bash
# Check if Docker daemon is running
docker info

# See all downloaded images
docker images

# See running containers
docker ps

# See ALL containers (including stopped ones)
docker ps -a

# Remove the hello-world container (get the container ID from docker ps -a)
docker rm <container_id>
```

---

## What's Next?
In the next chapter, we'll understand **what Docker actually is**, why it exists, and the core concepts that everything else builds on.

→ [Next: 01 — What is Docker & Why Do We Need It?](../01-what-is-docker/README.md)
