# 07 — Interview Rapid Fire & Resume Talking Points

## What You'll Learn
- 20 most-asked Docker interview questions for SDE interns (with crisp answers)
- How to talk about Docker in your MERN project on your resume
- Common gotchas interviewers love to test
- Docker vs Kubernetes — 1-minute explanation
- Full command cheat sheet

---

## 🎤 Top 20 Interview Questions — Rapid Fire

### Basics (Questions 1–7)

**Q1: What is Docker?**
> Docker is a platform for building, shipping, and running applications in containers. A container packages an application with all its dependencies, ensuring it runs consistently across any environment.

**Q2: What is the difference between a container and a virtual machine?**
> A VM virtualizes the hardware and runs a full guest OS — it's heavy (GBs) and slow to start. A container virtualizes the OS — it shares the host kernel and only packages the app + dependencies, making it lightweight (MBs) and fast to start (seconds). Containers provide process-level isolation, VMs provide hardware-level isolation.

**Q3: What is a Docker Image?**
> A Docker image is a read-only template that contains the application code, runtime, libraries, and configuration needed to run an application. It's built in layers, and each layer is cached for efficiency. You can think of it as a class — containers are instances of that class.

**Q4: What is the difference between an Image and a Container?**
> An image is a static, read-only blueprint. A container is a running instance of that image. You can create multiple containers from the same image. Images are created with `docker build`, containers with `docker run`.

**Q5: What is a Dockerfile?**
> A Dockerfile is a text file containing instructions to build a Docker image. It specifies the base image (`FROM`), copies files (`COPY`), installs dependencies (`RUN`), exposes ports (`EXPOSE`), and defines the startup command (`CMD`).

**Q6: What is Docker Hub?**
> Docker Hub is a public registry for Docker images, similar to npm for Node.js packages. You can pull official images (like `node`, `mongo`, `nginx`) and push your own images. It's the default registry Docker uses.

**Q7: What does `docker run -d -p 3000:5000 --name myapp image` do?**
> - `-d`: Runs the container in detached mode (background)
> - `-p 3000:5000`: Maps host port 3000 to container port 5000
> - `--name myapp`: Names the container "myapp"
> - Runs a container from the specified image

---

### Dockerfile & Build (Questions 8–12)

**Q8: What is the difference between `CMD` and `RUN`?**
> `RUN` executes commands at **build time** — the results become part of the image layer (e.g., `RUN npm install`). `CMD` defines the **default command** that runs when a container starts (e.g., `CMD ["node", "app.js"]`). You can have multiple `RUN` instructions, but only one `CMD`.

**Q9: What is the difference between `CMD` and `ENTRYPOINT`?**
> `CMD` sets the default command and can be easily overridden at runtime. `ENTRYPOINT` sets a fixed executable that always runs — the `CMD` becomes arguments to `ENTRYPOINT`. Use `CMD` when you want users to override the command; use `ENTRYPOINT` when the container should always run a specific program.

**Q10: What does `EXPOSE` do?**
> `EXPOSE` is a documentation instruction — it tells users which port the application listens on, but it doesn't actually publish the port. You still need `-p` in `docker run` to make the port accessible from the host.

**Q11: Why do we copy `package.json` before copying the rest of the code?**
> For **layer caching optimization**. Docker caches each layer and only rebuilds from the changed layer onward. By copying `package.json` first and running `npm install`, the dependency layer is cached and reused when only application code changes, making builds much faster.

**Q12: What is `.dockerignore`?**
> Similar to `.gitignore`, it specifies files to exclude from the Docker build context. It prevents large or sensitive files (like `node_modules`, `.git`, `.env`) from being sent to the Docker daemon, reducing build time and image size.

---

### Networking & Volumes (Questions 13–16)

**Q13: How do Docker containers communicate with each other?**
> By placing them on the same Docker network. With a custom bridge network, containers can discover each other by container name using Docker's built-in DNS. For example, Express can connect to MongoDB using `mongodb://mongo:27017` where `mongo` is the container name.

**Q14: What are Docker volumes? Why do we need them?**
> Volumes are a mechanism to persist data beyond a container's lifecycle. By default, data inside a container is lost when the container is removed. Volumes store data on the host filesystem, so it survives container restarts and removals. Common use: persisting database data like MongoDB's `/data/db`.

