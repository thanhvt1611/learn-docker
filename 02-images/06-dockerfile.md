# Bài 06: Dockerfile

## 📋 Mục Tiêu Bài Học

Sau bài học này, bạn sẽ:
- ✅ Hiểu cấu trúc của Dockerfile
- ✅ Thành thạo các instructions cơ bản
- ✅ Viết Dockerfile cho các ngôn ngữ khác nhau
- ✅ Hiểu sự khác biệt giữa CMD và ENTRYPOINT

---

## 📝 Dockerfile Là Gì?

**Dockerfile** là một text file chứa các instructions để build Docker image.

### Cấu Trúc Cơ Bản

```dockerfile
# Comment
INSTRUCTION arguments
```

### Ví Dụ Đơn Giản

```dockerfile
# Sử dụng base image
FROM node:18-alpine

# Đặt working directory
WORKDIR /app

# Copy files
COPY package*.json ./

# Cài dependencies
RUN npm install

# Copy source code
COPY . .

# Expose port
EXPOSE 3000

# Chạy ứng dụng
CMD ["node", "app.js"]
```

---

## 🔧 Các Instructions Cơ Bản

### 1. FROM - Base Image

**Syntax:**
```dockerfile
FROM <image>[:<tag>] [AS <name>]
```

**Ví dụ:**

```dockerfile
# Sử dụng official image
FROM node:18-alpine

# Sử dụng specific version
FROM python:3.11.5-slim

# Multi-stage build
FROM node:18 AS builder
FROM nginx:alpine AS production

# Sử dụng scratch (empty image)
FROM scratch
```

**Best Practices:**

```dockerfile
# ❌ Không nên
FROM node:latest              # Không predictable

# ✅ Nên
FROM node:18.17.0-alpine3.18  # Specific version
```

---

### 2. WORKDIR - Set Working Directory

**Syntax:**
```dockerfile
WORKDIR /path/to/directory
```

**Ví dụ:**

```dockerfile
FROM node:18-alpine

# Tạo và set working directory
WORKDIR /app

# Tất cả commands sau đây chạy trong /app
COPY . .
RUN npm install
```

**Tương đương với:**

```bash
mkdir -p /app
cd /app
```

---

### 3. COPY - Copy Files

**Syntax:**
```dockerfile
COPY [--chown=<user>:<group>] <src>... <dest>
```

**Ví dụ:**

```dockerfile
# Copy file
COPY package.json /app/

# Copy nhiều files
COPY package.json package-lock.json /app/

# Copy thư mục
COPY src/ /app/src/

# Copy tất cả
COPY . /app/

# Copy với ownership
COPY --chown=node:node . /app/

# Copy từ stage khác (multi-stage)
COPY --from=builder /app/dist /app/
```

**Wildcards:**

```dockerfile
# Copy tất cả .js files
COPY *.js /app/

# Copy tất cả files trong src/
COPY src/* /app/src/
```

---

### 4. ADD - Copy Files (Advanced)

**Syntax:**
```dockerfile
ADD <src>... <dest>
```

**Ví dụ:**

```dockerfile
# Giống COPY
ADD package.json /app/

# Auto-extract tar files
ADD app.tar.gz /app/

# Download từ URL
ADD https://example.com/file.tar.gz /app/
```

**COPY vs ADD:**

```dockerfile
# ✅ Dùng COPY cho local files
COPY package.json /app/

# ✅ Dùng ADD khi cần auto-extract
ADD archive.tar.gz /app/

# ❌ Không dùng ADD cho simple copy
ADD package.json /app/  # Nên dùng COPY
```

---

### 5. RUN - Execute Commands

**Syntax:**
```dockerfile
# Shell form
RUN <command>

# Exec form
RUN ["executable", "param1", "param2"]
```

**Ví dụ:**

```dockerfile
# Shell form (chạy trong /bin/sh -c)
RUN npm install
RUN apt-get update && apt-get install -y curl

# Exec form (không dùng shell)
RUN ["npm", "install"]
RUN ["/bin/bash", "-c", "echo hello"]

# Multi-line với \
RUN apt-get update && \
    apt-get install -y \
        curl \
        git \
        vim && \
    rm -rf /var/lib/apt/lists/*
```

**Best Practices:**

```dockerfile
# ❌ Nhiều RUN commands (nhiều layers)
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git
RUN rm -rf /var/lib/apt/lists/*

# ✅ Combine commands (ít layers hơn)
RUN apt-get update && \
    apt-get install -y curl git && \
    rm -rf /var/lib/apt/lists/*
```

---

### 6. CMD - Default Command

**Syntax:**
```dockerfile
# Exec form (preferred)
CMD ["executable", "param1", "param2"]

# Shell form
CMD command param1 param2

# As default parameters to ENTRYPOINT
CMD ["param1", "param2"]
```

