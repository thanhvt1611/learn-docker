# Bài 15: Docker Compose Nâng Cao

## 📋 Mục Tiêu Bài Học

Sau bài học này, bạn sẽ:
- ✅ Sử dụng extends và override files
- ✅ Implement health checks và dependencies
- ✅ Multi-stage builds với Compose
- ✅ Profiles cho different environments
- ✅ Secrets management
- ✅ Production-ready configurations

---

## 🔄 Extends và Override Files

### docker-compose.override.yml

Docker Compose tự động merge `docker-compose.yml` và `docker-compose.override.yml`.

**docker-compose.yml** (Base configuration):
```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
  
  api:
    build: ./api
    environment:
      NODE_ENV: production
```

**docker-compose.override.yml** (Development overrides):
```yaml
version: '3.8'

services:
  web:
    volumes:
      - ./html:/usr/share/nginx/html  # Mount local files
  
  api:
    volumes:
      - ./api:/app  # Hot reload
    environment:
      NODE_ENV: development  # Override to development
      DEBUG: "true"
    command: npm run dev  # Override command
```

```bash
# Tự động merge cả 2 files
docker compose up

# Kết quả:
# - web có volume mount
# - api có NODE_ENV=development, DEBUG=true, và npm run dev
```

### Multiple Compose Files

```bash
# Specify multiple files
docker compose -f docker-compose.yml -f docker-compose.prod.yml up

# Order matters: later files override earlier ones
docker compose \
  -f docker-compose.yml \
  -f docker-compose.override.yml \
  -f docker-compose.local.yml \
  up
```

### Ví Dụ: Environment-Specific Files

**docker-compose.yml** (Base):
```yaml
version: '3.8'

services:
  api:
    build: ./api
    environment:
      NODE_ENV: production
```

**docker-compose.dev.yml** (Development):
```yaml
version: '3.8'

services:
  api:
    volumes:
      - ./api:/app
    environment:
      NODE_ENV: development
      DEBUG: "true"
    command: npm run dev
```

**docker-compose.prod.yml** (Production):
```yaml
version: '3.8'

services:
  api:
    restart: unless-stopped
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '1'
          memory: 1G
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

```bash
# Development
docker compose -f docker-compose.yml -f docker-compose.dev.yml up

# Production
docker compose -f docker-compose.yml -f docker-compose.prod.yml up
```

---

## 🏥 Health Checks

### Định Nghĩa Health Checks

```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost/"]
      interval: 30s      # Check mỗi 30s
      timeout: 3s        # Timeout sau 3s
      retries: 3         # Retry 3 lần
      start_period: 10s  # Grace period 10s
  
  api:
    build: ./api
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 10s
  
  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

### Disable Health Check

```yaml
services:
  web:
    image: nginx
    healthcheck:
      disable: true
```

---

## 🔗 Advanced Dependencies

### depends_on với Conditions

```yaml
version: '3.8'

services:
  web:
    image: nginx
    depends_on:
      api:
        condition: service_healthy  # Wait until healthy
  
  api:
    build: ./api
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started  # Just wait for start
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 10s
      timeout: 3s
      retries: 3
  
  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5
  
  redis:
    image: redis:alpine
```

**Conditions:**
- `service_started`: Service đã start (default)
- `service_healthy`: Service healthy (cần healthcheck)
- `service_completed_successfully`: Service exit với code 0

---

## 🎭 Profiles

Profiles cho phép bạn chạy một subset của services.

### Định Nghĩa Profiles

```yaml
version: '3.8'

services:
  # Core services (luôn chạy)
  db:
    image: postgres:15
  
  api:
    build: ./api
    depends_on:
      - db
  
  web:
    build: ./web
    depends_on:
      - api
  
  # Debug tools (chỉ khi cần)
  adminer:
    image: adminer
    profiles: ["debug"]
    ports:
      - "8080:8080"
  
  # Testing tools
  test:
    build: ./api
    profiles: ["test"]
    command: npm test
  
  # Development tools
  docs:
    image: nginx:alpine
    profiles: ["dev", "docs"]
    volumes:
      - ./docs:/usr/share/nginx/html
    ports:
      - "8000:80"
```

