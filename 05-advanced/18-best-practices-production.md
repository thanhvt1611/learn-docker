# Bài 18: Best Practices cho Production

## 📋 Mục Tiêu

Hướng dẫn các best practices để deploy Docker containers lên production một cách an toàn và hiệu quả.

---

## 🏗️ Image Best Practices

### 1. Sử Dụng Official Base Images

```dockerfile
# ✅ Official images
FROM node:18-alpine
FROM python:3.11-slim
FROM postgres:15-alpine

# ❌ Unofficial images
FROM random-user/node
FROM unknown/python
```

### 2. Specify Exact Versions

```dockerfile
# ✅ Exact version
FROM node:18.19.0-alpine3.19

# ⚠️ Major version only
FROM node:18-alpine

# ❌ Latest tag
FROM node:latest
```

### 3. Use Minimal Base Images

```dockerfile
# ✅ Alpine (5MB)
FROM node:18-alpine

# ⚠️ Slim (100MB)
FROM node:18-slim

# ❌ Full (900MB)
FROM node:18
```

### 4. Multi-Stage Builds

```dockerfile
# ✅ Multi-stage build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/server.js"]

# Image size: ~150MB

# ❌ Single stage
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
CMD ["node", "dist/server.js"]

# Image size: ~500MB
```

### 5. Optimize Layer Caching

```dockerfile
# ✅ Dependencies trước, source code sau
FROM node:18-alpine
WORKDIR /app

# Layer này ít thay đổi
COPY package*.json ./
RUN npm ci --only=production

# Layer này thường thay đổi
COPY . .

CMD ["node", "server.js"]

# ❌ Copy tất cả trước
FROM node:18-alpine
WORKDIR /app
COPY . .  # Mỗi lần code thay đổi, phải rebuild tất cả
RUN npm ci --only=production
CMD ["node", "server.js"]
```

### 6. Minimize Layers

```dockerfile
# ✅ Combine RUN commands
RUN apt-get update && \
    apt-get install -y \
        package1 \
        package2 \
        package3 && \
    rm -rf /var/lib/apt/lists/*

# ❌ Multiple RUN commands
RUN apt-get update
RUN apt-get install -y package1
RUN apt-get install -y package2
RUN apt-get install -y package3
```

### 7. Remove Unnecessary Files

```dockerfile
# ✅ Clean up trong cùng layer
RUN apt-get update && \
    apt-get install -y build-essential && \
    npm install && \
    apt-get purge -y build-essential && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# ❌ Clean up ở layer khác (không giảm size)
RUN apt-get update
RUN apt-get install -y build-essential
RUN npm install
RUN apt-get purge -y build-essential  # Không hiệu quả
```

---

## 🔒 Security Best Practices

### 1. Non-Root User

```dockerfile
# ✅ Run as non-root user
FROM node:18-alpine

WORKDIR /app

# Create user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

COPY --chown=nodejs:nodejs . .

USER nodejs

CMD ["node", "server.js"]

# ❌ Run as root
FROM node:18-alpine
WORKDIR /app
COPY . .
CMD ["node", "server.js"]  # Runs as root
```

### 2. Không Lưu Secrets Trong Image

```dockerfile
# ❌ KHÔNG làm thế này
ENV API_KEY=secret123
ENV DATABASE_PASSWORD=password123

# ✅ Sử dụng runtime environment variables
# docker run -e API_KEY=$API_KEY myapp

# ✅ Hoặc Docker secrets (Swarm mode)
# docker secret create api_key api_key.txt
```

### 3. Scan Images for Vulnerabilities

```bash
# Sử dụng Docker Scout
docker scout cves myapp:latest

# Sử dụng Trivy
trivy image myapp:latest

# Sử dụng Snyk
snyk container test myapp:latest
```

### 4. Use Read-Only Filesystem

```yaml
# docker-compose.yml
services:
  app:
    image: myapp
    read_only: true
    tmpfs:
      - /tmp
      - /var/run
```

### 5. Drop Capabilities

