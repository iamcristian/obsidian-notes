---
tags:
  - software-engineering
  - devops
  - docker
  - docker-compose
created: 2026-01-02
status: 🔴
---
# 🐙 Docker Compose

> *"Define and run multi-container Docker applications."*

## 🎯 What is Docker Compose?

Docker Compose es una herramienta para definir y ejecutar aplicaciones multi-contenedor usando un archivo YAML declarativo.

---

## 📝 Basic docker-compose.yml

```yaml
# docker-compose.yml
version: '3.9'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
    depends_on:
      - db
      - redis
    volumes:
      - .:/app
      - /app/node_modules

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

---

## 🔧 Essential Commands

```bash
# Start services
docker-compose up              # Foreground
docker-compose up -d           # Detached (background)
docker-compose up --build      # Rebuild images

# Stop services
docker-compose down            # Stop and remove containers
docker-compose down -v         # Also remove volumes
docker-compose down --rmi all  # Also remove images

# View status
docker-compose ps              # List services
docker-compose logs            # View all logs
docker-compose logs -f app     # Follow specific service

# Execute commands
docker-compose exec app sh     # Shell into running container
docker-compose run app npm test # Run one-off command

# Scale services
docker-compose up -d --scale worker=3

# Restart
docker-compose restart
docker-compose restart app

# Build
docker-compose build
docker-compose build --no-cache
```

---

## 📋 Complete Service Configuration

```yaml
version: '3.9'

services:
  app:
    # Build configuration
    build:
      context: .
      dockerfile: Dockerfile
      args:
        - BUILD_ENV=production
      target: production

    # Or use existing image
    # image: myapp:latest

    # Container configuration
    container_name: myapp
    hostname: app
    restart: unless-stopped
    
    # Port mapping
    ports:
      - "3000:3000"
      - "9229:9229"  # Debug port
    
    # Environment
    environment:
      NODE_ENV: production
      LOG_LEVEL: info
    env_file:
      - .env
      - .env.production
    
    # Volumes
    volumes:
      - ./src:/app/src:ro              # Read-only bind mount
      - node_modules:/app/node_modules  # Named volume
      - type: bind
        source: ./config
        target: /app/config
        read_only: true
    
    # Networking
    networks:
      - frontend
      - backend
    
    # Dependencies
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    
    # Health check
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    
    # Resources
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
    
    # Logging
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
    
    # Labels
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.app.rule=Host(`app.local`)"
```

---

## 🌐 Networking

### Default Network
```yaml
# All services are on the same default network
# They can reach each other by service name

services:
  app:
    environment:
      DATABASE_URL: postgres://db:5432/mydb  # 'db' is the service name
      REDIS_URL: redis://redis:6379
  
  db:
    image: postgres:15
  
  redis:
    image: redis:7
```

### Custom Networks
```yaml
services:
  frontend:
    networks:
      - public
  
  api:
    networks:
      - public
      - private
  
  db:
    networks:
      - private

networks:
  public:
    driver: bridge
  private:
    driver: bridge
    internal: true  # No external access
```

### External Network
```yaml
services:
  app:
    networks:
      - existing_network

networks:
  existing_network:
    external: true
```

---

## 💾 Volume Patterns

### Named Volumes (Persistent Data)
```yaml
services:
  db:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
    driver: local
```

### Bind Mounts (Development)
```yaml
services:
  app:
    volumes:
      # Development: sync code
      - ./src:/app/src
      
      # Exclude node_modules (use volume instead)
      - /app/node_modules
      
      # Config files (read-only)
      - ./config:/app/config:ro
```

### tmpfs (Temporary/Sensitive)
```yaml
services:
  app:
    tmpfs:
      - /tmp
      - /app/temp
```

---

## 🔒 Environment Variables & Secrets

### Environment Files
```yaml
services:
  app:
    env_file:
      - .env           # Base config
      - .env.local     # Local overrides (gitignored)
```

```bash
# .env
DATABASE_URL=postgres://localhost:5432/mydb
REDIS_URL=redis://localhost:6379
LOG_LEVEL=debug
```

### Docker Secrets (Swarm Mode)
```yaml
services:
  app:
    secrets:
      - db_password
      - api_key

secrets:
  db_password:
    file: ./secrets/db_password.txt
  api_key:
    external: true
```

### Variable Substitution
```yaml
services:
  app:
    image: myapp:${TAG:-latest}
    ports:
      - "${PORT:-3000}:3000"
    environment:
      NODE_ENV: ${NODE_ENV:-development}
```

```bash
# Usage
TAG=v1.0.0 PORT=8080 docker-compose up
```

---

## 🏭 Development vs Production

### docker-compose.override.yml (Auto-loaded for development)
```yaml
# docker-compose.yml (base)
version: '3.9'
services:
  app:
    image: myapp:latest
    ports:
      - "3000:3000"
```

```yaml
# docker-compose.override.yml (development - auto-loaded)
version: '3.9'
services:
  app:
    build: .
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      NODE_ENV: development
    command: npm run dev
```

```yaml
# docker-compose.prod.yml (production)
version: '3.9'
services:
  app:
    restart: always
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 512M
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
```

```bash
# Development (uses docker-compose.override.yml automatically)
docker-compose up

# Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

---

## 🎯 Complete Example: Full Stack App

```yaml
# docker-compose.yml
version: '3.9'

services:
  # Frontend - React/Next.js
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://localhost:4000
    depends_on:
      - api
    networks:
      - frontend

  # Backend API - Node.js
  api:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "4000:4000"
    environment:
      - DATABASE_URL=postgres://user:password@db:5432/mydb
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - frontend
      - backend
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:4000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Background Worker
  worker:
    build:
      context: ./backend
      dockerfile: Dockerfile
    command: npm run worker
    environment:
      - DATABASE_URL=postgres://user:password@db:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    networks:
      - backend
    deploy:
      replicas: 2

  # PostgreSQL Database
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d mydb"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis Cache
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - backend

  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/certs:/etc/nginx/certs:ro
    depends_on:
      - frontend
      - api
    networks:
      - frontend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true

volumes:
  postgres_data:
  redis_data:
```

---

## 🐞 Debugging Tips

```bash
# View configuration after variable substitution
docker-compose config

# Validate compose file
docker-compose config --quiet

# View logs for specific service
docker-compose logs -f api

# Execute shell in running container
docker-compose exec api sh

# View resource usage
docker stats

# Check why container restarted
docker-compose logs --tail=100 api

# Rebuild without cache
docker-compose build --no-cache api
```

---

## 📋 Compose File Quick Reference

| Option | Description |
|--------|-------------|
| `build` | Build configuration |
| `image` | Image to use |
| `ports` | Port mappings |
| `volumes` | Volume mounts |
| `environment` | Environment variables |
| `env_file` | Environment file(s) |
| `depends_on` | Service dependencies |
| `networks` | Networks to join |
| `restart` | Restart policy |
| `healthcheck` | Health check config |
| `deploy` | Deployment config |
| `command` | Override default command |
| `entrypoint` | Override entrypoint |

---

← [[Programming/Software Engineering/DevOps/_Index|Back to DevOps]]
