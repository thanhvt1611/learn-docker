# Bài 20: Troubleshooting Docker

## 📋 Mục Tiêu

Hướng dẫn debug và giải quyết các vấn đề thường gặp khi làm việc với Docker.

---

## 🔍 Container Issues

### Container Không Start

**Triệu chứng:**
```bash
docker ps
# Container không xuất hiện
```

**Debug:**
```bash
# 1. Xem tất cả containers (bao gồm stopped)
docker ps -a

# 2. Xem logs
docker logs container-name

# 3. Inspect container
docker inspect container-name

# 4. Xem exit code
docker inspect container-name --format='{{.State.ExitCode}}'
```

**Nguyên nhân thường gặp:**

**1. Application crash:**
```bash
# Xem logs để tìm error
docker logs container-name

# Common errors:
# - Missing environment variables
# - Database connection failed
# - Port already in use
```

**2. Wrong command:**
```dockerfile
# ❌ Command không tồn tại
CMD ["nod", "server.js"]  # Typo: nod instead of node

# ✅ Correct command
CMD ["node", "server.js"]
```

**3. Missing dependencies:**
```dockerfile
# ❌ Missing npm install
FROM node:18-alpine
COPY . .
CMD ["node", "server.js"]  # Fail: no node_modules

# ✅ Install dependencies
FROM node:18-alpine
COPY package*.json ./
RUN npm ci
COPY . .
CMD ["node", "server.js"]
```

---

### Container Exits Immediately

**Debug:**
```bash
# Run với interactive mode để xem error
docker run -it container-name sh

# Hoặc override command
docker run -it container-name /bin/sh
```

**Nguyên nhân:**

**1. No foreground process:**
```dockerfile
# ❌ Process chạy background
CMD service nginx start  # Exits immediately

# ✅ Foreground process
CMD ["nginx", "-g", "daemon off;"]
```

**2. Script exits:**
```bash
#!/bin/sh
echo "Starting..."
# Script ends here, container exits

# ✅ Keep running
#!/bin/sh
echo "Starting..."
exec "$@"  # Execute main command
```

---

### Container Restart Loop

**Triệu chứng:**
```bash
docker ps
# STATUS: Restarting (1) 5 seconds ago
```

**Debug:**
```bash
# Xem logs
docker logs -f container-name

# Xem restart count
docker inspect container-name --format='{{.RestartCount}}'

# Disable restart để debug
docker update --restart=no container-name
```

**Giải pháp:**
```yaml
# Tăng start period cho health check
services:
  app:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      start_period: 30s  # Tăng thời gian khởi động
```

---

## 🌐 Network Issues

### Container Không Kết Nối Được

**Debug:**
```bash
# 1. Kiểm tra container có IP không
docker inspect container-name | grep IPAddress

# 2. Kiểm tra network
docker network inspect network-name

# 3. Ping giữa containers
docker exec container1 ping container2

# 4. Check DNS resolution
docker exec container1 nslookup container2

# 5. Test port connectivity
docker exec container1 nc -zv container2 3000
```

**Nguyên nhân:**

**1. Không cùng network:**
```bash
# Kiểm tra network
docker inspect container1 | grep NetworkID
docker inspect container2 | grep NetworkID

# Nếu khác nhau, connect vào cùng network
docker network connect mynetwork container2
```

**2. Default bridge không có DNS:**
```bash
# ❌ Default bridge
docker run -d --name web nginx
docker run -d --name app node
docker exec app ping web  # Fail: cannot resolve

# ✅ Custom network
docker network create mynet
docker run -d --name web --network mynet nginx
docker run -d --name app --network mynet node
docker exec app ping web  # Success
```

**3. Firewall blocking:**
```bash
# Check firewall rules
sudo iptables -L

# Allow Docker networks
sudo iptables -A INPUT -i docker0 -j ACCEPT
```

---

### Port Mapping Không Hoạt Động

**Debug:**
```bash
# 1. Kiểm tra port mapping
docker port container-name

# 2. Kiểm tra port đang listen
docker exec container-name netstat -tlnp

# 3. Test từ host
curl http://localhost:8080

# 4. Check process listening
sudo lsof -i :8080
```

**Nguyên nhân:**

**1. Port conflict:**
```bash
# Error: port is already allocated
# Giải pháp: Dùng port khác
docker run -d -p 8081:80 nginx  # Thay vì 8080
```

