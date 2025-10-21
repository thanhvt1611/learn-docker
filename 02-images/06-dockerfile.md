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

**Mục tiêu:** Tạo Dockerfile cho static website với nginx

**Yêu cầu:**
1. Sử dụng nginx:alpine
2. Copy HTML files vào /usr/share/nginx/html
3. Expose port 80
4. Build image
5. Run container và test

**Hướng dẫn chi tiết:**

```bash
# Bước 1: Tạo thư mục project
mkdir my-website
cd my-website

# Bước 2: Tạo file HTML
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>My Docker Website</title>
    <style>
        body { font-family: Arial; text-align: center; margin-top: 50px; }
        h1 { color: #333; }
    </style>
</head>
<body>
    <h1>Welcome to My Docker Website!</h1>
    <p>This is running inside a Docker container.</p>
</body>
</html>
EOF

# Bước 3: Tạo Dockerfile
cat > Dockerfile << 'EOF'
# Sử dụng nginx:alpine làm base image
FROM nginx:alpine

# Copy HTML files vào container
COPY index.html /usr/share/nginx/html/

# Expose port 80
EXPOSE 80

# CMD mặc định (nginx sẽ start tự động)
CMD ["nginx", "-g", "daemon off;"]
EOF

# Bước 4: Build image
docker build -t my-website:1.0 .
# Output:
# Sending build context to Docker daemon  2.048kB
# Step 1/4 : FROM nginx:alpine
# ...
# Successfully tagged my-website:1.0

# Bước 5: Verify image được tạo
docker images | grep my-website
# Output: my-website  1.0  ...

# Bước 6: Run container
docker run -d -p 8080:80 --name my-web my-website:1.0
# Output: container ID

# Bước 7: Test website
curl http://localhost:8080
# Output: HTML content

# Hoặc mở browser: http://localhost:8080

# Bước 8: Xem logs
docker logs my-web
# Output: Nginx logs

# Bước 9: Stop và remove
docker stop my-web
docker rm my-web
```

**Kiểm tra kết quả:**
- ✅ Dockerfile được tạo đúng
- ✅ Image build thành công
- ✅ Container chạy và accessible
- ✅ HTML content hiển thị đúng

---

### Bài 2: Node.js API

**Mục tiêu:** Tạo Dockerfile cho Node.js REST API

**Yêu cầu:**
1. Sử dụng node:18-alpine
2. Install dependencies
3. Copy source code
4. Expose port 3000
5. Run với non-root user
6. Build image
7. Run container và test

**Hướng dẫn chi tiết:**

```bash
# Bước 1: Tạo thư mục project
mkdir my-api
cd my-api

# Bước 2: Tạo package.json
cat > package.json << 'EOF'
{
  "name": "my-api",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
EOF

# Bước 3: Tạo app.js
cat > app.js << 'EOF'
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.json({ message: 'Hello from Docker!' });
});

app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
EOF

# Bước 4: Tạo Dockerfile
cat > Dockerfile << 'EOF'
# Sử dụng node:18-alpine
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Tạo non-root user
RUN addgroup -g 1001 appgroup && \
    adduser -D -u 1001 -G appgroup appuser

# Switch to non-root user
USER appuser

# Expose port
EXPOSE 3000

# Start app
CMD ["npm", "start"]
EOF

# Bước 5: Build image
docker build -t my-api:1.0 .
# Output: Successfully tagged my-api:1.0

# Bước 6: Verify image
docker images | grep my-api

# Bước 7: Run container
docker run -d -p 3000:3000 --name my-api-container my-api:1.0
# Output: container ID

# Bước 8: Xem logs
docker logs my-api-container
# Output: Server running on port 3000

# Bước 9: Test API
curl http://localhost:3000
# Output: {"message":"Hello from Docker!"}

curl http://localhost:3000/health
# Output: {"status":"healthy"}

# Bước 10: Verify non-root user
docker exec my-api-container whoami
# Output: appuser (không phải root!)

# Bước 11: Inspect image
docker inspect my-api:1.0 | grep -A 5 "User"
# Output: "User": "appuser"

# Bước 12: Stop và remove
docker stop my-api-container
docker rm my-api-container
```