**Ví dụ:**

```dockerfile
# Node.js app
CMD ["node", "app.js"]

# Python app
CMD ["python", "app.py"]

# Shell command
CMD npm start

# With parameters
CMD ["node", "server.js", "--port", "3000"]
```

**Lưu ý:**

```dockerfile
# Chỉ có 1 CMD trong Dockerfile
CMD ["node", "app.js"]
CMD ["npm", "start"]      # Cái này sẽ override cái trên

# CMD có thể bị override khi run
# Dockerfile:
CMD ["node", "app.js"]

# Run:
docker run myapp              # Chạy: node app.js
docker run myapp npm test     # Chạy: npm test (override CMD)
```

---

### 7. ENTRYPOINT - Configure Container Executable

**Syntax:**
```dockerfile
# Exec form (preferred)
ENTRYPOINT ["executable", "param1", "param2"]

# Shell form
ENTRYPOINT command param1 param2
```

**Ví dụ:**

```dockerfile
# Simple entrypoint
ENTRYPOINT ["node", "app.js"]

# With CMD as default params
ENTRYPOINT ["node"]
CMD ["app.js"]

# Script entrypoint
ENTRYPOINT ["/entrypoint.sh"]
```

**ENTRYPOINT vs CMD:**

```dockerfile
# Chỉ CMD
CMD ["node", "app.js"]
# docker run myapp           → node app.js
# docker run myapp npm test  → npm test (override)

# Chỉ ENTRYPOINT
ENTRYPOINT ["node", "app.js"]
# docker run myapp           → node app.js
# docker run myapp npm test  → node app.js npm test (append)

# ENTRYPOINT + CMD
ENTRYPOINT ["node"]
CMD ["app.js"]
# docker run myapp           → node app.js
# docker run myapp server.js → node server.js (override CMD)
```

---

### 8. ENV - Environment Variables

**Syntax:**
```dockerfile
ENV <key>=<value> ...
```

**Ví dụ:**

```dockerfile
# Single variable
ENV NODE_ENV=production

# Multiple variables
ENV NODE_ENV=production \
    PORT=3000 \
    LOG_LEVEL=info

# Sử dụng trong Dockerfile
ENV APP_HOME=/app
WORKDIR $APP_HOME
COPY . $APP_HOME
```

**Sử dụng trong container:**

```bash
docker run -e NODE_ENV=development myapp  # Override ENV
```

---

### 9. ARG - Build Arguments

**Syntax:**
```dockerfile
ARG <name>[=<default value>]
```

**Ví dụ:**

```dockerfile
# Define build argument
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}-alpine

ARG APP_PORT=3000
EXPOSE $APP_PORT

# With default value
ARG BUILD_DATE
ARG VERSION=1.0.0
LABEL build_date=$BUILD_DATE
LABEL version=$VERSION
```

**Build với arguments:**

```bash
docker build --build-arg NODE_VERSION=20 -t myapp .
docker build --build-arg VERSION=2.0.0 -t myapp:2.0.0 .
```

**ARG vs ENV:**

```dockerfile
# ARG: Chỉ available trong build time
ARG BUILD_ENV=production

# ENV: Available trong build time VÀ runtime
ENV NODE_ENV=production

# Combine ARG và ENV
ARG NODE_VERSION=18
ENV NODE_VERSION=$NODE_VERSION
```

---

### 10. EXPOSE - Document Ports

**Syntax:**
```dockerfile
EXPOSE <port> [<port>/<protocol>...]
```

**Ví dụ:**

```dockerfile
# Single port
EXPOSE 3000

# Multiple ports
EXPOSE 3000 8080

# With protocol
EXPOSE 80/tcp
EXPOSE 53/udp

# Range
EXPOSE 8000-8010
```

**Lưu ý:**

```dockerfile
# EXPOSE chỉ là documentation, không publish port
EXPOSE 3000

# Phải dùng -p khi run
docker run -p 3000:3000 myapp
```

---

### 11. VOLUME - Mount Points

**Syntax:**
```dockerfile
VOLUME ["/data"]
```

**Ví dụ:**

```dockerfile
# Single volume
VOLUME /var/lib/mysql

# Multiple volumes
VOLUME ["/data", "/logs"]

# With WORKDIR
WORKDIR /app
VOLUME /app/data
```

---

### 12. USER - Set User

**Syntax:**
```dockerfile
USER <user>[:<group>]
```

**Ví dụ:**

```dockerfile
# Tạo user và switch
RUN addgroup -g 1000 node && \
    adduser -D -u 1000 -G node node

USER node

# Hoặc dùng existing user
USER node:node

# Switch về root nếu cần
USER root
RUN apt-get update
USER node
```