**2. Application không listen đúng interface:**
```javascript
// ❌ Chỉ listen localhost
app.listen(3000, 'localhost');

// ✅ Listen tất cả interfaces
app.listen(3000, '0.0.0.0');
```

**3. Wrong port mapping:**
```bash
# ❌ Sai thứ tự
docker run -d -p 80:8080 nginx  # host:container

# ✅ Đúng
docker run -d -p 8080:80 nginx
```

---

## 💾 Volume Issues

### Data Không Persist

**Debug:**
```bash
# 1. Kiểm tra volume
docker volume ls

# 2. Inspect volume
docker volume inspect volume-name

# 3. Kiểm tra mount point
docker inspect container-name --format='{{json .Mounts}}' | jq
```

**Nguyên nhân:**

**1. Anonymous volume:**
```bash
# ❌ Anonymous volume (mỗi lần tạo container mới)
docker run -d -v /data postgres

# ✅ Named volume
docker run -d -v pgdata:/data postgres
```

**2. Wrong mount path:**
```bash
# ❌ Sai path
docker run -d -v pgdata:/var/lib/postgres postgres

# ✅ Đúng path
docker run -d -v pgdata:/var/lib/postgresql/data postgres
```

---

### Permission Denied

**Debug:**
```bash
# Kiểm tra permissions
docker exec container-name ls -la /data

# Kiểm tra user
docker exec container-name whoami
```

**Giải pháp:**

**1. Fix ownership:**
```dockerfile
FROM node:18-alpine

WORKDIR /app

RUN addgroup -g 1001 nodejs && \
    adduser -S nodejs -u 1001

COPY --chown=nodejs:nodejs . .

USER nodejs
```

**2. Volume permissions:**
```bash
# Chạy với specific user
docker run -d \
  --user $(id -u):$(id -g) \
  -v $(pwd):/app \
  myapp
```

---

## 🏗️ Build Issues

### Build Fails

**Debug:**
```bash
# Build với verbose output
docker build --progress=plain --no-cache -t myapp .

# Build specific stage
docker build --target builder -t myapp:builder .
```

**Nguyên nhân:**

**1. Context too large:**
```bash
# Error: Sending build context to Docker daemon  2.5GB

# Giải pháp: .dockerignore
node_modules
.git
*.log
dist
build
```

**2. Network timeout:**
```dockerfile
# ❌ Timeout khi download
RUN npm install

# ✅ Tăng timeout
RUN npm install --network-timeout=100000
```

**3. Cache issues:**
```bash
# Build không dùng cache
docker build --no-cache -t myapp .

# Hoặc prune build cache
docker builder prune
```

---

### Image Size Quá Lớn

**Debug:**
```bash
# Xem image size
docker images myapp

# Xem layers
docker history myapp

# Analyze với dive
dive myapp
```

**Giải pháp:**

**1. Multi-stage build:**
```dockerfile
# Before: 1.2GB
FROM node:18
COPY . .
RUN npm install
RUN npm run build
CMD ["node", "dist/server.js"]

# After: 150MB
FROM node:18-alpine AS builder
COPY . .
RUN npm ci && npm run build

FROM node:18-alpine
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/server.js"]
```

**2. Clean up trong cùng layer:**
```dockerfile
# ❌ Cleanup ở layer khác (không giảm size)
RUN apt-get update
RUN apt-get install -y build-essential
RUN npm install
RUN apt-get purge -y build-essential

# ✅ Cleanup trong cùng layer
RUN apt-get update && \
    apt-get install -y build-essential && \
    npm install && \
    apt-get purge -y build-essential && \
    rm -rf /var/lib/apt/lists/*
```

---

## 🔧 Docker Daemon Issues

### Docker Daemon Không Start

**Debug:**
```bash
# Check status
sudo systemctl status docker

# View logs
sudo journalctl -u docker

# Check config
sudo dockerd --validate
```

**Giải pháp:**

**1. Fix daemon.json:**
```bash
# Validate JSON
cat /etc/docker/daemon.json | jq

# Restart daemon
sudo systemctl restart docker
```

**2. Disk space:**
```bash
# Check disk space
df -h

# Clean up
docker system prune -a
```

---

### Out of Disk Space

**Debug:**
```bash
# Xem disk usage
docker system df

# Xem chi tiết
docker system df -v
```

**Giải pháp:**
```bash
# Remove unused containers
docker container prune

# Remove unused images
docker image prune -a

# Remove unused volumes
docker volume prune

# Remove everything
docker system prune -a --volumes
```

---

## 🐛 Common Errors

