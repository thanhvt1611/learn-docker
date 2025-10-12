# Bài 08: Multi-Stage Builds

## 📋 Mục Tiêu Bài Học

Sau bài học này, bạn sẽ:
- ✅ Hiểu Multi-Stage Builds là gì và tại sao cần
- ✅ Viết Dockerfile với multiple stages
- ✅ Tối ưu image size hiệu quả
- ✅ Áp dụng patterns phổ biến

---

## 🎯 Multi-Stage Builds Là Gì?

**Multi-Stage Builds** cho phép sử dụng nhiều `FROM` statements trong một Dockerfile, mỗi stage có thể sử dụng base image khác nhau.

### Vấn Đề Trước Khi Có Multi-Stage

```dockerfile
# ❌ Single-stage build (image rất lớn)
FROM node:18

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm install

# Copy source
COPY . .

# Build
RUN npm run build

# Image size: ~1.2GB
# Bao gồm: Node.js + npm + build tools + source code + dependencies
```

**Vấn đề:**
- Image quá lớn (chứa cả build tools không cần thiết)
- Chứa source code và dev dependencies
- Slow deployment và pull times

### Giải Pháp: Multi-Stage Builds

```dockerfile
# ✅ Multi-stage build (image nhỏ hơn nhiều)

# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./

# Image size: ~200MB
# Chỉ chứa: Runtime + production files
```

**Lợi ích:**
- ✅ Image size giảm 80-90%
- ✅ Không chứa build tools
- ✅ Không chứa source code
- ✅ Bảo mật hơn

---

## 🏗️ Cú Pháp Multi-Stage

### Basic Syntax

```dockerfile
# Stage 1
FROM image1 AS stage-name-1
# ... instructions

# Stage 2
FROM image2 AS stage-name-2
COPY --from=stage-name-1 /path/to/file /dest
# ... instructions

# Stage 3 (final)
FROM image3
COPY --from=stage-name-2 /path/to/file /dest
```

### Ví Dụ Đơn Giản

```dockerfile
# Build stage
FROM golang:1.21 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Production stage
FROM alpine:latest
COPY --from=builder /app/myapp /usr/local/bin/
CMD ["myapp"]
```

---

## 🎨 Patterns Phổ Biến

### Pattern 1: Build + Runtime (Node.js)

```dockerfile
# ============================================
# Build Stage
# ============================================
FROM node:18 AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install ALL dependencies (including devDependencies)
RUN npm ci

# Copy source code
COPY . .

# Build application
RUN npm run build

# ============================================
# Production Stage
# ============================================
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install ONLY production dependencies
RUN npm ci --only=production

# Copy built files from builder
COPY --from=builder /app/dist ./dist

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

USER nodejs

EXPOSE 3000

CMD ["node", "dist/index.js"]
```

**Kết quả:**
- Build stage: ~1.2GB (có dev tools)
- Final image: ~200MB (chỉ runtime)

### Pattern 2: Compile + Minimal Runtime (Go)

```dockerfile
# ============================================
# Build Stage
# ============================================
FROM golang:1.21-alpine AS builder

WORKDIR /app

# Copy go mod files
COPY go.mod go.sum ./
RUN go mod download

# Copy source code
COPY . .

# Build binary
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# ============================================
# Production Stage (Minimal)
# ============================================
FROM alpine:latest

RUN apk --no-cache add ca-certificates

WORKDIR /root/

# Copy binary from builder
COPY --from=builder /app/main .

EXPOSE 8080

CMD ["./main"]
```

**Kết quả:**
- Build stage: ~500MB
- Final image: ~10MB

### Pattern 3: Scratch Image (Ultra Minimal)

```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder

WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -ldflags '-extldflags "-static"' -o main .

# Production stage (scratch = empty image)
FROM scratch

COPY --from=builder /app/main /main
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

EXPOSE 8080

ENTRYPOINT ["/main"]
```

**Kết quả:**
- Final image: ~5MB (chỉ có binary!)

### Pattern 4: Test + Build + Production

```dockerfile
# ============================================
# Dependencies Stage
# ============================================
FROM node:18 AS dependencies

WORKDIR /app
COPY package*.json ./
RUN npm ci

# ============================================
# Test Stage
# ============================================
FROM dependencies AS test

COPY . .
RUN npm run test
RUN npm run lint

# ============================================
# Build Stage
# ============================================
FROM dependencies AS builder

COPY . .
RUN npm run build

# ============================================
# Production Stage
# ============================================
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY --from=builder /app/dist ./dist

USER node

CMD ["node", "dist/index.js"]
```

### Pattern 5: Multi-Language Build

```dockerfile
# ============================================
# Frontend Build (Node.js)
# ============================================
FROM node:18 AS frontend-builder

WORKDIR /app/frontend
COPY frontend/package*.json ./
RUN npm ci
COPY frontend/ ./
RUN npm run build

# ============================================
# Backend Build (Go)
# ============================================
FROM golang:1.21 AS backend-builder

WORKDIR /app/backend
COPY backend/go.* ./
RUN go mod download
COPY backend/ ./
RUN CGO_ENABLED=0 go build -o server

# ============================================
# Production Stage (Nginx + Backend)
# ============================================
FROM nginx:alpine

# Copy frontend build
COPY --from=frontend-builder /app/frontend/dist /usr/share/nginx/html

# Copy backend binary
COPY --from=backend-builder /app/backend/server /usr/local/bin/

# Copy nginx config
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

---

## 🎯 Ví Dụ Thực Tế

### React Application

```dockerfile
# ============================================
# Build Stage
# ============================================
FROM node:18-alpine AS builder

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci

