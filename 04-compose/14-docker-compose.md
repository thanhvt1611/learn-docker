# Bài 14: Docker Compose Cơ Bản

## 📋 Mục Tiêu Bài Học

Sau bài học này, bạn sẽ:
- ✅ Hiểu Docker Compose là gì và tại sao cần
- ✅ Viết docker-compose.yml file
- ✅ Quản lý multi-container applications
- ✅ Sử dụng các lệnh Docker Compose cơ bản
- ✅ Build và deploy ứng dụng với Compose

---

## 🎯 Docker Compose Là Gì?

**Docker Compose** là tool để định nghĩa và chạy multi-container Docker applications.

### Vấn Đề Trước Khi Có Compose

```bash
# Chạy database
docker run -d \
  --name postgres \
  --network mynetwork \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15

# Chạy redis
docker run -d \
  --name redis \
  --network mynetwork \
  redis:alpine

# Chạy backend
docker run -d \
  --name api \
  --network mynetwork \
  -e DATABASE_URL=postgresql://postgres:secret@postgres:5432/mydb \
  -e REDIS_URL=redis://redis:6379 \
  -p 3000:3000 \
  myapi:latest

# Chạy frontend
docker run -d \
  --name web \
  --network mynetwork \
  -e API_URL=http://api:3000 \
  -p 80:80 \
  myweb:latest

# 😫 Quá nhiều commands!
# 😫 Khó maintain
# 😫 Dễ sai sót
```

### Giải Pháp: Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:alpine

  api:
    image: myapi:latest
    environment:
      DATABASE_URL: postgresql://postgres:secret@postgres:5432/mydb
      REDIS_URL: redis://redis:6379
    ports:
      - "3000:3000"
    depends_on:
      - postgres
      - redis

  web:
    image: myweb:latest
    environment:
      API_URL: http://api:3000
    ports:
      - "80:80"
    depends_on:
      - api

volumes:
  pgdata:
```

```bash
# Chỉ cần 1 command!
docker compose up -d

# 🎉 Đơn giản hơn nhiều!
```

---

## 📝 docker-compose.yml File

### Cấu Trúc Cơ Bản

```yaml
version: '3.8'  # Compose file version

services:       # Định nghĩa containers
  service1:
    # Configuration cho service1
  service2:
    # Configuration cho service2

networks:       # Định nghĩa networks (optional)
  network1:

volumes:        # Định nghĩa volumes (optional)
  volume1:
```

### Version

```yaml
# Compose file format version
version: '3.8'  # Recommended (Docker Engine 19.03.0+)

# Các versions khác:
# version: '3.7'  # Docker Engine 18.06.0+
# version: '3.6'  # Docker Engine 18.02.0+
# version: '3'    # Generic version 3
```

---

## 🔧 Services

### Định Nghĩa Service

```yaml
services:
  web:
    # Image từ Docker Hub
    image: nginx:alpine
    
    # Hoặc build từ Dockerfile
    build: .
    
    # Container name
    container_name: my-web
    
    # Ports
    ports:
      - "80:80"
      - "443:443"
    
    # Environment variables
    environment:
      NODE_ENV: production
      PORT: 3000
    
    # Volumes
    volumes:
      - ./html:/usr/share/nginx/html
      - nginx-logs:/var/log/nginx
    
    # Networks
    networks:
      - frontend
    
    # Restart policy
    restart: unless-stopped
    
    # Dependencies
    depends_on:
      - api
```

### Image vs Build

```yaml
# Sử dụng existing image
services:
  db:
    image: postgres:15
    
  redis:
    image: redis:alpine

# Build từ Dockerfile
services:
  web:
    build: .
    # Tương đương: docker build .
    
  api:
    build:
      context: ./api
      dockerfile: Dockerfile.prod
      args:
        NODE_VERSION: 18
```

### Ports

```yaml
services:
  web:
    ports:
      # Short syntax
      - "80:80"           # host:container
      - "8080:80"         # different ports
      - "127.0.0.1:80:80" # specific IP
      - "3000-3005:3000-3005" # port range
      
      # Long syntax
      - target: 80        # container port
        published: 8080   # host port
        protocol: tcp
        mode: host
```

### Environment Variables

```yaml
services:
  web:
    # Method 1: Key-value pairs
    environment:
      NODE_ENV: production
      PORT: 3000
      DEBUG: "false"
    
    # Method 2: Array format
    environment:
      - NODE_ENV=production
      - PORT=3000
    
    # Method 3: From .env file
    env_file:
      - .env
      - .env.production
