# Bài 07: Build và Quản Lý Images

## 📋 Mục Tiêu Bài Học

Sau bài học này, bạn sẽ:
- ✅ Thành thạo lệnh docker build
- ✅ Hiểu build context và .dockerignore
- ✅ Quản lý image tags hiệu quả
- ✅ Push/Pull images từ registry

---

## 🔨 Docker Build

### Cú Pháp Cơ Bản

```bash
docker build [OPTIONS] PATH | URL | -
```

### Build Đơn Giản

```bash
# Build từ Dockerfile trong thư mục hiện tại
docker build .

# Build với tag
docker build -t myapp:latest .

# Build với multiple tags
docker build -t myapp:latest -t myapp:1.0.0 .

# Build từ Dockerfile khác
docker build -f Dockerfile.prod -t myapp:prod .

# Build từ URL
docker build -t myapp https://github.com/user/repo.git

# Build từ stdin
docker build -t myapp - < Dockerfile
```

### Build Options Quan Trọng

```bash
# Không sử dụng cache
docker build --no-cache -t myapp .

# Build với build arguments
docker build --build-arg VERSION=1.0.0 -t myapp .

# Chỉ định target stage (multi-stage)
docker build --target builder -t myapp:builder .

# Set build context
docker build -t myapp /path/to/context

# Quiet mode (chỉ show image ID)
docker build -q -t myapp .

# Show build progress
docker build --progress=plain -t myapp .
```

---

## 📁 Build Context

### Build Context Là Gì?

**Build Context** là tập hợp files và directories được gửi đến Docker daemon khi build.

```
my-project/
├── Dockerfile
├── src/
│   ├── app.js
│   └── utils.js
├── package.json
├── node_modules/        ← Không nên include
├── .git/                ← Không nên include
└── .dockerignore        ← Exclude files
```

### Ví Dụ Build Context

```bash
# Build context = thư mục hiện tại
docker build -t myapp .

# Output:
Sending build context to Docker daemon  150MB
# ↑ Tất cả files trong thư mục được gửi đến daemon
```

### Vấn Đề Với Build Context Lớn

```bash
# ❌ Build context quá lớn
Sending build context to Docker daemon  2.5GB
# Chậm, tốn bandwidth

# ✅ Sử dụng .dockerignore
Sending build context to Docker daemon  50MB
# Nhanh hơn nhiều!
```

---

## 🚫 .dockerignore File

### Cú Pháp

```
# .dockerignore
# Comment

# Ignore specific files
README.md
LICENSE

# Ignore directories
node_modules/
.git/
dist/

# Ignore patterns
*.log
*.tmp
**/*.md

# Negate (include lại)
!important.md
```

### Ví Dụ .dockerignore Cho Node.js

```
# .dockerignore

# Dependencies
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Testing
coverage/
.nyc_output/

# Production
dist/
build/

# Misc
.DS_Store
.env.local
.env.development.local
.env.test.local
.env.production.local

# IDE
.vscode/
.idea/
*.swp
*.swo

# Git
.git/
.gitignore

# Docker
Dockerfile*
docker-compose*.yml
.dockerignore

# Documentation
*.md
docs/
```

### Ví Dụ .dockerignore Cho Python

```
# .dockerignore

# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
ENV/

# Testing
.pytest_cache/
.coverage
htmlcov/

# Distribution
dist/
build/
*.egg-info/

# Misc
.env
.DS_Store

# IDE
.vscode/
.idea/
*.swp
```

---

## 🏷️ Image Tagging

### Tag Format

```
[REGISTRY/][NAMESPACE/]REPOSITORY[:TAG]
```

### Tagging Strategies

#### 1. Semantic Versioning

```bash
# Major.Minor.Patch
docker build -t myapp:1.0.0 .
docker build -t myapp:1.0 .
docker build -t myapp:1 .
docker build -t myapp:latest .

# Với prefix
docker build -t myapp:v1.0.0 .
```

#### 2. Git-Based Tagging

```bash
# Git commit SHA
GIT_SHA=$(git rev-parse --short HEAD)
docker build -t myapp:$GIT_SHA .

# Git branch
GIT_BRANCH=$(git rev-parse --abbrev-ref HEAD)
docker build -t myapp:$GIT_BRANCH .

# Git tag
GIT_TAG=$(git describe --tags --abbrev=0)
docker build -t myapp:$GIT_TAG .
```

#### 3. Environment-Based Tagging

```bash
# Development
docker build -t myapp:dev .

# Staging
docker build -t myapp:staging .

# Production
docker build -t myapp:prod .
docker build -t myapp:stable .
```

