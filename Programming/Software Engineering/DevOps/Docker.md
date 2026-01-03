---
tags:
  - software-engineering
  - devops
  - docker
  - containers
created: 2026-01-02
status: 🔴
---
# 🐳 Docker

> *"Build once, run anywhere."*

## 🎯 What is Docker?

Docker es una plataforma de **containerización** que empaqueta aplicaciones y sus dependencias en contenedores livianos, portables y consistentes.

---

## 🏗️ Container vs VM

```
┌─────────────────────────────────────────────────────────────────┐
│                    VIRTUAL MACHINES                              │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                       │
│  │   App    │  │   App    │  │   App    │                       │
│  ├──────────┤  ├──────────┤  ├──────────┤                       │
│  │  Bins/   │  │  Bins/   │  │  Bins/   │                       │
│  │  Libs    │  │  Libs    │  │  Libs    │                       │
│  ├──────────┤  ├──────────┤  ├──────────┤                       │
│  │ Guest OS │  │ Guest OS │  │ Guest OS │  ◄─ Full OS each      │
│  └──────────┘  └──────────┘  └──────────┘                       │
│  ┌─────────────────────────────────────────────────────┐        │
│  │              HYPERVISOR                              │        │
│  └─────────────────────────────────────────────────────┘        │
│  ┌─────────────────────────────────────────────────────┐        │
│  │                HOST OS                               │        │
│  └─────────────────────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                      CONTAINERS                                  │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                       │
│  │   App    │  │   App    │  │   App    │                       │
│  ├──────────┤  ├──────────┤  ├──────────┤                       │
│  │  Bins/   │  │  Bins/   │  │  Bins/   │  ◄─ Only bins/libs    │
│  │  Libs    │  │  Libs    │  │  Libs    │                       │
│  └──────────┘  └──────────┘  └──────────┘                       │
│  ┌─────────────────────────────────────────────────────┐        │
│  │              DOCKER ENGINE                           │        │
│  └─────────────────────────────────────────────────────┘        │
│  ┌─────────────────────────────────────────────────────┐        │
│  │                HOST OS                               │        │
│  └─────────────────────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────────────┘
```

| Aspect | Container | Virtual Machine |
|--------|-----------|-----------------|
| **Startup** | Seconds | Minutes |
| **Size** | MBs | GBs |
| **Performance** | Near native | Overhead |
| **Isolation** | Process level | Hardware level |
| **Portability** | High | Medium |

---

## 📝 Dockerfile Fundamentals

### Basic Structure
```dockerfile
# Base image
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Copy dependency files first (cache optimization)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Expose port
EXPOSE 3000

# Set environment variables
ENV NODE_ENV=production

# Define startup command
CMD ["node", "dist/main.js"]
```

### Multi-Stage Build (Production-Ready)
```dockerfile
# ============ BUILD STAGE ============
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# ============ PRODUCTION STAGE ============
FROM node:20-alpine AS production

WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup

# Copy only necessary files from builder
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./

# Switch to non-root user
USER appuser

EXPOSE 3000

CMD ["node", "dist/main.js"]
```

---

## 📜 Dockerfile Instructions Reference

```dockerfile
# FROM - Base image
FROM node:20-alpine

# WORKDIR - Working directory
WORKDIR /app

# COPY - Copy files from host to container
COPY source dest
COPY --chown=user:group source dest

# ADD - Like COPY but can extract tar and fetch URLs
ADD archive.tar.gz /app/
ADD https://example.com/file.txt /app/

# RUN - Execute commands during build
RUN npm install
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*

# ENV - Environment variables
ENV NODE_ENV=production
ENV PORT=3000

# ARG - Build-time variables
ARG VERSION=1.0.0
ARG BUILD_DATE

# EXPOSE - Document which ports the container listens on
EXPOSE 3000
EXPOSE 80/tcp 443/tcp

# USER - Set the user for RUN, CMD, ENTRYPOINT
USER appuser:appgroup

# CMD - Default command (can be overridden)
CMD ["node", "app.js"]
CMD node app.js

# ENTRYPOINT - Main executable (harder to override)
ENTRYPOINT ["node"]
CMD ["app.js"]  # Arguments to ENTRYPOINT

# VOLUME - Create mount point
VOLUME /data

# HEALTHCHECK - Container health monitoring
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# LABEL - Metadata
LABEL version="1.0"
LABEL maintainer="dev@example.com"
```

---

## 🔧 Essential Docker Commands