---

### 13. LABEL - Metadata

**Syntax:**
```dockerfile
LABEL <key>=<value> <key>=<value> ...
```

**Ví dụ:**

```dockerfile
LABEL maintainer="your-email@example.com"
LABEL version="1.0.0"
LABEL description="My awesome application"

# Multiple labels
LABEL org.opencontainers.image.authors="John Doe" \
      org.opencontainers.image.version="1.0.0" \
      org.opencontainers.image.created="2024-01-01"
```

---

## 🎯 Ví Dụ Dockerfile Cho Các Ngôn Ngữ

### Node.js Application

```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Build application (nếu cần)
RUN npm run build

# Production stage
FROM node:18-alpine

WORKDIR /app

# Copy from builder
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY package*.json ./

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

USER nodejs

EXPOSE 3000

CMD ["node", "dist/index.js"]
```

### Python Application

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .

# Create non-root user
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app

USER appuser

EXPOSE 8000

CMD ["python", "app.py"]
```

### Java Application

```dockerfile
# Build stage
FROM maven:3.9-eclipse-temurin-17 AS builder

WORKDIR /app

COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn package -DskipTests

# Runtime stage
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

COPY --from=builder /app/target/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Go Application

```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o main .

# Runtime stage
FROM alpine:latest

RUN apk --no-cache add ca-certificates

WORKDIR /root/

COPY --from=builder /app/main .

EXPOSE 8080

CMD ["./main"]
```

### .NET Application

```dockerfile
# Build stage
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS builder

WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:8.0

WORKDIR /app

COPY --from=builder /app/out .

EXPOSE 80

ENTRYPOINT ["dotnet", "MyApp.dll"]
```

---

## 💡 Best Practices

### 1. Order Instructions Properly

```dockerfile
# ❌ Không tốt (cache bị invalidate thường xuyên)
FROM node:18-alpine
COPY . /app
RUN npm install

# ✅ Tốt (tận dụng cache)
FROM node:18-alpine
COPY package*.json /app/
RUN npm install
COPY . /app
```

### 2. Minimize Layers

```dockerfile
# ❌ Nhiều layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git

# ✅ Ít layers
RUN apt-get update && \
    apt-get install -y curl git && \
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
```

### 4. Don't Run as Root

```dockerfile
# ✅ Tạo và sử dụng non-root user
RUN addgroup -g 1001 appgroup && \
    adduser -D -u 1001 -G appgroup appuser

USER appuser
```

### 5. Use Multi-Stage Builds

```dockerfile
# Build stage (lớn)
FROM node:18 AS builder
# ... build code

# Production stage (nhỏ)
FROM node:18-alpine
COPY --from=builder /app/dist ./dist
```

---

## 🎓 Bài Tập Thực Hành

### Bài 1: Simple Web Server

Tạo Dockerfile cho static website với nginx:

```dockerfile
# Viết Dockerfile để:
# 1. Sử dụng nginx:alpine
# 2. Copy HTML files vào /usr/share/nginx/html
# 3. Expose port 80
```

### Bài 2: Node.js API

Tạo Dockerfile cho Node.js REST API:

```dockerfile
# Viết Dockerfile để:
# 1. Sử dụng node:18-alpine
# 2. Install dependencies
# 3. Copy source code
# 4. Expose port 3000
# 5. Run với non-root user
```

### Bài 3: Multi-Stage Build

Tạo Dockerfile với multi-stage build cho React app:

```dockerfile
# Stage 1: Build
# - Sử dụng node:18
# - npm install và npm run build

# Stage 2: Production
# - Sử dụng nginx:alpine
# - Copy build files từ stage 1
```

---

## 📚 Tóm Tắt

### Instructions Quan Trọng

| Instruction | Mục Đích | Ví Dụ |
|-------------|----------|-------|
| `FROM` | Base image | `FROM node:18-alpine` |
| `WORKDIR` | Working directory | `WORKDIR /app` |
| `COPY` | Copy files | `COPY . /app` |
| `RUN` | Execute commands | `RUN npm install` |
| `CMD` | Default command | `CMD ["node", "app.js"]` |
| `ENTRYPOINT` | Main executable | `ENTRYPOINT ["node"]` |
| `ENV` | Environment vars | `ENV NODE_ENV=production` |
| `EXPOSE` | Document ports | `EXPOSE 3000` |

---

## 📚 Bài Tiếp Theo

Trong bài tiếp theo, chúng ta sẽ học cách **build và quản lý images**.

👉 [Bài 07: Build và Quản Lý Images](./07-build-va-quan-ly-images.md)

---

## 🔗 Tài Liệu Tham Khảo

- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [Best Practices for Writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

---

**Happy Coding! 🚀**

