# Bài 09: Best Practices Dockerfile

## 📋 Mục Tiêu Bài Học

Sau bài học này, bạn sẽ:
- ✅ Viết Dockerfile tối ưu và bảo mật
- ✅ Tận dụng build cache hiệu quả
- ✅ Giảm image size tối đa
- ✅ Áp dụng security best practices

---

## 🎯 Nguyên Tắc Chung

### 1. Use Official Base Images

```dockerfile
# ❌ Không nên - unofficial image
FROM random-user/node

# ✅ Nên - official image
FROM node:18-alpine

# ✅ Tốt nhất - specific version
FROM node:18.17.0-alpine3.18
```

### 2. Use Specific Tags

```dockerfile
# ❌ Không predictable
FROM node:latest

# ⚠️ Tốt hơn nhưng vẫn có thể thay đổi
FROM node:18

# ✅ Tốt nhất - specific version
FROM node:18.17.0-alpine3.18
```

### 3. Use Minimal Base Images

```dockerfile
# ❌ Quá lớn (993MB)
FROM node:18

# ⚠️ Tốt hơn (170MB)
FROM node:18-alpine

# ✅ Tốt nhất cho production
FROM node:18-alpine3.18
```

---

## 🚀 Tối Ưu Build Cache

### 1. Order Instructions Properly

```dockerfile
# ❌ Cache bị invalidate thường xuyên
FROM node:18-alpine
COPY . /app
RUN npm install
CMD ["node", "app.js"]

# ✅ Tận dụng cache tốt
FROM node:18-alpine
WORKDIR /app

# Copy dependencies files first
COPY package*.json ./
RUN npm ci

# Copy source code last (thay đổi thường xuyên)
COPY . .

CMD ["node", "app.js"]
```

**Giải thích:**
- `package.json` ít thay đổi → cache được giữ lâu
- Source code thay đổi thường xuyên → chỉ rebuild layer này

### 2. Combine RUN Commands

```dockerfile
# ❌ Nhiều layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git
RUN apt-get clean

# ✅ Một layer, cleanup ngay
RUN apt-get update && \
    apt-get install -y \
        curl \
        git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### 3. Use .dockerignore

```
# .dockerignore
node_modules
npm-debug.log
.git
.env
*.md
.vscode
coverage
dist
build
```

---

## 📦 Giảm Image Size

### 1. Multi-Stage Builds

```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]
```

### 2. Remove Unnecessary Files

```dockerfile
# ❌ Giữ cache và temp files
RUN apt-get update && \
    apt-get install -y curl

# ✅ Cleanup trong cùng layer
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

### 3. Use --no-install-recommends

```dockerfile
# ❌ Install tất cả recommended packages
RUN apt-get install -y python3

# ✅ Chỉ install essential packages
RUN apt-get update && \
    apt-get install -y --no-install-recommends python3 && \
    rm -rf /var/lib/apt/lists/*
```

### 4. Minimize Layers

```dockerfile
# ❌ Nhiều layers
COPY file1.txt /app/
COPY file2.txt /app/
COPY file3.txt /app/

# ✅ Một layer
COPY file1.txt file2.txt file3.txt /app/
```

---

## 🔒 Security Best Practices

### 1. Don't Run as Root

```dockerfile
# ❌ Chạy với root user
FROM node:18-alpine
COPY . /app
CMD ["node", "app.js"]

# ✅ Tạo và sử dụng non-root user
FROM node:18-alpine

# Tạo user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Set ownership
WORKDIR /app
COPY --chown=nodejs:nodejs . .

# Switch to user
USER nodejs

CMD ["node", "app.js"]
```

### 2. Use COPY Instead of ADD

```dockerfile
# ❌ ADD có thể extract tar và download URLs
ADD archive.tar.gz /app/

# ✅ COPY an toàn hơn cho local files
COPY archive.tar.gz /app/
```

### 3. Don't Store Secrets in Images

```dockerfile
# ❌ KHÔNG BAO GIỜ làm thế này
ENV API_KEY=secret123
COPY .env /app/

# ✅ Sử dụng build secrets hoặc runtime env
# Build time:
RUN --mount=type=secret,id=api_key \
    API_KEY=$(cat /run/secrets/api_key) npm install

# Runtime:
docker run -e API_KEY=secret123 myapp
```

### 4. Scan Images for Vulnerabilities

```bash
# Sử dụng Docker Scout
docker scout cves myapp:latest

# Hoặc Trivy
trivy image myapp:latest

# Hoặc Snyk
snyk container test myapp:latest
```

### 5. Use Specific Versions

```dockerfile
# ❌ Không cụ thể
RUN npm install express

# ✅ Lock versions
COPY package-lock.json ./
RUN npm ci
```

---

## 📝 Metadata và Documentation

### 1. Use LABEL

```dockerfile
LABEL maintainer="your-email@example.com"
LABEL version="1.0.0"
LABEL description="My awesome application"
LABEL org.opencontainers.image.source="https://github.com/user/repo"
LABEL org.opencontainers.image.licenses="MIT"
```

### 2. Document Exposed Ports

```dockerfile
# Document ports
EXPOSE 3000
EXPOSE 8080

# Add comment
# Port 3000: HTTP API
# Port 8080: Metrics
```

### 3. Add Comments

```dockerfile
# Base image
FROM node:18-alpine

# Install dependencies
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Copy application
COPY . .

# Run as non-root user
USER node

# Start application
CMD ["node", "server.js"]
```

---

## 🎨 Dockerfile Templates

### Node.js Production Template

