# Bài 16: Bài Tập Thực Hành

## 📋 Mục Tiêu

Bài học này cung cấp các bài tập thực hành từ cơ bản đến nâng cao để củng cố kiến thức Docker.

---

## 🎯 Cấp Độ 1: Cơ Bản

### Bài 1: Hello Docker

**Mục tiêu:** Làm quen với Docker commands cơ bản

**Yêu cầu:**
1. Pull image `hello-world`
2. Run container từ image
3. Xem logs của container
4. List tất cả containers (bao gồm stopped)
5. Remove container

**Hướng dẫn chi tiết:**

```bash
# Bước 1: Pull image hello-world
docker pull hello-world
# Output:
# Using default tag: latest
# latest: Pulling from library/hello-world
# ...
# Status: Downloaded newer image for hello-world:latest

# Bước 2: Verify image được pull
docker images | grep hello-world
# Output: hello-world  latest  ...

# Bước 3: Run container
docker run hello-world
# Output:
# Hello from Docker!
# This message shows that your installation appears to be working correctly.
# ...

# Bước 4: List tất cả containers (kể cả stopped)
docker ps -a
# Output:
# CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                     PORTS   NAMES
# a1b2c3d4e5f6   hello-world   "/hello"   10 seconds ago  Exited (0) 9 seconds ago           vibrant_newton

# Bước 5: Xem logs của container
docker logs a1b2c3d4e5f6
# Hoặc dùng container name:
docker logs vibrant_newton
# Output: Hello from Docker! ...

# Bước 6: Get container ID
CONTAINER_ID=$(docker ps -a -q --filter "ancestor=hello-world" | head -1)
echo $CONTAINER_ID

# Bước 7: Remove container
docker rm $CONTAINER_ID
# Output: container ID

# Bước 8: Verify container đã xóa
docker ps -a | grep hello-world
# Output: (trống - không có container)

# Bước 9: Verify image vẫn còn
docker images | grep hello-world
# Output: hello-world vẫn có
```

**Kiểm tra kết quả:**
- ✅ Image được pull thành công
- ✅ Container chạy và output đúng
- ✅ Container được list
- ✅ Container được remove thành công

---

### Bài 2: Web Server Đơn Giản

**Mục tiêu:** Chạy web server và expose port

**Yêu cầu:**
1. Run Nginx container
2. Expose port 8080 trên host → port 80 trong container
3. Truy cập http://localhost:8080 để verify
4. Xem logs của Nginx
5. Stop và remove container

**Hướng dẫn chi tiết:**

```bash
# Bước 1: Pull nginx:alpine image
docker pull nginx:alpine
# Output: Downloaded newer image for nginx:alpine

# Bước 2: Run Nginx container
docker run -d -p 8080:80 --name my-nginx nginx:alpine
# -d: Detached mode (chạy background)
# -p 8080:80: Port mapping (host:container)
# --name my-nginx: Đặt tên container
# Output: container ID

# Bước 3: Verify container chạy
docker ps
# Output:
# CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                  NAMES
# a1b2c3d4e5f6   nginx:alpine   "/docker-entrypoint.…"   5 seconds ago   Up 4 seconds   0.0.0.0:8080->80/tcp   my-nginx

# Bước 4: Test web server với curl
curl http://localhost:8080
# Output: Nginx welcome page HTML

# Bước 5: Hoặc mở browser
# Truy cập: http://localhost:8080
# Bạn sẽ thấy trang "Welcome to nginx!"

# Bước 6: Xem logs của Nginx
docker logs my-nginx
# Output: Nginx startup logs

# Bước 7: Follow logs (real-time)
docker logs -f my-nginx
# Output: Logs sẽ update khi có request
# Ctrl+C để thoát

# Bước 8: Xem chi tiết container
docker inspect my-nginx | grep -A 5 "PortBindings"
# Output: Port mapping configuration

# Bước 9: Stop container
docker stop my-nginx
# Output: my-nginx

# Bước 10: Verify container stopped
docker ps
# Output: (trống - my-nginx không chạy)

# Bước 11: Verify container vẫn tồn tại (stopped)
docker ps -a | grep my-nginx
# Output: my-nginx với status "Exited"

# Bước 12: Remove container
docker rm my-nginx
# Output: my-nginx

# Bước 13: Verify container đã xóa
docker ps -a | grep my-nginx
# Output: (trống)
```

**Kiểm tra kết quả:**
- ✅ Nginx container chạy thành công
- ✅ Port mapping hoạt động (8080 → 80)
- ✅ Web server accessible qua http://localhost:8080
- ✅ Logs hiển thị đúng
- ✅ Container được stop và remove thành công

---

### Bài 3: Custom HTML Page

**Mục tiêu:** Mount local files vào container

**Yêu cầu:**
1. Tạo file `index.html` với nội dung tùy ý
2. Run Nginx container
3. Mount file `index.html` vào `/usr/share/nginx/html/index.html`
4. Truy cập và verify nội dung

**Hướng dẫn chi tiết:**