# Copy source and build
COPY . .
RUN npm run build

# ============================================
# Production Stage
# ============================================
FROM nginx:alpine

# Copy built files
COPY --from=builder /app/build /usr/share/nginx/html

# Copy nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**nginx.conf:**
```nginx
server {
    listen 80;
    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;
    }
}
```

### Python Application

```dockerfile
# ============================================
# Build Stage
# ============================================
FROM python:3.11 AS builder

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# ============================================
# Production Stage
# ============================================
FROM python:3.11-slim

WORKDIR /app

# Copy dependencies from builder
COPY --from=builder /root/.local /root/.local

# Copy application
COPY . .

# Make sure scripts in .local are usable
ENV PATH=/root/.local/bin:$PATH

EXPOSE 8000

CMD ["python", "app.py"]
```

### Java Application

```dockerfile
# ============================================
# Build Stage
# ============================================
FROM maven:3.9-eclipse-temurin-17 AS builder

WORKDIR /app

# Copy pom.xml and download dependencies
COPY pom.xml .
RUN mvn dependency:go-offline

# Copy source and build
COPY src ./src
RUN mvn package -DskipTests

# ============================================
# Production Stage
# ============================================
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

# Copy jar from builder
COPY --from=builder /app/target/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## 🔧 Advanced Techniques

### 1. Named Stages

```dockerfile
# Đặt tên cho stages để dễ reference
FROM node:18 AS dependencies
FROM node:18 AS builder
FROM node:18 AS tester
FROM node:18-alpine AS production

# Copy từ named stage
COPY --from=dependencies /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
```

### 2. Build Specific Stage

```bash
# Build chỉ stage cụ thể
docker build --target builder -t myapp:builder .
docker build --target test -t myapp:test .
docker build --target production -t myapp:prod .

# Use case: CI/CD
# - Build test stage để run tests
# - Build production stage để deploy
```

### 3. Copy From External Image

```dockerfile
# Copy từ image khác (không phải stage)
FROM alpine:latest

# Copy từ nginx image
COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf

# Copy từ specific image
COPY --from=myregistry/myapp:v1.0 /app/config /config
```

### 4. Build Arguments Across Stages

```dockerfile
# Define ARG before FROM (global)
ARG NODE_VERSION=18

# Stage 1
FROM node:${NODE_VERSION} AS builder
ARG BUILD_ENV=production
RUN echo "Building for ${BUILD_ENV}"

# Stage 2
FROM node:${NODE_VERSION}-alpine
ARG BUILD_ENV=production
LABEL build_env=${BUILD_ENV}
```

---

## 📊 So Sánh Image Size

### Before Multi-Stage

```dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
CMD ["node", "dist/index.js"]
```

```bash
docker build -t myapp:single .
docker images myapp:single
# REPOSITORY   TAG      SIZE
# myapp        single   1.2GB
```

### After Multi-Stage

```dockerfile
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]
```

```bash
docker build -t myapp:multi .
docker images myapp:multi
# REPOSITORY   TAG      SIZE
# myapp        multi    180MB

# Tiết kiệm: 1.2GB - 180MB = 1020MB (85%)
```

---

## 💡 Best Practices

### 1. Order Stages Properly

```dockerfile
# ✅ Dependencies → Build → Test → Production
FROM node:18 AS dependencies
FROM node:18 AS builder
FROM node:18 AS test
FROM node:18-alpine AS production
```

### 2. Use Specific Base Images

```dockerfile
# ❌ Không cụ thể
FROM node AS builder

# ✅ Cụ thể
FROM node:18.17.0-alpine3.18 AS builder
```

### 3. Minimize Final Stage

```dockerfile
# ✅ Chỉ copy những gì cần thiết
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules

# ❌ Không copy toàn bộ
COPY --from=builder /app ./
```

### 4. Clean Up in Build Stage

```dockerfile
FROM node:18 AS builder
RUN npm install && \
    npm run build && \
    npm prune --production  # Remove dev dependencies
```

### 5. Use .dockerignore

```
# Giảm build context
node_modules
.git
*.log
```

---

## 🎓 Bài Tập Thực Hành

### Bài 1: Convert Single to Multi-Stage

Chuyển đổi Dockerfile sau sang multi-stage:

```dockerfile
FROM python:3.11
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

### Bài 2: Optimize React App

Tạo multi-stage Dockerfile cho React app với:
- Stage 1: Build với node:18
- Stage 2: Serve với nginx:alpine

### Bài 3: Multi-Language Build

Tạo Dockerfile với:
- Stage 1: Build frontend (React)
- Stage 2: Build backend (Go)
- Stage 3: Combine cả hai

---

## 📚 Tóm Tắt

### Key Takeaways

1. **Multi-Stage** giảm image size 80-90%
2. **Separate** build và runtime environments
3. **Copy** chỉ artifacts cần thiết
4. **Use** alpine hoặc slim images cho final stage
5. **Name** stages để dễ reference

### Commands

```bash
# Build specific stage
docker build --target stage-name -t image:tag .

# Build final stage (default)
docker build -t image:tag .
```

---

## 📚 Bài Tiếp Theo

Trong bài tiếp theo, chúng ta sẽ học **Best Practices** khi viết Dockerfile.

👉 [Bài 09: Best Practices Dockerfile](./09-best-practices-dockerfile.md)

---

## 🔗 Tài Liệu Tham Khảo

- [Multi-Stage Builds](https://docs.docker.com/build/building/multi-stage/)
- [Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

---

**Happy Optimizing! 🚀**

