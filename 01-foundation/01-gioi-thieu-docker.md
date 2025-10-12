# Bài 01: Giới Thiệu Docker

## 📋 Mục Tiêu Bài Học

Sau bài học này, bạn sẽ:
- ✅ Hiểu Docker là gì và tại sao cần sử dụng Docker
- ✅ Phân biệt được Container và Virtual Machine
- ✅ Nắm được kiến trúc cơ bản của Docker
- ✅ Biết các use cases phổ biến của Docker

---

## 🐳 Docker Là Gì?

**Docker** là một nền tảng mở (open platform) cho việc phát triển, vận chuyển và chạy ứng dụng. Docker cho phép bạn tách ứng dụng khỏi hạ tầng (infrastructure) để có thể phân phối phần mềm nhanh chóng.

### Định Nghĩa Chính Thức

> *"Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly."*
> 
> — Docker Official Documentation

### Giải Thích Đơn Giản

Hãy tưởng tượng Docker như một **"container vận chuyển hàng hóa"** trong thế giới phần mềm:

- 📦 **Container vật lý**: Đóng gói hàng hóa, vận chuyển bằng tàu/xe, có thể chuyển đổi giữa các phương tiện
- 🐳 **Docker Container**: Đóng gói ứng dụng + dependencies, chạy trên bất kỳ máy nào có Docker

---

## 🤔 Tại Sao Cần Docker?

### Vấn Đề Truyền Thống

#### 1. "It Works On My Machine" Problem

```
Developer: "Code chạy ngon trên máy tôi!"
Tester: "Sao trên máy tôi lỗi?"
DevOps: "Production lại crash?"
```

**Nguyên nhân:**
- Khác phiên bản ngôn ngữ (Node 14 vs Node 18)
- Khác dependencies (library versions)
- Khác hệ điều hành (Windows vs Linux)
- Khác cấu hình môi trường

#### 2. Dependency Hell

```javascript
// Ứng dụng A cần Python 2.7
// Ứng dụng B cần Python 3.9
// Làm sao chạy cả 2 trên cùng 1 server?
```

#### 3. Lãng Phí Tài Nguyên

- Mỗi ứng dụng cần 1 VM riêng
- VM nặng, khởi động chậm
- Chi phí cao

### Giải Pháp Của Docker

✅ **Consistency**: "Build once, run anywhere"  
✅ **Isolation**: Mỗi container độc lập  
✅ **Lightweight**: Nhẹ hơn VM rất nhiều  
✅ **Fast**: Khởi động trong vài giây  
✅ **Portable**: Chạy trên bất kỳ đâu có Docker  

---

## 🆚 Container vs Virtual Machine

### So Sánh Trực Quan

```
┌─────────────────────────────────────────────────────────────┐
│                    VIRTUAL MACHINES                         │
├─────────────────────────────────────────────────────────────┤
│  App A  │  App B  │  App C                                  │
│  Bins/  │  Bins/  │  Bins/                                  │
│  Libs   │  Libs   │  Libs                                   │
├─────────┼─────────┼─────────┐                               │
│ Guest OS│ Guest OS│ Guest OS│  ← Mỗi VM có OS riêng (nặng)  │
├─────────┴─────────┴─────────┤                               │
│      Hypervisor (VMware, VirtualBox)                        │
├─────────────────────────────────────────────────────────────┤
│              Host Operating System                          │
├─────────────────────────────────────────────────────────────┤
│              Infrastructure (Hardware)                      │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                      CONTAINERS                             │
├─────────────────────────────────────────────────────────────┤
│  App A  │  App B  │  App C                                  │
│  Bins/  │  Bins/  │  Bins/                                  │
│  Libs   │  Libs   │  Libs                                   │
├─────────┴─────────┴─────────┤                               │
│      Docker Engine          │  ← Chia sẻ OS kernel (nhẹ)    │
├─────────────────────────────────────────────────────────────┤
│              Host Operating System                          │
├─────────────────────────────────────────────────────────────┤
│              Infrastructure (Hardware)                      │
└─────────────────────────────────────────────────────────────┘
```

### Bảng So Sánh Chi Tiết

| Tiêu Chí | Virtual Machine | Docker Container |
|----------|----------------|------------------|
| **Kích thước** | GB (gigabytes) | MB (megabytes) |
| **Khởi động** | Phút | Giây |
| **Hiệu năng** | Chậm hơn (overhead) | Gần như native |
| **Isolation** | Hoàn toàn (OS-level) | Process-level |
| **Tài nguyên** | Cần nhiều RAM/CPU | Ít hơn nhiều |
| **Portability** | Khó (image lớn) | Dễ (image nhỏ) |
| **Use case** | Chạy nhiều OS khác nhau | Chạy nhiều apps trên cùng OS |

### Khi Nào Dùng Gì?

**Dùng Virtual Machine khi:**
- ❌ Cần chạy nhiều OS khác nhau (Windows + Linux)
- ❌ Cần isolation hoàn toàn về security
- ❌ Chạy ứng dụng legacy cần full OS

**Dùng Docker Container khi:**
- ✅ Microservices architecture
- ✅ CI/CD pipelines
- ✅ Development environments
- ✅ Scaling applications
- ✅ Cloud-native applications

---

## 🏗️ Kiến Trúc Docker

### Docker Architecture Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                      DOCKER CLIENT                           │
│                   (docker command)                           │
│                                                              │
│  $ docker build                                              │
│  $ docker pull                                               │
│  $ docker run                                                │
└────────────────────┬─────────────────────────────────────────┘
                     │ REST API
                     ▼
