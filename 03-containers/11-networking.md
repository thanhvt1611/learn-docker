# Bài 11: Docker Networking

## 📋 Mục Tiêu Bài Học

Sau bài học này, bạn sẽ:
- ✅ Hiểu các loại Docker networks
- ✅ Tạo và quản lý custom networks
- ✅ Kết nối containers với nhau
- ✅ Expose ports và port mapping
- ✅ Troubleshoot network issues

---

## 🌐 Docker Network Drivers

Docker hỗ trợ 5 loại network drivers:

```
┌─────────────────────────────────────────────────────────┐
│  1. bridge    - Default, containers trên cùng host     │
│  2. host      - Dùng network của host                  │
│  3. overlay   - Multi-host networking (Swarm)          │
│  4. macvlan   - Assign MAC address cho container       │
│  5. none      - Disable networking                     │
└─────────────────────────────────────────────────────────┘
```

---

## 🌉 Bridge Network (Default)

### Cách Hoạt Động

```
┌─────────────────────────────────────────────────────────┐
│                    HOST MACHINE                         │
│                                                         │
│  ┌───────────────────────────────────────────────────┐ │
│  │         Docker Bridge Network (172.17.0.0/16)     │ │
│  │                                                   │ │
│  │  ┌──────────────┐  ┌──────────────┐             │ │
│  │  │ Container 1  │  │ Container 2  │             │ │
│  │  │ 172.17.0.2   │  │ 172.17.0.3   │             │ │
│  │  └──────┬───────┘  └──────┬───────┘             │ │
│  │         │                  │                     │ │
│  │         └──────────┬───────┘                     │ │
│  │                    │                             │ │
│  │              docker0 bridge                      │ │
│  │              172.17.0.1                          │ │
│  └────────────────────┼───────────────────────────────┘ │
│                       │                                 │
│                  Host Network                           │
│                  (eth0: 192.168.1.100)                  │
└───────────────────────┼─────────────────────────────────┘
                        │
                   Internet
```

### Default Bridge

```bash
# Chạy container trên default bridge
docker run -d --name web nginx

# Kiểm tra network
docker inspect web | grep IPAddress
# "IPAddress": "172.17.0.2"

# Containers trên default bridge KHÔNG thể ping nhau bằng tên
docker run -d --name app node
docker exec web ping app  # ❌ Không work
```

### Custom Bridge Network

```bash
# Tạo custom bridge network
docker network create mynetwork

# Chạy containers trên custom network
docker run -d --name web --network mynetwork nginx
docker run -d --name app --network mynetwork node

# Containers có thể ping nhau bằng tên (DNS)
docker exec web ping app  # ✅ Work!
docker exec app ping web  # ✅ Work!
```

### Ví Dụ: Web + Database

```bash
# 1. Tạo network
docker network create app-network

# 2. Chạy database
docker run -d \
  --name postgres \
  --network app-network \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# 3. Chạy web app
docker run -d \
  --name web \
  --network app-network \
  -p 3000:3000 \
  -e DATABASE_URL=postgresql://postgres:secret@postgres:5432/mydb \
  myapp:latest

# Web app có thể connect đến database qua hostname "postgres"
```

---

## 🖥️ Host Network

### Cách Hoạt Động

```
┌─────────────────────────────────────────────────────────┐
│                    HOST MACHINE                         │
│                  (192.168.1.100)                        │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Container (host network)                        │  │
│  │  - Dùng trực tiếp network của host              │  │
│  │  - Không có network isolation                   │  │
│  │  - Port conflicts với host                      │  │
│  └──────────────────────────────────────────────────┘  │
│                                                         │
│                  Host Network Stack                     │
│                  eth0: 192.168.1.100                    │
└─────────────────────────────────────────────────────────┘
```

### Sử Dụng Host Network

```bash
# Chạy container với host network
docker run -d --network host nginx

# Container bind trực tiếp port 80 của host
# Không cần -p flag
# curl http://localhost:80 → nginx

# ⚠️ Lưu ý: Chỉ work trên Linux
# macOS và Windows không hỗ trợ host network
```