### Error: "Cannot connect to Docker daemon"

```bash
# Check Docker is running
sudo systemctl status docker

# Start Docker
sudo systemctl start docker

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker
```

---

### Error: "No space left on device"

```bash
# Check disk space
df -h

# Clean Docker
docker system prune -a --volumes

# Change Docker data directory
# /etc/docker/daemon.json
{
  "data-root": "/new/path/docker"
}
```

---

### Error: "OCI runtime create failed"

```bash
# Common causes:
# 1. Invalid command
# 2. Missing dependencies
# 3. Permission issues

# Debug
docker run -it --entrypoint sh myapp
```

---

### Error: "Conflict. The container name is already in use"

```bash
# Remove existing container
docker rm container-name

# Or use different name
docker run --name container-name-2 myapp
```

---

## 🛠️ Debugging Tools

### docker logs

```bash
# View logs
docker logs container-name

# Follow logs
docker logs -f container-name

# Tail logs
docker logs --tail 100 container-name

# Logs với timestamps
docker logs -t container-name

# Logs từ thời điểm cụ thể
docker logs --since 2024-01-01 container-name
docker logs --since 1h container-name
```

### docker exec

```bash
# Shell vào container
docker exec -it container-name sh

# Run command
docker exec container-name ls -la

# Run as root
docker exec -u root container-name sh
```

### docker inspect

```bash
# Full info
docker inspect container-name

# Specific field
docker inspect container-name --format='{{.State.Status}}'
docker inspect container-name --format='{{.NetworkSettings.IPAddress}}'

# JSON output
docker inspect container-name | jq
```

### docker stats

```bash
# Real-time resource usage
docker stats

# Specific containers
docker stats container1 container2

# No stream
docker stats --no-stream
```

### docker events

```bash
# Monitor Docker events
docker events

# Filter events
docker events --filter 'type=container'
docker events --filter 'event=start'
```

---

## 📊 Performance Issues

### High CPU Usage

**Debug:**
```bash
# Check CPU usage
docker stats

# Top processes trong container
docker exec container-name top

# Profile application
docker exec container-name node --prof app.js
```

**Giải pháp:**
```yaml
# Set CPU limits
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '2'
```

---

### High Memory Usage

**Debug:**
```bash
# Check memory
docker stats

# Memory info
docker exec container-name free -m
```

**Giải pháp:**
```yaml
# Set memory limits
services:
  app:
    deploy:
      resources:
        limits:
          memory: 2G
```

---

## 💡 Debugging Checklist

### Container Issues
- [ ] Check logs: `docker logs`
- [ ] Check status: `docker ps -a`
- [ ] Inspect container: `docker inspect`
- [ ] Check resources: `docker stats`
- [ ] Shell into container: `docker exec -it`

### Network Issues
- [ ] Check IP: `docker inspect | grep IPAddress`
- [ ] Check network: `docker network inspect`
- [ ] Test connectivity: `ping`, `curl`, `nc`
- [ ] Check DNS: `nslookup`
- [ ] Check ports: `docker port`

### Volume Issues
- [ ] List volumes: `docker volume ls`
- [ ] Inspect volume: `docker volume inspect`
- [ ] Check permissions: `ls -la`
- [ ] Check mounts: `docker inspect | grep Mounts`

### Build Issues
- [ ] Check .dockerignore
- [ ] Build without cache: `--no-cache`
- [ ] Check context size
- [ ] Validate Dockerfile: `hadolint`

---

## 🔗 Tài Liệu Tham Khảo

- [Docker Troubleshooting](https://docs.docker.com/config/daemon/troubleshoot/)
- [Docker Logs](https://docs.docker.com/config/containers/logging/)
- [Docker Debug](https://docs.docker.com/engine/reference/commandline/debug/)

---

## 🎓 Kết Thúc Khóa Học

Chúc mừng! Bạn đã hoàn thành toàn bộ khóa học Docker! 🎉

**Bạn đã học:**
- ✅ Docker cơ bản và nâng cao
- ✅ Dockerfile và image optimization
- ✅ Container management
- ✅ Networking và volumes
- ✅ Docker Compose
- ✅ Production best practices
- ✅ Security
- ✅ Troubleshooting

**Bước tiếp theo:**
1. Thực hành với các projects thực tế
2. Deploy ứng dụng lên production
3. Tìm hiểu Kubernetes (container orchestration)
4. Tham gia Docker community

👉 [Quay lại README](../README.md)

---

**Happy Dockering! 🐳**