```yaml
services:
  app:
    image: myapp
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE  # Only if needed
```

---

## 🚀 Performance Best Practices

### 1. Health Checks

```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY . .

HEALTHCHECK --interval=30s \
            --timeout=3s \
            --start-period=10s \
            --retries=3 \
  CMD node healthcheck.js

CMD ["node", "server.js"]
```

**healthcheck.js:**
```javascript
const http = require('http');

const options = {
  host: 'localhost',
  port: 3000,
  path: '/health',
  timeout: 2000
};

http.get(options, (res) => {
  process.exit(res.statusCode === 200 ? 0 : 1);
}).on('error', () => {
  process.exit(1);
});
```

### 2. Resource Limits

```yaml
# docker-compose.yml
services:
  app:
    image: myapp
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G
```

### 3. Restart Policies

```yaml
services:
  app:
    image: myapp
    restart: unless-stopped  # Production
    
  dev:
    image: myapp
    restart: "no"  # Development
```

### 4. Logging Configuration

```yaml
services:
  app:
    image: myapp
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
        labels: "production"
```

### 5. Use BuildKit

```bash
# Enable BuildKit
export DOCKER_BUILDKIT=1

# Build với BuildKit
docker build -t myapp .

# BuildKit features:
# - Parallel builds
# - Better caching
# - Secrets support
# - SSH forwarding
```

---

## 🌐 Networking Best Practices

### 1. Network Segmentation

```yaml
version: '3.8'

services:
  web:
    image: nginx
    networks:
      - frontend
  
  api:
    image: myapi
    networks:
      - frontend
      - backend
  
  db:
    image: postgres
    networks:
      - backend  # Không expose ra frontend

networks:
  frontend:
  backend:
    internal: true  # Không có external access
```

### 2. Least Privilege Ports

```yaml
services:
  # ✅ Chỉ expose ports cần thiết
  web:
    image: nginx
    ports:
      - "80:80"
      - "443:443"
  
  # ✅ Internal services không expose ports
  api:
    image: myapi
    # Không có ports, chỉ access qua network
  
  db:
    image: postgres
    # Không expose port 5432 ra ngoài
```

### 3. Use DNS Names

```yaml
services:
  api:
    environment:
      # ✅ Sử dụng service names
      DATABASE_URL: postgresql://postgres:secret@db:5432/mydb
      REDIS_URL: redis://redis:6379
      
      # ❌ Hardcode IPs
      # DATABASE_URL: postgresql://postgres:secret@172.18.0.2:5432/mydb
```

---

## 💾 Storage Best Practices

### 1. Named Volumes

```yaml
# ✅ Named volumes
services:
  db:
    image: postgres
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
    driver: local

# ❌ Anonymous volumes
services:
  db:
    image: postgres
    volumes:
      - /var/lib/postgresql/data
```

### 2. Backup Strategy

```bash
# Backup volume
docker run --rm \
  -v pgdata:/data \
  -v $(pwd)/backups:/backup \
  alpine \
  tar czf /backup/pgdata-$(date +%Y%m%d).tar.gz -C /data .

# Restore volume
docker run --rm \
  -v pgdata:/data \
  -v $(pwd)/backups:/backup \
  alpine \
  tar xzf /backup/pgdata-20240101.tar.gz -C /data
```

### 3. Read-Only Mounts

```yaml
services:
  web:
    image: nginx
    volumes:
      # ✅ Config files read-only
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
      
      # ✅ Logs writable
      - nginx-logs:/var/log/nginx
```

---

## 🔄 CI/CD Best Practices

### 1. Automated Builds

```yaml
# .github/workflows/docker.yml
name: Docker Build

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: |
            myuser/myapp:latest
            myuser/myapp:${{ github.sha }}
          cache-from: type=registry,ref=myuser/myapp:latest
          cache-to: type=inline
```

### 2. Image Tagging Strategy

