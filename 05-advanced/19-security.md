# Bài 19: Docker Security

## 📋 Mục Tiêu

Hướng dẫn bảo mật Docker containers và infrastructure.

---

## 🔒 Image Security

### 1. Sử Dụng Trusted Base Images

```dockerfile
# ✅ Official images từ Docker Hub
FROM node:18-alpine
FROM python:3.11-slim
FROM postgres:15-alpine

# ✅ Verified publishers
FROM bitnami/nginx
FROM redhat/ubi8

# ❌ Random user images
FROM random-user/node
FROM unknown/python
```

### 2. Scan Images for Vulnerabilities

```bash
# Docker Scout (built-in)
docker scout cves myapp:latest
docker scout recommendations myapp:latest

# Trivy
trivy image myapp:latest
trivy image --severity HIGH,CRITICAL myapp:latest

# Snyk
snyk container test myapp:latest

# Clair
clairctl analyze myapp:latest

# Anchore
anchore-cli image add myapp:latest
anchore-cli image vuln myapp:latest all
```

### 3. Keep Images Updated

```bash
# ✅ Regularly update base images
docker pull node:18-alpine
docker build --no-cache -t myapp:latest .

# ✅ Automate updates với Dependabot
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"
```

### 4. Minimize Image Size

```dockerfile
# ✅ Multi-stage build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/server.js"]

# Smaller image = smaller attack surface
```

---

## 👤 User Security

### 1. Non-Root User

```dockerfile
# ✅ Create and use non-root user
FROM node:18-alpine

WORKDIR /app

# Create user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Copy files với ownership
COPY --chown=nodejs:nodejs package*.json ./
RUN npm ci --only=production

COPY --chown=nodejs:nodejs . .

# Switch to non-root user
USER nodejs

EXPOSE 3000
CMD ["node", "server.js"]
```

### 2. Verify User

```bash
# Check user trong container
docker run --rm myapp whoami
# Output: nodejs (not root)

# Inspect user
docker inspect myapp | grep User
```

### 3. Read-Only Root Filesystem

```yaml
# docker-compose.yml
services:
  app:
    image: myapp
    read_only: true
    tmpfs:
      - /tmp
      - /var/run
```

---

## 🔐 Secrets Management

### 1. KHÔNG Hardcode Secrets

```dockerfile
# ❌ KHÔNG làm thế này
ENV API_KEY=secret123
ENV DATABASE_PASSWORD=password123
RUN echo "password123" > /app/secret.txt

# ✅ Sử dụng runtime environment variables
# docker run -e API_KEY=$API_KEY myapp
```

### 2. Docker Secrets (Swarm Mode)

```bash
# Create secret
echo "my-secret-password" | docker secret create db_password -

# Use in service
docker service create \
  --name myapp \
  --secret db_password \
  myapp:latest
```

**Trong container:**
```javascript
const fs = require('fs');
const dbPassword = fs.readFileSync('/run/secrets/db_password', 'utf8');
```

### 3. Build Secrets (BuildKit)

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

# Use secret during build
RUN --mount=type=secret,id=npm_token \
    NPM_TOKEN=$(cat /run/secrets/npm_token) npm install
```

```bash
# Build với secret
docker build --secret id=npm_token,src=.npmrc -t myapp .
```

### 4. Environment Files

```bash
# .env (KHÔNG commit vào Git)
DATABASE_PASSWORD=secret123
API_KEY=secret-key

# .gitignore
.env
.env.local
.env.*.local
secrets/
*.key
*.pem
```

### 5. External Secret Management

```yaml
# Sử dụng HashiCorp Vault
services:
  app:
    image: myapp
    environment:
      VAULT_ADDR: http://vault:8200
      VAULT_TOKEN: ${VAULT_TOKEN}
    command: >
      sh -c "
        export DB_PASSWORD=$(vault kv get -field=password secret/db)
        node server.js
      "
```

---

## 🌐 Network Security

### 1. Network Segmentation

```yaml
version: '3.8'

services:
  # Public-facing
  nginx:
    image: nginx
    networks:
      - frontend
    ports:
      - "80:80"
      - "443:443"
  
  # Internal API
  api:
    image: myapi
    networks:
      - frontend
      - backend
    # Không expose ports
  
  # Database
  db:
    image: postgres
    networks:
      - backend  # Chỉ backend network
    # Không expose ports

networks:
  frontend:
  backend:
    internal: true  # Không có external access
```

### 2. Least Privilege Ports

```yaml
services:
  # ✅ Chỉ expose ports cần thiết
  web:
    image: nginx
    ports:
      - "80:80"
      - "443:443"
  
  # ✅ Internal services
  api:
    image: myapi
    # Không expose ports, chỉ access qua network
  
  db:
    image: postgres
    # KHÔNG expose 5432 ra ngoài
```

### 3. Firewall Rules

```bash
# UFW (Ubuntu)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw deny 5432/tcp  # Block PostgreSQL from outside

# iptables
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -A INPUT -p tcp --dport 5432 -j DROP
```

---

## 🛡️ Container Security

### 1. Drop Capabilities

```yaml
services:
  app:
    image: myapp
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE  # Chỉ add nếu cần
```

### 2. Security Options

```yaml
services:
  app:
    image: myapp
    security_opt:
      - no-new-privileges:true
      - apparmor:docker-default
      - seccomp:default
```

### 3. Resource Limits

```yaml
services:
  app:
    image: myapp
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
          pids: 100  # Limit number of processes
```

### 4. Prevent Privilege Escalation

```yaml
services:
  app:
    image: myapp
    security_opt:
      - no-new-privileges:true
