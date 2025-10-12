# Bài 02: Cài Đặt Docker

## 📋 Mục Tiêu Bài Học

Sau bài học này, bạn sẽ:
- ✅ Biết cách cài đặt Docker trên Windows, macOS, Linux
- ✅ Hiểu sự khác biệt giữa Docker Desktop và Docker Engine
- ✅ Kiểm tra Docker đã cài đặt thành công
- ✅ Chạy container đầu tiên

---

## 🖥️ Yêu Cầu Hệ Thống

### Windows

**Docker Desktop for Windows:**
- Windows 10/11 64-bit: Pro, Enterprise, hoặc Education
- WSL 2 feature enabled
- 4GB RAM minimum (8GB recommended)
- BIOS-level hardware virtualization enabled

**Hoặc Docker Desktop với WSL 2:**
- Windows 10/11 Home cũng được
- WSL 2 backend

### macOS

**Docker Desktop for Mac:**
- macOS 11 Big Sur hoặc mới hơn
- Apple Silicon (M1/M2) hoặc Intel chip
- 4GB RAM minimum (8GB recommended)

### Linux

**Docker Engine:**
- Ubuntu, Debian, CentOS, Fedora, hoặc các distros khác
- 64-bit kernel version 3.10+
- 2GB RAM minimum

---

## 🪟 Cài Đặt Trên Windows

### Phương Pháp 1: Docker Desktop (Khuyên Dùng)

#### Bước 1: Download Docker Desktop

```
1. Truy cập: https://www.docker.com/products/docker-desktop
2. Click "Download for Windows"
3. Chạy file "Docker Desktop Installer.exe"
```

#### Bước 2: Cài Đặt

```
1. Double-click file installer
2. Chọn "Use WSL 2 instead of Hyper-V" (recommended)
3. Click "OK" để bắt đầu cài đặt
4. Đợi quá trình cài đặt hoàn tất
5. Click "Close and restart"
```

#### Bước 3: Cấu Hình WSL 2 (Nếu Chưa Có)

Mở PowerShell với quyền Administrator:

```powershell
# Enable WSL
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# Enable Virtual Machine Platform
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# Restart máy
Restart-Computer

# Sau khi restart, set WSL 2 làm default
wsl --set-default-version 2

# Install Ubuntu (optional nhưng recommended)
wsl --install -d Ubuntu
```

#### Bước 4: Khởi Động Docker Desktop

```
1. Mở Docker Desktop từ Start Menu
2. Đợi Docker Engine khởi động (icon Docker ở system tray)
3. Khi icon Docker không còn "animating" = đã sẵn sàng
```

### Phương Pháp 2: Docker Toolbox (Legacy - Không Khuyên Dùng)

Chỉ dùng nếu máy không hỗ trợ Hyper-V hoặc WSL 2.

---

## 🍎 Cài Đặt Trên macOS

### Bước 1: Download Docker Desktop

```
1. Truy cập: https://www.docker.com/products/docker-desktop
2. Chọn chip của bạn:
   - "Mac with Intel chip"
   - "Mac with Apple chip" (M1/M2/M3)
3. Download file .dmg
```

### Bước 2: Cài Đặt

```
1. Double-click file "Docker.dmg"
2. Kéo icon Docker vào thư mục Applications
3. Mở Docker từ Applications
4. Click "Open" khi macOS hỏi xác nhận
5. Nhập password để cấp quyền
```

### Bước 3: Khởi Động

```
1. Docker Desktop sẽ tự động start
2. Icon Docker xuất hiện ở menu bar (góc trên bên phải)
3. Đợi cho đến khi status = "Docker Desktop is running"
```

### Cài Đặt Qua Homebrew (Alternative)

```bash
# Install Homebrew (nếu chưa có)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Docker Desktop
brew install --cask docker

# Mở Docker Desktop
open /Applications/Docker.app
```

---

## 🐧 Cài Đặt Trên Linux

### Ubuntu / Debian

#### Bước 1: Gỡ Các Phiên Bản Cũ (Nếu Có)

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

#### Bước 2: Cài Đặt Dependencies

```bash
sudo apt-get update

sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

#### Bước 3: Thêm Docker's Official GPG Key

```bash
sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

#### Bước 4: Setup Repository

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### Bước 5: Cài Đặt Docker Engine

```bash
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

#### Bước 6: Verify Installation

```bash
sudo docker run hello-world
```

#### Bước 7: Chạy Docker Không Cần Sudo (Optional)

```bash
# Tạo docker group
sudo groupadd docker

# Thêm user vào docker group
sudo usermod -aG docker $USER

# Activate changes
newgrp docker

# Test
docker run hello-world
```

### CentOS / RHEL / Fedora

```bash
# Remove old versions
sudo yum remove docker docker-client docker-client-latest \
  docker-common docker-latest docker-latest-logrotate \
  docker-logrotate docker-engine

# Install yum-utils
sudo yum install -y yum-utils

# Add Docker repository
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# Install Docker Engine
sudo yum install docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin

# Start Docker
sudo systemctl start docker

# Enable Docker to start on boot
sudo systemctl enable docker