```dockerfile
# ============================================
# Build Stage
# ============================================
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy source code
COPY . .

# Build application
RUN npm run build

# ============================================
# Production Stage
# ============================================
FROM node:18-alpine

# Install dumb-init for proper signal handling
RUN apk add --no-cache dumb-init

# Create app directory
WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Copy package files
COPY --chown=nodejs:nodejs package*.json ./

# Install production dependencies only
RUN npm ci --only=production && \
    npm cache clean --force

# Copy built application from builder
COPY --chown=nodejs:nodejs --from=builder /app/dist ./dist

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD node healthcheck.js

# Use dumb-init to handle signals properly
ENTRYPOINT ["dumb-init", "--"]

# Start application
CMD ["node", "dist/index.js"]
```

### Python Production Template

```dockerfile
# ============================================
# Build Stage
# ============================================
FROM python:3.11-slim AS builder

WORKDIR /app

# Install build dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        gcc \
        python3-dev && \
    rm -rf /var/lib/apt/lists/*

# Copy requirements
COPY requirements.txt .

# Install Python dependencies
RUN pip install --user --no-cache-dir -r requirements.txt

# ============================================
# Production Stage
# ============================================
FROM python:3.11-slim

WORKDIR /app

# Copy Python dependencies from builder
COPY --from=builder /root/.local /root/.local

# Copy application
COPY . .

# Create non-root user
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

# Make sure scripts in .local are usable
ENV PATH=/root/.local/bin:$PATH

# Expose port
EXPOSE 8000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD python healthcheck.py || exit 1

# Start application
CMD ["python", "app.py"]
```

### Go Production Template

```dockerfile
# ============================================
# Build Stage
# ============================================
FROM golang:1.21-alpine AS builder

# Install build dependencies
RUN apk add --no-cache git ca-certificates

WORKDIR /app

# Copy go mod files
COPY go.mod go.sum ./
RUN go mod download

# Copy source code
COPY . .

# Build binary
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
    go build -ldflags="-w -s" -o main .

# ============================================
# Production Stage
# ============================================
FROM alpine:latest

# Install ca-certificates for HTTPS
RUN apk --no-cache add ca-certificates

WORKDIR /root/

# Copy binary from builder
COPY --from=builder /app/main .

# Create non-root user
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup

USER appuser

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health || exit 1

# Start application
CMD ["./main"]
```

---

## 🔍 Linting và Validation

### 1. Use Hadolint

```bash
# Install hadolint
brew install hadolint  # macOS
# hoặc
docker pull hadolint/hadolint

# Lint Dockerfile
hadolint Dockerfile

# Hoặc dùng Docker
docker run --rm -i hadolint/hadolint < Dockerfile
```

### 2. Common Hadolint Rules

```dockerfile
# DL3006: Always tag the version of an image explicitly
FROM node:18-alpine  # ✅

# DL3008: Pin versions in apt-get install
RUN apt-get install -y curl=7.68.0-1  # ✅

# DL3009: Delete the apt-get lists after installing
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*  # ✅

# DL3015: Avoid additional packages by using --no-install-recommends
RUN apt-get install -y --no-install-recommends curl  # ✅

# DL3059: Multiple consecutive RUN instructions
# Combine them into one  # ✅
```

---

## 💡 Advanced Tips

### 1. Use BuildKit

```bash
# Enable BuildKit
export DOCKER_BUILDKIT=1

# Hoặc
docker buildx build -t myapp .
```

**Lợi ích:**
- Build nhanh hơn
- Better caching
- Parallel builds
- Build secrets support

### 2. Build Secrets

```dockerfile
# Sử dụng build secrets (BuildKit)
RUN --mount=type=secret,id=npm_token \
    NPM_TOKEN=$(cat /run/secrets/npm_token) npm install
```

```bash
# Build với secret
docker build --secret id=npm_token,src=.npmrc -t myapp .
```

### 3. Cache Mounts

```dockerfile
# Cache npm packages
RUN --mount=type=cache,target=/root/.npm \
    npm install

# Cache go modules
RUN --mount=type=cache,target=/go/pkg/mod \
    go mod download
```

### 4. SSH Mounts

```dockerfile
# Clone private repo
RUN --mount=type=ssh \
    git clone git@github.com:user/private-repo.git
```

---

## 📚 Checklist

### ✅ Security
- [ ] Không chạy với root user
- [ ] Không lưu secrets trong image
- [ ] Sử dụng official base images
- [ ] Scan vulnerabilities
- [ ] Pin package versions

### ✅ Performance
- [ ] Sử dụng multi-stage builds
- [ ] Tối ưu layer ordering
- [ ] Combine RUN commands
- [ ] Sử dụng .dockerignore
- [ ] Cleanup trong cùng layer

### ✅ Maintainability
- [ ] Sử dụng specific tags
- [ ] Thêm labels
- [ ] Thêm comments
- [ ] Document exposed ports
- [ ] Add health checks

---

## 📚 Tóm Tắt

### Key Takeaways

1. **Security**: Không run as root, không lưu secrets
2. **Size**: Multi-stage builds, alpine images, cleanup
3. **Cache**: Order instructions properly
4. **Quality**: Lint với hadolint, scan vulnerabilities
5. **Documentation**: Labels, comments, health checks

---

## 📚 Module Tiếp Theo

Chúc mừng! Bạn đã hoàn thành **Module 2: Docker Images**. 

Trong module tiếp theo, chúng ta sẽ học về **Docker Containers** - cách vận hành và quản lý containers.

👉 [Module 3: Docker Containers](../03-containers/10-quan-ly-containers.md)

---

## 🔗 Tài Liệu Tham Khảo

- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Hadolint](https://github.com/hadolint/hadolint)
- [Docker Security](https://docs.docker.com/engine/security/)

---

**Happy Building! 🚀**