### Container Lifecycle
```bash
# Run container
docker run [options] image [command]
docker run -d --name myapp -p 3000:3000 myimage

# Common run options
-d                    # Detached mode (background)
-it                   # Interactive with TTY
--name NAME           # Container name
-p 8080:3000          # Port mapping (host:container)
-v /host:/container   # Volume mount
-e VAR=value          # Environment variable
--rm                  # Remove container when it stops
--network network     # Connect to network

# Container management
docker ps                    # List running containers
docker ps -a                 # List all containers
docker stop container        # Stop container
docker start container       # Start stopped container
docker restart container     # Restart container
docker rm container          # Remove container
docker rm -f container       # Force remove running container

# Execute in running container
docker exec -it container bash
docker exec container ls /app

# View logs
docker logs container
docker logs -f container     # Follow logs
docker logs --tail 100 container

# Inspect container
docker inspect container
docker stats                 # Resource usage
```

### Image Management
```bash
# Build image
docker build -t name:tag .
docker build -t myapp:v1.0 -f Dockerfile.prod .

# List images
docker images
docker images -a             # Include intermediate

# Remove images
docker rmi image
docker image prune           # Remove dangling images
docker image prune -a        # Remove all unused

# Tag and push
docker tag image:tag registry/image:tag
docker push registry/image:tag

# Pull image
docker pull image:tag
```

### Volume Management
```bash
# Create volume
docker volume create mydata

# List volumes
docker volume ls

# Inspect volume
docker volume inspect mydata

# Remove volume
docker volume rm mydata
docker volume prune          # Remove unused

# Use volume
docker run -v mydata:/app/data myimage
docker run -v $(pwd):/app myimage  # Bind mount
```

### Network Management
```bash
# List networks
docker network ls

# Create network
docker network create mynetwork

# Connect container to network
docker network connect mynetwork container

# Disconnect
docker network disconnect mynetwork container

# Inspect
docker network inspect mynetwork
```

---

## 🏆 Dockerfile Best Practices

### ✅ Layer Optimization
```dockerfile
# ❌ BAD - Invalidates cache frequently
COPY . .
RUN npm install

# ✅ GOOD - Dependencies cached separately
COPY package*.json ./
RUN npm ci
COPY . .
```

### ✅ Minimize Image Size
```dockerfile
# ❌ BAD - Large image with dev dependencies
FROM node:20
RUN npm install

# ✅ GOOD - Alpine + production only
FROM node:20-alpine
RUN npm ci --only=production
```

### ✅ Security Practices
```dockerfile
# ✅ Run as non-root
RUN adduser -D appuser
USER appuser

# ✅ Use specific versions
FROM node:20.10.0-alpine3.19

# ✅ Don't include secrets
# Use build args or runtime env vars instead
```

### ✅ Reduce Build Context
```
# .dockerignore
node_modules
npm-debug.log
Dockerfile*
.git
.gitignore
.env
*.md
dist
coverage
.nyc_output
```

---

## 🏥 Health Checks

```dockerfile
# HTTP health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Custom script
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD /app/healthcheck.sh
```

```typescript
// Express health endpoint
app.get('/health', (req, res) => {
  const healthcheck = {
    uptime: process.uptime(),
    message: 'OK',
    timestamp: Date.now()
  };
  try {
    // Check DB connection, external services, etc.
    res.send(healthcheck);
  } catch (error) {
    healthcheck.message = error;
    res.status(503).send(healthcheck);
  }
});
```

---

## 📋 Common Patterns

### Node.js Production
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
USER nodejs
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

### Python Production
```dockerfile
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

FROM python:3.11-slim
WORKDIR /app
RUN useradd -m appuser
COPY --from=builder /root/.local /home/appuser/.local
COPY . .
USER appuser
ENV PATH=/home/appuser/.local/bin:$PATH
EXPOSE 8000
CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:8000", "app:app"]
```

### Go Production
```dockerfile
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /app/main

FROM scratch
COPY --from=builder /app/main /main
EXPOSE 8080
ENTRYPOINT ["/main"]
```

---

## 🔍 Debugging Containers

```bash
# Enter running container
docker exec -it container /bin/sh

# View process tree
docker top container

# Copy files from container
docker cp container:/path/file ./local

# View filesystem changes
docker diff container

# Export container filesystem
docker export container > container.tar

# Check events
docker events --filter container=name
```

---

## 📚 Related

- [[Programming/Software Engineering/DevOps/Docker Compose|Docker Compose]] - Multi-container applications
- [[Programming/Software Engineering/DevOps/Kubernetes|Kubernetes]] - Container orchestration

---

← [[Programming/Software Engineering/DevOps/_Index|Back to DevOps]]
