# Bài 13: Environment Variables

## 📋 Mục Tiêu Bài Học

Sau bài học này, bạn sẽ:
- ✅ Hiểu cách sử dụng environment variables
- ✅ Quản lý configuration với .env files
- ✅ Bảo mật secrets và sensitive data
- ✅ Override variables trong runtime
- ✅ Best practices cho configuration management

---

## 🔧 Environment Variables Là Gì?

**Environment Variables** là các biến được sử dụng để configure ứng dụng mà không cần thay đổi code.

### Tại Sao Cần Environment Variables?

```javascript
// ❌ Hardcode configuration
const config = {
  database: 'postgresql://localhost:5432/mydb',
  apiKey: 'secret123',
  port: 3000
};

// ✅ Sử dụng environment variables
const config = {
  database: process.env.DATABASE_URL,
  apiKey: process.env.API_KEY,
  port: process.env.PORT || 3000
};
```

**Lợi ích:**
- ✅ Tách configuration khỏi code
- ✅ Dễ dàng thay đổi giữa environments (dev/staging/prod)
- ✅ Bảo mật secrets
- ✅ Không cần rebuild image khi thay đổi config

---

## 🎯 Cách Sử Dụng Environment Variables

### 1. Trong Dockerfile (Build Time)

```dockerfile
# Set default environment variables
FROM node:18-alpine

# ENV instruction
ENV NODE_ENV=production
ENV PORT=3000
ENV LOG_LEVEL=info

# Multiple variables
ENV NODE_ENV=production \
    PORT=3000 \
    LOG_LEVEL=info

WORKDIR /app
COPY . .

# Sử dụng trong Dockerfile
RUN echo "Building for ${NODE_ENV}"

CMD ["node", "server.js"]
```

### 2. Khi Run Container (Runtime)

```bash
# Method 1: -e flag
docker run -d \
  -e NODE_ENV=production \
  -e PORT=3000 \
  -e DATABASE_URL=postgresql://db:5432/mydb \
  myapp

# Method 2: Multiple -e flags
docker run -d \
  -e NODE_ENV=production \
  -e PORT=3000 \
  -e API_KEY=secret123 \
  myapp

# Method 3: --env flag (same as -e)
docker run -d \
  --env NODE_ENV=production \
  --env PORT=3000 \
  myapp
```

### 3. Từ File (.env)

```bash
# .env file
NODE_ENV=production
PORT=3000
DATABASE_URL=postgresql://db:5432/mydb
API_KEY=secret123
LOG_LEVEL=info

# Run với --env-file
docker run -d \
  --env-file .env \
  myapp

# Multiple env files
docker run -d \
  --env-file .env.common \
  --env-file .env.production \
  myapp
```

---

## 📁 .env Files

### Cấu Trúc .env File

```bash
# .env

# Database
DATABASE_URL=postgresql://postgres:secret@db:5432/mydb
DATABASE_POOL_SIZE=10

# Redis
REDIS_URL=redis://redis:6379
REDIS_TTL=3600

# API Keys
API_KEY=your-api-key-here
SECRET_KEY=your-secret-key-here

# Application
NODE_ENV=production
PORT=3000
LOG_LEVEL=info

# Feature Flags
FEATURE_NEW_UI=true
FEATURE_ANALYTICS=false

# Comments are allowed
# Empty lines are ignored
```

### Multiple Environment Files

```bash
# .env.common (shared across environments)
LOG_LEVEL=info
PORT=3000

# .env.development
NODE_ENV=development
DATABASE_URL=postgresql://localhost:5432/mydb_dev
DEBUG=true

# .env.production
NODE_ENV=production
DATABASE_URL=postgresql://db:5432/mydb_prod
DEBUG=false

# Run với specific environment
docker run -d \
  --env-file .env.common \
  --env-file .env.production \
  myapp
```

### .env File Best Practices

```bash
# ✅ Good .env file structure

# Group related variables
# Database Configuration
DATABASE_URL=postgresql://db:5432/mydb
DATABASE_POOL_SIZE=10

# API Configuration
API_URL=https://api.example.com
API_TIMEOUT=30000

# Use descriptive names
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587

# Document variables
# Maximum number of retries for failed requests
MAX_RETRIES=3

# Use defaults in code, not in .env
# PORT=3000  ← Set in code as fallback
```

---

## 🔒 Bảo Mật Secrets

### ❌ KHÔNG Nên Làm

```bash
# ❌ Commit .env vào Git
git add .env
git commit -m "Add config"

# ❌ Hardcode secrets trong Dockerfile
ENV API_KEY=secret123

# ❌ Log secrets
console.log(process.env.API_KEY)

# ❌ Expose secrets trong image
docker inspect myapp | grep API_KEY
```

### ✅ Nên Làm