#### 4. Date-Based Tagging

```bash
# YYYY-MM-DD
DATE=$(date +%Y-%m-%d)
docker build -t myapp:$DATE .

# YYYY-MM-DD-HHMMSS
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
docker build -t myapp:$TIMESTAMP .
```

### Multi-Tagging

```bash
# Tag một image với nhiều tags
docker build \
  -t myapp:1.0.0 \
  -t myapp:1.0 \
  -t myapp:1 \
  -t myapp:latest \
  .

# Hoặc tag sau khi build
docker build -t myapp:1.0.0 .
docker tag myapp:1.0.0 myapp:1.0
docker tag myapp:1.0.0 myapp:1
docker tag myapp:1.0.0 myapp:latest
```

---

## 📤 Push Images

### Push Lên Docker Hub

```bash
# 1. Login
docker login
# Username: your-username
# Password: your-password

# 2. Tag image với username
docker tag myapp:latest your-username/myapp:latest

# 3. Push
docker push your-username/myapp:latest

# 4. Push tất cả tags
docker push --all-tags your-username/myapp

# 5. Logout
docker logout
```

### Push Lên Private Registry

```bash
# 1. Login vào private registry
docker login myregistry.com
# hoặc
docker login myregistry.com:5000

# 2. Tag image
docker tag myapp:latest myregistry.com/myapp:latest

# 3. Push
docker push myregistry.com/myapp:latest
```

### Push Lên Cloud Registries

#### AWS ECR

```bash
# 1. Login (AWS CLI required)
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  123456789.dkr.ecr.us-east-1.amazonaws.com

# 2. Tag
docker tag myapp:latest \
  123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# 3. Push
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
```

#### Google GCR

```bash
# 1. Configure gcloud
gcloud auth configure-docker

# 2. Tag
docker tag myapp:latest gcr.io/my-project/myapp:latest

# 3. Push
docker push gcr.io/my-project/myapp:latest
```

#### Azure ACR

```bash
# 1. Login
az acr login --name myregistry

# 2. Tag
docker tag myapp:latest myregistry.azurecr.io/myapp:latest

# 3. Push
docker push myregistry.azurecr.io/myapp:latest
```

---

## 📥 Pull Images

### Pull Từ Docker Hub

```bash
# Pull latest
docker pull nginx

# Pull specific version
docker pull nginx:1.25

# Pull từ user repository
docker pull username/myapp:latest

# Pull tất cả tags
docker pull --all-tags nginx

# Pull cho platform cụ thể
docker pull --platform linux/amd64 nginx
```

### Pull Từ Private Registry

```bash
# Login trước
docker login myregistry.com

# Pull
docker pull myregistry.com/myapp:latest
```

---

## 💾 Save và Load Images

### Save Image To File

```bash
# Save single image
docker save myapp:latest -o myapp.tar

# Save với gzip
docker save myapp:latest | gzip > myapp.tar.gz

# Save multiple images
docker save -o images.tar myapp:latest nginx:alpine

# Xem size
ls -lh myapp.tar
```

### Load Image From File

```bash
# Load image
docker load -i myapp.tar

# Load từ gzip
gunzip -c myapp.tar.gz | docker load

# Load từ stdin
cat myapp.tar | docker load
```

### Use Cases

```bash
# Transfer image giữa servers không có internet
# Server A:
docker save myapp:latest | gzip > myapp.tar.gz
scp myapp.tar.gz user@serverB:/tmp/

# Server B:
gunzip -c /tmp/myapp.tar.gz | docker load
```

---

## 🔍 Inspect và Analyze Images

### Docker Inspect

```bash
# Xem chi tiết image
docker inspect myapp:latest

# Lấy thông tin cụ thể
docker inspect --format='{{.Size}}' myapp:latest
docker inspect --format='{{.Architecture}}' myapp:latest
docker inspect --format='{{.Config.Env}}' myapp:latest
```

### Docker History

```bash
# Xem layers
docker history myapp:latest

# Xem full commands
docker history --no-trunc myapp:latest

# Format output
docker history --format "table {{.ID}}\t{{.Size}}\t{{.CreatedBy}}" myapp:latest
```

### Dive Tool (Third-party)

```bash
# Install dive
# macOS
brew install dive

# Linux
wget https://github.com/wagoodman/dive/releases/download/v0.11.0/dive_0.11.0_linux_amd64.deb
sudo apt install ./dive_0.11.0_linux_amd64.deb

# Analyze image
dive myapp:latest
```