# Verify
sudo docker run hello-world
```

### Script Tự Động (Convenience Script)

```bash
# Download và chạy script
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Cleanup
rm get-docker.sh
```

⚠️ **Lưu ý**: Chỉ dùng script này cho môi trường development/testing, không dùng cho production.

---

## ✅ Kiểm Tra Cài Đặt

### 1. Kiểm Tra Version

```bash
# Kiểm tra Docker version
docker --version
# Output: Docker version 24.0.7, build afdd53b

# Kiểm tra Docker Compose version
docker compose version
# Output: Docker Compose version v2.23.0

# Xem thông tin chi tiết
docker version
```

**Output mẫu:**

```
Client:
 Version:           24.0.7
 API version:       1.43
 Go version:        go1.20.10
 Git commit:        afdd53b
 Built:             Thu Oct 26 09:07:41 2023
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          24.0.7
  API version:      1.43 (minimum version 1.12)
  Go version:       go1.20.10
  Git commit:       311b9ff
  Built:            Thu Oct 26 09:07:41 2023
  OS/Arch:          linux/amd64
```

### 2. Kiểm Tra Docker Info

```bash
docker info
```

**Thông tin quan trọng:**
- Server Version
- Storage Driver
- Cgroup Driver
- Number of containers/images
- Docker Root Dir

### 3. Chạy Hello World Container

```bash
docker run hello-world
```

**Output thành công:**

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c1ec31eb5944: Pull complete
Digest: sha256:4bd78111b6914a99dbc560e6a20eab57ff6655aea4a80c50b0c5491968cbc2e6
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.
```

✅ Nếu thấy message này = Docker đã cài đặt thành công!

---

## 🎯 Chạy Container Đầu Tiên

### Ví Dụ 1: Nginx Web Server

```bash
# Pull và run nginx
docker run -d -p 8080:80 --name my-nginx nginx

# Giải thích:
# -d: Chạy ở background (detached mode)
# -p 8080:80: Map port 8080 (host) -> 80 (container)
# --name my-nginx: Đặt tên container
# nginx: Image name

# Kiểm tra container đang chạy
docker ps

# Truy cập http://localhost:8080 trên browser
# Bạn sẽ thấy trang "Welcome to nginx!"

# Stop container
docker stop my-nginx

# Remove container
docker rm my-nginx
```

### Ví Dụ 2: Ubuntu Interactive Shell

```bash
# Chạy Ubuntu container với interactive shell
docker run -it ubuntu bash

# Giải thích:
# -i: Interactive mode
# -t: Allocate a pseudo-TTY
# ubuntu: Image name
# bash: Command to run

# Bây giờ bạn đang ở trong Ubuntu container!
# Thử các lệnh:
cat /etc/os-release
ls /
whoami

# Thoát container
exit
```

### Ví Dụ 3: Node.js Application

```bash
# Chạy Node.js REPL
docker run -it node

# Bạn sẽ vào Node.js interactive shell
> console.log("Hello from Docker!")
> process.version
> .exit
```

---

## 🔧 Cấu Hình Docker Desktop

### Settings Quan Trọng

#### 1. Resources (Tài Nguyên)

```
Docker Desktop > Settings > Resources

- CPUs: 2-4 CPUs (tùy máy)
- Memory: 4-8 GB
- Swap: 1-2 GB
- Disk image size: 60 GB+
```

#### 2. Docker Engine (Advanced)

```json
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "features": {
    "buildkit": true
  }
}
```

#### 3. Kubernetes (Optional)

```
Docker Desktop > Settings > Kubernetes
☑ Enable Kubernetes
```

---

## 🐛 Troubleshooting

### Windows

**Lỗi: "WSL 2 installation is incomplete"**

```powershell
# Update WSL kernel
wsl --update

# Restart Docker Desktop
```

**Lỗi: "Docker Desktop requires Windows 10 Pro/Enterprise"**

```
Giải pháp: Enable WSL 2 backend
Docker Desktop > Settings > General
☑ Use the WSL 2 based engine
```

### macOS

**Lỗi: "Docker.app is damaged"**

```bash
# Remove quarantine attribute
xattr -d com.apple.quarantine /Applications/Docker.app
```

### Linux

**Lỗi: "permission denied while trying to connect to Docker daemon"**

```bash
# Thêm user vào docker group
sudo usermod -aG docker $USER

# Logout và login lại
# Hoặc chạy:
newgrp docker
```

**Lỗi: "Cannot connect to the Docker daemon"**

```bash
# Start Docker service
sudo systemctl start docker

# Check status
sudo systemctl status docker
```

---

## 📚 Bài Tiếp Theo

Bây giờ Docker đã sẵn sàng! Trong bài tiếp theo, chúng ta sẽ tìm hiểu các **khái niệm cơ bản** của Docker.

👉 [Bài 03: Khái Niệm Cơ Bản](./03-khai-niem-co-ban.md)

---

## 🔗 Tài Liệu Tham Khảo

- [Install Docker Desktop on Windows](https://docs.docker.com/desktop/install/windows-install/)
- [Install Docker Desktop on Mac](https://docs.docker.com/desktop/install/mac-install/)
- [Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [Post-installation steps for Linux](https://docs.docker.com/engine/install/linux-postinstall/)

---

**Happy Dockering! 🐳**

