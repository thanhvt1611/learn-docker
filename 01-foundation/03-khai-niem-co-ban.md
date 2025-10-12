# Bài 03: Khái Niệm Cơ Bản

## 📋 Mục Tiêu Bài Học

Sau bài học này, bạn sẽ:
- ✅ Hiểu rõ Container, Image, Registry là gì
- ✅ Nắm được Docker Architecture chi tiết
- ✅ Phân biệt Image vs Container
- ✅ Hiểu Docker lifecycle

---

## 🐳 Các Khái Niệm Cốt Lõi

### 1. Docker Image

#### Định Nghĩa

**Docker Image** là một template read-only (chỉ đọc) chứa tất cả những gì cần để chạy một ứng dụng:
- Code của ứng dụng
- Runtime (Node.js, Python, Java...)
- Libraries và dependencies
- Environment variables
- Configuration files

#### Ví Dụ Thực Tế

```
Hãy tưởng tượng Image như một "công thức nấu ăn":
- Công thức bánh pizza (Image) 
- Có đầy đủ nguyên liệu và hướng dẫn
- Nhưng chưa phải là pizza thật (Container)
```

#### Image Layers (Lớp)

Docker Image được xây dựng từ nhiều layers (lớp):

```
┌─────────────────────────────────────┐
│  Layer 5: Your Application Code    │  ← Thêm code của bạn
├─────────────────────────────────────┤
│  Layer 4: npm install dependencies │  ← Cài dependencies
├─────────────────────────────────────┤
│  Layer 3: Copy package.json         │  ← Copy config files
├─────────────────────────────────────┤
│  Layer 2: Install Node.js           │  ← Cài runtime
├─────────────────────────────────────┤
│  Layer 1: Ubuntu Base OS            │  ← Base image
└─────────────────────────────────────┘
```

**Đặc điểm:**
- Mỗi layer là read-only
- Layers được cache để tái sử dụng
- Chỉ layers thay đổi mới rebuild

#### Ví Dụ Code

```bash
# List tất cả images
docker images

# Output:
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    605c77e624dd   2 weeks ago    141MB
node          18        a283f62cb84b   3 weeks ago    993MB
ubuntu        22.04     3b418d7b466a   4 weeks ago    77.8MB

# Pull image từ Docker Hub
docker pull nginx:latest

# Xem chi tiết image
docker inspect nginx:latest

# Xem history của image (các layers)
docker history nginx:latest
```

---

### 2. Docker Container

#### Định Nghĩa

**Docker Container** là một instance đang chạy (running instance) của một Docker Image.

```
Image  →  Container
(Template)  (Running Instance)

Giống như:
Class  →  Object (trong OOP)
Recipe →  Dish (trong nấu ăn)
Blueprint → House (trong xây dựng)
```

#### Đặc Điểm

- ✅ **Isolated**: Độc lập với host và containers khác
- ✅ **Lightweight**: Chia sẻ OS kernel, không cần full OS
- ✅ **Portable**: Chạy trên bất kỳ đâu có Docker
- ✅ **Ephemeral**: Có thể tạo/xóa nhanh chóng

#### Container Lifecycle

```
┌──────────┐
│  Image   │
└────┬─────┘
     │ docker run
     ▼
┌──────────┐     docker start      ┌──────────┐
│ Created  │ ──────────────────▶   │ Running  │
└──────────┘                       └────┬─────┘
     ▲                                  │
     │                                  │ docker stop
     │                                  ▼
     │                             ┌──────────┐
     │                             │ Stopped  │
     │                             └────┬─────┘
     │                                  │
     │                                  │ docker rm
     │                                  ▼
     └──────────────────────────── ┌──────────┐
                                   │ Removed  │
                                   └──────────┘
```

#### Ví Dụ Code

```bash
# Tạo và chạy container từ image
docker run -d --name my-nginx nginx
# -d: detached mode (chạy background)
# --name: đặt tên container

# List containers đang chạy
docker ps

# List tất cả containers (kể cả stopped)
docker ps -a

# Stop container
docker stop my-nginx

# Start lại container
docker start my-nginx

# Restart container
docker restart my-nginx

# Xóa container
docker rm my-nginx

# Xóa container đang chạy (force)
docker rm -f my-nginx
```

---

### 3. Docker Registry

#### Định Nghĩa

**Docker Registry** là nơi lưu trữ và phân phối Docker Images.

