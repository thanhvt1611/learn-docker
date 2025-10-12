# 🐳 Khóa Học Docker Toàn Diện - Từ Cơ Bản Đến Nâng Cao

> **Tài liệu học Docker chi tiết dành cho lập trình viên** - Bao gồm lý thuyết, ví dụ thực tế và bài tập thực hành

---

## 📚 Giới Thiệu

Đây là bộ tài liệu học Docker được thiết kế đặc biệt cho lập trình viên, từ người mới bắt đầu đến người có kinh nghiệm muốn nâng cao kỹ năng. Tài liệu được xây dựng dựa trên **Docker Official Documentation** và các **best practices** trong thực tế.

### 🎯 Mục Tiêu Khóa Học

- ✅ Hiểu rõ Docker là gì và tại sao cần sử dụng Docker
- ✅ Thành thạo các lệnh Docker cơ bản và nâng cao
- ✅ Viết Dockerfile hiệu quả và tối ưu
- ✅ Quản lý containers, images, networks, volumes
- ✅ Sử dụng Docker Compose cho ứng dụng multi-container
- ✅ Áp dụng best practices cho môi trường production
- ✅ Containerize ứng dụng với nhiều ngôn ngữ lập trình

### 👥 Đối Tượng Học

- Lập trình viên mới bắt đầu với Docker
- DevOps Engineers muốn nắm vững Docker
- Backend/Frontend Developers muốn containerize ứng dụng
- System Administrators quan tâm đến container technology

---

## 📖 Cấu Trúc Khóa Học

Khóa học được chia thành **5 modules** với **20 bài học** chi tiết:

### 📘 Module 1: Docker Cơ Bản (Foundation)
**Thời gian:** 2 tuần | **Độ khó:** ⭐ Beginner

| Bài | Tên Bài Học | Nội Dung Chính |
|-----|-------------|----------------|
| 01 | [Giới Thiệu Docker](./01-foundation/01-gioi-thieu-docker.md) | Docker là gì? Tại sao cần Docker? So sánh VM vs Container |
| 02 | [Cài Đặt Docker](./01-foundation/02-cai-dat-docker.md) | Cài Docker trên Windows, macOS, Linux |
| 03 | [Khái Niệm Cơ Bản](./01-foundation/03-khai-niem-co-ban.md) | Container, Image, Registry, Docker Architecture |
| 04 | [Lệnh Docker Cơ Bản](./01-foundation/04-lenh-docker-co-ban.md) | run, ps, stop, rm, logs, exec, inspect |

### 🔨 Module 2: Docker Images (Building)
**Thời gian:** 2 tuần | **Độ khó:** ⭐⭐ Intermediate

| Bài | Tên Bài Học | Nội Dung Chính |
|-----|-------------|----------------|
| 05 | [Docker Images](./02-images/05-docker-images.md) | Image layers, Image history, Base images |
| 06 | [Dockerfile](./02-images/06-dockerfile.md) | FROM, RUN, COPY, ADD, CMD, ENTRYPOINT, ENV, WORKDIR |
| 07 | [Build và Quản Lý Images](./02-images/07-build-va-quan-ly-images.md) | docker build, tag, push, pull, save, load |
| 08 | [Multi-Stage Builds](./02-images/08-multi-stage-builds.md) | Tối ưu hóa image size, Build patterns |
| 09 | [Best Practices Dockerfile](./02-images/09-best-practices-dockerfile.md) | Security, Optimization, Caching strategies |

### 🚀 Module 3: Docker Containers (Running)
**Thời gian:** 2 tuần | **Độ khó:** ⭐⭐ Intermediate

| Bài | Tên Bài Học | Nội Dung Chính |
|-----|-------------|----------------|
| 10 | [Quản Lý Containers](./03-containers/10-quan-ly-containers.md) | Lifecycle, Start/Stop, Restart policies, Health checks |
| 11 | [Networking](./03-containers/11-networking.md) | Bridge, Host, Overlay, None, Custom networks |
| 12 | [Volumes và Storage](./03-containers/12-volumes-va-storage.md) | Volumes, Bind mounts, tmpfs, Storage drivers |
| 13 | [Environment Variables](./03-containers/13-environment-variables.md) | ENV, .env files, Secrets management |