### Sử Dụng Profiles

```bash
# Chỉ chạy core services (db, api, web)
docker compose up

# Chạy với debug profile
docker compose --profile debug up

# Chạy với multiple profiles
docker compose --profile dev --profile debug up

# Chạy specific service với profile
docker compose run --rm test
```

---

## 🔐 Secrets Management

### Docker Secrets (Swarm Mode)

```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    secrets:
      - db_password
      - db_user
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
      POSTGRES_USER_FILE: /run/secrets/db_user

secrets:
  db_password:
    file: ./secrets/db_password.txt
  db_user:
    file: ./secrets/db_user.txt
```

### Alternative: Environment Files

```yaml
version: '3.8'

services:
  api:
    build: ./api
    env_file:
      - .env.common
      - .env.secrets  # Không commit file này
```

**.env.secrets:**
```bash
DATABASE_PASSWORD=super-secret-password
API_KEY=secret-api-key
JWT_SECRET=jwt-secret-key
```

**.gitignore:**
```
.env.secrets
.env.local
secrets/
```

---

## 🏗️ Build Configuration

### Build Arguments

```yaml
version: '3.8'

services:
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
      args:
        NODE_VERSION: 18
        BUILD_DATE: ${BUILD_DATE}
        GIT_COMMIT: ${GIT_COMMIT}
      target: production  # Multi-stage build target
      cache_from:
        - myapp/api:latest
```

**Dockerfile:**
```dockerfile
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}-alpine AS base

ARG BUILD_DATE
ARG GIT_COMMIT
LABEL build_date=${BUILD_DATE}
LABEL git_commit=${GIT_COMMIT}

# ... rest of Dockerfile
```

### Build Context

```yaml
services:
  api:
    build:
      context: ./api
      dockerfile: ../Dockerfile.api  # Dockerfile ở ngoài context
      
  web:
    build:
      context: .  # Root directory
      dockerfile: ./web/Dockerfile
```

---

## 🌐 Advanced Networking

### Custom Networks

```yaml
version: '3.8'

services:
  web:
    image: nginx
    networks:
      - frontend
  
  api:
    build: ./api
    networks:
      - frontend
      - backend
  
  db:
    image: postgres:15
    networks:
      - backend

networks:
  frontend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
  
  backend:
    driver: bridge
    internal: true  # No external access
```

### Network Aliases

```yaml
services:
  api:
    image: myapi
    networks:
      backend:
        aliases:
          - api-server
          - backend-api
```

### External Networks

```yaml
services:
  web:
    image: nginx
    networks:
      - existing-network

networks:
  existing-network:
    external: true
    name: my-pre-existing-network
```

---

## 💾 Advanced Volumes

### Volume Configuration

```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    volumes:
      - type: volume
        source: pgdata
        target: /var/lib/postgresql/data
        volume:
          nocopy: true  # Don't copy data from container
      
      - type: bind
        source: ./config
        target: /etc/config
        read_only: true
        bind:
          propagation: shared

volumes:
  pgdata:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /path/on/host
```

### External Volumes

```yaml
services:
  db:
    image: postgres:15
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
    external: true
    name: my-existing-volume
```

---

## 🎯 Ví Dụ Production-Ready

### Full-Stack Application

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - nginx-logs:/var/log/nginx
    depends_on:
      web:
        condition: service_healthy
    networks:
      - frontend
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost/health"]
      interval: 30s
      timeout: 3s
      retries: 3
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  # Frontend (React)
  web:
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
      args:
        REACT_APP_API_URL: ${API_URL}
    restart: unless-stopped
    environment:
      NODE_ENV: production
    networks:
      - frontend
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost/"]
      interval: 30s
      timeout: 3s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M

  # Backend API (Node.js)
  api:
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
    restart: unless-stopped
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://postgres:${DB_PASSWORD}@postgres:5432/${DB_NAME}
      REDIS_URL: redis://redis:6379
      JWT_SECRET: ${JWT_SECRET}
    env_file:
      - .env.secrets
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - frontend
      - backend
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 10s
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 1G
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  # PostgreSQL Database
  postgres:
    image: postgres:15-alpine
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./postgres/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  # Redis Cache
  redis:
    image: redis:alpine
    restart: unless-stopped
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis-data:/data
    networks:
      - backend
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

  # Database Admin (only with debug profile)
  adminer:
    image: adminer
    restart: unless-stopped
    profiles: ["debug"]
    ports:
      - "8080:8080"
    networks:
      - backend
    depends_on:
      - postgres

  # Backup Service
  backup:
    image: postgres:15-alpine
    profiles: ["backup"]
    volumes:
      - postgres-data:/var/lib/postgresql/data:ro
      - ./backups:/backups
    networks:
      - backend
    command: >
      sh -c "pg_dump -h postgres -U ${DB_USER} ${DB_NAME} > /backups/backup-$$(date +%Y%m%d-%H%M%S).sql"

