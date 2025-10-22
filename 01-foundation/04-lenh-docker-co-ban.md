# Bài 04: Lệnh Docker Cơ Bản

## 📋 Mục Tiêu Bài Học

Sau bài học này, bạn sẽ:
- ✅ Thành thạo các lệnh Docker cơ bản
- ✅ Quản lý containers và images
- ✅ Debug containers hiệu quả
- ✅ Hiểu các options quan trọng

---

## 🎯 Nhóm Lệnh Docker

Docker commands được chia thành các nhóm chính:

```
docker
├── Container Management (run, start, stop, rm...)
├── Image Management (build, pull, push, images...)
├── System Management (info, version, system...)
├── Network Management (network create, ls...)
├── Volume Management (volume create, ls...)
└── Logs & Debugging (logs, exec, inspect...)
```

---

## 📦 Container Management

### 1. docker run - Tạo và Chạy Container

**Syntax:**
```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

**Ví dụ cơ bản:**

```bash
# Chạy container đơn giản
docker run hello-world

# Chạy container interactive
docker run -it ubuntu bash

# Chạy container ở background (detached)
docker run -d nginx

# Chạy với tên cụ thể
docker run -d --name my-nginx nginx

# Chạy với port mapping
docker run -d -p 8080:80 nginx

# Chạy với environment variables
docker run -d -e MYSQL_ROOT_PASSWORD=secret mysql

# Chạy với volume mount
docker run -d -v mydata:/data nginx

# Kết hợp nhiều options
docker run -d \
  --name my-app \
  -p 3000:3000 \
  -e NODE_ENV=production \
  -v $(pwd):/app \
  node:18
```

**Options quan trọng:**

| Option | Mô Tả | Ví Dụ |
|--------|-------|-------|
| `-d` | Detached mode (background) | `docker run -d nginx` |
| `-it` | Interactive + TTY | `docker run -it ubuntu bash` |
| `--name` | Đặt tên container | `docker run --name web nginx` |
| `-p` | Port mapping | `docker run -p 8080:80 nginx` |
| `-e` | Environment variable | `docker run -e KEY=value nginx` |
| `-v` | Volume mount | `docker run -v data:/data nginx` |
| `--rm` | Auto remove khi stop | `docker run --rm nginx` |
| `--network` | Chỉ định network | `docker run --network mynet nginx` |

### 2. docker ps - List Containers

```bash
# List containers đang chạy
docker ps

# List tất cả containers (kể cả stopped)
docker ps -a

# List với format tùy chỉnh
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"

# List chỉ container IDs
docker ps -q

# List với filter
docker ps --filter "status=running"
docker ps --filter "name=nginx"
```

**Output mẫu:**

```
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                  NAMES
a1b2c3d4e5f6   nginx     "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   0.0.0.0:8080->80/tcp   my-nginx
```

### 3. docker start/stop/restart

```bash
# Start container đã stopped
docker start my-nginx

# Start nhiều containers
docker start container1 container2 container3

# Stop container đang chạy
docker stop my-nginx

# Stop với timeout (default 10s)
docker stop -t 30 my-nginx

# Restart container
docker restart my-nginx

# Pause container (tạm dừng processes)
docker pause my-nginx

# Unpause container
docker unpause my-nginx
```

### 4. docker rm - Xóa Container

```bash
# Xóa container đã stopped
docker rm my-nginx

# Xóa container đang chạy (force)
docker rm -f my-nginx

# Xóa nhiều containers
docker rm container1 container2 container3

# Xóa tất cả stopped containers
docker container prune

# Xóa tất cả containers (cẩn thận!)
docker rm -f $(docker ps -aq)
```

### 5. docker exec - Chạy Command Trong Container

```bash
# Chạy command trong container đang chạy
docker exec my-nginx ls /usr/share/nginx/html

# Vào shell của container
docker exec -it my-nginx bash

# Chạy command với user cụ thể
docker exec -u root my-nginx whoami

# Chạy command với environment variable
docker exec -e VAR=value my-nginx env
```

**Use cases phổ biến:**

```bash
# Debug container
docker exec -it my-app bash