┌──────────────────────────────────────────────────────────────┐
│                    DOCKER DAEMON (dockerd)                   │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Container 1 │  │  Container 2 │  │  Container 3 │      │
│  │              │  │              │  │              │      │
│  │  App + Libs  │  │  App + Libs  │  │  App + Libs  │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                              │
│  ┌────────────────────────────────────────────────┐         │
│  │              Images Storage                    │         │
│  │  [nginx]  [node]  [postgres]  [myapp]         │         │
│  └────────────────────────────────────────────────┘         │
└────────────────────┬─────────────────────────────────────────┘
                     │
                     ▼
┌──────────────────────────────────────────────────────────────┐
│                    DOCKER REGISTRY                           │
│                    (Docker Hub)                              │
│                                                              │
│  Public Images: nginx, node, python, mysql, ...             │
│  Private Images: your-company/app                           │
└──────────────────────────────────────────────────────────────┘
```

### Các Thành Phần Chính

#### 1. **Docker Client** (docker CLI)
- Công cụ dòng lệnh để tương tác với Docker
- Gửi commands đến Docker Daemon
- Ví dụ: `docker run`, `docker build`, `docker ps`

#### 2. **Docker Daemon** (dockerd)
- Chạy trên host machine
- Quản lý containers, images, networks, volumes
- Lắng nghe Docker API requests

#### 3. **Docker Registry** (Docker Hub)
- Nơi lưu trữ Docker images
- Docker Hub: registry công khai
- Có thể tự host private registry

#### 4. **Docker Objects**
- **Images**: Template để tạo containers
- **Containers**: Instance chạy từ image
- **Networks**: Kết nối giữa containers
- **Volumes**: Lưu trữ persistent data

---

## 💼 Use Cases Phổ Biến

### 1. Development Environment

**Vấn đề:**
```bash
# Developer mới join team
"Cài Node.js, MongoDB, Redis, PostgreSQL, Elasticsearch..."
"Mất 2 ngày setup môi trường!"
```

**Giải pháp Docker:**
```bash
# Chỉ cần 1 lệnh
docker-compose up

# Môi trường dev giống hệt production
# Onboarding developer mới trong 5 phút!
```

### 2. Microservices Architecture

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Frontend  │───▶│   API       │───▶│  Database   │
│  (React)    │    │  (Node.js)  │    │ (PostgreSQL)│
│  Container  │    │  Container  │    │  Container  │
└─────────────┘    └─────────────┘    └─────────────┘
       │                  │                   │
       └──────────────────┴───────────────────┘
                    Docker Network
```

### 3. CI/CD Pipeline

```yaml
# .gitlab-ci.yml
test:
  image: node:18
  script:
    - npm install
    - npm test

build:
  image: docker:latest
  script:
    - docker build -t myapp .
    - docker push myapp

deploy:
  script:
    - docker pull myapp
    - docker run -d myapp
```

### 4. Scaling Applications

```bash
# Chạy 1 instance
docker run myapp

# Cần scale? Chạy thêm instances
docker run myapp  # Instance 2
docker run myapp  # Instance 3
docker run myapp  # Instance 4

# Hoặc dùng Docker Swarm/Kubernetes
docker service scale myapp=10
```

---

## 🎯 Lợi Ích Của Docker

### Cho Developers

✅ **Consistent Environment**: Dev = Test = Production  
✅ **Fast Setup**: Onboarding nhanh  
✅ **Isolation**: Test nhiều versions cùng lúc  
✅ **Easy Sharing**: Share môi trường qua Dockerfile  

### Cho DevOps

✅ **Easy Deployment**: Deploy nhanh, rollback dễ  
✅ **Scalability**: Scale horizontal dễ dàng  
✅ **Resource Efficiency**: Chạy nhiều apps trên ít servers  
✅ **Portability**: Chạy trên bất kỳ cloud nào  

### Cho Business

✅ **Faster Time to Market**: Ship features nhanh hơn  
✅ **Cost Reduction**: Tiết kiệm infrastructure  
✅ **Better Resource Utilization**: Tối ưu servers  
✅ **Improved Reliability**: Ít bugs hơn  

---

## 📊 Docker Trong Thực Tế

### Các Công Ty Sử Dụng Docker

- 🏢 **Netflix**: Microservices platform
- 🏢 **Uber**: Deployment automation
- 🏢 **Spotify**: Development environments
- 🏢 **PayPal**: CI/CD pipelines
- 🏢 **Alibaba**: Cloud infrastructure

### Thống Kê

- 📈 **13 million+** developers sử dụng Docker
- 📈 **13 billion+** container images downloaded/month
- 📈 **Top 3** container platform thế giới

---

## 🎓 Tóm Tắt

### Key Takeaways

1. **Docker** = Platform để containerize applications
2. **Container** ≠ VM (nhẹ hơn, nhanh hơn)
3. **Docker giải quyết**: "Works on my machine" problem
4. **Use cases**: Development, Microservices, CI/CD, Scaling
5. **Lợi ích**: Consistency, Portability, Efficiency

### Thuật Ngữ Quan Trọng

- **Container**: Môi trường chạy ứng dụng độc lập
- **Image**: Template để tạo container
- **Docker Engine**: Phần mềm chạy containers
- **Docker Hub**: Registry lưu trữ images
- **Dockerfile**: File định nghĩa cách build image

---

## 📚 Bài Tiếp Theo

Trong bài tiếp theo, chúng ta sẽ học cách **cài đặt Docker** trên các hệ điều hành khác nhau.

👉 [Bài 02: Cài Đặt Docker](./02-cai-dat-docker.md)

---

## 🔗 Tài Liệu Tham Khảo

- [Docker Official Documentation](https://docs.docker.com/)
- [Docker Overview](https://docs.docker.com/get-started/docker-overview/)
- [What is a Container?](https://www.docker.com/resources/what-container/)

---

**Happy Learning! 🚀**

