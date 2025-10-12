# Bài 12: Volumes và Storage

## 📋 Mục Tiêu Bài Học

Sau bài học này, bạn sẽ:
- ✅ Hiểu cách Docker lưu trữ data
- ✅ Sử dụng Volumes, Bind Mounts, tmpfs
- ✅ Persist data giữa container restarts
- ✅ Share data giữa containers
- ✅ Backup và restore volumes

---

## 💾 Docker Storage Overview

### 3 Loại Storage

```
┌─────────────────────────────────────────────────────────┐
│  1. Volumes        - Managed by Docker (recommended)    │
│  2. Bind Mounts    - Mount host directory               │
│  3. tmpfs Mounts   - In-memory storage (Linux only)     │
└─────────────────────────────────────────────────────────┘
```

### So Sánh

| Feature | Volumes | Bind Mounts | tmpfs |
|---------|---------|-------------|-------|
| **Managed by** | Docker | User | Docker |
| **Location** | /var/lib/docker/volumes | Anywhere on host | Memory |
| **Portable** | ✅ Yes | ❌ No | ❌ No |
| **Performance** | ✅ Good | ✅ Good | ⚡ Fastest |
| **Persist** | ✅ Yes | ✅ Yes | ❌ No |
| **Share** | ✅ Easy | ⚠️ Manual | ❌ No |
| **Backup** | ✅ Easy | ⚠️ Manual | ❌ N/A |

---

## 📦 Volumes (Recommended)

### Cách Hoạt Động

```
┌─────────────────────────────────────────────────────────┐
│                    HOST MACHINE                         │
│                                                         │
│  /var/lib/docker/volumes/                              │
│  ├── mydata/                                           │
│  │   └── _data/          ← Volume data                │
│  │       ├── file1.txt                                │
│  │       └── file2.txt                                │
│  │                                                     │
│  ┌──────────────────────────────────────────────────┐ │
│  │  Container                                       │ │
│  │  /app/data  ──────────────▶  mydata volume      │ │
│  └──────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

### Tạo và Sử Dụng Volumes

```bash
# Tạo volume
docker volume create mydata

# List volumes
docker volume ls

# Inspect volume
docker volume inspect mydata

# Output:
[
    {
        "CreatedAt": "2024-01-01T10:00:00Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/mydata/_data",
        "Name": "mydata",
        "Options": {},
        "Scope": "local"
    }
]
```

### Mount Volume Vào Container

```bash
# Method 1: -v flag
docker run -d \
  --name web \
  -v mydata:/app/data \
  nginx

# Method 2: --mount flag (recommended, more explicit)
docker run -d \
  --name web \
  --mount source=mydata,target=/app/data \
  nginx

# Auto-create volume nếu chưa tồn tại
docker run -d \
  --name web \
  -v mydata:/app/data \
  nginx
# Volume "mydata" sẽ được tạo tự động
```

### Ví Dụ: Database với Volume

```bash
# PostgreSQL với persistent data
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15

# Data được lưu trong volume "pgdata"
# Khi container bị xóa, data vẫn còn

# Stop và remove container
docker stop postgres
docker rm postgres

# Start lại với cùng volume
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15

# Data vẫn còn nguyên! ✅
```

### Read-Only Volumes

```bash
# Mount volume read-only
docker run -d \
  --name web \
  -v mydata:/app/data:ro \
  nginx

# Container không thể write vào volume
```

### Remove Volumes

```bash
# Xóa volume
docker volume rm mydata

# Xóa tất cả unused volumes
docker volume prune

# Xóa volume khi remove container
docker run --rm -v mydata:/data nginx
# Volume cũng bị xóa khi container stop
```

---

## 📁 Bind Mounts

### Cách Hoạt Động

```
┌─────────────────────────────────────────────────────────┐
│                    HOST MACHINE                         │
│                                                         │
│  /home/user/myapp/                                     │
│  ├── src/                                              │
│  │   ├── app.js                                       │
│  │   └── utils.js                                     │
│  │                                                     │
│  ┌──────────────────────────────────────────────────┐ │
│  │  Container                                       │ │
│  │  /app  ──────────────▶  /home/user/myapp/src    │ │
│  │                                                  │ │
│  │  Changes in container = Changes on host         │ │
│  └──────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

### Sử Dụng Bind Mounts

```bash
# Method 1: -v flag
docker run -d \
  --name web \
  -v /host/path:/container/path \
  nginx

# Method 2: --mount flag (recommended)
docker run -d \
  --name web \
  --mount type=bind,source=/host/path,target=/container/path \
  nginx

# Với current directory
docker run -d \
  --name web \
  -v $(pwd):/app \
  nginx

# Windows PowerShell
docker run -d -v ${PWD}:/app nginx
```

### Ví Dụ: Development Environment

```bash
# Mount source code cho hot reload
docker run -dit \
  --name dev-app \
  -v $(pwd)/src:/app/src \
  -v $(pwd)/package.json:/app/package.json \
  -p 3000:3000 \
  node:18-alpine \
  npm run dev

# Thay đổi code trên host → Tự động reload trong container
```