```

### Volumes

```yaml
services:
  db:
    volumes:
      # Named volume
      - pgdata:/var/lib/postgresql/data
      
      # Bind mount
      - ./config:/etc/config
      - ./data:/data
      
      # Anonymous volume
      - /var/log
      
      # Read-only
      - ./config:/etc/config:ro
      
      # Long syntax
      - type: volume
        source: pgdata
        target: /var/lib/postgresql/data
      
      - type: bind
        source: ./config
        target: /etc/config
        read_only: true

volumes:
  pgdata:  # Declare named volume
```

### Networks

```yaml
services:
  web:
    networks:
      - frontend
      - backend
  
  api:
    networks:
      - backend
  
  db:
    networks:
      - backend

networks:
  frontend:
  backend:
```

### Depends On

```yaml
services:
  web:
    depends_on:
      - api
      - redis
    # Web sẽ start sau api và redis
  
  api:
    depends_on:
      - db
    # API sẽ start sau db
  
  db:
    image: postgres:15
    # DB start đầu tiên
```

### Restart Policies

```yaml
services:
  web:
    restart: "no"              # Không restart (default)
    
  api:
    restart: always            # Luôn restart
    
  worker:
    restart: on-failure        # Restart khi fail
    
  db:
    restart: unless-stopped    # Restart trừ khi stop manually
```

---

## 🎯 Ví Dụ Thực Tế

### Ví Dụ 1: WordPress + MySQL

```yaml
version: '3.8'

services:
  db:
    image: mysql:8.0
    container_name: wordpress-db
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - wordpress-network

  wordpress:
    image: wordpress:latest
    container_name: wordpress
    restart: unless-stopped
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress-data:/var/www/html
    depends_on:
      - db
    networks:
      - wordpress-network

volumes:
  db-data:
  wordpress-data:

networks:
  wordpress-network:
```

### Ví Dụ 2: MERN Stack

```yaml
version: '3.8'

services:
  # MongoDB
  mongo:
    image: mongo:7
    container_name: mern-mongo
    restart: unless-stopped
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: secret
    volumes:
      - mongo-data:/data/db
    networks:
      - mern-network

  # Backend (Node.js + Express)
  backend:
    build: ./backend
    container_name: mern-backend
    restart: unless-stopped
    ports:
      - "5000:5000"
    environment:
      NODE_ENV: production
      MONGODB_URI: mongodb://admin:secret@mongo:27017/myapp?authSource=admin
      PORT: 5000
    depends_on:
      - mongo
    networks:
      - mern-network

  # Frontend (React)
  frontend:
    build: ./frontend
    container_name: mern-frontend
    restart: unless-stopped
    ports:
      - "3000:80"
    environment:
      REACT_APP_API_URL: http://localhost:5000
    depends_on:
      - backend
    networks:
      - mern-network

volumes:
  mongo-data:

networks:
  mern-network:
```

### Ví Dụ 3: Microservices

```yaml
version: '3.8'

services:
  # API Gateway
  gateway:
    build: ./gateway
    ports:
      - "80:80"
    environment:
      USER_SERVICE_URL: http://user-service:3001
      ORDER_SERVICE_URL: http://order-service:3002
      PRODUCT_SERVICE_URL: http://product-service:3003
    depends_on:
      - user-service
      - order-service
      - product-service
    networks:
      - microservices

  # User Service
  user-service:
    build: ./services/user
    environment:
      DATABASE_URL: postgresql://postgres:secret@postgres:5432/users
      REDIS_URL: redis://redis:6379
    depends_on:
      - postgres
      - redis
    networks:
      - microservices

  # Order Service
  order-service:
    build: ./services/order
    environment:
      DATABASE_URL: postgresql://postgres:secret@postgres:5432/orders
      REDIS_URL: redis://redis:6379
    depends_on:
      - postgres
      - redis
    networks:
      - microservices

  # Product Service
  product-service:
    build: ./services/product
    environment:
      DATABASE_URL: postgresql://postgres:secret@postgres:5432/products
      REDIS_URL: redis://redis:6379
    depends_on:
      - postgres
      - redis
    networks:
      - microservices

  # PostgreSQL
  postgres:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - microservices

  # Redis
  redis:
    image: redis:alpine
    networks:
      - microservices

volumes:
  postgres-data:

networks:
  microservices:
```

---

## 🚀 Docker Compose Commands

### docker compose up

```bash
# Start tất cả services
docker compose up

# Start ở background (detached)
docker compose up -d

# Build images trước khi start
docker compose up --build

# Start specific services
docker compose up web api

# Force recreate containers
docker compose up --force-recreate

