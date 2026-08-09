# 🐳 Docker — SDE Intern Interview Prep

A structured, hands-on Docker learning guide tailored for someone with MERN stack on their resume. Goes from zero to interview-ready.

**Approach**: Concept first (WHY), then commands (HOW), then interview Q&A at the end of every chapter.

---

## 📚 Syllabus

| #  | Chapter                                                        | What You'll Learn                                                           |
|----|----------------------------------------------------------------|-----------------------------------------------------------------------------|
| 00 | [Setup & Installation](./00-setup/README.md)                   | Install Docker Desktop, verify it works, understand Engine vs CLI vs Desktop |
| 01 | [What is Docker & Why?](./01-what-is-docker/README.md)         | The "works on my machine" problem, VMs vs containers, architecture, images vs containers |
| 02 | [Images & Essential Commands](./02-images-and-commands/README.md) | Pull/run/stop/remove containers, all important flags (`-d`, `-p`, `-it`, `--name`), hands-on with Nginx, Node, MongoDB |
| 03 | [Writing Your First Dockerfile](./03-dockerfile/README.md)     | Dockerfile instructions (FROM, WORKDIR, COPY, RUN, EXPOSE, CMD), layer caching, .dockerignore, Dockerize an Express API |
| 04 | [Docker Networking](./04-networking/README.md)                 | Default vs custom bridge, container DNS, connecting Express ↔ MongoDB       |
| 05 | [Volumes & Data Persistence](./05-volumes/README.md)           | Named volumes vs bind mounts, persist MongoDB data, development live reload |
| 06 | [Docker Compose — The YML File](./06-docker-compose/README.md) | Write docker-compose.yml from scratch, every field explained, full MERN stack in one file |
| 07 | [Interview Rapid Fire](./07-interview-cheatsheet/README.md)    | 20 interview Q&As, resume talking points, "tell me about Docker" script, command cheat sheet |

---

## 🎯 How to Use This Guide

1. **Read in order** — each chapter builds on the previous one
2. **Try every command** — reading alone won't stick, run the hands-on exercises
3. **Focus on Chapter 07 before the interview** — it has the 20 most common questions with answers
4. **The MERN compose example in Chapter 06** is your go-to reference for "how I used Docker in my project"

---

## ⏱️ Time Estimate

| Speed        | Time to complete |
|-------------|------------------|
| Quick read   | 2–3 hours        |
| With hands-on| 4–5 hours        |
| Deep study   | 6–8 hours        |

---

## 📁 Project Structure

```
Docker/
├── README.md                          ← You are here (syllabus)
├── 00-setup/README.md                 ← Installation guide
├── 01-what-is-docker/README.md        ← Core concepts
├── 02-images-and-commands/README.md   ← Commands deep dive
├── 03-dockerfile/                     ← Dockerfile tutorial
│   ├── README.md
│   ├── app.js                         ← Demo Express app
│   ├── package.json
│   ├── Dockerfile                     ← Working Dockerfile
│   └── .dockerignore
├── 04-networking/README.md            ← Container networking
├── 05-volumes/README.md               ← Data persistence
├── 06-docker-compose/                 ← Docker Compose tutorial
│   ├── README.md
│   ├── docker-compose.yml             ← Working compose file
│   ├── backend/
│   │   ├── Dockerfile
│   │   ├── package.json
│   │   ├── app.js
│   │   └── .dockerignore
│   └── frontend/
│       ├── Dockerfile
│       ├── package.json
│       ├── src/index.html
│       └── .dockerignore
└── 07-interview-cheatsheet/README.md  ← Interview prep & cheat sheet
```