```bash
# Bước 1: Tạo thư mục project
mkdir my-website
cd my-website

# Bước 2: Tạo file index.html
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>My Custom Website</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin-top: 50px;
            background-color: #f0f0f0;
        }
        h1 { color: #333; }
        p { color: #666; }
    </style>
</head>
<body>
    <h1>Hello Docker!</h1>
    <p>This is my custom HTML page mounted into Nginx container.</p>
    <p>Current time: <span id="time"></span></p>
    <script>
        document.getElementById('time').textContent = new Date().toLocaleString();
    </script>
</body>
</html>
EOF

# Bước 3: Verify file được tạo
cat index.html
# Output: HTML content

# Bước 4: Get current directory path
PWD_PATH=$(pwd)
echo $PWD_PATH
# Output: /path/to/my-website

# Bước 5: Run Nginx container với volume mount
docker run -d \
  -p 8080:80 \
  --name my-website \
  -v $PWD_PATH/index.html:/usr/share/nginx/html/index.html:ro \
  nginx:alpine
# -v: Volume mount
# :ro: Read-only mode
# Output: container ID

# Bước 6: Verify container chạy
docker ps | grep my-website
# Output: my-website container

# Bước 7: Test website
curl http://localhost:8080
# Output: Custom HTML content

# Bước 8: Hoặc mở browser
# Truy cập: http://localhost:8080
# Bạn sẽ thấy "Hello Docker!" và custom content

# Bước 9: Verify volume mount
docker inspect my-website | grep -A 10 "Mounts"
# Output: Sẽ thấy volume mount configuration

# Bước 10: Modify HTML file (test live update)
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Updated Website</title>
</head>
<body>
    <h1>Updated Content!</h1>
    <p>This file was updated while container is running.</p>
</body>
</html>
EOF

# Bước 11: Refresh browser hoặc curl lại
curl http://localhost:8080
# Output: Updated content (vì volume mount là live!)

# Bước 12: Xem logs
docker logs my-website
# Output: Nginx logs

# Bước 13: Stop container
docker stop my-website
# Output: my-website

# Bước 14: Remove container
docker rm my-website
# Output: my-website

# Bước 15: Verify HTML file vẫn còn (volume mount không xóa)
cat index.html
# Output: HTML content vẫn có
```

**Kiểm tra kết quả:**
- ✅ HTML file được tạo
- ✅ Container chạy với volume mount
- ✅ Custom HTML content hiển thị đúng
- ✅ Live update hoạt động (thay đổi file → thấy ngay)
- ✅ Container được stop và remove thành công
- ✅ HTML file vẫn tồn tại trên host

---

### Bài 4: Environment Variables

**Mục tiêu:** Sử dụng environment variables

**Yêu cầu:**
1. Run PostgreSQL container
2. Set environment variables:
   - POSTGRES_USER=myuser
   - POSTGRES_PASSWORD=mypassword
   - POSTGRES_DB=mydb
3. Verify database được tạo bằng cách exec vào container

**Hướng dẫn chi tiết:**

```bash
# Bước 1: Run PostgreSQL container với environment variables
docker run -d \
  --name postgres \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_PASSWORD=mypassword \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  postgres:15
# -e: Set environment variable
# Output: container ID

# Bước 2: Verify container chạy
docker ps | grep postgres
# Output: postgres container

# Bước 3: Xem logs để confirm PostgreSQL started
docker logs postgres
# Tìm dòng: "database system is ready to accept connections"

# Bước 4: Inspect container để xem environment variables
docker inspect postgres | grep -A 20 "Env"
# Output: Sẽ thấy POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB

# Bước 5: Connect vào PostgreSQL
docker exec -it postgres psql -U myuser -d mydb
# Bây giờ bạn ở trong PostgreSQL shell

# Bước 6: Verify database (trong psql shell)
\l
# Output: List databases, sẽ thấy "mydb"

# Bước 7: Verify user (trong psql shell)
\du
# Output: List users, sẽ thấy "myuser"

# Bước 8: Tạo table (trong psql shell)
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100)
);

# Bước 9: Insert data (trong psql shell)
INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com');
INSERT INTO users (name, email) VALUES ('Jane Smith', 'jane@example.com');

# Bước 10: Query data (trong psql shell)
SELECT * FROM users;
# Output:
#  id |    name    |       email
# ----+------------+-------------------
#   1 | John Doe   | john@example.com
#   2 | Jane Smith | jane@example.com

# Bước 11: Thoát psql shell
\q

# Bước 12: Connect lại từ host (từ terminal khác)
docker exec -it postgres psql -U myuser -d mydb -c "SELECT * FROM users;"
# Output: Data vẫn còn

# Bước 13: Test connection từ host (nếu có psql client)
psql -h localhost -U myuser -d mydb
# Nhập password: mypassword
# Bây giờ bạn ở trong PostgreSQL shell

# Bước 14: Thoát
\q

# Bước 15: Stop container
docker stop postgres
# Output: postgres

# Bước 16: Remove container
docker rm postgres
# Output: postgres
```

**Kiểm tra kết quả:**
- ✅ PostgreSQL container chạy thành công
- ✅ Environment variables được set đúng
- ✅ Database được tạo với tên "mydb"
- ✅ User được tạo với tên "myuser"
- ✅ Có thể connect vào database
- ✅ Có thể tạo table và insert data

---

### Bài 5: Persistent Data

**Mục tiêu:** Sử dụng volumes để persist data

**Yêu cầu:**
1. Tạo named volume `pgdata`
2. Run PostgreSQL với volume
3. Tạo table và insert data
4. Stop và remove container
5. Run lại container với cùng volume
6. Verify data vẫn còn

**Hướng dẫn chi tiết:**