# Xem logs file
docker exec my-app cat /var/log/app.log

# Chạy database commands
docker exec -it my-postgres psql -U postgres

# Check processes
docker exec my-app ps aux
```

### 6. docker logs - Xem Logs

```bash
# Xem logs của container
docker logs my-nginx

# Follow logs (real-time)
docker logs -f my-nginx

# Xem 100 dòng cuối
docker logs --tail 100 my-nginx

# Xem logs với timestamps
docker logs -t my-nginx

# Xem logs từ thời điểm cụ thể
docker logs --since 2024-01-01 my-nginx
docker logs --since 1h my-nginx

# Kết hợp options
docker logs -f --tail 50 --since 10m my-nginx
```

### 7. docker inspect - Xem Chi Tiết

```bash
# Xem tất cả thông tin container
docker inspect my-nginx

# Lấy thông tin cụ thể với format
docker inspect --format='{{.State.Status}}' my-nginx

# Lấy IP address
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' my-nginx

# Lấy environment variables
docker inspect --format='{{.Config.Env}}' my-nginx
```

### 8. docker stats - Xem Resource Usage

```bash
# Xem resource usage của tất cả containers
docker stats

# Xem resource usage của container cụ thể
docker stats my-nginx

# Xem 1 lần (không stream)
docker stats --no-stream
```

**Output mẫu:**

```
CONTAINER ID   NAME       CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O
a1b2c3d4e5f6   my-nginx   0.01%     2.5MiB / 7.775GiB     0.03%     1.2kB / 0B        0B / 0B
```

---

## 🖼️ Image Management

### 1. docker images - List Images

```bash
# List tất cả images
docker images

# List với filter
docker images nginx
docker images --filter "dangling=true"

# List chỉ image IDs
docker images -q

# List với format tùy chỉnh
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

### 2. docker pull - Download Image

```bash
# Pull image mới nhất
docker pull nginx

# Pull version cụ thể
docker pull nginx:1.25

# Pull từ registry khác
docker pull gcr.io/google-samples/hello-app:1.0

# Pull tất cả tags
docker pull --all-tags nginx
```

### 3. docker push - Upload Image

```bash
# Login vào Docker Hub
docker login

# Tag image
docker tag myapp:latest username/myapp:v1.0

# Push image
docker push username/myapp:v1.0

# Push tất cả tags
docker push --all-tags username/myapp
```

### 4. docker build - Build Image

```bash
# Build từ Dockerfile trong thư mục hiện tại
docker build -t myapp:v1.0 .

# Build với Dockerfile khác
docker build -f Dockerfile.prod -t myapp:prod .

# Build với build args
docker build --build-arg VERSION=1.0 -t myapp .

# Build không dùng cache
docker build --no-cache -t myapp .
```

### 5. docker rmi - Xóa Image

```bash
# Xóa image
docker rmi nginx:latest

# Xóa image force (kể cả khi có container)
docker rmi -f nginx:latest

# Xóa nhiều images
docker rmi image1 image2 image3

# Xóa tất cả dangling images
docker image prune

# Xóa tất cả unused images
docker image prune -a
```

### 6. docker tag - Tag Image

```bash
# Tag image với tên mới
docker tag nginx:latest myregistry/nginx:v1.0

# Tag với multiple tags
docker tag myapp:latest myapp:v1.0
docker tag myapp:latest myapp:stable
```

### 7. docker history - Xem Image History

```bash
# Xem layers của image
docker history nginx:latest

# Xem với format đầy đủ
docker history --no-trunc nginx:latest
```

---

## 🔧 System Management

### 1. docker version

```bash
# Xem version của Docker
docker version

# Xem version ngắn gọn
docker --version
```

### 2. docker info

```bash
# Xem thông tin hệ thống Docker
docker info

# Xem thông tin cụ thể
docker info --format '{{.ServerVersion}}'
```

### 3. docker system

```bash
# Xem disk usage
docker system df

# Xem chi tiết
docker system df -v

# Cleanup tất cả unused data
docker system prune

# Cleanup bao gồm cả volumes
docker system prune --volumes

# Cleanup tất cả (cẩn thận!)
docker system prune -a
```