### Read-Only Bind Mounts

```bash
# Mount read-only
docker run -d \
  --name web \
  -v $(pwd)/config:/etc/nginx/conf.d:ro \
  nginx

# Container không thể modify files
```

### Bind Mount với Specific User

```bash
# Mount với ownership
docker run -d \
  --name web \
  --mount type=bind,source=$(pwd),target=/app,readonly \
  --user $(id -u):$(id -g) \
  nginx
```

---

## 💨 tmpfs Mounts (Linux Only)

### Cách Hoạt Động

```
┌─────────────────────────────────────────────────────────┐
│                    HOST MACHINE                         │
│                                                         │
│  ┌──────────────────────────────────────────────────┐ │
│  │  Container                                       │ │
│  │                                                  │ │
│  │  /app/cache  ──────────▶  RAM (tmpfs)          │ │
│  │                                                  │ │
│  │  - Stored in memory                             │ │
│  │  - Very fast                                    │ │
│  │  - Lost when container stops                   │ │
│  └──────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

### Sử Dụng tmpfs

```bash
# Mount tmpfs
docker run -d \
  --name web \
  --tmpfs /app/cache \
  nginx

# Với options
docker run -d \
  --name web \
  --tmpfs /app/cache:rw,size=100m,mode=1777 \
  nginx

# --mount syntax
docker run -d \
  --name web \
  --mount type=tmpfs,target=/app/cache,tmpfs-size=100m \
  nginx
```

### Use Cases

```bash
# 1. Temporary cache
docker run -d \
  --tmpfs /tmp \
  myapp

# 2. Sensitive data (không lưu vào disk)
docker run -d \
  --tmpfs /run/secrets:rw,size=10m \
  myapp

# 3. Build cache
docker run -d \
  --tmpfs /app/.cache:rw,size=500m \
  builder
```

---

## 🎯 Ví Dụ Thực Tế

### Ví Dụ 1: WordPress với MySQL

```bash
# 1. Tạo network
docker network create wordpress

# 2. MySQL với volume
docker run -d \
  --name mysql \
  --network wordpress \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=wordpress \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0

# 3. WordPress với volume
docker run -d \
  --name wordpress \
  --network wordpress \
  -p 80:80 \
  -e WORDPRESS_DB_HOST=mysql \
  -e WORDPRESS_DB_PASSWORD=secret \
  -v wordpress-data:/var/www/html \
  wordpress:latest

# Data được persist trong volumes:
# - mysql-data: Database
# - wordpress-data: WordPress files
```

### Ví Dụ 2: Development với Hot Reload

```bash
# Node.js app với hot reload
docker run -dit \
  --name dev \
  -v $(pwd):/app \
  -v /app/node_modules \
  -p 3000:3000 \
  -e NODE_ENV=development \
  node:18-alpine \
  sh -c "npm install && npm run dev"

# Giải thích:
# -v $(pwd):/app              → Mount source code
# -v /app/node_modules        → Anonymous volume cho node_modules
#                               (không override bằng host)
```

### Ví Dụ 3: Nginx với Config và Logs

```bash
# Nginx với custom config và persistent logs
docker run -d \
  --name nginx \
  -p 80:80 \
  -v $(pwd)/nginx.conf:/etc/nginx/nginx.conf:ro \
  -v $(pwd)/html:/usr/share/nginx/html:ro \
  -v nginx-logs:/var/log/nginx \
  nginx:alpine

# - nginx.conf: Read-only config
# - html: Read-only static files
# - nginx-logs: Persistent logs
```

### Ví Dụ 4: Multi-Container với Shared Volume

```bash
# Tạo shared volume
docker volume create shared-data

# Container 1: Writer
docker run -d \
  --name writer \
  -v shared-data:/data \
  alpine \
  sh -c "while true; do date >> /data/log.txt; sleep 5; done"

# Container 2: Reader
docker run -d \
  --name reader \
  -v shared-data:/data:ro \
  alpine \
  sh -c "tail -f /data/log.txt"

# Cả 2 containers share cùng volume
```

---

## 💾 Backup và Restore

### Backup Volume

```bash
# Method 1: Sử dụng container
docker run --rm \
  -v mydata:/data \
  -v $(pwd):/backup \
  alpine \
  tar czf /backup/mydata-backup.tar.gz -C /data .

# Method 2: Backup trực tiếp
docker run --rm \
  -v mydata:/source:ro \
  -v $(pwd):/backup \
  alpine \
  tar czf /backup/backup.tar.gz -C /source .
```

### Restore Volume

```bash
# Tạo volume mới
docker volume create mydata-restored

# Restore data
docker run --rm \
  -v mydata-restored:/data \
  -v $(pwd):/backup \
  alpine \
  tar xzf /backup/mydata-backup.tar.gz -C /data