```bash
# Bước 1: Tạo named volume
docker volume create pgdata
# Output: pgdata

# Bước 2: Verify volume được tạo
docker volume ls | grep pgdata
# Output: pgdata

# Bước 3: Inspect volume
docker volume inspect pgdata
# Output: Volume configuration

# Bước 4: Run PostgreSQL container với volume
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=testdb \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15
# -v pgdata:/var/lib/postgresql/data: Mount volume
# Output: container ID

# Bước 5: Verify container chạy
docker ps | grep postgres
# Output: postgres container

# Bước 6: Xem logs
docker logs postgres
# Tìm: "database system is ready to accept connections"

# Bước 7: Connect vào PostgreSQL
docker exec -it postgres psql -U postgres -d testdb
# Bây giờ bạn ở trong PostgreSQL shell

# Bước 8: Tạo table (trong psql shell)
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100),
  price DECIMAL(10, 2)
);

# Bước 9: Insert data (trong psql shell)
INSERT INTO products (name, price) VALUES ('Laptop', 999.99);
INSERT INTO products (name, price) VALUES ('Mouse', 29.99);
INSERT INTO products (name, price) VALUES ('Keyboard', 79.99);

# Bước 10: Verify data (trong psql shell)
SELECT * FROM products;
# Output:
#  id |   name   | price
# ----+----------+--------
#   1 | Laptop   | 999.99
#   2 | Mouse    |  29.99
#   3 | Keyboard |  79.99

# Bước 11: Thoát psql shell
\q

# Bước 12: Stop container
docker stop postgres
# Output: postgres

# Bước 13: Remove container (data vẫn giữ trong volume!)
docker rm postgres
# Output: postgres

# Bước 14: Verify container đã xóa
docker ps -a | grep postgres
# Output: (trống)

# Bước 15: Verify volume vẫn tồn tại
docker volume ls | grep pgdata
# Output: pgdata vẫn có

# Bước 16: Run lại container với cùng volume
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=testdb \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15
# Output: container ID (mới)

# Bước 17: Verify container chạy
docker ps | grep postgres
# Output: postgres container (mới)

# Bước 18: Connect vào PostgreSQL
docker exec -it postgres psql -U postgres -d testdb
# Bây giờ bạn ở trong PostgreSQL shell

# Bước 19: Verify data vẫn còn! (trong psql shell)
SELECT * FROM products;
# Output:
#  id |   name   | price
# ----+----------+--------
#   1 | Laptop   | 999.99
#   2 | Mouse    |  29.99
#   3 | Keyboard |  79.99
# 🎉 Data vẫn còn!

# Bước 20: Thoát psql shell
\q

# Bước 21: Cleanup
docker stop postgres
docker rm postgres
docker volume rm pgdata
# Output: postgres, pgdata

# Bước 22: Verify cleanup
docker ps -a | grep postgres
docker volume ls | grep pgdata
# Output: (trống)
```

**Kiểm tra kết quả:**
- ✅ Named volume được tạo
- ✅ PostgreSQL container chạy với volume
- ✅ Table và data được tạo
- ✅ Container được stop và remove
- ✅ Volume vẫn tồn tại
- ✅ Container mới chạy với cùng volume
- ✅ Data vẫn còn sau khi container được remove!

**Bonus - Backup volume:**

```bash
# Backup volume data
docker run --rm -v pgdata:/data -v $(pwd):/backup \
  alpine tar czf /backup/pgdata-backup.tar.gz -C /data .

# Restore volume data
docker run --rm -v pgdata:/data -v $(pwd):/backup \
  alpine tar xzf /backup/pgdata-backup.tar.gz -C /data
```

---

## 🚀 Cấp Độ 2: Trung Bình

### Bài 6: Build Custom Image

**Mục tiêu:** Tạo Dockerfile và build image

**Yêu cầu:**
1. Tạo Node.js app đơn giản (Express server)
2. Viết Dockerfile
3. Build image với tag `myapp:1.0`
4. Run container từ image
5. Test API endpoint

**Hướng dẫn chi tiết:**

```bash
# Bước 1: Tạo thư mục project
mkdir my-app
cd my-app

# Bước 2: Tạo package.json
cat > package.json << 'EOF'
{
  "name": "my-app",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
EOF

# Bước 3: Tạo app.js
cat > app.js << 'EOF'
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.json({ message: 'Hello from Docker!' });
});

app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
EOF

# Bước 4: Tạo Dockerfile
cat > Dockerfile << 'EOF'
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "app.js"]
EOF

# Bước 5: Tạo .dockerignore
cat > .dockerignore << 'EOF'
node_modules
npm-debug.log
.git
.env
EOF

# Bước 6: Verify files
ls -la
# Output: package.json, app.js, Dockerfile, .dockerignore

# Bước 7: Build image
docker build -t myapp:1.0 .
# Output:
# Sending build context to Docker daemon  ...
# Step 1/6 : FROM node:18-alpine
# ...
# Successfully tagged myapp:1.0

# Bước 8: Verify image được tạo
docker images | grep myapp
# Output: myapp  1.0  ...

# Bước 9: Inspect image
docker inspect myapp:1.0 | grep -A 5 "Cmd"
# Output: CMD ["node", "app.js"]

# Bước 10: Run container
docker run -d -p 3000:3000 --name my-app-container myapp:1.0
# Output: container ID

# Bước 11: Verify container chạy
docker ps | grep my-app-container
# Output: my-app-container

# Bước 12: Xem logs
docker logs my-app-container
# Output: Server running on port 3000

# Bước 13: Test API endpoint 1
curl http://localhost:3000
# Output: {"message":"Hello from Docker!"}

# Bước 14: Test API endpoint 2
curl http://localhost:3000/health
# Output: {"status":"healthy"}

# Bước 15: Test với jq (pretty print)
curl http://localhost:3000 | jq .
# Output:
# {
#   "message": "Hello from Docker!"
# }

# Bước 16: Xem resource usage
docker stats my-app-container --no-stream
# Output: CPU, Memory usage

# Bước 17: Exec vào container
docker exec -it my-app-container sh
# Bây giờ bạn ở trong container shell

# Bước 18: Trong container, check Node version
node --version
# Output: v18.x.x

# Bước 19: Thoát container
exit

# Bước 20: Stop container
docker stop my-app-container
# Output: my-app-container

# Bước 21: Remove container
docker rm my-app-container
# Output: my-app-container

# Bước 22: Verify image vẫn còn
docker images | grep myapp
# Output: myapp:1.0 vẫn có
```