# Scale services
docker compose up --scale worker=3
```

### docker compose down

```bash
# Stop và remove containers, networks
docker compose down

# Remove volumes cũng
docker compose down -v

# Remove images cũng
docker compose down --rmi all

# Remove only local images
docker compose down --rmi local
```

### docker compose ps

```bash
# List running services
docker compose ps

# List all services (including stopped)
docker compose ps -a

# Show services status
docker compose ps --services
```

### docker compose logs

```bash
# Xem logs của tất cả services
docker compose logs

# Follow logs
docker compose logs -f

# Logs của specific service
docker compose logs web

# Tail logs
docker compose logs --tail=100 web

# Logs với timestamps
docker compose logs -t
```

### docker compose exec

```bash
# Execute command trong service
docker compose exec web sh

# Run command
docker compose exec api npm test

# With user
docker compose exec -u root web sh
```

### docker compose build

```bash
# Build tất cả services
docker compose build

# Build specific service
docker compose build web

# Build không dùng cache
docker compose build --no-cache

# Build với build args
docker compose build --build-arg VERSION=1.0.0
```

### docker compose start/stop/restart

```bash
# Start services
docker compose start

# Stop services (không remove)
docker compose stop

# Restart services
docker compose restart

# Restart specific service
docker compose restart web
```

### docker compose pull

```bash
# Pull tất cả images
docker compose pull

# Pull specific service
docker compose pull web
```

### docker compose config

```bash
# Validate và view compose file
docker compose config

# View với resolved values
docker compose config --resolve-image-digests
```

---

## 🔍 .env File với Compose

### .env File

```bash
# .env
NODE_ENV=production
DB_PASSWORD=secret123
API_PORT=3000
WEB_PORT=80
```

### Sử Dụng Trong docker-compose.yml

```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
  
  api:
    build: ./api
    ports:
      - "${API_PORT}:3000"
    environment:
      NODE_ENV: ${NODE_ENV}
      DATABASE_URL: postgresql://postgres:${DB_PASSWORD}@db:5432/mydb
  
  web:
    build: ./web
    ports:
      - "${WEB_PORT}:80"
    environment:
      API_URL: http://localhost:${API_PORT}
```

### Default Values

```yaml
services:
  web:
    ports:
      - "${WEB_PORT:-80}:80"  # Default: 80
    environment:
      NODE_ENV: ${NODE_ENV:-production}  # Default: production
```

---

## 💡 Best Practices

### 1. Sử Dụng Version 3.8+

```yaml
# ✅ Specific version
version: '3.8'

# ❌ Old version
version: '2'
```

### 2. Named Volumes

```yaml
# ✅ Named volumes
volumes:
  - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:

# ❌ Anonymous volumes
volumes:
  - /var/lib/postgresql/data
```

### 3. Health Checks

```yaml
services:
  web:
    image: nginx
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 10s
```

### 4. Resource Limits

```yaml
services:
  web:
    image: nginx
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

### 5. Logging

```yaml
services:
  web:
    image: nginx
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

---

## 🎓 Bài Tập Thực Hành

### Bài 1: Simple Web App

Tạo docker-compose.yml cho:
- Nginx web server (port 80)
- Node.js API (port 3000)
- PostgreSQL database

### Bài 2: Full Stack App

Tạo docker-compose.yml cho:
- React frontend
- Express backend
- MongoDB database
- Redis cache

### Bài 3: Development Environment

Tạo docker-compose.yml với:
- Hot reload cho frontend và backend
- Bind mounts cho source code
- Named volumes cho node_modules

---

## 📚 Tóm Tắt

### Key Concepts

1. **docker-compose.yml**: Định nghĩa multi-container app
2. **Services**: Containers trong application
3. **Networks**: Kết nối giữa services
4. **Volumes**: Persistent storage
5. **Environment**: Configuration variables

### Essential Commands

```bash
docker compose up -d        # Start services
docker compose down         # Stop và remove
docker compose ps           # List services
docker compose logs -f      # View logs
docker compose exec         # Execute command
docker compose build        # Build images
```

---

## 📚 Bài Tiếp Theo

Trong bài tiếp theo, chúng ta sẽ học **Docker Compose Nâng Cao** với nhiều tính năng advanced hơn.

👉 [Bài 15: Docker Compose Nâng Cao](./15-docker-compose-advanced.md)

---

## 🔗 Tài Liệu Tham Khảo

- [Docker Compose Overview](https://docs.docker.com/compose/)
- [Compose File Reference](https://docs.docker.com/compose/compose-file/)
- [Compose CLI Reference](https://docs.docker.com/compose/reference/)

---

**Happy Composing! 🎼**