**Kiểm tra kết quả:**
- ✅ Dockerfile được tạo đúng
- ✅ Image build thành công
- ✅ Container chạy
- ✅ API endpoints hoạt động
- ✅ Non-root user được sử dụng
- ✅ Dependencies được install

---

### Bài 3: Multi-Stage Build

**Mục tiêu:** Tạo Dockerfile với multi-stage build cho React app

**Yêu cầu:**
1. Stage 1: Build React app
2. Stage 2: Serve với Nginx
3. So sánh image size
4. Build image
5. Run container và test

**Hướng dẫn chi tiết:**

```bash
# Bước 1: Tạo thư mục project
mkdir my-react-app
cd my-react-app

# Bước 2: Tạo package.json
cat > package.json << 'EOF'
{
  "name": "my-react-app",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-scripts": "5.0.1"
  },
  "scripts": {
    "build": "react-scripts build",
    "start": "react-scripts start"
  },
  "eslintConfig": {
    "extends": ["react-app"]
  },
  "browserslist": {
    "production": [">0.2%", "not dead", "not op_mini all"],
    "development": ["last 1 chrome version"]
  }
}
EOF

# Bước 3: Tạo public/index.html
mkdir -p public
cat > public/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>My React App</title>
</head>
<body>
    <div id="root"></div>
</body>
</html>
EOF

# Bước 4: Tạo src/index.js
mkdir -p src
cat > src/index.js << 'EOF'
import React from 'react';
import ReactDOM from 'react-dom/client';

const App = () => (
  <div style={{ textAlign: 'center', marginTop: '50px' }}>
    <h1>Hello from React in Docker!</h1>
    <p>Multi-stage build example</p>
  </div>
);

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
EOF

# Bước 5: Tạo .dockerignore
cat > .dockerignore << 'EOF'
node_modules
npm-debug.log
build
.git
.env
EOF

# Bước 6: Tạo Dockerfile với multi-stage build
cat > Dockerfile << 'EOF'
# ===== Stage 1: Build =====
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy source code
COPY . .

# Build app
RUN npm run build

# ===== Stage 2: Production =====
FROM nginx:alpine

# Copy built files từ stage 1
COPY --from=builder /app/build /usr/share/nginx/html

# Copy nginx config (optional)
RUN echo 'server { listen 80; location / { root /usr/share/nginx/html; try_files $uri /index.html; } }' \
    > /etc/nginx/conf.d/default.conf

# Expose port
EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
EOF

# Bước 7: Build image
docker build -t my-react-app:1.0 .
# Output: Successfully tagged my-react-app:1.0
# Lưu ý: Build sẽ mất vài phút vì cần install dependencies

# Bước 8: Xem image size
docker images | grep my-react-app
# Output: my-react-app  1.0  ...  (khoảng 50-100MB)

# So sánh với single-stage build (nếu có):
# Single-stage: ~500MB (chứa node_modules)
# Multi-stage: ~50MB (chỉ chứa build output)

# Bước 9: Run container
docker run -d -p 8080:80 --name my-react my-react-app:1.0
# Output: container ID

# Bước 10: Test app
curl http://localhost:8080
# Output: HTML content

# Hoặc mở browser: http://localhost:8080

# Bước 11: Xem layers của image
docker history my-react-app:1.0
# Output: Sẽ thấy 2 stages

# Bước 12: Inspect image
docker inspect my-react-app:1.0 | grep -A 10 "RootFS"

# Bước 13: Stop và remove
docker stop my-react
docker rm my-react
```

**Kiểm tra kết quả:**
- ✅ Dockerfile multi-stage được tạo đúng
- ✅ Image build thành công
- ✅ Image size nhỏ (chỉ chứa build output)
- ✅ Container chạy
- ✅ App accessible
- ✅ Layers được tối ưu

**Bonus - So sánh Single-stage vs Multi-stage:**

```bash
# Tạo Dockerfile single-stage
cat > Dockerfile.single << 'EOF'
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
EXPOSE 80
CMD ["npm", "start"]
EOF

# Build single-stage
docker build -f Dockerfile.single -t my-react-app:single .

# So sánh size
docker images | grep my-react-app
# my-react-app:1.0      (multi-stage)  ~50MB
# my-react-app:single   (single-stage) ~500MB

# Multi-stage nhỏ hơn 10 lần!
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