**Kiểm tra kết quả:**
- ✅ Node.js app được tạo
- ✅ Dockerfile được tạo đúng
- ✅ Image build thành công
- ✅ Container chạy từ image
- ✅ API endpoints hoạt động
- ✅ Logs hiển thị đúng

**Bonus - Tag và push image:**

```bash
# Tag image với Docker Hub username
docker tag myapp:1.0 username/myapp:1.0

# Login vào Docker Hub
docker login

# Push image
docker push username/myapp:1.0

# Verify push
docker images | grep username/myapp
```

---

### Bài 7: Multi-Container với Networks

**Mục tiêu:** Kết nối nhiều containers qua network

**Yêu cầu:**
1. Tạo custom network `mynetwork`
2. Run PostgreSQL container trên network
3. Run Node.js API container trên cùng network
4. API connect đến PostgreSQL qua hostname
5. Verify connection

**Hướng dẫn chi tiết:**

```bash
# Bước 1: Tạo custom network
docker network create mynetwork
# Output: network ID

# Bước 2: Verify network được tạo
docker network ls | grep mynetwork
# Output: mynetwork

# Bước 3: Inspect network
docker network inspect mynetwork
# Output: Network configuration (trống - chưa có containers)

# Bước 4: Run PostgreSQL container trên network
docker run -d \
  --name postgres \
  --network mynetwork \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=mydb \
  postgres:15
# Output: container ID

# Bước 5: Verify PostgreSQL container chạy
docker ps | grep postgres
# Output: postgres container

# Bước 6: Xem logs
docker logs postgres
# Tìm: "database system is ready to accept connections"

# Bước 7: Tạo Node.js API app
mkdir my-api
cd my-api

# Bước 8: Tạo package.json
cat > package.json << 'EOF'
{
  "name": "my-api",
  "version": "1.0.0",
  "main": "app.js",
  "dependencies": {
    "express": "^4.18.2",
    "pg": "^8.10.0"
  }
}
EOF

# Bước 9: Tạo app.js
cat > app.js << 'EOF'
const express = require('express');
const { Client } = require('pg');

const app = express();

// PostgreSQL connection
const client = new Client({
  user: 'postgres',
  password: 'secret',
  host: 'postgres',  // ← Hostname của PostgreSQL container
  port: 5432,
  database: 'mydb'
});

client.connect();

app.get('/', (req, res) => {
  res.json({ message: 'Hello from API!' });
});

app.get('/db-test', async (req, res) => {
  try {
    const result = await client.query('SELECT NOW()');
    res.json({
      message: 'Connected to PostgreSQL!',
      timestamp: result.rows[0].now
    });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`API running on port ${PORT}`);
});
EOF

# Bước 10: Tạo Dockerfile
cat > Dockerfile << 'EOF'
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "app.js"]
EOF

# Bước 11: Build API image
docker build -t my-api:1.0 .
# Output: Successfully tagged my-api:1.0

# Bước 12: Run API container trên cùng network
docker run -d \
  --name api \
  --network mynetwork \
  -p 3000:3000 \
  -e DATABASE_URL=postgresql://postgres:secret@postgres:5432/mydb \
  my-api:1.0
# Output: container ID

# Bước 13: Verify API container chạy
docker ps | grep api
# Output: api container

# Bước 14: Xem logs
docker logs api
# Output: API running on port 3000

# Bước 15: Inspect network để xem containers
docker network inspect mynetwork
# Output: Sẽ thấy 2 containers (postgres, api)

# Bước 16: Test API endpoint 1
curl http://localhost:3000
# Output: {"message":"Hello from API!"}

# Bước 17: Test API endpoint 2 (database connection)
curl http://localhost:3000/db-test
# Output: {"message":"Connected to PostgreSQL!","timestamp":"..."}
# 🎉 API connected to PostgreSQL!

# Bước 18: Test network connectivity từ API container
docker exec api ping -c 1 postgres
# Output: PING postgres ... (thành công!)

# Bước 19: Test network connectivity từ PostgreSQL container
docker exec postgres ping -c 1 api
# Output: PING api ... (thành công!)

# Bước 20: Xem network interfaces
docker exec api ip addr
# Output: Network configuration

# Bước 21: Xem DNS resolution
docker exec api nslookup postgres
# Output: postgres resolves to IP address

# Bước 22: Stop containers
docker stop api postgres
# Output: api, postgres

# Bước 23: Remove containers
docker rm api postgres
# Output: api, postgres

# Bước 24: Remove network
docker network rm mynetwork
# Output: mynetwork

# Bước 25: Verify cleanup
docker ps -a | grep -E "api|postgres"
docker network ls | grep mynetwork
# Output: (trống)
```