**Q15: What is the difference between a named volume and a bind mount?**
> A named volume is managed by Docker — you reference it by name and Docker handles storage. A bind mount maps a specific host path to a container path. Named volumes are for persistence (databases), bind mounts are for development (live code syncing).

**Q16: What happens to data when you run `docker compose down` vs `docker compose down -v`?**
> `docker compose down` removes containers and networks but **preserves volumes** (data is safe). `docker compose down -v` also deletes volumes, destroying all persisted data. Use `-v` only when you want a complete clean slate.

---

### Docker Compose (Questions 17–20)

**Q17: What is Docker Compose?**
> Docker Compose is a tool for defining and running multi-container applications. You describe all services, networks, and volumes in a `docker-compose.yml` file, then start everything with `docker compose up`. It simplifies managing complex application stacks.

**Q18: What does `depends_on` do?**
> It controls startup order — the dependent service starts first. However, it only waits for the container to start, not for the application inside to be "ready." For example, MongoDB's container may start before it's accepting connections.

**Q19: What is the difference between `docker compose up` and `docker compose up --build`?**
> `docker compose up` starts services using existing images; it won't rebuild if the Dockerfile or code has changed. `docker compose up --build` forces a rebuild of images before starting, ensuring the latest code is included.

**Q20: How do services communicate in Docker Compose?**
> Compose auto-creates a network for all services. Services reach each other using the **service name** as hostname. If you have a service called `mongo`, Express connects to it as `mongodb://mongo:27017`. Docker DNS resolves the service name to the correct IP.

---

## 📝 Resume Talking Points — "Tell me about Docker in your project"

### Sample Answer (MERN Stack Project)

> "In my MERN stack project, I used Docker to containerize all three components — the React frontend, Express.js backend API, and MongoDB database. 
>
> I wrote individual **Dockerfiles** for the frontend and backend, using `node:18-alpine` as the base image to keep the image size small. I optimized the build by copying `package.json` first for layer caching, so dependency installation was cached across builds.
>
> I used **Docker Compose** to orchestrate all three services. The `docker-compose.yml` defined the services, their port mappings, environment variables, and a named volume for MongoDB data persistence. The backend connected to MongoDB using the service name as the hostname — Docker's built-in DNS resolved it automatically.
>
> This setup meant any developer could clone the repo, run `docker compose up`, and have the entire stack running in seconds — regardless of their OS or what was installed on their machine."

### If They Ask Follow-ups:

**"Why Docker and not just run everything locally?"**
> "Docker eliminates the 'works on my machine' problem. Different developers had different Node.js versions and MongoDB versions. Docker ensured everyone ran the exact same environment."

**"How did you handle data persistence?"**
> "I used a named Docker volume mounted to MongoDB's `/data/db` directory. This ensured database data survived container restarts and `docker compose down`."

**"How did the services communicate?"**
> "All services were on the same Docker Compose network. The backend connected to MongoDB using the service name — `mongodb://mongo:27017/myapp`. Docker Compose creates a default network and provides DNS resolution for service names."

**"What's in your Dockerfile?"**
> "I used `FROM node:18-alpine` for a lightweight base, set a `WORKDIR`, copied `package.json` first and ran `npm install` for layer caching, then copied the rest of the code. I used `EXPOSE` to document the port and `CMD` to start the app."

---

## ⚠️ Common Gotchas Interviewers Love to Test

### 1. "Why can't you use `localhost` to connect containers?"
Inside a container, `localhost` refers to **that container itself**, not the host machine. To connect to another container, use the **container name** or **service name** on a shared Docker network.

### 2. "Does `EXPOSE` actually publish the port?"
No! `EXPOSE` is just metadata/documentation. You MUST use `-p` in `docker run` or `ports:` in `docker-compose.yml` to actually make the port accessible.

### 3. "What happens if you don't use volumes with a database?"
All data is stored in the container's writable layer. When the container is removed (`docker rm`), all data is permanently lost. Volumes solve this.