### Khi Nào Dùng Host Network?

✅ **Dùng khi:**
- Cần performance cao (no NAT overhead)
- Cần bind nhiều ports
- Network-intensive applications

❌ **Không dùng khi:**
- Cần network isolation
- Chạy nhiều containers cùng port
- Trên macOS/Windows

---

## 🔗 Overlay Network (Multi-Host)

### Cách Hoạt Động

```
┌─────────────────────┐         ┌─────────────────────┐
│    Host 1           │         │    Host 2           │
│                     │         │                     │
│  ┌──────────────┐   │         │  ┌──────────────┐   │
│  │ Container A  │   │         │  │ Container B  │   │
│  │ 10.0.0.2     │   │         │  │ 10.0.0.3     │   │
│  └──────┬───────┘   │         │  └──────┬───────┘   │
│         │           │         │         │           │
│    Overlay Network  │◄───────►│    Overlay Network  │
│    (10.0.0.0/24)    │  VXLAN  │    (10.0.0.0/24)    │
└─────────────────────┘         └─────────────────────┘
```

### Tạo Overlay Network (Docker Swarm)

```bash
# Initialize swarm
docker swarm init

# Tạo overlay network
docker network create -d overlay myoverlay

# Deploy service trên overlay network
docker service create \
  --name web \
  --network myoverlay \
  --replicas 3 \
  nginx
```

---

## 🚫 None Network

```bash
# Container không có network
docker run -d --network none nginx

# Container hoàn toàn isolated
# Không có network interface (chỉ có loopback)
```

---

## 🔧 Quản Lý Networks

### Tạo Network

```bash
# Tạo bridge network (default driver)
docker network create mynetwork

# Tạo với driver cụ thể
docker network create -d bridge mybridge
docker network create -d overlay myoverlay

# Tạo với subnet và gateway
docker network create \
  --subnet=172.20.0.0/16 \
  --gateway=172.20.0.1 \
  mynetwork

# Tạo với IP range
docker network create \
  --subnet=172.20.0.0/16 \
  --ip-range=172.20.240.0/20 \
  mynetwork
```

### List Networks

```bash
# List tất cả networks
docker network ls

# Output:
NETWORK ID     NAME      DRIVER    SCOPE
abc123def456   bridge    bridge    local
def456ghi789   host      host      local
ghi789jkl012   none      null      local
jkl012mno345   mynet     bridge    local
```

### Inspect Network

```bash
# Xem chi tiết network
docker network inspect mynetwork

# Xem containers trong network
docker network inspect mynetwork --format='{{range .Containers}}{{.Name}} {{end}}'
```

### Remove Network

```bash
# Xóa network
docker network rm mynetwork

# Xóa tất cả unused networks
docker network prune

# Xóa với force
docker network rm -f mynetwork
```

---

## 🔌 Connect/Disconnect Containers

### Connect Container Vào Network

```bash
# Chạy container trên network
docker run -d --name web --network mynetwork nginx

# Hoặc connect sau khi chạy
docker run -d --name web nginx
docker network connect mynetwork web

# Connect với IP cụ thể
docker network connect --ip 172.20.0.10 mynetwork web

# Connect với alias
docker network connect --alias webserver mynetwork web
```

### Disconnect Container Khỏi Network

```bash
# Disconnect
docker network disconnect mynetwork web

# Force disconnect
docker network disconnect -f mynetwork web
```

### Container Trên Nhiều Networks

```bash
# Container có thể ở nhiều networks
docker network create frontend
docker network create backend

# Web server ở cả frontend và backend
docker run -d --name web nginx
docker network connect frontend web
docker network connect backend web

# API chỉ ở backend
docker run -d --name api --network backend myapi

# Database chỉ ở backend
docker run -d --name db --network backend postgres
```

---

## 🔓 Port Mapping

### Publish Ports