**Kiểm tra kết quả:**
- ✅ Custom network được tạo
- ✅ PostgreSQL container chạy trên network
- ✅ API container chạy trên cùng network
- ✅ API có thể connect đến PostgreSQL qua hostname
- ✅ Containers có thể ping nhau
- ✅ DNS resolution hoạt động

**Bonus - Docker Compose version:**

```bash
# Tạo docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: mydb
    networks:
      - mynetwork

  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgresql://postgres:secret@postgres:5432/mydb
    depends_on:
      - postgres
    networks:
      - mynetwork

networks:
  mynetwork:
EOF

# Run với docker-compose
docker compose up -d

# Test
curl http://localhost:3000/db-test

# Cleanup
docker compose down
```

---

### Bài 8: Docker Compose - WordPress

**Mục tiêu:** Deploy WordPress với Docker Compose

**Yêu cầu:**
1. Tạo `docker-compose.yml`
2. Services: WordPress + MySQL
3. Persistent volumes cho cả 2 services
4. Custom network
5. Deploy và access WordPress

**Hướng dẫn chi tiết:**

```bash
# Bước 1: Tạo thư mục project
mkdir wordpress-app
cd wordpress-app

# Bước 2: Tạo docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  db:
    image: mysql:8.0
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppass
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - wordpress-net
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  wordpress:
    image: wordpress:latest
    restart: unless-stopped
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppass
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp-data:/var/www/html
    depends_on:
      db:
        condition: service_healthy
    networks:
      - wordpress-net

volumes:
  db-data:
  wp-data:

networks:
  wordpress-net:
EOF

# Bước 3: Verify docker-compose.yml
cat docker-compose.yml

# Bước 4: Start services
docker compose up -d
# Output:
# Creating network "wordpress-app_wordpress-net" with the default driver
# Creating volume "wordpress-app_db-data" with default driver
# Creating volume "wordpress-app_wp-data" with default driver
# Creating wordpress-app-db-1 ... done
# Creating wordpress-app-wordpress-1 ... done

# Bước 5: Verify services chạy
docker compose ps
# Output:
# NAME                      COMMAND                  SERVICE      STATUS
# wordpress-app-db-1        "docker-entrypoint.s…"   db           Up 10 seconds (healthy)
# wordpress-app-wordpress-1 "docker-entrypoint.s…"   wordpress    Up 5 seconds

# Bước 6: Xem logs
docker compose logs
# Output: Logs từ cả 2 services

# Bước 7: Follow logs (real-time)
docker compose logs -f
# Ctrl+C để thoát

# Bước 8: Xem logs của service cụ thể
docker compose logs db
docker compose logs wordpress

# Bước 9: Verify volumes được tạo
docker volume ls | grep wordpress-app
# Output: wordpress-app_db-data, wordpress-app_wp-data

# Bước 10: Verify network được tạo
docker network ls | grep wordpress-app
# Output: wordpress-app_wordpress-net

# Bước 11: Inspect network
docker network inspect wordpress-app_wordpress-net
# Output: Sẽ thấy 2 containers kết nối

# Bước 12: Access WordPress
# Mở browser: http://localhost:8080
# Bạn sẽ thấy WordPress setup page

# Bước 13: Setup WordPress (trong browser)
# - Language: English
# - Site Title: My Blog
# - Username: admin
# - Password: password123
# - Email: admin@example.com
# - Click "Install WordPress"

# Bước 14: Login vào WordPress
# URL: http://localhost:8080/wp-login.php
# Username: admin
# Password: password123

# Bước 15: Verify database connection
docker compose exec db mysql -u wpuser -pwppass wordpress -e "SELECT * FROM wp_users;"
# Output: WordPress users table

# Bước 16: Verify volumes có data
docker volume inspect wordpress-app_db-data
# Output: Mountpoint sẽ chứa MySQL data

# Bước 17: Stop services (data vẫn giữ)
docker compose stop
# Output: Stopping services

# Bước 18: Verify services stopped
docker compose ps
# Output: Services với status "Exited"

# Bước 19: Start lại services
docker compose start
# Output: Starting services

# Bước 20: Verify data vẫn còn
# Mở browser: http://localhost:8080
# WordPress vẫn có data!

# Bước 21: Scale services (nâng cao)
docker compose up -d --scale wordpress=2
# Output: 2 WordPress instances

# Bước 22: Verify
docker compose ps
# Output: 2 WordPress containers

# Bước 23: Cleanup (xóa containers nhưng giữ volumes)
docker compose stop
docker compose rm
# Output: Containers removed

# Bước 24: Verify volumes vẫn còn
docker volume ls | grep wordpress-app
# Output: Volumes vẫn có

# Bước 25: Cleanup hoàn toàn (xóa cả volumes)
docker compose down -v
# Output: Containers, networks, volumes removed

# Bước 26: Verify cleanup
docker compose ps
docker volume ls | grep wordpress-app
docker network ls | grep wordpress-app
# Output: (trống)
```

**Kiểm tra kết quả:**
- ✅ docker-compose.yml được tạo đúng
- ✅ Services start thành công
- ✅ MySQL database chạy
- ✅ WordPress chạy
- ✅ Volumes được tạo
- ✅ Network được tạo
- ✅ WordPress accessible qua http://localhost:8080
- ✅ Data persist sau khi stop/start
- ✅ Cleanup hoạt động