---

## 🌐 Network Management

```bash
# List networks
docker network ls

# Tạo network
docker network create mynetwork

# Inspect network
docker network inspect mynetwork

# Connect container vào network
docker network connect mynetwork my-nginx

# Disconnect container khỏi network
docker network disconnect mynetwork my-nginx

# Xóa network
docker network rm mynetwork

# Xóa unused networks
docker network prune
```

---

## 💾 Volume Management

```bash
# List volumes
docker volume ls

# Tạo volume
docker volume create mydata

# Inspect volume
docker volume inspect mydata

# Xóa volume
docker volume rm mydata

# Xóa unused volumes
docker volume prune
```

---

## 🎯 Ví Dụ Thực Hành

### Ví Dụ 1: Web Server với Nginx

```bash
# 1. Pull image
docker pull nginx:latest

# 2. Chạy container
docker run -d \
  --name my-web \
  -p 8080:80 \
  -v $(pwd)/html:/usr/share/nginx/html \
  nginx:latest

# 3. Kiểm tra container
docker ps

# 4. Xem logs
docker logs -f my-web

# 5. Test
curl http://localhost:8080

# 6. Vào container
docker exec -it my-web bash

# 7. Stop và remove
docker stop my-web
docker rm my-web
```

### Ví Dụ 2: Database Container

```bash
# 1. Chạy PostgreSQL
docker run -d \
  --name my-db \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=myapp \
  -p 5432:5432 \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15

# 2. Kiểm tra logs
docker logs my-db

# 3. Connect vào database
docker exec -it my-db psql -U postgres -d myapp

# 4. Backup database
docker exec my-db pg_dump -U postgres myapp > backup.sql

# 5. Xem resource usage
docker stats my-db
```

### Ví Dụ 3: Development Environment

```bash
# 1. Chạy Node.js container
docker run -it --rm \
  --name node-dev \
  -v $(pwd):/app \
  -w /app \
  -p 3000:3000 \
  node:18 \
  bash

# Trong container:
npm init -y
npm install express
node app.js

# 2. Từ terminal khác, xem logs
docker logs -f node-dev

# 3. Xem processes
docker exec node-dev ps aux
```

---

## 💡 Tips & Tricks

### 1. Aliases Hữu Ích

```bash
# Thêm vào ~/.bashrc hoặc ~/.zshrc

# Stop tất cả containers
alias dstop='docker stop $(docker ps -q)'

# Remove tất cả containers
alias drm='docker rm $(docker ps -aq)'

# Remove tất cả images
alias drmi='docker rmi $(docker images -q)'

# Docker cleanup
alias dclean='docker system prune -af'

# Vào container gần nhất
alias dexec='docker exec -it $(docker ps -q -l) bash'
```

### 2. Shortcuts

```bash
# Sử dụng container ID ngắn
docker stop a1b  # Thay vì a1b2c3d4e5f6

# Sử dụng container name
docker stop my-nginx  # Dễ nhớ hơn ID

# Auto-remove container sau khi stop
docker run --rm nginx
```

### 3. Debugging

```bash
# Xem tại sao container exit
docker logs container_name

# Xem processes trong container
docker top container_name

# Xem file changes
docker diff container_name

# Xem port mappings
docker port container_name
```

---

## 📚 Cheat Sheet

### Container Lifecycle

```bash
docker run IMAGE          # Tạo + Start
docker start CONTAINER    # Start stopped container
docker stop CONTAINER     # Stop gracefully
docker kill CONTAINER     # Force stop
docker restart CONTAINER  # Stop + Start
docker rm CONTAINER       # Xóa container
```

### Monitoring

```bash
docker ps                 # List running containers
docker logs CONTAINER     # Xem logs
docker stats              # Resource usage
docker top CONTAINER      # Processes
docker inspect CONTAINER  # Chi tiết
```

### Images

```bash
docker images            # List images
docker pull IMAGE        # Download image
docker build -t NAME .   # Build image
docker push IMAGE        # Upload image
docker rmi IMAGE         # Xóa image
```

### Cleanup