```bash
# ✅ Semantic versioning
docker tag myapp:latest myapp:1.2.3
docker tag myapp:latest myapp:1.2
docker tag myapp:latest myapp:1

# ✅ Git commit SHA
docker tag myapp:latest myapp:${GIT_COMMIT}

# ✅ Environment
docker tag myapp:latest myapp:production
docker tag myapp:latest myapp:staging

# ❌ Chỉ dùng latest
docker tag myapp:latest
```

### 3. Testing in CI

```yaml
# .github/workflows/test.yml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build test image
        run: docker build -t myapp:test .
      
      - name: Run tests
        run: |
          docker run --rm myapp:test npm test
          docker run --rm myapp:test npm run lint
      
      - name: Security scan
        run: |
          docker run --rm \
            -v /var/run/docker.sock:/var/run/docker.sock \
            aquasec/trivy image myapp:test
```

---

## 📊 Monitoring Best Practices

### 1. Container Metrics

```yaml
version: '3.8'

services:
  # Application
  app:
    image: myapp
    labels:
      - "prometheus.scrape=true"
      - "prometheus.port=3000"
      - "prometheus.path=/metrics"
  
  # Prometheus
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    ports:
      - "9090:9090"
  
  # Grafana
  grafana:
    image: grafana/grafana
    environment:
      GF_SECURITY_ADMIN_PASSWORD: ${GRAFANA_PASSWORD}
    volumes:
      - grafana-data:/var/lib/grafana
    ports:
      - "3000:3000"
  
  # cAdvisor
  cadvisor:
    image: gcr.io/cadvisor/cadvisor
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    ports:
      - "8080:8080"

volumes:
  prometheus-data:
  grafana-data:
```

### 2. Centralized Logging

```yaml
services:
  app:
    image: myapp
    logging:
      driver: "fluentd"
      options:
        fluentd-address: localhost:24224
        tag: myapp
  
  fluentd:
    image: fluent/fluentd
    volumes:
      - ./fluentd.conf:/fluentd/etc/fluent.conf
    ports:
      - "24224:24224"
```

---

## 🎯 Production Deployment Checklist

### Pre-Deployment

- [ ] Images được scan for vulnerabilities
- [ ] Tests pass trong CI/CD
- [ ] Health checks implemented
- [ ] Resource limits configured
- [ ] Logging configured
- [ ] Monitoring setup
- [ ] Backup strategy in place
- [ ] Secrets managed securely
- [ ] Documentation updated

### Deployment

- [ ] Use specific image tags (không dùng `latest`)
- [ ] Deploy to staging trước
- [ ] Run smoke tests
- [ ] Monitor metrics và logs
- [ ] Có rollback plan

### Post-Deployment

- [ ] Verify health checks
- [ ] Check resource usage
- [ ] Monitor error rates
- [ ] Review logs
- [ ] Update documentation

---

## 🏭 Production-Ready Example

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:1.25-alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - nginx-logs:/var/log/nginx
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

  app:
    image: myapp:1.2.3  # Specific version
    restart: unless-stopped
    environment:
      NODE_ENV: production
    env_file:
      - .env.production
    networks:
      - frontend
      - backend
    depends_on:
      db:
        condition: service_healthy
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
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    read_only: true
    tmpfs:
      - /tmp
    cap_drop:
      - ALL

  db:
    image: postgres:15.5-alpine  # Specific version
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    volumes:
      - postgres-data:/var/lib/postgresql/data
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

secrets:
  db_password:
    file: ./secrets/db_password.txt

volumes:
  postgres-data:
    driver: local
  nginx-logs:
    driver: local

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true
```

---

## 💡 Key Takeaways

1. **Security First**: Non-root users, no secrets in images, scan vulnerabilities
2. **Optimize Images**: Multi-stage builds, minimal base images, layer caching
3. **Production Ready**: Health checks, resource limits, restart policies
4. **Monitoring**: Metrics, logging, alerting
5. **Automation**: CI/CD, automated testing, automated deployments

---

## 📚 Bài Tiếp Theo

👉 [Bài 19: Security](./19-security.md)

---

**Happy Production! 🚀**