**Bonus - Thêm Nginx reverse proxy:**

```yaml
version: '3.8'

services:
  db:
    image: mysql:8.0
    # ... (như trên)

  wordpress:
    image: wordpress:latest
    # ... (như trên)
    expose:
      - "80"  # Không expose port, chỉ cho nginx access

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - wordpress
    networks:
      - wordpress-net

volumes:
  db-data:
  wp-data:

networks:
  wordpress-net:
```

---

### Bài 9: Multi-Stage Build

**Mục tiêu:** Optimize image size với multi-stage build

**Yêu cầu:**
1. Tạo React app
2. Viết Dockerfile với multi-stage build
3. Stage 1: Build app
4. Stage 2: Serve với Nginx
5. So sánh image size

**Hướng dẫn chi tiết:**

```bash
# Bước 1: Tạo thư mục project
mkdir react-app
cd react-app

# Bước 2: Tạo package.json
cat > package.json << 'EOF'
{
  "name": "react-app",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-scripts": "5.0.1"
  },
  "scripts": {
    "build": "react-scripts build",
    "start": "react-scripts start"
  },
  "eslintConfig": {
    "extends": ["react-app"]
  },
  "browserslist": {
    "production": [">0.2%", "not dead", "not op_mini all"],
    "development": ["last 1 chrome version"]
  }
}
EOF

# Bước 3: Tạo public/index.html
mkdir -p public
cat > public/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <title>React App</title>
</head>
<body>
    <div id="root"></div>
</body>
</html>
EOF

# Bước 4: Tạo src/index.js
mkdir -p src
cat > src/index.js << 'EOF'
import React from 'react';
import ReactDOM from 'react-dom/client';

const App = () => (
  <div style={{ textAlign: 'center', marginTop: '50px', fontFamily: 'Arial' }}>
    <h1>Hello from React in Docker!</h1>
    <p>This is a multi-stage build example.</p>
    <p>Image size is optimized!</p>
  </div>
);

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
EOF

# Bước 5: Tạo .dockerignore
cat > .dockerignore << 'EOF'
node_modules
npm-debug.log
build
.git
.env
.DS_Store
EOF

# Bước 6: Tạo Dockerfile với multi-stage build
cat > Dockerfile << 'EOF'
# ===== Stage 1: Build =====
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy source code
COPY . .

# Build app
RUN npm run build

# ===== Stage 2: Production =====
FROM nginx:alpine

# Copy built files từ stage 1
COPY --from=builder /app/build /usr/share/nginx/html

# Copy nginx config
RUN echo 'server { \
  listen 80; \
  location / { \
    root /usr/share/nginx/html; \
    try_files $uri /index.html; \
  } \
}' > /etc/nginx/conf.d/default.conf

# Expose port
EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
EOF

# Bước 7: Verify files
ls -la
# Output: package.json, src/, public/, Dockerfile, .dockerignore

# Bước 8: Build image
docker build -t react-app:1.0 .
# Output: Successfully tagged react-app:1.0
# Lưu ý: Build sẽ mất vài phút

# Bước 9: Xem image size
docker images | grep react-app
# Output: react-app  1.0  ...  ~50MB

# Bước 10: Xem layers của image
docker history react-app:1.0
# Output: Sẽ thấy 2 stages

# Bước 11: Tạo Dockerfile single-stage để so sánh
cat > Dockerfile.single << 'EOF'
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
EXPOSE 80
CMD ["npm", "start"]
EOF

# Bước 12: Build single-stage image
docker build -f Dockerfile.single -t react-app:single .
# Output: Successfully tagged react-app:single

# Bước 13: So sánh image size
docker images | grep react-app
# Output:
# react-app  1.0      ...  ~50MB   (multi-stage)
# react-app  single   ...  ~500MB  (single-stage)
# Multi-stage nhỏ hơn 10 lần!

# Bước 14: Run multi-stage image
docker run -d -p 8080:80 --name react-app-multi react-app:1.0
# Output: container ID

# Bước 15: Verify container chạy
docker ps | grep react-app-multi
# Output: react-app-multi

# Bước 16: Test app
curl http://localhost:8080
# Output: HTML content

# Hoặc mở browser: http://localhost:8080

# Bước 17: Run single-stage image
docker run -d -p 8081:80 --name react-app-single react-app:single
# Output: container ID

# Bước 18: Test single-stage app
curl http://localhost:8081
# Output: HTML content

# Bước 19: So sánh resource usage
docker stats --no-stream
# Output: CPU, Memory của cả 2 containers
# Multi-stage sẽ nhỏ hơn

# Bước 20: Inspect multi-stage image
docker inspect react-app:1.0 | grep -A 5 "RootFS"
# Output: Layers information

# Bước 21: Inspect single-stage image
docker inspect react-app:single | grep -A 5 "RootFS"
# Output: Layers information (nhiều hơn)

# Bước 22: Stop containers
docker stop react-app-multi react-app-single
# Output: container names

# Bước 23: Remove containers
docker rm react-app-multi react-app-single
# Output: container names

# Bước 24: Remove images
docker rmi react-app:1.0 react-app:single
# Output: image IDs

# Bước 25: Verify cleanup
docker images | grep react-app
# Output: (trống)
```