#### 1. Sử Dụng .gitignore

```bash
# .gitignore
.env
.env.local
.env.*.local
*.key
*.pem
secrets/
```

#### 2. Template File

```bash
# .env.example (commit vào Git)
DATABASE_URL=postgresql://user:password@host:5432/dbname
API_KEY=your-api-key-here
SECRET_KEY=your-secret-key-here

# .env (không commit, copy từ .env.example)
DATABASE_URL=postgresql://postgres:secret@db:5432/mydb
API_KEY=actual-api-key
SECRET_KEY=actual-secret-key
```

#### 3. Docker Secrets (Swarm Mode)

```bash
# Tạo secret
echo "my-secret-password" | docker secret create db_password -

# Sử dụng trong service
docker service create \
  --name myapp \
  --secret db_password \
  myapp

# Trong container, secret ở /run/secrets/db_password
```

#### 4. Build Secrets (BuildKit)

```dockerfile
# Dockerfile
RUN --mount=type=secret,id=npm_token \
    NPM_TOKEN=$(cat /run/secrets/npm_token) npm install
```

```bash
# Build với secret
docker build --secret id=npm_token,src=.npmrc -t myapp .
```

#### 5. Environment Variables Từ Host

```bash
# Sử dụng env vars từ host
docker run -d \
  -e API_KEY=$API_KEY \
  -e DATABASE_URL=$DATABASE_URL \
  myapp

# Hoặc export trước
export API_KEY=secret123
docker run -d -e API_KEY myapp
```

---

## 🎯 Ví Dụ Thực Tế

### Ví Dụ 1: Node.js Application

**Dockerfile:**
```dockerfile
FROM node:18-alpine

WORKDIR /app

# Default environment variables
ENV NODE_ENV=production
ENV PORT=3000

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

**server.js:**
```javascript
const express = require('express');
const app = express();

// Read from environment variables
const config = {
  env: process.env.NODE_ENV || 'development',
  port: parseInt(process.env.PORT) || 3000,
  database: process.env.DATABASE_URL,
  redis: process.env.REDIS_URL,
  apiKey: process.env.API_KEY,
  logLevel: process.env.LOG_LEVEL || 'info'
};

// Validate required variables
if (!config.database) {
  throw new Error('DATABASE_URL is required');
}

app.listen(config.port, () => {
  console.log(`Server running on port ${config.port}`);
  console.log(`Environment: ${config.env}`);
});
```

**.env:**
```bash
NODE_ENV=production
PORT=3000
DATABASE_URL=postgresql://postgres:secret@db:5432/mydb
REDIS_URL=redis://redis:6379
API_KEY=your-api-key
LOG_LEVEL=info
```

**Run:**
```bash
docker run -d \
  --name myapp \
  --env-file .env \
  -p 3000:3000 \
  myapp
```

### Ví Dụ 2: Python Flask Application

**Dockerfile:**
```dockerfile
FROM python:3.11-slim

WORKDIR /app

ENV FLASK_APP=app.py
ENV FLASK_ENV=production

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["flask", "run", "--host=0.0.0.0"]
```

**app.py:**
```python
import os
from flask import Flask

app = Flask(__name__)

# Configuration from environment
class Config:
    ENV = os.getenv('FLASK_ENV', 'production')
    DEBUG = os.getenv('DEBUG', 'False') == 'True'
    DATABASE_URL = os.getenv('DATABASE_URL')
    SECRET_KEY = os.getenv('SECRET_KEY')
    API_KEY = os.getenv('API_KEY')
    
    # Validate required variables
    @classmethod
    def validate(cls):
        required = ['DATABASE_URL', 'SECRET_KEY']
        missing = [var for var in required if not getattr(cls, var)]
        if missing:
            raise ValueError(f"Missing required env vars: {missing}")

Config.validate()
app.config.from_object(Config)

@app.route('/')
def index():
    return f"Running in {Config.ENV} mode"

if __name__ == '__main__':
    app.run()
```

### Ví Dụ 3: Multi-Container với Docker Compose

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: ${DB_USER:-postgres}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME:-mydb}
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}

  web:
    build: .
    environment:
      NODE_ENV: ${NODE_ENV:-production}
      PORT: ${PORT:-3000}
      DATABASE_URL: postgresql://${DB_USER:-postgres}:${DB_PASSWORD}@db:5432/${DB_NAME:-mydb}
      REDIS_URL: redis://:${REDIS_PASSWORD}@redis:6379
      API_KEY: ${API_KEY}
      SECRET_KEY: ${SECRET_KEY}
    ports:
      - "${PORT:-3000}:3000"
    depends_on:
      - db
      - redis
    env_file:
      - .env

volumes:
  pgdata:
```