### 🎼 Module 4: Docker Compose (Multi-container)
**Thời gian:** 2 tuần | **Độ khó:** ⭐⭐⭐ Intermediate-Advanced

| Bài | Tên Bài Học | Nội Dung Chính |
|-----|-------------|----------------|
| 14 | [Docker Compose Cơ Bản](./04-compose/14-docker-compose.md) | docker-compose.yml, Services, Up/Down, Logs |
| 15 | [Docker Compose Nâng Cao](./04-compose/15-docker-compose-advanced.md) | Networks, Volumes, Depends_on, Profiles, Extends |

### 🎓 Module 5: Thực Hành và Nâng Cao (Advanced)
**Thời gian:** 2 tuần | **Độ khó:** ⭐⭐⭐⭐ Advanced

| Bài | Tên Bài Học | Nội Dung Chính |
|-----|-------------|----------------|
| 16 | [Bài Tập Thực Hành](./05-advanced/16-bai-tap-thuc-hanh.md) | 30+ bài tập từ cơ bản đến nâng cao |
| 17 | [Docker Cho Các Ngôn Ngữ](./05-advanced/17-docker-cho-cac-ngon-ngu.md) | Node.js, Python, Java, Go, .NET, PHP |
| 18 | [Best Practices Production](./05-advanced/18-best-practices-production.md) | Security, Performance, Monitoring, Logging |
| 19 | [Security](./05-advanced/19-security.md) | Image scanning, User permissions, Secrets |
| 20 | [Troubleshooting](./05-advanced/20-troubleshooting.md) | Debug containers, Common errors, Solutions |

---

## 🗓️ Lộ Trình Học Đề Xuất

### 📅 **10 Tuần - Học Toàn Diện**

```
Tuần 1-2:  Module 1 - Nắm vững foundation
Tuần 3-4:  Module 2 - Thành thạo building images
Tuần 5-6:  Module 3 - Vận hành containers
Tuần 7-8:  Module 4 - Multi-container applications
Tuần 9-10: Module 5 - Thực hành và nâng cao
```

### ⚡ **4 Tuần - Học Nhanh (Core)**

```
Tuần 1: Bài 01-04 + Bài 05-06
Tuần 2: Bài 07-10 + Bài 11-12
Tuần 3: Bài 14-15 + Thực hành
Tuần 4: Bài 16-17 + Projects
```

### 🎯 **Học Theo Mục Đích**

**Chỉ cần containerize app:**
- Bài 01, 03, 04, 06, 07, 10

**Full-stack development:**
- Bài 01-04, 06-07, 10-15, 17

**DevOps/Production:**
- Toàn bộ 20 bài + focus vào 18-20

---

## 💡 Đặc Điểm Nổi Bật

### ✨ Mỗi Bài Học Bao Gồm:

- 📖 **Lý thuyết rõ ràng**: Giải thích khái niệm dễ hiểu
- 💻 **Ví dụ thực tế**: Code examples với nhiều ngôn ngữ
- 🎨 **Diagrams**: Hình ảnh minh họa trực quan
- 🏋️ **Bài tập**: Từ cơ bản đến nâng cao
- 💡 **Tips & Tricks**: Best practices, common mistakes
- 🔗 **Tài liệu tham khảo**: Links đến docs chính thức

### 🎯 Phương Pháp Học Hiệu Quả:

1. **Đọc lý thuyết** (20%)
2. **Xem ví dụ** (30%)
3. **Thực hành ngay** (50%)
4. **Làm bài tập** (Quan trọng nhất!)

---

## 🚀 Bắt Đầu Nhanh