```bash
# Map port host:container
docker run -d -p 8080:80 nginx
# Host port 8080 → Container port 80

# Map tất cả interfaces
docker run -d -p 80:80 nginx

# Map specific IP
docker run -d -p 127.0.0.1:8080:80 nginx

# Map random port
docker run -d -p 80 nginx
# Docker tự chọn port trên host

# Map nhiều ports
docker run -d \
  -p 80:80 \
  -p 443:443 \
  nginx

# Map UDP port
docker run -d -p 53:53/udp dns-server

# Map port range
docker run -d -p 8000-8010:8000-8010 myapp
```

### Expose Ports (Documentation Only)

```dockerfile
# Trong Dockerfile
EXPOSE 80
EXPOSE 443

# Chỉ là documentation, không publish port
# Vẫn cần -p khi run
docker run -d -p 80:80 myapp
```

### Xem Port Mappings

```bash
# Xem ports của container
docker port web

# Output:
80/tcp -> 0.0.0.0:8080
443/tcp -> 0.0.0.0:8443

# Xem specific port
docker port web 80
# 0.0.0.0:8080
```

---

## 🎯 Ví Dụ Thực Tế

### Ví Dụ 1: Full-Stack Application

```bash
# 1. Tạo networks
docker network create frontend
docker network create backend

# 2. Database (chỉ backend)
docker run -d \
  --name postgres \
  --network backend \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# 3. API (backend + frontend)
docker run -d \
  --name api \
  --network backend \
  -e DATABASE_URL=postgresql://postgres:secret@postgres:5432/mydb \
  myapi:latest

docker network connect frontend api

# 4. Frontend (chỉ frontend)
docker run -d \
  --name web \
  --network frontend \
  -p 80:80 \
  -e API_URL=http://api:3000 \
  myfrontend:latest

# 5. Nginx reverse proxy (frontend)
docker run -d \
  --name nginx \
  --network frontend \
  -p 80:80 \
  -p 443:443 \
  nginx:alpine
```

**Network topology:**
```
Internet
   │
   ▼
┌─────────┐
│  Nginx  │ (frontend network)
└────┬────┘
     │
     ▼
┌─────────┐
│   Web   │ (frontend network)
└────┬────┘
     │
     ▼
┌─────────┐
│   API   │ (frontend + backend networks)
└────┬────┘
     │
     ▼
┌──────────┐
│ Postgres │ (backend network)
└──────────┘
```

### Ví Dụ 2: Microservices

```bash
# Tạo network
docker network create microservices

# Service 1: User Service
docker run -d \
  --name user-service \
  --network microservices \
  -e DATABASE_URL=mongodb://mongo:27017/users \
  user-service:latest

# Service 2: Order Service
docker run -d \
  --name order-service \
  --network microservices \
  -e DATABASE_URL=mongodb://mongo:27017/orders \
  -e USER_SERVICE_URL=http://user-service:3000 \
  order-service:latest

# Service 3: Payment Service
docker run -d \
  --name payment-service \
  --network microservices \
  -e ORDER_SERVICE_URL=http://order-service:3000 \
  payment-service:latest

# Database
docker run -d \
  --name mongo \
  --network microservices \
  mongo:7

# API Gateway
docker run -d \
  --name api-gateway \
  --network microservices \
  -p 80:80 \
  -e USER_SERVICE=http://user-service:3000 \
  -e ORDER_SERVICE=http://order-service:3000 \
  -e PAYMENT_SERVICE=http://payment-service:3000 \
  api-gateway:latest
```

### Ví Dụ 3: Development Environment

```bash
# Network cho dev
docker network create dev

# Database
docker run -d \
  --name dev-db \
  --network dev \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=dev \
  postgres:15

# Redis
docker run -d \
  --name dev-redis \
  --network dev \
  -p 6379:6379 \
  redis:alpine

# App (với hot reload)
docker run -d \
  --name dev-app \
  --network dev \
  -p 3000:3000 \
  -v $(pwd):/app \
  -e DATABASE_URL=postgresql://postgres:dev@dev-db:5432/mydb \
  -e REDIS_URL=redis://dev-redis:6379 \
  node:18-alpine \
  npm run dev
```