```
┌─────────────────────────────────────────────┐
│          DOCKER REGISTRY (Docker Hub)       │
│                                             │
│  Public Images:                             │
│  - nginx, node, python, mysql, redis...     │
│                                             │
│  Your Images:                               │
│  - username/myapp:v1.0                      │
│  - company/api:latest                       │
└─────────────────────────────────────────────┘
         ▲                    │
         │ push               │ pull
         │                    ▼
    ┌─────────────────────────────┐
    │    Your Local Machine       │
    │    (Docker Engine)          │
    └─────────────────────────────┘
```

#### Các Loại Registry

**1. Docker Hub (Public)**
- Registry mặc định của Docker
- Miễn phí cho public images
- URL: https://hub.docker.com

**2. Private Registry**
- Tự host registry riêng
- Bảo mật hơn cho doanh nghiệp
- Ví dụ: Harbor, GitLab Container Registry

**3. Cloud Registry**
- AWS ECR (Elastic Container Registry)
- Google GCR (Google Container Registry)
- Azure ACR (Azure Container Registry)

#### Ví Dụ Code

```bash
# Pull image từ Docker Hub
docker pull nginx:latest
docker pull node:18-alpine

# Pull từ specific registry
docker pull gcr.io/google-samples/hello-app:1.0

# Login vào Docker Hub
docker login
# Nhập username và password

# Tag image
docker tag myapp:latest username/myapp:v1.0

# Push image lên Docker Hub
docker push username/myapp:v1.0

# Logout
docker logout
```

---

### 4. Docker Compose

#### Định Nghĩa

**Docker Compose** là công cụ để định nghĩa và chạy multi-container applications.

#### Ví Dụ

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Frontend
  web:
    image: nginx:latest
    ports:
      - "80:80"
    depends_on:
      - api

  # Backend API
  api:
    image: node:18
    ports:
      - "3000:3000"
    depends_on:
      - db

  # Database
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

```bash
# Start tất cả services
docker compose up -d

# Stop tất cả services
docker compose down

# Xem logs
docker compose logs -f
```

---

## 🏗️ Docker Architecture Chi Tiết

### Kiến Trúc Tổng Quan

```
┌────────────────────────────────────────────────────────────────┐
│                        DOCKER HOST                             │
│                                                                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │              DOCKER DAEMON (dockerd)                     │ │
│  │                                                          │ │
│  │  ┌────────────────────────────────────────────────────┐ │ │
│  │  │           CONTAINERS                               │ │ │
│  │  │                                                    │ │ │
│  │  │  ┌──────────┐  ┌──────────┐  ┌──────────┐        │ │ │
│  │  │  │Container1│  │Container2│  │Container3│        │ │ │
│  │  │  │          │  │          │  │          │        │ │ │
│  │  │  │  nginx   │  │  node    │  │ postgres │        │ │ │
│  │  │  └──────────┘  └──────────┘  └──────────┘        │ │ │
│  │  └────────────────────────────────────────────────────┘ │ │
│  │                                                          │ │
│  │  ┌────────────────────────────────────────────────────┐ │ │
│  │  │           IMAGES                                   │ │ │
│  │  │  [nginx:latest] [node:18] [postgres:15]           │ │ │
│  │  └────────────────────────────────────────────────────┘ │ │
│  │                                                          │ │
│  │  ┌────────────────────────────────────────────────────┐ │ │
│  │  │           VOLUMES                                  │ │ │
│  │  │  [db-data] [app-logs] [config]                    │ │ │
│  │  └────────────────────────────────────────────────────┘ │ │
│  │                                                          │ │
│  │  ┌────────────────────────────────────────────────────┐ │ │
│  │  │           NETWORKS                                 │ │ │
│  │  │  [bridge] [host] [custom-network]                 │ │ │
│  │  └────────────────────────────────────────────────────┘ │ │
│  └──────────────────────────────────────────────────────────┘ │
│                          ▲                                    │
│                          │ REST API / Unix Socket             │
│                          │                                    │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │              DOCKER CLIENT (docker CLI)                 │ │
│  │  $ docker run                                           │ │
│  │  $ docker build                                         │ │
│  │  $ docker pull                                          │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
                          ▲
                          │ HTTPS
                          │
┌────────────────────────────────────────────────────────────────┐
│                   DOCKER REGISTRY (Docker Hub)                 │
│  Public Images: nginx, node, python, mysql, redis...          │
└────────────────────────────────────────────────────────────────┘
```

### Các Thành Phần

#### 1. Docker Client
- CLI tool: `docker`
- Gửi commands đến Docker Daemon
- Có thể kết nối đến remote daemon

#### 2. Docker Daemon (dockerd)
- Chạy trên host machine
- Quản lý Docker objects
- Lắng nghe Docker API requests
- Giao tiếp với other daemons

