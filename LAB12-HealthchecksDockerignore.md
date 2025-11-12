# LAB-12: Healthchecks and .dockerignore Best Practices

> **Important 2024:** Without healthchecks, Docker cannot detect if your containerized service is unhealthy! Learn how to implement proper health monitoring and optimize your build context.

## Table of Contents
- [Overview](#overview)
- [Part 1: Healthchecks](#part1)
  - [LAB-12.1: Basic HTTP Healthcheck](#lab1)
  - [LAB-12.2: Database Healthcheck](#lab2)
  - [LAB-12.3: Custom Healthcheck Script](#lab3)
  - [LAB-12.4: Healthchecks in Docker Compose v2](#lab4)
  - [LAB-12.5: Dependent Services with Healthchecks](#lab5)
- [Part 2: .dockerignore](#part2)
  - [LAB-12.6: Understanding Build Context](#lab6)
  - [LAB-12.7: Creating Comprehensive .dockerignore](#lab7)
  - [LAB-12.8: Testing .dockerignore Effectiveness](#lab8)
- [Best Practices](#bestpractices)

---

## Overview <a name="overview"></a>

### Healthchecks
A container might be "running" but the application inside could be unresponsive. Healthchecks tell Docker how to test if your container is working correctly.

### .dockerignore
The `.dockerignore` file tells Docker which files to exclude from the build context. This:
- Makes builds faster
- Reduces image size
- Prevents accidentally including secrets
- Improves layer caching

---

# Part 1: Healthchecks <a name="part1"></a>

## LAB-12.1: Basic HTTP Healthcheck <a name="lab1"></a>

### Step 1: Container without healthcheck (BAD)

Create `app-no-health.js`:

```javascript
const http = require('http');

let isHealthy = true;

const server = http.createServer((req, res) => {
  if (req.url === '/') {
    res.writeHead(200);
    res.end('Hello World!\n');
  } else if (req.url === '/crash') {
    // Simulate app becoming unhealthy
    isHealthy = false;
    res.writeHead(200);
    res.end('App is now unhealthy!\n');
  } else if (req.url === '/health') {
    if (isHealthy) {
      res.writeHead(200);
      res.end('OK\n');
    } else {
      res.writeHead(500);
      res.end('NOT OK\n');
    }
  }
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

Create `Dockerfile.no-health`:

```dockerfile
FROM node:20-alpine

WORKDIR /app
COPY app-no-health.js .

EXPOSE 3000
CMD ["node", "app-no-health.js"]
```

Build and run:

```bash
docker build -f Dockerfile.no-health -t app-nohealth .
docker run -d -p 3000:3000 --name nohealth app-nohealth

# Check status
docker ps
```

Status shows "Up" - but let's break the app:

```bash
# Make the app unhealthy
curl http://localhost:3000/crash

# Check if app still works
curl http://localhost:3000/health
# Returns 500 error!

# But Docker still thinks it's healthy
docker ps
# Still shows "Up"!
```

### Step 2: Container with healthcheck (GOOD)

Create `Dockerfile.with-health`:

```dockerfile
FROM node:20-alpine

# Install curl for healthcheck
RUN apk add --no-cache curl

WORKDIR /app
COPY app-no-health.js .

EXPOSE 3000

# Add healthcheck
HEALTHCHECK --interval=10s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

CMD ["node", "app-no-health.js"]
```

Build and run:

```bash
docker build -f Dockerfile.with-health -t app-health .
docker run -d -p 3001:3000 --name withhealth app-health

# Wait 10 seconds for first healthcheck
sleep 10

# Check status
docker ps
# Shows "healthy" in STATUS column!

# Break the app
curl http://localhost:3001/crash

# Wait for healthcheck to detect (30+ seconds)
sleep 35

# Check status again
docker ps
# Shows "unhealthy"!
```

**Result:** Docker now knows the app is unhealthy!

### Step 3: Inspect healthcheck details

```bash
# View detailed health information
docker inspect --format='{{json .State.Health}}' withhealth | jq

# View health logs
docker inspect --format='{{range .State.Health.Log}}{{.Output}}{{end}}' withhealth
```

### Step 4: Cleanup

```bash
docker stop nohealth withhealth
docker rm nohealth withhealth
```

---

## LAB-12.2: Database Healthcheck <a name="lab2"></a>

### Step 1: PostgreSQL with healthcheck

Create `docker-compose-db.yml`:

```yaml
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: testuser
      POSTGRES_PASSWORD: testpass
      POSTGRES_DB: testdb
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U testuser -d testdb"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
```

Start and monitor:

```bash
docker compose -f docker-compose-db.yml up -d

# Watch healthcheck status
watch -n 1 'docker compose -f docker-compose-db.yml ps'
```

You'll see the status change from "starting" to "healthy"!

### Step 2: MySQL with healthcheck

Create `docker-compose-mysql.yml`:

```yaml
services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: testdb
      MYSQL_USER: testuser
      MYSQL_PASSWORD: testpass
    ports:
      - "3306:3306"
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-prootpass"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
```

Start it:

```bash
docker compose -f docker-compose-mysql.yml up -d
docker compose -f docker-compose-mysql.yml ps
```

### Step 3: Cleanup

```bash
docker compose -f docker-compose-db.yml down
docker compose -f docker-compose-mysql.yml down
```

---

## LAB-12.3: Custom Healthcheck Script <a name="lab3"></a>

For complex applications, use a custom healthcheck script.

### Step 1: Create application with dependencies

Create `complex-app.py`:

```python
from http.server import HTTPServer, BaseHTTPRequestHandler
import json

class HealthHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/health':
            # Complex health check logic
            health_status = self.check_health()

            if health_status['healthy']:
                self.send_response(200)
            else:
                self.send_response(503)

            self.send_header('Content-type', 'application/json')
            self.end_headers()
            self.wfile.write(json.dumps(health_status).encode())
        else:
            self.send_response(200)
            self.end_headers()
            self.wfile.write(b'Hello World!')

    def check_health(self):
        # Check multiple components
        checks = {
            'database': self.check_database(),
            'cache': self.check_cache(),
            'disk_space': self.check_disk_space()
        }

        all_healthy = all(checks.values())

        return {
            'healthy': all_healthy,
            'checks': checks
        }

    def check_database(self):
        # Simulate database check
        return True

    def check_cache(self):
        # Simulate cache check
        return True

    def check_disk_space(self):
        # Simulate disk space check
        import shutil
        stat = shutil.disk_usage('/')
        free_percent = (stat.free / stat.total) * 100
        return free_percent > 10

if __name__ == '__main__':
    server = HTTPServer(('0.0.0.0', 8080), HealthHandler)
    print('Server starting on port 8080...')
    server.serve_forever()
```

### Step 2: Create Dockerfile with healthcheck

Create `Dockerfile.complex`:

```dockerfile
FROM python:3.11-alpine

WORKDIR /app
COPY complex-app.py .

EXPOSE 8080

# Comprehensive healthcheck
HEALTHCHECK --interval=15s --timeout=5s --start-period=10s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health || exit 1

CMD ["python", "complex-app.py"]
```

### Step 3: Test it

```bash
docker build -f Dockerfile.complex -t complex-app .
docker run -d -p 8080:8080 --name complex complex-app

# Check health endpoint
curl http://localhost:8080/health | jq

# Monitor health status
docker ps
sleep 15
docker inspect --format='{{.State.Health.Status}}' complex
```

### Step 4: Cleanup

```bash
docker stop complex
docker rm complex
```

---

## LAB-12.4: Healthchecks in Docker Compose v2 <a name="lab4"></a>

Create `docker-compose-health.yml`:

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost/"]
      interval: 10s
      timeout: 3s
      retries: 3
      start_period: 5s

  api:
    build:
      context: .
      dockerfile: Dockerfile.with-health
    ports:
      - "3000:3000"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 10s
      timeout: 3s
      retries: 3
      start_period: 5s

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5
```

Start and monitor:

```bash
docker compose -f docker-compose-health.yml up -d

# Watch all services become healthy
docker compose -f docker-compose-health.yml ps

# Cleanup
docker compose -f docker-compose-health.yml down
```

---

## LAB-12.5: Dependent Services with Healthchecks <a name="lab5"></a>

**NEW in Compose v2:** Wait for services to be healthy before starting dependent services!

Create `docker-compose-depends.yml`:

```yaml
services:
  database:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 5s
      timeout: 3s
      retries: 5

  backend:
    image: nginx:alpine
    depends_on:
      database:
        condition: service_healthy
    ports:
      - "8080:80"

  frontend:
    image: nginx:alpine
    depends_on:
      backend:
        condition: service_started
    ports:
      - "8081:80"
```

Start it:

```bash
docker compose -f docker-compose-depends.yml up -d

# Watch startup order
docker compose -f docker-compose-depends.yml logs -f
```

You'll see:
1. Database starts first
2. Backend waits for database to be **healthy**
3. Frontend starts after backend

Cleanup:

```bash
docker compose -f docker-compose-depends.yml down
```

---

# Part 2: .dockerignore <a name="part2"></a>

## LAB-12.6: Understanding Build Context <a name="lab6"></a>

### Step 1: Create a messy project

```bash
mkdir dockerignore-test
cd dockerignore-test

# Create app files
echo "console.log('Hello');" > app.js

# Create files that shouldn't be in image
mkdir node_modules
echo "large dependency" > node_modules/package.txt

mkdir .git
echo "git history" > .git/config

echo "SECRET_KEY=abc123" > .env
echo "password=secret" > secrets.txt

# Create large files
dd if=/dev/zero of=large-file.bin bs=1M count=100

# Create build artifacts
mkdir dist
echo "built file" > dist/app.min.js
```

### Step 2: Build without .dockerignore

Create `Dockerfile`:

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY . .
CMD ["node", "app.js"]
```

Build and watch:

```bash
# Watch the build context size
docker build -t test-no-ignore .
```

You'll see: "Sending build context to Docker daemon: **~105MB**"

**Problem:** Everything gets sent to Docker daemon, including secrets!

---

## LAB-12.7: Creating Comprehensive .dockerignore <a name="lab7"></a>

### Step 1: Create .dockerignore

Create `.dockerignore`:

```
# Git
.git
.gitignore
.gitattributes

# Dependencies
node_modules/
venv/
__pycache__/
*.pyc
vendor/

# Build outputs
dist/
build/
target/
out/
*.egg-info/

# IDE
.vscode/
.idea/
*.swp
*.swo
*~
.DS_Store

# Testing
coverage/
.coverage
*.test
test/
tests/
spec/

# CI/CD
.github/
.gitlab-ci.yml
.circleci/
Jenkinsfile

# Docker
docker-compose*.yml
Dockerfile*
.dockerignore

# Documentation
README.md
CHANGELOG.md
LICENSE
docs/
*.md

# Secrets (CRITICAL!)
.env
.env.*
*.pem
*.key
*.crt
secrets/
secrets.txt
credentials.json

# Logs
*.log
logs/

# OS
Thumbs.db
.DS_Store

# Large files
*.bin
*.tar
*.zip
*.gz
*.mp4

# Keep specific files
!.env.example
!README.docker.md
```

### Step 2: Build with .dockerignore

```bash
docker build -t test-with-ignore .
```

You'll see: "Sending build context to Docker daemon: **~3KB**"

**Result:** **35x smaller** build context! Secrets excluded!

### Step 3: Verify secrets are not in image

```bash
# Try to find secrets in image
docker run --rm test-with-ignore ls -la
docker run --rm test-with-ignore cat .env
# Should fail - file not found!
```

---

## LAB-12.8: Testing .dockerignore Effectiveness <a name="lab8"></a>

### Step 1: Create test script

Create `check-ignore.sh`:

```bash
#!/bin/bash

echo "=== Building Docker image ==="
docker build -t ignore-test . 2>&1 | grep "Sending build context"

echo ""
echo "=== Checking what's in the image ==="
docker run --rm ignore-test ls -la /app

echo ""
echo "=== Checking for secrets ==="
if docker run --rm ignore-test test -f /app/.env 2>/dev/null; then
    echo "❌ WARNING: .env file found in image!"
else
    echo "✅ .env file not in image (good)"
fi

if docker run --rm ignore-test test -f /app/secrets.txt 2>/dev/null; then
    echo "❌ WARNING: secrets.txt found in image!"
else
    echo "✅ secrets.txt not in image (good)"
fi

echo ""
echo "=== Checking for node_modules ==="
if docker run --rm ignore-test test -d /app/node_modules 2>/dev/null; then
    echo "❌ WARNING: node_modules found in image!"
else
    echo "✅ node_modules not in image (good)"
fi
```

Run it:

```bash
chmod +x check-ignore.sh
./check-ignore.sh
```

---

## Best Practices <a name="bestpractices"></a>

### Healthcheck Best Practices

1. **Always include healthchecks** in production images
2. **Set appropriate intervals:**
   - Web apps: 10-30s
   - Databases: 5-10s
   - Batch jobs: 1-5 minutes
3. **Use start_period** for slow-starting apps
4. **Keep checks lightweight** (fast response)
5. **Check actual functionality**, not just "is process running"
6. **Use depends_on with condition: service_healthy** in Compose v2

### Healthcheck Parameters Explained

```dockerfile
HEALTHCHECK --interval=30s    # Run check every 30s
            --timeout=3s      # Check must complete in 3s
            --start-period=40s # Don't check for first 40s
            --retries=3       # Need 3 failures to mark unhealthy
  CMD curl -f http://localhost/health || exit 1
```

### .dockerignore Best Practices

1. **Always create .dockerignore** for every Dockerfile
2. **Start with comprehensive template** (see LAB-12.7)
3. **Exclude secrets** (.env, *.pem, *.key)
4. **Exclude dependencies** (node_modules, venv)
5. **Exclude build outputs** (dist/, build/)
6. **Exclude version control** (.git/)
7. **Use ! to include exceptions**
8. **Test effectiveness** (check build context size)

### Common Mistakes to Avoid

❌ **Don't:**
- Skip healthchecks in production
- Include secrets in images
- Send entire project to Docker daemon
- Use `latest` without healthcheck
- Make healthchecks too slow

✅ **Do:**
- Add healthchecks to all services
- Use .dockerignore in every project
- Check that services are actually working
- Set appropriate timeouts and intervals
- Test health endpoints before deploying

---

## Summary

You've learned:
- ✅ Implementing HTTP, database, and custom healthchecks
- ✅ Using healthchecks in Docker Compose v2
- ✅ Creating dependent services with health conditions
- ✅ Creating comprehensive .dockerignore files
- ✅ Excluding secrets and unnecessary files
- ✅ Testing .dockerignore effectiveness

**Key Takeaway:** Healthchecks and .dockerignore are essential for production-ready containers!

---

## Real-World Example: Complete Production Setup

Create `production-example/`:

**.dockerignore:**
```
node_modules/
.git/
.env
*.log
test/
```

**Dockerfile:**
```dockerfile
FROM node:20-alpine

RUN apk add --no-cache curl

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci --only=production

# Copy app
COPY . .

# Healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Non-root user
RUN addgroup -S appuser && adduser -S appuser -G appuser
USER appuser

EXPOSE 3000
CMD ["node", "server.js"]
```

**docker-compose.yml:**
```yaml
services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5

  app:
    build: .
    depends_on:
      db:
        condition: service_healthy
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgres://db:5432/myapp
```

This is production-ready! 🚀

---

## References

- [Docker Healthcheck Documentation](https://docs.docker.com/engine/reference/builder/#healthcheck)
- [Docker Compose Healthcheck](https://docs.docker.com/compose/compose-file/#healthcheck)
- [.dockerignore Documentation](https://docs.docker.com/engine/reference/builder/#dockerignore-file)