**.env:**
```bash
# Database
DB_USER=postgres
DB_PASSWORD=secret123
DB_NAME=mydb

# Redis
REDIS_PASSWORD=redis-secret

# Application
NODE_ENV=production
PORT=3000
API_KEY=your-api-key
SECRET_KEY=your-secret-key
```

**Run:**
```bash
docker compose up -d
```

---

## 🔍 Debug Environment Variables

### Xem Environment Variables

```bash
# Method 1: docker inspect
docker inspect myapp | grep -A 20 Env

# Method 2: docker exec
docker exec myapp env

# Method 3: Specific variable
docker exec myapp printenv NODE_ENV

# Method 4: Trong container
docker exec -it myapp sh
echo $NODE_ENV
printenv
```

### Verify Variables

```bash
# Test với temporary container
docker run --rm \
  -e TEST_VAR=hello \
  alpine \
  sh -c 'echo $TEST_VAR'

# Output: hello
```

---

## 💡 Best Practices

### 1. Sử Dụng .env Files

```bash
# ✅ Organize với .env files
docker run --env-file .env myapp

# ❌ Nhiều -e flags
docker run -e VAR1=val1 -e VAR2=val2 -e VAR3=val3 myapp
```

### 2. Provide Defaults

```javascript
// ✅ Có default values
const port = process.env.PORT || 3000;
const logLevel = process.env.LOG_LEVEL || 'info';

// ❌ Không có defaults
const port = process.env.PORT;  // undefined nếu không set
```

### 3. Validate Required Variables

```javascript
// ✅ Validate khi start
const required = ['DATABASE_URL', 'API_KEY'];
const missing = required.filter(key => !process.env[key]);
if (missing.length > 0) {
  throw new Error(`Missing required env vars: ${missing.join(', ')}`);
}

// ❌ Không validate
const db = process.env.DATABASE_URL;  // Có thể undefined
```

### 4. Type Conversion

```javascript
// ✅ Convert types properly
const port = parseInt(process.env.PORT) || 3000;
const debug = process.env.DEBUG === 'true';
const maxRetries = parseInt(process.env.MAX_RETRIES) || 3;

// ❌ Không convert
const port = process.env.PORT;  // String "3000"
const debug = process.env.DEBUG;  // String "true"
```

### 5. Document Variables

```bash
# .env.example
# Database connection string
# Format: postgresql://user:password@host:port/database
DATABASE_URL=postgresql://user:password@localhost:5432/mydb

# API key for external service
# Get from: https://example.com/api-keys
API_KEY=your-api-key-here

# Log level (debug, info, warn, error)
LOG_LEVEL=info
```

### 6. Separate Secrets

```bash
# .env (general config)
NODE_ENV=production
PORT=3000
LOG_LEVEL=info

# .env.secrets (sensitive data, không commit)
DATABASE_PASSWORD=secret123
API_KEY=secret-key
SECRET_KEY=jwt-secret
```

---

## 🎓 Bài Tập Thực Hành

### Bài 1: Basic Configuration

```bash
# 1. Tạo Node.js app đọc env vars
# 2. Tạo .env file với config
# 3. Run container với --env-file
# 4. Verify variables trong container
```

### Bài 2: Multi-Environment

```bash
# 1. Tạo .env.development và .env.production
# 2. Run container với mỗi environment
# 3. Verify app behavior khác nhau
```

### Bài 3: Docker Compose

```bash
# 1. Tạo docker-compose.yml với env vars
# 2. Tạo .env file
# 3. Run với docker compose
# 4. Verify containers có đúng config
```

---

## 📚 Tóm Tắt

### Methods để Set Environment Variables

| Method | Command | Use Case |
|--------|---------|----------|
| **Dockerfile** | `ENV KEY=value` | Default values |
| **-e flag** | `docker run -e KEY=value` | Single variable |
| **--env-file** | `docker run --env-file .env` | Multiple variables |
| **Compose** | `environment:` in YAML | Multi-container |

### Key Concepts

1. **Separation**: Tách config khỏi code
2. **Security**: Không commit secrets
3. **Validation**: Validate required variables
4. **Defaults**: Provide sensible defaults
5. **Documentation**: Document all variables

---

## 📚 Module Tiếp Theo

Chúc mừng! Bạn đã hoàn thành **Module 3: Docker Containers**.

Trong module tiếp theo, chúng ta sẽ học về **Docker Compose** - tool để quản lý multi-container applications.

👉 [Module 4: Docker Compose](../04-compose/14-docker-compose.md)

---

## 🔗 Tài Liệu Tham Khảo

- [Environment Variables](https://docs.docker.com/engine/reference/commandline/run/#env)
- [Docker Secrets](https://docs.docker.com/engine/swarm/secrets/)
- [12-Factor App Config](https://12factor.net/config)

---

**Happy Configuring! ⚙️**