**Kiểm tra kết quả:**
- ✅ React app được tạo
- ✅ Dockerfile multi-stage được tạo
- ✅ Image build thành công
- ✅ Multi-stage image nhỏ hơn single-stage
- ✅ App chạy đúng
- ✅ Resource usage tối ưu

**Bonus - Analyze image layers:**

```bash
# Sử dụng dive để analyze image
docker run --rm -it \
  -v /var/run/docker.sock:/var/run/docker.sock \
  wagoodman/dive:latest react-app:1.0

# Hoặc sử dụng docker history
docker history react-app:1.0 --no-trunc

# Hoặc sử dụng docker inspect
docker inspect react-app:1.0 --format='{{json .RootFS.Layers}}' | jq .
```

---

### Bài 10: Health Checks

**Mục tiêu:** Implement health checks

**Yêu cầu:**
1. Tạo API với `/health` endpoint
2. Thêm HEALTHCHECK vào Dockerfile
3. Build và run container
4. Verify health status

**Hướng dẫn chi tiết:**

```bash
# Bước 1: Tạo thư mục project
mkdir health-check-app
cd health-check-app

# Bước 2: Tạo package.json
cat > package.json << 'EOF'
{
  "name": "health-check-app",
  "version": "1.0.0",
  "main": "server.js",
  "dependencies": {
    "express": "^4.18.2"
  }
}
EOF

# Bước 3: Tạo server.js
cat > server.js << 'EOF'
const express = require('express');
const app = express();

let isHealthy = true;

app.get('/', (req, res) => {
  res.json({ message: 'Hello from API!' });
});

app.get('/health', (req, res) => {
  if (isHealthy) {
    res.status(200).json({ status: 'healthy' });
  } else {
    res.status(503).json({ status: 'unhealthy' });
  }
});

app.get('/toggle-health', (req, res) => {
  isHealthy = !isHealthy;
  res.json({ status: isHealthy ? 'healthy' : 'unhealthy' });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
EOF

# Bước 4: Tạo healthcheck.js
cat > healthcheck.js << 'EOF'
const http = require('http');

const options = {
  hostname: 'localhost',
  port: 3000,
  path: '/health',
  method: 'GET',
  timeout: 2000
};

const req = http.request(options, (res) => {
  if (res.statusCode === 200) {
    process.exit(0);  // Healthy
  } else {
    process.exit(1);  // Unhealthy
  }
});

req.on('error', () => {
  process.exit(1);  // Error
});

req.on('timeout', () => {
  req.destroy();
  process.exit(1);  // Timeout
});

req.end();
EOF

# Bước 5: Tạo Dockerfile với HEALTHCHECK
cat > Dockerfile << 'EOF'
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci --only=production

COPY . .

# HEALTHCHECK configuration
HEALTHCHECK --interval=10s --timeout=3s --retries=3 \
  CMD node healthcheck.js

EXPOSE 3000

CMD ["node", "server.js"]
EOF

# Bước 6: Build image
docker build -t health-check-app:1.0 .
# Output: Successfully tagged health-check-app:1.0

# Bước 7: Run container
docker run -d -p 3000:3000 --name health-app health-check-app:1.0
# Output: container ID

# Bước 8: Verify container chạy
docker ps | grep health-app
# Output: health-app container

# Bước 9: Xem logs
docker logs health-app
# Output: Server running on port 3000

# Bước 10: Test health endpoint
curl http://localhost:3000/health
# Output: {"status":"healthy"}

# Bước 11: Inspect container để xem health status
docker inspect health-app | grep -A 10 "Health"
# Output:
# "Health": {
#   "Status": "healthy",
#   "FailingStreak": 0,
#   "Log": [...]
# }

# Bước 12: Wait a bit và check lại
sleep 15
docker inspect health-app | grep -A 5 "Health"
# Output: Status vẫn "healthy"

# Bước 13: Toggle health status (simulate unhealthy)
curl http://localhost:3000/toggle-health
# Output: {"status":"unhealthy"}

# Bước 14: Check health status ngay lập tức
docker inspect health-app | grep -A 5 "Health"
# Output: Status vẫn "healthy" (chưa update)

# Bước 15: Wait cho health check chạy lại
sleep 15
docker inspect health-app | grep -A 5 "Health"
# Output: Status sẽ thay đổi thành "unhealthy"

# Bước 16: Xem health check logs
docker inspect health-app | grep -A 20 "Health"
# Output: Sẽ thấy log của health checks

# Bước 17: Toggle lại thành healthy
curl http://localhost:3000/toggle-health
# Output: {"status":"healthy"}

# Bước 18: Wait và verify
sleep 15
docker inspect health-app | grep -A 5 "Health"
# Output: Status sẽ thay đổi thành "healthy"

# Bước 19: Stop container
docker stop health-app
# Output: health-app

# Bước 20: Remove container
docker rm health-app
# Output: health-app

# Bước 21: Verify cleanup
docker ps -a | grep health-app
# Output: (trống)
```

**Kiểm tra kết quả:**
- ✅ API được tạo với /health endpoint
- ✅ Dockerfile có HEALTHCHECK
- ✅ Image build thành công
- ✅ Container chạy
- ✅ Health status được track
- ✅ Health status update khi thay đổi

**Bonus - Docker Compose với health checks:**

