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

**Gợi ý:**
```bash
docker pull hello-world
docker run hello-world
docker ps -a
docker rm <container-id>
```

---

### Bài 2: Web Server Đơn Giản

**Mục tiêu:** Chạy web server và expose port

**Yêu cầu:**
1. Run Nginx container
2. Expose port 8080 trên host → port 80 trong container
3. Truy cập http://localhost:8080 để verify
4. Xem logs của Nginx
5. Stop và remove container

**Gợi ý:**
```bash
docker run -d -p 8080:80 --name my-nginx nginx:alpine
curl http://localhost:8080
docker logs my-nginx
docker stop my-nginx
docker rm my-nginx
```

---

### Bài 3: Custom HTML Page

**Mục tiêu:** Mount local files vào container

**Yêu cầu:**
1. Tạo file `index.html` với nội dung tùy ý
2. Run Nginx container
3. Mount file `index.html` vào `/usr/share/nginx/html/index.html`
4. Truy cập và verify nội dung

**Gợi ý:**
```bash
echo "<h1>Hello Docker!</h1>" > index.html
docker run -d -p 8080:80 -v $(pwd)/index.html:/usr/share/nginx/html/index.html:ro nginx:alpine
```

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

**Gợi ý:**
```bash
docker run -d \
  --name postgres \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_PASSWORD=mypassword \
  -e POSTGRES_DB=mydb \
  postgres:15

docker exec -it postgres psql -U myuser -d mydb
```

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

**Gợi ý:**
```bash
docker volume create pgdata
docker run -d --name postgres -v pgdata:/var/lib/postgresql/data -e POSTGRES_PASSWORD=secret postgres:15
# Insert data...
docker stop postgres && docker rm postgres
docker run -d --name postgres -v pgdata:/var/lib/postgresql/data -e POSTGRES_PASSWORD=secret postgres:15
# Verify data...
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

**app.js:**
```javascript
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
```

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "app.js"]
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

**Gợi ý:**
```bash
docker network create mynetwork

docker run -d \
  --name postgres \
  --network mynetwork \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

docker run -d \
  --name api \
  --network mynetwork \
  -p 3000:3000 \
  -e DATABASE_URL=postgresql://postgres:secret@postgres:5432/mydb \
  myapi:latest
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

**docker-compose.yml:**
```yaml
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
      - db
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

**Dockerfile:**
```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM nginx:alpine
COPY --from=builder /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

### Bài 10: Health Checks

**Mục tiêu:** Implement health checks

**Yêu cầu:**
1. Tạo API với `/health` endpoint
2. Thêm HEALTHCHECK vào Dockerfile
3. Build và run container
4. Verify health status

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .

HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD node healthcheck.js

EXPOSE 3000
CMD ["node", "server.js"]
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

