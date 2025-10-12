# Bài 05: Docker Images

## 📋 Mục Tiêu Bài Học

Sau bài học này, bạn sẽ:
- ✅ Hiểu sâu về Docker Images và Image Layers
- ✅ Nắm được cách Docker lưu trữ images
- ✅ Biết cách tối ưu image size
- ✅ Hiểu Image naming và tagging

---

## 🖼️ Docker Image Là Gì?

### Định Nghĩa Chi Tiết

**Docker Image** là một package read-only chứa:
- ✅ Application code
- ✅ Runtime environment (Node.js, Python, Java...)
- ✅ System libraries
- ✅ Dependencies
- ✅ Configuration files
- ✅ Environment variables

### Đặc Điểm

- **Immutable**: Không thể thay đổi sau khi build
- **Layered**: Được xây dựng từ nhiều layers
- **Reusable**: Có thể tái sử dụng cho nhiều containers
- **Shareable**: Có thể chia sẻ qua registry

---

## 🏗️ Image Layers (Lớp)

### Kiến Trúc Layers

```
┌─────────────────────────────────────────────────┐
│  Container Layer (Read-Write)                   │  ← Writable layer
├─────────────────────────────────────────────────┤
│  Layer 5: COPY app.js /app/                     │  ← 100 KB
├─────────────────────────────────────────────────┤
│  Layer 4: RUN npm install                       │  ← 50 MB
├─────────────────────────────────────────────────┤
│  Layer 3: COPY package.json /app/               │  ← 1 KB
├─────────────────────────────────────────────────┤
│  Layer 2: RUN apt-get install nodejs            │  ← 200 MB
├─────────────────────────────────────────────────┤
│  Layer 1: FROM ubuntu:22.04                     │  ← 77 MB
└─────────────────────────────────────────────────┘
                Base Image
```

### Cách Hoạt Động

**1. Mỗi instruction trong Dockerfile tạo 1 layer:**

```dockerfile
FROM node:18-alpine        # Layer 1
WORKDIR /app              # Layer 2
COPY package*.json ./     # Layer 3
RUN npm install           # Layer 4
COPY . .                  # Layer 5
CMD ["node", "app.js"]    # Layer 6 (metadata, không tạo layer mới)
```

**2. Layers được cache:**

```bash
# Build lần 1
Step 1/6 : FROM node:18-alpine
 ---> Pulling from library/node
Step 2/6 : WORKDIR /app
 ---> Running in abc123
Step 3/6 : COPY package*.json ./
 ---> Running in def456
Step 4/6 : RUN npm install
 ---> Running in ghi789
# ... mất 2 phút

# Build lần 2 (không thay đổi gì)
Step 1/6 : FROM node:18-alpine
 ---> Using cache
Step 2/6 : WORKDIR /app
 ---> Using cache
Step 3/6 : COPY package*.json ./
 ---> Using cache
Step 4/6 : RUN npm install
 ---> Using cache
# ... chỉ mất vài giây!
```

**3. Layers được share giữa images:**

```
Image A (nginx:latest)          Image B (nginx:1.25)
├─ Layer 1: Ubuntu              ├─ Layer 1: Ubuntu (shared!)
├─ Layer 2: Nginx 1.25          ├─ Layer 2: Nginx 1.25 (shared!)
└─ Layer 3: Config A            └─ Layer 3: Config B

Disk usage: 
- Không phải: 200MB + 200MB = 400MB
- Mà là: 200MB + 10MB = 210MB (tiết kiệm!)
```

### Xem Image Layers

```bash
# Xem history của image
docker history nginx:latest

# Output:
IMAGE          CREATED       CREATED BY                                      SIZE
605c77e624dd   2 weeks ago   /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B
<missing>      2 weeks ago   /bin/sh -c #(nop)  STOPSIGNAL SIGQUIT           0B
<missing>      2 weeks ago   /bin/sh -c #(nop)  EXPOSE 80                    0B
<missing>      2 weeks ago   /bin/sh -c ln -sf /dev/stdout /var/log/nginx…   22B
<missing>      2 weeks ago   /bin/sh -c set -x     && addgroup --system -…   61.1MB
<missing>      2 weeks ago   /bin/sh -c #(nop)  ENV PKG_RELEASE=1~bookwor…   0B

# Xem chi tiết layer
docker inspect nginx:latest
```

---

## 🏷️ Image Naming và Tagging