### 4. "Why use `alpine` images?"
Alpine Linux is a minimal Linux distribution (~5MB). Using alpine variants (`node:18-alpine`) results in much smaller images compared to full images (`node:18` is ~350MB, `node:18-alpine` is ~178MB). Smaller images = faster pulls, less disk, faster CI/CD.

### 5. "Difference between `docker compose stop` and `docker compose down`?"
- `stop` → Stops containers but keeps them (you can `start` again)
- `down` → Stops AND removes containers and networks (but keeps volumes)
- `down -v` → Removes everything including volumes

### 6. "`docker run` vs `docker start`?"
- `docker run` = Creates a **new container** and starts it
- `docker start` = Starts an **existing stopped container** (no new container created)

---

## Docker vs Kubernetes — 1-Minute Explanation

Interviewers sometimes ask this just to see if you know the difference.

| Aspect              | Docker                                       | Kubernetes (K8s)                           |
|---------------------|----------------------------------------------|--------------------------------------------|
| **What it does**    | Builds and runs individual containers        | Orchestrates many containers across servers |
| **Scale**           | Single machine                               | Multiple machines (cluster)                |
| **Use case**        | Development, small deployments               | Production at scale (100s of containers)   |
| **Analogy**         | A single ship carrying cargo                 | A fleet of ships managed by a harbor master |
| **Complexity**      | Simple                                       | Complex                                    |

### One-liner Answer:
> "Docker is for creating and running containers. Kubernetes is for orchestrating and managing containers at scale across multiple machines. Docker is the container runtime; Kubernetes is the container orchestrator. You use Docker to build the containers, and Kubernetes to run and manage them in production."

> 💡 For an SDE intern interview, knowing this distinction is enough. You don't need to know Kubernetes commands or architecture.

---

## 📋 Complete Command Cheat Sheet

### Images
```bash
docker pull image:tag              # Download an image
docker images                      # List local images
docker rmi image                   # Remove an image
docker image prune -a              # Remove all unused images
docker build -t name:tag .         # Build from Dockerfile
```

### Containers
```bash
docker run -d --name n -p H:C img # Run a container
docker ps                          # List running containers
docker ps -a                       # List ALL containers
docker stop name                   # Stop a container
docker start name                  # Start a stopped container
docker rm name                     # Remove a container
docker rm -f name                  # Force remove (even if running)
docker container prune             # Remove all stopped containers
```

### Inspect & Debug
```bash
docker logs name                   # View logs
docker logs -f name                # Follow logs (real-time)
docker exec -it name sh            # Shell into container
docker inspect name                # Detailed container info
```

### Networking
```bash
docker network ls                  # List networks
docker network create name         # Create a network
docker network rm name             # Remove a network
docker network inspect name        # Network details
```

### Volumes
```bash
docker volume ls                   # List volumes
docker volume create name          # Create a volume
docker volume rm name              # Remove a volume
docker volume inspect name         # Volume details
docker volume prune                # Remove unused volumes
```

### Docker Compose
```bash
docker compose up                  # Start (foreground)
docker compose up -d               # Start (background)
docker compose up --build          # Rebuild and start
docker compose down                # Stop and remove
docker compose down -v             # Stop, remove, delete volumes
docker compose ps                  # List services
docker compose logs -f service     # Follow service logs
docker compose exec service sh     # Shell into service
docker compose build               # Build images
docker compose stop                # Stop without removing
docker compose restart             # Restart services
```

---

## 🎓 Summary — What You've Learned

| Chapter | Topic                    | Key Takeaway                                          |
|---------|--------------------------|-------------------------------------------------------|
| 00      | Setup                    | Docker Desktop bundles Engine + CLI + GUI              |
| 01      | What is Docker           | Containers = lightweight, isolated app environments    |
| 02      | Images & Commands        | Images are blueprints, containers are running instances |
| 03      | Dockerfile               | Recipe to build images; layer caching matters          |
| 04      | Networking               | Custom networks enable DNS-based container discovery   |
| 05      | Volumes                  | Named volumes persist data beyond container lifecycle  |
| 06      | Docker Compose           | One YAML file to orchestrate multi-container apps      |
| 07      | Interview Prep           | You're ready! 🚀                                      |

---

Good luck with your interview! 🎯🐳