```

---

## 📝 Logging và Auditing

### 1. Container Logs

```yaml
services:
  app:
    image: myapp
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
        labels: "production,security"
```

### 2. Audit Logs

```bash
# Enable Docker daemon audit logging
# /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "audit-log-path": "/var/log/docker-audit.log",
  "audit-log-format": "json"
}
```

### 3. Centralized Logging

```yaml
services:
  app:
    image: myapp
    logging:
      driver: "syslog"
      options:
        syslog-address: "tcp://logserver:514"
        tag: "myapp"
```

---

## 🔍 Security Scanning

### 1. Dockerfile Linting

```bash
# Hadolint
docker run --rm -i hadolint/hadolint < Dockerfile

# Example output:
# DL3006: Always tag the version of an image explicitly
# DL3008: Pin versions in apt get install
```

### 2. Runtime Security

```bash
# Falco - Runtime security
docker run -d \
  --name falco \
  --privileged \
  -v /var/run/docker.sock:/host/var/run/docker.sock \
  -v /dev:/host/dev \
  -v /proc:/host/proc:ro \
  falcosecurity/falco
```

### 3. Compliance Scanning

```bash
# Docker Bench Security
docker run -it --net host --pid host --userns host --cap-add audit_control \
  -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
  -v /var/lib:/var/lib \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /usr/lib/systemd:/usr/lib/systemd \
  -v /etc:/etc --label docker_bench_security \
  docker/docker-bench-security
```

---

## 🔐 Docker Content Trust

### 1. Enable Content Trust

```bash
# Enable Docker Content Trust
export DOCKER_CONTENT_TRUST=1

# Pull signed images only
docker pull nginx:alpine

# Push signed images
docker push myuser/myapp:latest
```

### 2. Sign Images

```bash
# Generate keys
docker trust key generate mykey

# Sign and push
docker trust sign myuser/myapp:latest
```

---

## 🏭 Production Security Checklist

### Image Security
- [ ] Use official/verified base images
- [ ] Scan images for vulnerabilities
- [ ] Use specific version tags
- [ ] Multi-stage builds
- [ ] Minimal base images (Alpine)
- [ ] Regular updates

### User Security
- [ ] Non-root user
- [ ] Read-only filesystem
- [ ] Drop unnecessary capabilities
- [ ] No new privileges

### Secrets Management
- [ ] No secrets in images
- [ ] Use Docker secrets or external vault
- [ ] .env files in .gitignore
- [ ] Rotate secrets regularly

### Network Security
- [ ] Network segmentation
- [ ] Internal networks for databases
- [ ] Minimal port exposure
- [ ] Firewall rules
- [ ] TLS/SSL for external traffic

### Runtime Security
- [ ] Resource limits
- [ ] Health checks
- [ ] Logging enabled
- [ ] Monitoring setup
- [ ] Security scanning in CI/CD

### Compliance
- [ ] Docker Bench Security passed
- [ ] CIS Docker Benchmark compliance
- [ ] Regular security audits
- [ ] Incident response plan

---

## 🎯 Secure Production Example

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:1.25-alpine  # Specific version
    restart: unless-stopped
    ports:
      - "443:443"  # Only HTTPS
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    networks:
      - frontend
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    security_opt:
      - no-new-privileges:true
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  app:
    image: myapp:1.2.3  # Specific version
    restart: unless-stopped
    secrets:
      - db_password
      - api_key
    environment:
      NODE_ENV: production
      DATABASE_PASSWORD_FILE: /run/secrets/db_password
      API_KEY_FILE: /run/secrets/api_key
    networks:
      - frontend
      - backend
    read_only: true
    tmpfs:
      - /tmp
    cap_drop:
      - ALL
    security_opt:
      - no-new-privileges:true
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
          pids: 100
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 3s
      retries: 3
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  db:
    image: postgres:15.5-alpine  # Specific version
    restart: unless-stopped
    secrets:
      - db_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - backend  # Only backend network
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - DAC_OVERRIDE
      - SETGID
      - SETUID
    security_opt:
      - no-new-privileges:true
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

secrets:
  db_password:
    file: ./secrets/db_password.txt
  api_key:
    file: ./secrets/api_key.txt

volumes:
  postgres-data:
    driver: local

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No external access
```

---

## 🛠️ Security Tools

### Scanning Tools
- **Trivy**: Vulnerability scanner
- **Snyk**: Security platform
- **Clair**: Static analysis
- **Anchore**: Policy-based scanning

### Runtime Security
- **Falco**: Runtime threat detection
- **Sysdig**: Container security
- **Aqua Security**: Full platform

### Compliance
- **Docker Bench**: CIS benchmark
- **OpenSCAP**: Security compliance

### Secrets Management
- **HashiCorp Vault**: Secrets management
- **AWS Secrets Manager**: Cloud secrets
- **Azure Key Vault**: Cloud secrets

---

## 💡 Key Takeaways

1. **Defense in Depth**: Multiple layers of security
2. **Least Privilege**: Minimal permissions and access
3. **Secrets Management**: Never hardcode secrets
4. **Regular Updates**: Keep images and dependencies updated
5. **Monitoring**: Log everything, monitor continuously
6. **Automation**: Security scanning in CI/CD

---

## 📚 Bài Tiếp Theo

👉 [Bài 20: Troubleshooting](./20-troubleshooting.md)

---

## 🔗 Tài Liệu Tham Khảo

- [Docker Security Best Practices](https://docs.docker.com/engine/security/)
- [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker)
- [OWASP Docker Security](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)

---

**Stay Secure! 🔒**