```bash
docker container prune   # Xóa stopped containers
docker image prune       # Xóa dangling images
docker volume prune      # Xóa unused volumes
docker system prune      # Cleanup tất cả
```

---

## 🎓 Bài Tập Thực Hành

### Bài 1: Cơ Bản - Nginx Web Server

**Mục tiêu:** Làm quen với các lệnh Docker cơ bản

**Yêu cầu:**
1. Pull image nginx
2. Chạy container với port 8080 (host) → 80 (container)
3. Kiểm tra container đang chạy
4. Xem logs
5. Stop và remove container

**Hướng dẫn chi tiết:**

```bash
# Bước 1: Pull image nginx
docker pull nginx:latest
# Output: Downloaded newer image for nginx:latest

# Bước 2: Chạy container
docker run -d -p 8080:80 --name my-nginx nginx:latest
# -d: Chạy ở background (detached mode)
# -p 8080:80: Map port 8080 (host) → 80 (container)
# --name my-nginx: Đặt tên container
# Output: a1b2c3d4e5f6... (container ID)

# Bước 3: Kiểm tra container đang chạy
docker ps
# Output:
# CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                  NAMES
# a1b2c3d4e5f6   nginx     "/docker-entrypoint.…"   10 seconds ago  Up 9 seconds   0.0.0.0:8080->80/tcp   my-nginx

# Bước 4: Xem logs
docker logs my-nginx
# Output: Nginx startup logs

# Bước 4b: Follow logs (real-time)
docker logs -f my-nginx
# Ctrl+C để thoát

# Bước 5: Test web server
curl http://localhost:8080
# Hoặc mở browser: http://localhost:8080
# Output: Nginx welcome page

# Bước 6: Stop container
docker stop my-nginx
# Output: my-nginx

# Bước 7: Verify container đã stop
docker ps
# Output: (trống - không có container chạy)

# Bước 8: Xem tất cả containers (kể cả stopped)
docker ps -a
# Output: my-nginx sẽ có status "Exited"

# Bước 9: Remove container
docker rm my-nginx
# Output: my-nginx

# Bước 10: Verify container đã xóa
docker ps -a
# Output: (my-nginx không còn)
```

