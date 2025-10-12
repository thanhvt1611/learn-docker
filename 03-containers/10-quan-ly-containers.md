# Bài 10: Quản Lý Containers

## 📋 Mục Tiêu Bài Học

Sau bài học này, bạn sẽ:
- ✅ Hiểu container lifecycle chi tiết
- ✅ Quản lý containers hiệu quả
- ✅ Sử dụng restart policies
- ✅ Implement health checks
- ✅ Giới hạn tài nguyên containers

---

## 🔄 Container Lifecycle

### Các Trạng Thái Container

```
┌──────────┐
│  Image   │
└────┬─────┘
     │ docker run / docker create
     ▼
┌──────────┐
│ Created  │ ← Container được tạo nhưng chưa start
└────┬─────┘
     │ docker start
     ▼
┌──────────┐     docker pause      ┌──────────┐
│ Running  │ ◄──────────────────── │  Paused  │
└────┬─────┘ ──────────────────▶   └──────────┘
     │           docker unpause
     │
     │ docker stop / docker kill
     ▼
┌──────────┐
│ Stopped  │ ← Container dừng, có thể start lại
└────┬─────┘
     │ docker rm
     ▼
┌──────────┐
│ Removed  │ ← Container bị xóa hoàn toàn
└──────────┘
```

### Lifecycle Commands

```bash
# Tạo container (không start)
docker create --name myapp nginx

# Tạo và start ngay
docker run --name myapp nginx

# Start container đã stopped
docker start myapp

# Stop container (graceful shutdown)
docker stop myapp

# Kill container (force stop)
docker kill myapp

# Restart container
docker restart myapp

# Pause container (freeze processes)
docker pause myapp

# Unpause container
docker unpause myapp

# Remove container
docker rm myapp

# Remove container đang chạy
docker rm -f myapp
```

---

## 🚀 Chạy Containers

### docker run - Chi Tiết

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

### Run Modes

#### 1. Foreground Mode (Default)

```bash
# Chạy ở foreground, attach STDOUT/STDERR
docker run nginx

# Output hiển thị trực tiếp
# Ctrl+C để stop
```

#### 2. Detached Mode (-d)

```bash
# Chạy ở background
docker run -d nginx

# Output: container ID
a1b2c3d4e5f6...

# Container chạy background, terminal free
```

#### 3. Interactive Mode (-it)

```bash
# Interactive + TTY
docker run -it ubuntu bash

# Bạn vào shell của container
root@a1b2c3d4e5f6:/# ls
root@a1b2c3d4e5f6:/# exit
```

#### 4. Interactive + Detached

```bash
# Chạy background nhưng có thể attach sau
docker run -dit --name myubuntu ubuntu bash

# Attach vào container
docker attach myubuntu

# Detach mà không stop: Ctrl+P, Ctrl+Q
```

### Common Options

```bash
# Đặt tên container
docker run --name web nginx

# Port mapping
docker run -p 8080:80 nginx

# Environment variables
docker run -e NODE_ENV=production node

# Volume mount
docker run -v /host/path:/container/path nginx

# Network
docker run --network mynetwork nginx

# Auto remove khi stop
docker run --rm nginx

# Restart policy
docker run --restart unless-stopped nginx

# Resource limits
docker run --memory="512m" --cpus="1.5" nginx

# Working directory
docker run -w /app node

# User
docker run --user node node

# Hostname
docker run --hostname myapp nginx
```

---

## 🔁 Restart Policies

### Các Loại Restart Policies

```bash
# 1. no (default) - Không tự động restart
docker run --restart no nginx

# 2. on-failure - Restart khi exit code != 0
docker run --restart on-failure nginx

# 3. on-failure với max retries
docker run --restart on-failure:5 nginx

# 4. always - Luôn restart (kể cả khi stop manually)
docker run --restart always nginx

# 5. unless-stopped - Restart trừ khi stop manually
docker run --restart unless-stopped nginx
```

### So Sánh Restart Policies

| Policy | Container Crashes | Manual Stop | Docker Restart |
|--------|------------------|-------------|----------------|
| `no` | ❌ Không restart | ❌ | ❌ |
| `on-failure` | ✅ Restart | ❌ | ❌ |
| `always` | ✅ Restart | ✅ Restart | ✅ Restart |
| `unless-stopped` | ✅ Restart | ❌ | ✅ Restart |

### Ví Dụ Thực Tế

```bash
# Web server - luôn chạy
docker run -d \
  --name web \
  --restart unless-stopped \
  -p 80:80 \
  nginx

# Database - luôn chạy
docker run -d \
  --name db \
  --restart always \
  -e POSTGRES_PASSWORD=secret \
  postgres

# Worker job - retry khi fail
docker run -d \
  --name worker \
  --restart on-failure:3 \
  myapp:worker

# Development - không restart
docker run -d \
  --name dev \
  --restart no \
  myapp:dev
```

### Update Restart Policy