volumes:
  postgres-data:
    driver: local
  redis-data:
    driver: local
  nginx-logs:
    driver: local

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No external access
```

**.env:**
```bash
# Database
DB_NAME=myapp
DB_USER=postgres
DB_PASSWORD=change-me-in-production

# Redis
REDIS_PASSWORD=change-me-in-production

# API
API_URL=https://api.example.com
JWT_SECRET=change-me-in-production
```

**Usage:**
```bash
# Production
docker compose up -d

# With debug tools
docker compose --profile debug up -d

# Run backup
docker compose run --rm backup

# View logs
docker compose logs -f api

# Scale API
docker compose up -d --scale api=3
```

---

## 🔧 Useful Patterns

### Init Containers

```yaml
services:
  # Run migrations before starting API
  migrate:
    build: ./api
    command: npm run migrate
    depends_on:
      db:
        condition: service_healthy
    networks:
      - backend
  
  api:
    build: ./api
    depends_on:
      migrate:
        condition: service_completed_successfully
    networks:
      - backend
```

### Sidecar Pattern

```yaml
services:
  app:
    build: ./app
    networks:
      - app-network
  
  # Logging sidecar
  log-collector:
    image: fluentd
    volumes:
      - ./logs:/logs
    networks:
      - app-network
  
  # Monitoring sidecar
  metrics:
    image: prometheus/node-exporter
    networks:
      - app-network
```

---

## 💡 Best Practices

### 1. Use Health Checks

```yaml
# ✅ Always implement health checks
services:
  api:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
```

### 2. Resource Limits

```yaml
# ✅ Set resource limits
services:
  api:
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 1G
```

### 3. Logging Configuration

```yaml
# ✅ Configure logging
services:
  api:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

### 4. Restart Policies

```yaml
# ✅ Use appropriate restart policies
services:
  api:
    restart: unless-stopped  # Production
  
  dev:
    restart: "no"  # Development
```

### 5. Network Segmentation

```yaml
# ✅ Separate frontend and backend
networks:
  frontend:
  backend:
    internal: true  # No external access
```

---

## 📚 Tóm Tắt

### Advanced Features

1. **Override Files**: Environment-specific configurations
2. **Health Checks**: Monitor service health
3. **Profiles**: Conditional service execution
4. **Secrets**: Secure sensitive data
5. **Advanced Networking**: Network segmentation
6. **Resource Limits**: Control resource usage

### Production Checklist

- ✅ Health checks cho tất cả services
- ✅ Resource limits
- ✅ Restart policies
- ✅ Logging configuration
- ✅ Network segmentation
- ✅ Secrets management
- ✅ Backup strategy

---

## 📚 Module Tiếp Theo

Chúc mừng! Bạn đã hoàn thành **Module 4: Docker Compose**.

Trong module tiếp theo, chúng ta sẽ học về **Thực Hành và Nâng Cao** với production best practices, security, và troubleshooting.

👉 [Module 5: Thực Hành và Nâng Cao](../05-advanced/16-bai-tap-thuc-hanh.md)

---

## 🔗 Tài Liệu Tham Khảo

- [Compose File Reference](https://docs.docker.com/compose/compose-file/)
- [Compose Profiles](https://docs.docker.com/compose/profiles/)
- [Compose Secrets](https://docs.docker.com/compose/use-secrets/)

---

**Happy Advanced Composing! 🎼✨**