#### 3. Docker Objects

**Images:**
- Read-only templates
- Built from Dockerfile
- Stored locally hoặc trong registry

**Containers:**
- Runnable instances của images
- Có thể start, stop, move, delete
- Isolated từ host và containers khác

**Networks:**
- Kết nối containers với nhau
- Types: bridge, host, overlay, none

**Volumes:**
- Persistent data storage
- Tách biệt khỏi container lifecycle

---

## 🔄 Image vs Container

### So Sánh Trực Quan

```
┌─────────────────────────────────────────────────────────────┐
│                        IMAGE                                │
│  - Read-only template                                       │
│  - Stored on disk                                           │
│  - Can be shared                                            │
│  - Immutable (không thay đổi)                               │
│                                                             │
│  Example: nginx:latest                                      │
└─────────────────────────────────────────────────────────────┘
                          │
                          │ docker run
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                      CONTAINER                              │
│  - Running instance                                         │
│  - Has writable layer                                       │
│  - Can be started/stopped                                   │
│  - Mutable (có thể thay đổi)                                │
│                                                             │
│  Example: my-nginx (running nginx:latest)                  │
└─────────────────────────────────────────────────────────────┘
```

### Bảng So Sánh

| Tiêu Chí | Image | Container |
|----------|-------|-----------|
| **Trạng thái** | Static (tĩnh) | Dynamic (động) |
| **Lưu trữ** | Disk | Memory + Disk |
| **Có thể sửa?** | ❌ Read-only | ✅ Writable layer |
| **Lifecycle** | Permanent | Temporary |
| **Số lượng** | 1 image | Nhiều containers từ 1 image |

### Ví Dụ Thực Tế

```bash
# 1 Image có thể tạo nhiều Containers

# Pull image
docker pull nginx:latest

# Tạo container 1
docker run -d --name web1 -p 8081:80 nginx

# Tạo container 2 từ cùng image
docker run -d --name web2 -p 8082:80 nginx

# Tạo container 3 từ cùng image
docker run -d --name web3 -p 8083:80 nginx

# Bây giờ có:
# - 1 image: nginx:latest
# - 3 containers: web1, web2, web3
```

---

## 🎯 Docker Workflow

### Development Workflow

```
1. Write Code
   ↓
2. Create Dockerfile
   ↓
3. Build Image
   $ docker build -t myapp:v1 .
   ↓
4. Run Container
   $ docker run -d -p 3000:3000 myapp:v1
   ↓
5. Test Application
   ↓
6. Push to Registry
   $ docker push username/myapp:v1
   ↓
7. Deploy to Production
   $ docker pull username/myapp:v1
   $ docker run -d username/myapp:v1
```

---

## 💡 Ví Dụ Thực Hành

### Ví Dụ 1: Node.js Application

```bash
# 1. Pull Node.js image
docker pull node:18-alpine

# 2. Chạy container với volume mount
docker run -it --rm \
  -v $(pwd):/app \
  -w /app \
  node:18-alpine \
  sh

# Trong container:
# npm init -y
# npm install express
# node app.js
```

### Ví Dụ 2: Database Container

```bash
# Chạy PostgreSQL
docker run -d \
  --name my-postgres \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15

# Connect vào database
docker exec -it my-postgres psql -U postgres -d mydb
```

---

## 📚 Tóm Tắt

### Key Takeaways

1. **Image** = Template (read-only)
2. **Container** = Running instance (writable)
3. **Registry** = Nơi lưu trữ images
4. **Docker Compose** = Multi-container tool
5. **1 Image** → **Nhiều Containers**

### Thuật Ngữ Quan Trọng

- **Layer**: Lớp trong image
- **Tag**: Phiên bản của image (latest, v1.0, ...)
- **Repository**: Tập hợp các images cùng tên
- **Daemon**: Docker service chạy background
- **Client**: Docker CLI tool

---

## 📚 Bài Tiếp Theo

Trong bài tiếp theo, chúng ta sẽ học các **lệnh Docker cơ bản** để thao tác với containers và images.

👉 [Bài 04: Lệnh Docker Cơ Bản](./04-lenh-docker-co-ban.md)

---

## 🔗 Tài Liệu Tham Khảo

- [Docker Objects](https://docs.docker.com/get-started/docker-overview/#docker-objects)
- [Docker Architecture](https://docs.docker.com/get-started/docker-overview/#docker-architecture)
- [What is a Container?](https://www.docker.com/resources/what-container/)

---

**Happy Learning! 🚀**