```bash
# Update restart policy của container đang chạy
docker update --restart unless-stopped myapp

# Update nhiều containers
docker update --restart always web db cache
```

---

## 🏥 Health Checks

### Định Nghĩa Health Check

**Health check** kiểm tra container có healthy không, tự động restart nếu unhealthy.

### Health Check Trong Dockerfile

```dockerfile
FROM nginx:alpine

# Health check mỗi 30s
HEALTHCHECK --interval=30s \
            --timeout=3s \
            --start-period=5s \
            --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost/ || exit 1
```

**Options:**
- `--interval`: Thời gian giữa các lần check (default: 30s)
- `--timeout`: Timeout cho mỗi check (default: 30s)
- `--start-period`: Thời gian khởi động (default: 0s)
- `--retries`: Số lần retry trước khi unhealthy (default: 3)

### Health Check Khi Run

```bash
# Override health check từ Dockerfile
docker run -d \
  --name web \
  --health-cmd="curl -f http://localhost/ || exit 1" \
  --health-interval=30s \
  --health-timeout=3s \
  --health-retries=3 \
  --health-start-period=5s \
  nginx

# Disable health check
docker run -d --no-healthcheck nginx
```

### Ví Dụ Health Checks

#### Node.js API

```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY . .
RUN npm ci --only=production

EXPOSE 3000

# Health check endpoint
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
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

const request = http.request(options, (res) => {
  console.log(`STATUS: ${res.statusCode}`);
  process.exit(res.statusCode === 200 ? 0 : 1);
});

request.on('error', (err) => {
  console.error('ERROR:', err);
  process.exit(1);
});

request.end();
```

#### Python Flask

```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .

EXPOSE 5000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD python -c "import requests; requests.get('http://localhost:5000/health', timeout=2)" || exit 1

CMD ["python", "app.py"]
```

#### Database (PostgreSQL)

```bash
docker run -d \
  --name postgres \
  --health-cmd="pg_isready -U postgres" \
  --health-interval=10s \
  --health-timeout=5s \
  --health-retries=5 \
  -e POSTGRES_PASSWORD=secret \
  postgres:15
```

### Kiểm Tra Health Status

```bash
# Xem health status
docker ps
# CONTAINER ID   IMAGE     STATUS
# a1b2c3d4e5f6   nginx     Up 2 minutes (healthy)

# Xem chi tiết health check
docker inspect --format='{{json .State.Health}}' web | jq

# Output:
{
  "Status": "healthy",
  "FailingStreak": 0,
  "Log": [
    {
      "Start": "2024-01-01T10:00:00Z",
      "End": "2024-01-01T10:00:01Z",
      "ExitCode": 0,
      "Output": "..."
    }
  ]
}
```

---

## 💻 Resource Limits

### Memory Limits

```bash
# Giới hạn memory
docker run -d --memory="512m" nginx

# Memory + swap
docker run -d --memory="512m" --memory-swap="1g" nginx

# Memory reservation (soft limit)
docker run -d --memory-reservation="256m" nginx

# OOM killer disable
docker run -d --memory="512m" --oom-kill-disable nginx
```

### CPU Limits

```bash
# Giới hạn số CPUs
docker run -d --cpus="1.5" nginx

# CPU shares (relative weight)
docker run -d --cpu-shares=512 nginx

# CPU period và quota
docker run -d --cpu-period=100000 --cpu-quota=50000 nginx

# Chỉ định CPU cores cụ thể
docker run -d --cpuset-cpus="0,1" nginx
```

### Disk I/O Limits

```bash
# Block IO weight
docker run -d --blkio-weight=500 nginx

# Read/Write BPS limit
docker run -d \
  --device-read-bps=/dev/sda:1mb \
  --device-write-bps=/dev/sda:1mb \
  nginx

# Read/Write IOPS limit
docker run -d \
  --device-read-iops=/dev/sda:1000 \
  --device-write-iops=/dev/sda:1000 \
  nginx
```

### Ví Dụ Resource Limits

```bash
# Web server với resource limits
docker run -d \
  --name web \
  --memory="1g" \
  --memory-reservation="512m" \
  --cpus="2" \
  --restart unless-stopped \
  -p 80:80 \
  nginx

# Database với resource limits
docker run -d \
  --name db \
  --memory="2g" \
  --cpus="4" \
  --blkio-weight=1000 \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# Worker với limited resources
docker run -d \
  --name worker \
  --memory="512m" \
  --cpus="0.5" \
  --restart on-failure:3 \
  myapp:worker
```

### Update Resource Limits

```bash
# Update memory limit
docker update --memory="1g" web

# Update CPU limit
docker update --cpus="2" web

# Update nhiều containers
docker update --memory="512m" --cpus="1" worker1 worker2 worker3
```

---

## 📊 Monitoring Containers

### docker stats

```bash
# Xem resource usage real-time
docker stats

# Xem specific containers
docker stats web db cache

# Xem 1 lần (không stream)
docker stats --no-stream

# Format output
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

### docker top

```bash
# Xem processes trong container
docker top web