---

## 🎯 Ví Dụ Thực Hành

### Ví Dụ 1: Build và Push Workflow

```bash
# 1. Tạo Dockerfile
cat > Dockerfile <<EOF
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "app.js"]
EOF

# 2. Tạo .dockerignore
cat > .dockerignore <<EOF
node_modules
npm-debug.log
.git
.env
EOF

# 3. Build với multiple tags
VERSION=1.0.0
docker build \
  -t myapp:$VERSION \
  -t myapp:latest \
  .

# 4. Test local
docker run -d -p 3000:3000 myapp:latest

# 5. Login Docker Hub
docker login

# 6. Tag cho Docker Hub
docker tag myapp:$VERSION username/myapp:$VERSION
docker tag myapp:latest username/myapp:latest

# 7. Push
docker push username/myapp:$VERSION
docker push username/myapp:latest

# 8. Verify
docker pull username/myapp:latest
```

### Ví Dụ 2: CI/CD Build Script

```bash
#!/bin/bash
# build.sh

set -e

# Variables
IMAGE_NAME="myapp"
REGISTRY="myregistry.com"
VERSION=$(git describe --tags --always)
BRANCH=$(git rev-parse --abbrev-ref HEAD)

# Build
echo "Building $IMAGE_NAME:$VERSION..."
docker build \
  --build-arg VERSION=$VERSION \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  -t $IMAGE_NAME:$VERSION \
  -t $IMAGE_NAME:$BRANCH \
  .

# Tag for registry
docker tag $IMAGE_NAME:$VERSION $REGISTRY/$IMAGE_NAME:$VERSION
docker tag $IMAGE_NAME:$BRANCH $REGISTRY/$IMAGE_NAME:$BRANCH

# Push
echo "Pushing to registry..."
docker push $REGISTRY/$IMAGE_NAME:$VERSION
docker push $REGISTRY/$IMAGE_NAME:$BRANCH

# Tag latest if main branch
if [ "$BRANCH" == "main" ]; then
  docker tag $IMAGE_NAME:$VERSION $REGISTRY/$IMAGE_NAME:latest
  docker push $REGISTRY/$IMAGE_NAME:latest
fi

echo "Build complete!"
```

### Ví Dụ 3: Backup và Restore Images

```bash
# Backup tất cả images
docker images --format "{{.Repository}}:{{.Tag}}" | \
  grep -v "<none>" | \
  xargs -I {} docker save {} -o {}.tar

# Hoặc backup vào 1 file
docker save $(docker images --format "{{.Repository}}:{{.Tag}}" | grep -v "<none>") \
  -o all-images.tar

# Restore
docker load -i all-images.tar
```

---

## 💡 Best Practices

### 1. Sử Dụng .dockerignore

```
# Luôn tạo .dockerignore để giảm build context
node_modules/
.git/
*.log
```

### 2. Tag Properly

```bash
# ✅ Sử dụng semantic versioning
docker build -t myapp:1.0.0 .

# ❌ Chỉ dùng latest
docker build -t myapp:latest .
```

### 3. Build Arguments

```dockerfile
# Sử dụng ARG cho flexibility
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}-alpine
```

### 4. Multi-Stage Builds

```dockerfile
# Giảm image size
FROM node:18 AS builder
# ... build

FROM node:18-alpine
COPY --from=builder /app/dist ./dist
```

### 5. Cache Optimization

```dockerfile
# Copy dependencies trước source code
COPY package*.json ./
RUN npm install
COPY . .
```

---

## 📚 Tóm Tắt

### Commands Quan Trọng

```bash
docker build -t name:tag .        # Build image
docker tag source target          # Tag image
docker push name:tag              # Push to registry
docker pull name:tag              # Pull from registry
docker save -o file.tar image     # Save to file
docker load -i file.tar           # Load from file
docker history image              # View layers
docker inspect image              # View details
```

---

## 📚 Bài Tiếp Theo

Trong bài tiếp theo, chúng ta sẽ học về **Multi-Stage Builds** để tối ưu image size.

👉 [Bài 08: Multi-Stage Builds](./08-multi-stage-builds.md)

---

## 🔗 Tài Liệu Tham Khảo

- [Docker Build Reference](https://docs.docker.com/engine/reference/commandline/build/)
- [.dockerignore File](https://docs.docker.com/engine/reference/builder/#dockerignore-file)
- [Docker Tag](https://docs.docker.com/engine/reference/commandline/tag/)

---

**Happy Building! 🚀**