---

## 🔍 DNS Resolution

### Container Name Resolution

```bash
# Containers trên cùng custom network có thể ping nhau bằng tên
docker network create mynet

docker run -d --name web --network mynet nginx
docker run -d --name app --network mynet node

# Từ container web
docker exec web ping app  # ✅ Work
docker exec web curl http://app:3000  # ✅ Work

# DNS resolution:
# app → 172.18.0.3
```

### Network Aliases

```bash
# Tạo alias cho container
docker run -d \
  --name web \
  --network mynet \
  --network-alias webserver \
  --network-alias www \
  nginx

# Có thể access qua nhiều tên
docker exec app ping web        # ✅
docker exec app ping webserver  # ✅
docker exec app ping www        # ✅
```

### Link Containers (Legacy)

```bash
# ⚠️ Legacy feature, không khuyên dùng
docker run -d --name db postgres
docker run -d --name web --link db:database nginx

# Thay vào đó, dùng custom networks
```

---

## 🐛 Troubleshooting

### Kiểm Tra Network Connectivity

```bash
# 1. Kiểm tra container có IP không
docker inspect web | grep IPAddress

# 2. Ping giữa containers
docker exec web ping app

# 3. Kiểm tra DNS
docker exec web nslookup app

# 4. Kiểm tra port
docker exec web nc -zv app 3000

# 5. Curl endpoint
docker exec web curl http://app:3000/health
```

### Common Issues

**Issue 1: Containers không ping được nhau**
```bash
# Kiểm tra cùng network không
docker inspect web | grep NetworkID
docker inspect app | grep NetworkID

# Nếu khác network, connect vào cùng network
docker network connect mynet app
```

**Issue 2: Port conflict**
```bash
# Error: port is already allocated
# Giải pháp: Dùng port khác
docker run -d -p 8080:80 nginx  # Thay vì port 80
```

**Issue 3: Cannot resolve hostname**
```bash
# Default bridge không có DNS
# Giải pháp: Dùng custom bridge network
docker network create mynet
docker run -d --name web --network mynet nginx
```

---

## 💡 Best Practices

### 1. Sử Dụng Custom Networks

```bash
# ✅ Tạo custom network
docker network create myapp

# ❌ Dùng default bridge
docker run -d nginx
```

### 2. Network Segmentation

```bash
# ✅ Tách frontend và backend
docker network create frontend
docker network create backend
```

### 3. Least Privilege

```bash
# ✅ Database chỉ ở backend network
docker run -d --name db --network backend postgres

# ❌ Database exposed ra internet
docker run -d -p 5432:5432 postgres
```

### 4. Use DNS Names

```bash
# ✅ Dùng container name
DATABASE_URL=postgresql://postgres:secret@db:5432/mydb

# ❌ Hardcode IP
DATABASE_URL=postgresql://postgres:secret@172.18.0.2:5432/mydb
```

---

## 📚 Tóm Tắt

### Network Drivers

| Driver | Use Case | Scope |
|--------|----------|-------|
| `bridge` | Single host, default | Local |
| `host` | No isolation, performance | Local |
| `overlay` | Multi-host (Swarm) | Swarm |
| `macvlan` | Legacy apps | Local |
| `none` | No networking | Local |

### Key Commands

```bash
docker network create       # Tạo network
docker network ls          # List networks
docker network inspect     # Xem chi tiết
docker network connect     # Connect container
docker network disconnect  # Disconnect container
docker network rm          # Xóa network
```

---

## 📚 Bài Tiếp Theo

Trong bài tiếp theo, chúng ta sẽ học về **Volumes và Storage** - cách persist data trong Docker.

👉 [Bài 12: Volumes và Storage](./12-volumes-va-storage.md)

---

## 🔗 Tài Liệu Tham Khảo

- [Docker Networking](https://docs.docker.com/network/)
- [Network Drivers](https://docs.docker.com/network/drivers/)
- [Container Networking](https://docs.docker.com/config/containers/container-networking/)

---

**Happy Networking! 🌐**