```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    healthcheck:
      test: ["CMD", "node", "healthcheck.js"]
      interval: 10s
      timeout: 3s
      retries: 3
      start_period: 5s
    restart: unless-stopped

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

---

## 🎓 Cấp Độ 3: Nâng Cao

### Bài 11: Full-Stack MERN Application

**Mục tiêu:** Deploy complete MERN stack

**Yêu cầu:**
1. MongoDB database
2. Express API
3. React frontend
4. Nginx reverse proxy
5. Docker Compose với multiple networks
6. Health checks cho tất cả services
7. Persistent volumes

**Cấu trúc:**
```
project/
├── docker-compose.yml
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   └── src/
└── nginx/
    └── nginx.conf
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  mongo:
    image: mongo:7
    restart: unless-stopped
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: secret
    volumes:
      - mongo-data:/data/db
    networks:
      - backend
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh localhost:27017/test --quiet
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build: ./backend
    restart: unless-stopped
    environment:
      NODE_ENV: production
      MONGODB_URI: mongodb://admin:secret@mongo:27017/myapp?authSource=admin
      PORT: 5000
    depends_on:
      mongo:
        condition: service_healthy
    networks:
      - backend
      - frontend
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:5000/health"]
      interval: 30s
      timeout: 3s
      retries: 3

  frontend:
    build: ./frontend
    restart: unless-stopped
    depends_on:
      - backend
    networks:
      - frontend

  nginx:
    image: nginx:alpine
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - frontend
      - backend
    networks:
      - frontend

volumes:
  mongo-data:

networks:
  frontend:
  backend:
```

---

### Bài 12: Microservices Architecture

**Mục tiêu:** Build microservices system

**Yêu cầu:**
1. API Gateway
2. User Service
3. Product Service
4. Order Service
5. Shared PostgreSQL
6. Shared Redis
7. Service discovery
8. Health checks

**Kiến trúc:**
```
Client → API Gateway → User Service    → PostgreSQL
                    → Product Service  → PostgreSQL
                    → Order Service    → PostgreSQL
                                       → Redis
```

---

### Bài 13: CI/CD Pipeline

**Mục tiêu:** Setup CI/CD với Docker

**Yêu cầu:**
1. Tạo GitHub Actions workflow
2. Build Docker image
3. Run tests trong container
4. Push image lên Docker Hub
5. Deploy với Docker Compose

**.github/workflows/docker.yml:**
```yaml
name: Docker CI/CD

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .
      
      - name: Run tests
        run: docker run --rm myapp:${{ github.sha }} npm test
      
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Push image
        run: |
          docker tag myapp:${{ github.sha }} myuser/myapp:latest
          docker push myuser/myapp:latest
```

---

### Bài 14: Monitoring Stack

**Mục tiêu:** Setup monitoring với Prometheus + Grafana

**Yêu cầu:**
1. Prometheus để collect metrics
2. Grafana để visualize
3. Node Exporter cho system metrics
4. cAdvisor cho container metrics
5. Alert rules

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
    volumes:
      - grafana-data:/var/lib/grafana
    depends_on:
      - prometheus

  node-exporter:
    image: prom/node-exporter
    ports:
      - "9100:9100"

  cadvisor:
    image: gcr.io/cadvisor/cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro

volumes:
  prometheus-data:
  grafana-data:
```

---

### Bài 15: Production Deployment

**Mục tiêu:** Deploy production-ready application

**Yêu cầu:**
1. Multi-stage builds
2. Health checks
3. Resource limits
4. Logging configuration
5. Secrets management
6. Backup strategy
7. SSL/TLS
8. Auto-restart policies

**docker-compose.prod.yml:**
```yaml
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.prod
    restart: unless-stopped
    ports:
      - "443:443"
    environment:
      NODE_ENV: production
    env_file:
      - .env.production
    volumes:
      - ./ssl:/etc/ssl:ro
    healthcheck:
      test: ["CMD", "curl", "-f", "https://localhost/health"]
      interval: 30s
      timeout: 3s
      retries: 3
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

---

## 🎯 Projects Thực Tế

### Project 1: Blog Platform

**Stack:** Next.js + PostgreSQL + Redis

**Features:**
- User authentication
- CRUD posts
- Comments
- Image upload
- Caching với Redis
- Full-text search

---

### Project 2: E-commerce API

**Stack:** Node.js + MongoDB + Redis + Elasticsearch

**Features:**
- Product catalog
- Shopping cart
- Order management
- Payment integration
- Search với Elasticsearch
- Session với Redis

---

### Project 3: Real-time Chat

**Stack:** Node.js + Socket.io + Redis + MongoDB

**Features:**
- Real-time messaging
- Multiple rooms
- User presence
- Message history
- File sharing

---

## 💡 Tips

### Debugging

```bash
# Xem logs
docker compose logs -f service-name

# Exec vào container
docker compose exec service-name sh

# Inspect container
docker inspect container-name

# Xem resource usage
docker stats

# Network troubleshooting
docker network inspect network-name
```

### Performance

```bash
# Build với cache
docker build --cache-from myapp:latest -t myapp:new .

# Prune unused resources
docker system prune -a

# Xem disk usage
docker system df
```

---

## 📚 Tài Liệu Tham Khảo

- [Docker Samples](https://github.com/docker/awesome-compose)
- [Play with Docker](https://labs.play-with-docker.com/)
- [Docker Curriculum](https://docker-curriculum.com/)

---

## 📚 Bài Tiếp Theo

👉 [Bài 17: Docker Cho Các Ngôn Ngữ](./17-docker-cho-cac-ngon-ngu.md)

---

**Happy Practicing! 🚀**