### Image Name Format

```
[REGISTRY/][NAMESPACE/]REPOSITORY[:TAG]

Ví dụ:
docker.io/library/nginx:latest
│         │       │      │
│         │       │      └─ Tag (version)
│         │       └──────── Repository (image name)
│         └──────────────── Namespace (user/org)
└────────────────────────── Registry (Docker Hub)
```

### Các Thành Phần

**1. Registry (Optional)**
```bash
docker.io/nginx              # Docker Hub (default)
gcr.io/my-project/app        # Google Container Registry
myregistry.com:5000/app      # Private registry
```

**2. Namespace (Optional)**
```bash
library/nginx                # Official images
username/myapp               # User images
company/product              # Organization images
```

**3. Repository (Required)**
```bash
nginx                        # Image name
myapp
api-server
```

**4. Tag (Optional, default: latest)**
```bash
nginx:latest                 # Latest version
nginx:1.25                   # Specific version
nginx:1.25-alpine            # Version + variant
myapp:v1.0.0                 # Semantic versioning
myapp:dev                    # Environment
myapp:sha-abc123             # Git commit
```

### Best Practices cho Tagging

```bash
# ❌ Không nên
docker build -t myapp .                    # Chỉ dùng latest
docker build -t myapp:1 .                  # Version không rõ ràng

# ✅ Nên
docker build -t myapp:1.0.0 .              # Semantic versioning
docker build -t myapp:1.0.0-alpine .       # Version + variant
docker build -t myapp:latest .             # Kèm theo latest

# ✅ Multi-tagging
docker build -t myapp:1.0.0 \
             -t myapp:1.0 \
             -t myapp:1 \
             -t myapp:latest .
```

---

## 📦 Base Images

### Official Images

```bash
# Operating Systems
ubuntu:22.04
debian:bullseye
alpine:3.18                  # Nhỏ nhất (~5MB)

# Programming Languages
node:18                      # Node.js
python:3.11                  # Python
openjdk:17                   # Java
golang:1.21                  # Go
php:8.2                      # PHP

# Databases
postgres:15
mysql:8.0
mongodb:7.0
redis:7.2

# Web Servers
nginx:latest
httpd:2.4                    # Apache
```

### Alpine vs Debian

```bash
# Debian-based (lớn hơn, đầy đủ hơn)
node:18                      # ~993MB
python:3.11                  # ~920MB

# Alpine-based (nhỏ hơn, tối giản)
node:18-alpine               # ~170MB
python:3.11-alpine           # ~50MB

# So sánh:
FROM node:18                 # Image size: 993MB
FROM node:18-alpine          # Image size: 170MB (tiết kiệm 82%!)
```

**Khi nào dùng Alpine?**

✅ **Dùng Alpine khi:**
- Production environments (tiết kiệm bandwidth)
- Microservices (nhiều containers)
- CI/CD pipelines (build nhanh hơn)

❌ **Không dùng Alpine khi:**
- Cần nhiều system libraries
- Compatibility issues với một số packages
- Development environment (debugging khó hơn)

---

## 💾 Image Storage

### Nơi Lưu Trữ Images

```bash
# Linux
/var/lib/docker/

# macOS (Docker Desktop)
~/Library/Containers/com.docker.docker/Data/

# Windows (Docker Desktop)
C:\ProgramData\Docker\
```

### Storage Drivers

```bash
# Xem storage driver hiện tại
docker info | grep "Storage Driver"

# Common drivers:
# - overlay2 (recommended, Linux)
# - aufs (older Linux)
# - devicemapper (RHEL/CentOS)
# - windowsfilter (Windows)
```

### Disk Usage

```bash
# Xem disk usage
docker system df

# Output:
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          10        5         2.5GB     1.2GB (48%)
Containers      15        3         100MB     80MB (80%)
Local Volumes   5         2         500MB     300MB (60%)
Build Cache     20        0         1GB       1GB (100%)

# Xem chi tiết
docker system df -v
```

---

## 🔍 Làm Việc Với Images

### Pull Images

```bash
# Pull latest version
docker pull nginx

# Pull specific version
docker pull nginx:1.25

# Pull từ registry khác
docker pull gcr.io/google-samples/hello-app:1.0

# Pull tất cả tags (cẩn thận!)
docker pull --all-tags nginx

# Pull với platform cụ thể
docker pull --platform linux/amd64 nginx
```