**Kiểm tra kết quả:**
- ✅ Container chạy thành công
- ✅ Port mapping hoạt động (truy cập được http://localhost:8080)
- ✅ Logs hiển thị đúng
- ✅ Container được stop và remove thành công

---

### Bài 2: Trung Bình - MySQL Database

**Mục tiêu:** Làm việc với database container, environment variables, và volumes

**Yêu cầu:**
1. Chạy MySQL container với:
   - Password: mypassword
   - Database: testdb
   - Port: 3306
   - Volume: mysql-data
2. Connect vào MySQL
3. Tạo table và insert data
4. Backup database
5. Stop container nhưng giữ data

**Hướng dẫn chi tiết:**

```bash
# Bước 1: Tạo volume cho database
docker volume create mysql-data
# Output: mysql-data

# Bước 2: Chạy MySQL container
docker run -d \
  --name my-mysql \
  -e MYSQL_ROOT_PASSWORD=mypassword \
  -e MYSQL_DATABASE=testdb \
  -p 3306:3306 \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0
# -e: Set environment variables
# -v: Mount volume
# Output: container ID

# Bước 3: Kiểm tra container chạy
docker ps
# Verify my-mysql đang chạy

# Bước 4: Xem logs để confirm MySQL started
docker logs my-mysql
# Tìm dòng: "ready for connections"

# Bước 5: Connect vào MySQL
docker exec -it my-mysql mysql -u root -p
# Nhập password: mypassword
# Bây giờ bạn ở trong MySQL shell

# Bước 6: Tạo table (trong MySQL shell)
USE testdb;
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100)
);

# Bước 7: Insert data
INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com');
INSERT INTO users (name, email) VALUES ('Jane Smith', 'jane@example.com');

# Bước 8: Verify data
SELECT * FROM users;
# Output:
# +----+------------+-------------------+
# | id | name       | email             |
# +----+------------+-------------------+
# |  1 | John Doe   | john@example.com  |
# |  2 | Jane Smith | jane@example.com  |
# +----+------------+-------------------+

# Bước 9: Thoát MySQL shell
exit

# Bước 10: Backup database (từ host terminal)
docker exec my-mysql mysqldump -u root -pmypassword testdb > backup.sql
# Output: backup.sql file được tạo

# Bước 11: Verify backup
cat backup.sql | head -20
# Bạn sẽ thấy SQL dump content

# Bước 12: Stop container (data vẫn giữ trong volume)
docker stop my-mysql
# Output: my-mysql

# Bước 13: Verify container stopped
docker ps
# Output: (trống)

# Bước 14: Verify volume vẫn tồn tại
docker volume ls
# Output: mysql-data vẫn có

# Bước 15: Start lại container
docker start my-mysql

# Bước 16: Verify data vẫn còn
docker exec -it my-mysql mysql -u root -pmypassword testdb -e "SELECT * FROM users;"
# Output: Data vẫn còn!
```

**Kiểm tra kết quả:**
- ✅ MySQL container chạy thành công
- ✅ Database và table được tạo
- ✅ Data được insert thành công
- ✅ Backup file được tạo
- ✅ Data persist sau khi stop/start container

---

### Bài 3: Nâng Cao - Multi-Container Network

**Mục tiêu:** Kết nối nhiều containers qua network

**Yêu cầu:**
1. Tạo custom network
2. Chạy 3 containers:
   - nginx (port 80)
   - node app (port 3000)
   - redis (port 6379)
3. Connect chúng vào cùng network
4. Test kết nối giữa containers
5. Monitor resource usage
6. Cleanup tất cả

**Hướng dẫn chi tiết:**

```bash
# Bước 1: Tạo custom network
docker network create myapp-network
# Output: network ID

# Bước 2: Verify network được tạo
docker network ls
# Output: myapp-network sẽ hiển thị

# Bước 3: Chạy Redis container
docker run -d \
  --name redis \
  --network myapp-network \
  redis:alpine
# Output: container ID

# Bước 4a: Tạo folder project
mkdir my-node-app
cd my-node-app
# Output: Folder my-node-app được tạo

# Bước 4b: Tạo package.json
npm init -y
# Output: package.json được tạo

# Bước 4c: Cài đặt dependencies
npm install express redis
# Output: node_modules được tạo, package-lock.json được cập nhật

# Bước 4d: Tạo file index.js (Khuyến nghị - Sử dụng Redis URL)
cat > index.js << 'EOF'
const express = require('express');
const redis = require('redis');
const app = express();

// Kết nối Redis sử dụng URL format
// Format: redis://[hostname]:[port]
const redisUrl = process.env.REDIS_URL || 'redis://localhost:6379';
const client = redis.createClient({
  url: redisUrl
});

// Xử lý lỗi Redis
client.on('error', (err) => console.log('Redis Client Error', err));
client.on('connect', () => console.log('Redis Client Connected'));

app.get('/', (req, res) => {
  res.json({msg: 'Hello from Node!'});
});

app.get('/redis', async (req, res) => {
  try {
    await client.set('test', 'value');
    const val = await client.get('test');
    res.json({redis: val});
  } catch (err) {
    res.status(500).json({error: err.message});
  }
});

// Kết nối Redis khi app khởi động
const startServer = async () => {
  try {
    await client.connect();
    console.log('Connected to Redis at:', redisUrl);

    app.listen(3000, () => {
      console.log('Server running on 3000');
    });
  } catch (err) {
    console.error('Failed to connect to Redis:', err);
    process.exit(1);
  }
};

startServer();
EOF
# Output: index.js được tạo

# Bước 4e: Chạy Node.js app container với volume mount
docker run -d \
  --name node-app \
  --network myapp-network \
  -p 3000:3000 \
  -e REDIS_URL=redis://redis:6379 \
  -v $(pwd):/app \
  -w /app \
  node:latest \
  node index.js
# -e REDIS_URL=redis://redis:6379: Set Redis URL (redis là tên container Redis)
# -v $(pwd):/app: Mount folder hiện tại vào /app trong container
# -w /app: Set working directory là /app
# node:latest: Sử dụng image Node.js mới nhất
# Output: container ID

# Bước 5: Chạy Nginx container
docker run -d \
  --name nginx \
  --network myapp-network \
  -p 80:80 \
  nginx:alpine
# Output: container ID

# Bước 6: Verify tất cả containers chạy
docker ps
# Output: 3 containers (redis, node-app, nginx)

# Bước 7: Inspect network để xem containers
docker network inspect myapp-network
# Output: Sẽ thấy 3 containers kết nối

# Bước 8: Test kết nối từ node-app đến redis
docker exec node-app sh -c "ping -c 1 redis"
# Output: PING redis ... (thành công)

# Bước 9: Test Node.js app
curl http://localhost:3000
# Output: {"msg":"Hello from Node!"}

# Bước 10: Test Redis connection
curl http://localhost:3000/redis
# Output: {"redis":"value"}

# Bước 11: Monitor resource usage
docker stats
# Output: CPU, Memory, Network I/O của mỗi container
# Ctrl+C để thoát

# Bước 12: Xem logs của node-app
docker logs node-app
# Output: Server logs

# Bước 13: Stop tất cả containers
docker stop redis node-app nginx
# Output: container names

# Bước 14: Remove tất cả containers
docker rm redis node-app nginx
# Output: container names

# Bước 15: Remove network
docker network rm myapp-network
# Output: myapp-network

# Bước 16: Verify cleanup
docker ps -a
docker network ls
# Output: Không có containers/networks
```

**Kiểm tra kết quả:**
- ✅ Folder project được tạo với package.json, index.js
- ✅ Dependencies được cài đặt thành công
- ✅ Custom network được tạo
- ✅ 3 containers chạy trên cùng network
- ✅ Containers có thể kết nối với nhau qua hostname
- ✅ Port mapping hoạt động (test: curl http://localhost:3000)
- ✅ Volume mount hoạt động (code trên host được sử dụng trong container)
- ✅ Resource monitoring hoạt động
- ✅ Cleanup thành công

**Bonus - Nâng cao hơn - Sử dụng Dockerfile:**

```bash
# Bước 1: Tạo Dockerfile trong folder my-node-app
cat > Dockerfile << 'EOF'
FROM node:latest
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
EOF

# Bước 2: Build image
docker build -t my-node-app .
# Output: Successfully tagged my-node-app:latest

# Bước 3: Chạy container từ image vừa build
docker run -d \
  --name node-app \
  --network myapp-network \
  -p 3000:3000 \
  -e REDIS_URL=redis://redis:6379 \
  my-node-app
# Output: container ID
```

**Bonus - Nâng cao hơn - Sử dụng Docker Compose:**

```bash
# Bước 1: Tạo docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  redis:
    image: redis:alpine
    networks:
      - myapp-network

  node-app:
    image: node:latest
    ports:
      - "3000:3000"
    environment:
      REDIS_URL: redis://redis:6379
    volumes:
      - .:/app
      - /app/node_modules
    working_dir: /app
    command: node index.js
    networks:
      - myapp-network
    depends_on:
      - redis

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    networks:
      - myapp-network
    depends_on:
      - node-app

networks:
  myapp-network:
EOF

# Bước 2: Chạy tất cả services
docker compose up -d
# Output: Creating redis, node-app, nginx...

# Bước 3: Xem logs
docker compose logs -f

# Bước 4: Cleanup
docker compose down
# Output: Removing containers, networks...
```

---

## 📚 Bài Tiếp Theo

Bây giờ bạn đã thành thạo các lệnh cơ bản! Trong module tiếp theo, chúng ta sẽ học về **Docker Images** chi tiết.

👉 [Bài 05: Docker Images](../02-images/05-docker-images.md)

---

## 🔗 Tài Liệu Tham Khảo

- [Docker CLI Reference](https://docs.docker.com/engine/reference/commandline/cli/)
- [Docker Run Reference](https://docs.docker.com/engine/reference/run/)
- [Docker Command Cheat Sheet](https://docs.docker.com/get-started/docker_cheatsheet.pdf)

---

**Happy Dockering! 🐳**