# Với custom format
docker top web aux
```

### docker logs

```bash
# Xem logs
docker logs web

# Follow logs
docker logs -f web

# Tail logs
docker logs --tail 100 web

# Logs với timestamps
docker logs -t web

# Logs từ thời điểm cụ thể
docker logs --since 2024-01-01 web
docker logs --since 1h web
```

---

## 🎯 Ví Dụ Thực Hành

### Ví Dụ 1: Production Web Server

```bash
# Chạy nginx với full configuration
docker run -d \
  --name production-web \
  --restart unless-stopped \
  --memory="1g" \
  --cpus="2" \
  --health-cmd="curl -f http://localhost/ || exit 1" \
  --health-interval=30s \
  --health-timeout=3s \
  --health-retries=3 \
  -p 80:80 \
  -p 443:443 \
  -v /etc/nginx/conf.d:/etc/nginx/conf.d:ro \
  -v /var/www/html:/usr/share/nginx/html:ro \
  nginx:alpine

# Kiểm tra status
docker ps
docker stats production-web --no-stream
docker logs -f production-web
```

### Ví Dụ 2: Database Container

```bash
# PostgreSQL với resource limits và health check
docker run -d \
  --name postgres-db \
  --restart always \
  --memory="4g" \
  --cpus="4" \
  --health-cmd="pg_isready -U postgres" \
  --health-interval=10s \
  --health-timeout=5s \
  --health-retries=5 \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=myapp \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15

# Monitor
docker stats postgres-db
docker logs -f postgres-db
```

### Ví Dụ 3: Development Container

```bash
# Development container với auto-reload
docker run -dit \
  --name dev-app \
  --restart no \
  --memory="512m" \
  --cpus="1" \
  -p 3000:3000 \
  -v $(pwd):/app \
  -w /app \
  -e NODE_ENV=development \
  node:18-alpine \
  npm run dev

# Attach vào container
docker attach dev-app

# Hoặc exec command
docker exec -it dev-app sh
```

---

## 💡 Best Practices

### 1. Luôn Đặt Tên Container

```bash
# ❌ Không đặt tên
docker run -d nginx

# ✅ Đặt tên rõ ràng
docker run -d --name production-web nginx
```

### 2. Sử Dụng Restart Policies

```bash
# ✅ Production containers
docker run -d --restart unless-stopped nginx

# ✅ Critical services
docker run -d --restart always postgres
```

### 3. Implement Health Checks

```bash
# ✅ Luôn có health check
docker run -d \
  --health-cmd="curl -f http://localhost/ || exit 1" \
  nginx
```

### 4. Set Resource Limits

```bash
# ✅ Giới hạn resources
docker run -d \
  --memory="1g" \
  --cpus="2" \
  nginx
```

### 5. Use --rm Cho Temporary Containers

```bash
# ✅ Auto cleanup
docker run --rm -it ubuntu bash
```

---

## 🎓 Bài Tập Thực Hành

### Bài 1: Container Lifecycle

```bash
# 1. Tạo container không start
# 2. Start container
# 3. Pause container
# 4. Unpause container
# 5. Stop container
# 6. Start lại
# 7. Remove container
```

### Bài 2: Health Check

```bash
# Tạo Dockerfile với health check cho:
# - Node.js API với endpoint /health
# - Check mỗi 30s
# - Timeout 3s
# - Retry 3 lần
```

### Bài 3: Resource Management

```bash
# Chạy 3 containers với resource limits:
# - Web: 1GB RAM, 2 CPUs
# - API: 512MB RAM, 1 CPU
# - Worker: 256MB RAM, 0.5 CPU
# Monitor resource usage
```

---

## 📚 Tóm Tắt

### Key Commands

```bash
docker run              # Tạo và start container
docker start/stop       # Start/Stop container
docker restart          # Restart container
docker pause/unpause    # Pause/Unpause container
docker rm               # Remove container
docker stats            # Monitor resources
docker logs             # Xem logs
docker top              # Xem processes
```

### Key Concepts

1. **Lifecycle**: Created → Running → Stopped → Removed
2. **Restart Policies**: no, on-failure, always, unless-stopped
3. **Health Checks**: Tự động kiểm tra container health
4. **Resource Limits**: Memory, CPU, Disk I/O

---

## 📚 Bài Tiếp Theo

Trong bài tiếp theo, chúng ta sẽ học về **Docker Networking** - cách containers giao tiếp với nhau.

👉 [Bài 11: Networking](./11-networking.md)

---

## 🔗 Tài Liệu Tham Khảo

- [Docker Run Reference](https://docs.docker.com/engine/reference/run/)
- [Container Lifecycle](https://docs.docker.com/engine/reference/commandline/container/)
- [Health Checks](https://docs.docker.com/engine/reference/builder/#healthcheck)
- [Resource Constraints](https://docs.docker.com/config/containers/resource_constraints/)

---

**Happy Managing! 🚀**