```

### Copy Data Giữa Volumes

```bash
# Copy từ volume1 sang volume2
docker run --rm \
  -v volume1:/source:ro \
  -v volume2:/dest \
  alpine \
  sh -c "cp -av /source/. /dest/"
```

---

## 🔍 Inspect và Debug

### Xem Volume Usage

```bash
# List volumes
docker volume ls

# Inspect volume
docker volume inspect mydata

# Xem containers đang dùng volume
docker ps -a --filter volume=mydata

# Xem disk usage
docker system df -v
```

### Access Volume Data

```bash
# Method 1: Qua container
docker run --rm -it \
  -v mydata:/data \
  alpine \
  sh

# Method 2: Trực tiếp trên host (Linux)
sudo ls /var/lib/docker/volumes/mydata/_data

# Method 3: Docker Desktop (macOS/Windows)
docker run --rm -it \
  -v /var/lib/docker:/docker \
  alpine \
  ls /docker/volumes/mydata/_data
```

---

## 🎨 Docker Compose với Volumes

```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    volumes:
      # Named volume
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: secret

  web:
    image: nginx:alpine
    volumes:
      # Bind mount
      - ./html:/usr/share/nginx/html:ro
      # Named volume
      - nginx-logs:/var/log/nginx
    ports:
      - "80:80"

  app:
    image: node:18-alpine
    volumes:
      # Bind mount cho development
      - ./src:/app/src
      # Anonymous volume cho node_modules
      - /app/node_modules
      # tmpfs mount
      - type: tmpfs
        target: /app/cache
        tmpfs:
          size: 100m

volumes:
  # Declare named volumes
  pgdata:
  nginx-logs:
```

---

## 💡 Best Practices

### 1. Sử Dụng Named Volumes

```bash
# ✅ Named volume (dễ quản lý)
docker run -v mydata:/data nginx

# ❌ Anonymous volume (khó track)
docker run -v /data nginx
```

### 2. Volumes Cho Production Data

```bash
# ✅ Volumes cho databases
docker run -v pgdata:/var/lib/postgresql/data postgres

# ❌ Bind mounts cho databases
docker run -v $(pwd)/data:/var/lib/postgresql/data postgres
```

### 3. Bind Mounts Cho Development

```bash
# ✅ Bind mount cho source code
docker run -v $(pwd)/src:/app/src node

# ❌ Volume cho source code (không sync)
docker run -v src:/app/src node
```

### 4. Read-Only Khi Có Thể

```bash
# ✅ Config files read-only
docker run -v $(pwd)/config:/etc/app:ro myapp

# ❌ Writable config
docker run -v $(pwd)/config:/etc/app myapp
```

### 5. Backup Thường Xuyên

```bash
# ✅ Automated backup
docker run --rm \
  -v mydata:/data:ro \
  -v $(pwd)/backups:/backup \
  alpine \
  tar czf /backup/backup-$(date +%Y%m%d).tar.gz -C /data .
```

### 6. Cleanup Unused Volumes

```bash
# ✅ Định kỳ cleanup
docker volume prune

# Xem volumes không dùng
docker volume ls -f dangling=true
```

---

## 🎓 Bài Tập Thực Hành

### Bài 1: Database Persistence

```bash
# 1. Chạy PostgreSQL với volume
# 2. Tạo database và table
# 3. Insert data
# 4. Stop và remove container
# 5. Start lại container với cùng volume
# 6. Verify data vẫn còn
```

### Bài 2: Development Environment

```bash
# 1. Tạo Node.js app
# 2. Mount source code với bind mount
# 3. Mount node_modules với anonymous volume
# 4. Thay đổi code và verify hot reload
```

### Bài 3: Backup và Restore

```bash
# 1. Tạo volume với data
# 2. Backup volume ra file .tar.gz
# 3. Xóa volume
# 4. Restore từ backup
# 5. Verify data
```

---

## 📚 Tóm Tắt

### Storage Types

| Type | Command | Use Case |
|------|---------|----------|
| **Volume** | `-v name:/path` | Production data |
| **Bind Mount** | `-v /host:/container` | Development |
| **tmpfs** | `--tmpfs /path` | Temporary data |

### Key Commands

```bash
docker volume create        # Tạo volume
docker volume ls           # List volumes
docker volume inspect      # Xem chi tiết
docker volume rm           # Xóa volume
docker volume prune        # Xóa unused volumes
```

---

## 📚 Bài Tiếp Theo

Trong bài tiếp theo, chúng ta sẽ học về **Environment Variables** - cách quản lý configuration.

👉 [Bài 13: Environment Variables](./13-environment-variables.md)

---

## 🔗 Tài Liệu Tham Khảo

- [Docker Volumes](https://docs.docker.com/storage/volumes/)
- [Bind Mounts](https://docs.docker.com/storage/bind-mounts/)
- [tmpfs Mounts](https://docs.docker.com/storage/tmpfs/)

---

**Happy Storing! 💾**