### List Images

```bash
# List tất cả images
docker images

# List với filter
docker images nginx
docker images --filter "dangling=true"
docker images --filter "before=nginx:latest"

# Format output
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"

# List image IDs
docker images -q
```

### Inspect Images

```bash
# Xem chi tiết image
docker inspect nginx:latest

# Lấy thông tin cụ thể
docker inspect --format='{{.Architecture}}' nginx
docker inspect --format='{{.Os}}' nginx
docker inspect --format='{{.Size}}' nginx

# Xem environment variables
docker inspect --format='{{.Config.Env}}' nginx

# Xem exposed ports
docker inspect --format='{{.Config.ExposedPorts}}' nginx
```

### Tag Images

```bash
# Tag image với tên mới
docker tag nginx:latest myregistry/nginx:v1.0

# Tag cho multiple registries
docker tag myapp:latest docker.io/username/myapp:latest
docker tag myapp:latest gcr.io/project/myapp:latest
```

### Remove Images

```bash
# Xóa image
docker rmi nginx:latest

# Xóa force (kể cả khi có containers)
docker rmi -f nginx:latest

# Xóa nhiều images
docker rmi nginx:latest node:18 python:3.11

# Xóa tất cả dangling images
docker image prune

# Xóa tất cả unused images
docker image prune -a

# Xóa images theo filter
docker images --filter "dangling=true" -q | xargs docker rmi
```

---

## 🎯 Ví Dụ Thực Hành

### Ví Dụ 1: Khám Phá Image Layers

```bash
# 1. Pull một image
docker pull nginx:alpine

# 2. Xem layers
docker history nginx:alpine

# 3. So sánh với version khác
docker pull nginx:latest
docker history nginx:latest

# 4. So sánh size
docker images nginx
```

### Ví Dụ 2: Image Tagging

```bash
# 1. Pull base image
docker pull node:18-alpine

# 2. Tag với nhiều versions
docker tag node:18-alpine myapp:1.0.0
docker tag node:18-alpine myapp:1.0
docker tag node:18-alpine myapp:1
docker tag node:18-alpine myapp:latest

# 3. Xem tất cả tags
docker images myapp

# 4. Push lên registry (nếu đã login)
docker push myapp:1.0.0
```

### Ví Dụ 3: Cleanup Images

```bash
# 1. Xem disk usage
docker system df

# 2. Xem dangling images
docker images --filter "dangling=true"

# 3. Xóa dangling images
docker image prune

# 4. Xóa tất cả unused images
docker image prune -a

# 5. Kiểm tra lại
docker system df
```

---

## 💡 Tips & Best Practices

### 1. Chọn Base Image Phù Hợp

```dockerfile
# ❌ Quá lớn
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y nodejs

# ✅ Tối ưu
FROM node:18-alpine
```

### 2. Sử Dụng Specific Tags

```dockerfile
# ❌ Không nên
FROM node:latest              # Không predictable

# ✅ Nên
FROM node:18.17.0-alpine3.18  # Specific version
```

### 3. Minimize Layers

```dockerfile
# ❌ Nhiều layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git

# ✅ Ít layers hơn
RUN apt-get update && \
    apt-get install -y \
        curl \
        git && \
    rm -rf /var/lib/apt/lists/*
```

### 4. Leverage Build Cache

```dockerfile
# ❌ Cache bị invalidate thường xuyên
COPY . /app
RUN npm install

# ✅ Tận dụng cache
COPY package*.json /app/
RUN npm install
COPY . /app
```

---

## 📚 Tóm Tắt

### Key Takeaways

1. **Image** = Read-only template với layers
2. **Layers** được cache và share giữa images
3. **Tagging** quan trọng cho version management
4. **Alpine** images nhỏ hơn nhiều so với Debian
5. **Cleanup** thường xuyên để tiết kiệm disk space

---

## 📚 Bài Tiếp Theo

Trong bài tiếp theo, chúng ta sẽ học cách viết **Dockerfile** để tạo custom images.

👉 [Bài 06: Dockerfile](./06-dockerfile.md)

---

## 🔗 Tài Liệu Tham Khảo

- [Docker Images Documentation](https://docs.docker.com/engine/reference/commandline/images/)
- [Image Layers](https://docs.docker.com/storage/storagedriver/)
- [Best Practices for Writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

---

**Happy Learning! 🚀**