### Bước 1: Cài Đặt Docker
```bash
# Kiểm tra Docker đã cài chưa
docker --version
docker-compose --version

# Nếu chưa có, xem hướng dẫn tại:
# 📄 01-foundation/02-cai-dat-docker.md
```

### Bước 2: Hello World
```bash
# Chạy container đầu tiên
docker run hello-world

# Chạy container interactive
docker run -it ubuntu bash
```

### Bước 3: Bắt Đầu Học
```bash
# Đọc bài đầu tiên
# 📄 01-foundation/01-gioi-thieu-docker.md
```

---

## 📂 Cấu Trúc Thư Mục

```
learn-docker/
├── README.md                          # File này
├── 01-foundation/                     # Module 1: Cơ bản
│   ├── 01-gioi-thieu-docker.md
│   ├── 02-cai-dat-docker.md
│   ├── 03-khai-niem-co-ban.md
│   └── 04-lenh-docker-co-ban.md
├── 02-images/                         # Module 2: Images
│   ├── 05-docker-images.md
│   ├── 06-dockerfile.md
│   ├── 07-build-va-quan-ly-images.md
│   ├── 08-multi-stage-builds.md
│   └── 09-best-practices-dockerfile.md
├── 03-containers/                     # Module 3: Containers
│   ├── 10-quan-ly-containers.md
│   ├── 11-networking.md
│   ├── 12-volumes-va-storage.md
│   └── 13-environment-variables.md
├── 04-compose/                        # Module 4: Docker Compose
│   ├── 14-docker-compose.md
│   └── 15-docker-compose-advanced.md
├── 05-advanced/                       # Module 5: Nâng cao
│   ├── 16-bai-tap-thuc-hanh.md
│   ├── 17-docker-cho-cac-ngon-ngu.md
│   ├── 18-best-practices-production.md
│   ├── 19-security.md
│   └── 20-troubleshooting.md
└── examples/                          # Code examples
    ├── nodejs/
    ├── python/
    ├── java/
    ├── go/
    └── dotnet/
```

---

## 🛠️ Công Cụ Cần Thiết

- **Docker Desktop** (Windows/macOS) hoặc **Docker Engine** (Linux)
- **Docker Compose** (thường đi kèm Docker Desktop)
- **Code Editor**: VS Code (khuyên dùng với Docker extension)
- **Terminal/Command Line**
- **Git** (để clone examples)

---

## 📚 Tài Liệu Tham Khảo

- 🔗 [Docker Official Documentation](https://docs.docker.com/)
- 🔗 [Docker Hub](https://hub.docker.com/)
- 🔗 [Docker GitHub](https://github.com/docker)
- 🔗 [Play with Docker](https://labs.play-with-docker.com/)
- 🔗 [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)

---

## 🤝 Đóng Góp

Nếu bạn phát hiện lỗi hoặc muốn đóng góp cải thiện tài liệu, hãy tạo issue hoặc pull request.

---

## 📝 Ghi Chú

- Tài liệu được cập nhật theo Docker version mới nhất
- Tất cả ví dụ đều được test trên Docker Desktop
- Bài tập có đáp án và hướng dẫn chi tiết

---

## 🎓 Sau Khi Hoàn Thành Khóa Học

Bạn sẽ có thể:

- ✅ Containerize bất kỳ ứng dụng nào
- ✅ Viết Dockerfile tối ưu và bảo mật
- ✅ Triển khai multi-container applications
- ✅ Debug và troubleshoot Docker issues
- ✅ Áp dụng best practices cho production
- ✅ Tích hợp Docker vào CI/CD pipeline

---

## 📞 Liên Hệ & Hỗ Trợ

- 💬 Tạo issue nếu có thắc mắc
- 📧 Email: [your-email@example.com]
- 🌐 Website: [your-website.com]

---

**Chúc bạn học tốt! 🚀**

*"The best way to learn Docker is by doing. Start containerizing your applications today!"*

---

**Version:** 1.0.0  
**Last Updated:** 2025-10-11  
**License:** MIT

