# LAB-11: Docker Security Best Practices

> **CRITICAL 2024:** According to the 2024 Docker Security Report, vulnerabilities in container images increased by 25% year-over-year. Security is not optional!

## Table of Contents
- [Overview](#overview)
- [Security Principles](#principles)
- [LAB-11.1: Using Distroless Images](#lab1)
- [LAB-11.2: Running as Non-Root User](#lab2)
- [LAB-11.3: Vulnerability Scanning with Docker Scout](#lab3)
- [LAB-11.4: Vulnerability Scanning with Trivy](#lab4)
- [LAB-11.5: Multi-Stage Builds for Security](#lab5)
- [LAB-11.6: Using Build Secrets Securely](#lab6)
- [LAB-11.7: Security Hardening at Runtime](#lab7)
- [Security Checklist](#checklist)

## Overview <a name="overview"></a>

Docker container security involves multiple layers:
1. **Image Security:** Use minimal, secure base images
2. **Build Security:** Don't include secrets in images
3. **Runtime Security:** Run with least privileges
4. **Scanning:** Regularly scan for vulnerabilities
5. **Updates:** Keep images updated with security patches

## Security Principles <a name="principles"></a>

1. **Principle of Least Privilege:** Run with minimal permissions
2. **Defense in Depth:** Multiple security layers
3. **Minimal Attack Surface:** Smaller images = fewer vulnerabilities
4. **Immutability:** Don't modify running containers
5. **Regular Scanning:** Continuously check for vulnerabilities

---

## LAB-11.1: Using Distroless Images <a name="lab1"></a>

**Distroless images** contain only your application and runtime dependencies—no shell, no package managers, no unnecessary tools.

### Benefits:
- **90%+ reduction** in potential vulnerabilities
- **10-50MB** instead of 100-500MB
- **No shell** means attackers can't execute commands

### Step 1: Traditional Node.js Image (INSECURE)

Create `app-traditional.js`:

```javascript
const http = require('http');
const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello from traditional image!\n');
});
server.listen(3000);
console.log('Server running on port 3000');
```

Create `Dockerfile.traditional`:

```dockerfile
FROM node:20

WORKDIR /app
COPY app-traditional.js .

EXPOSE 3000
CMD ["node", "app-traditional.js"]
```

Build and check size:

```bash
docker build -f Dockerfile.traditional -t app-traditional .
docker images app-traditional
```

You'll see the image is **~1GB**!

### Step 2: Distroless Image (SECURE)

Create `Dockerfile.distroless`:

```dockerfile
FROM node:20 AS builder

WORKDIR /app
COPY app-traditional.js .

# Final stage - Distroless
FROM gcr.io/distroless/nodejs20-debian12

COPY --from=builder /app/app-traditional.js /app/app-traditional.js

WORKDIR /app
EXPOSE 3000
CMD ["app-traditional.js"]
```

Build and compare:

```bash
docker build -f Dockerfile.distroless -t app-distroless .
docker images | grep app-
```

The distroless image is **~180MB** - much smaller!

### Step 3: Test both images

```bash
# Traditional image
docker run -d -p 3001:3000 --name trad app-traditional
curl http://localhost:3001

# Distroless image
docker run -d -p 3002:3000 --name dist app-distroless
curl http://localhost:3002
```

### Step 4: Try to access shell (security test)

```bash
# Traditional - Shell access (DANGEROUS!)
docker exec -it trad sh
# You get a shell! Attacker can run commands!
exit

# Distroless - No shell (SECURE!)
docker exec -it dist sh
# Error: exec failed: unable to start container process: exec: "sh": executable file not found
```

**Result:** Distroless prevents shell access, significantly improving security!

### Step 5: Cleanup

```bash
docker stop trad dist
docker rm trad dist
```

---

## LAB-11.2: Running as Non-Root User <a name="lab2"></a>

> **WARNING:** 58% of container images run as root (UID 0). This is a major security risk!

### Step 1: Check current user (INSECURE)

Create `Dockerfile.root`:

```dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y curl

CMD ["whoami"]
```

Build and run:

```bash
docker build -f Dockerfile.root -t test-root .
docker run test-root
```

Output: `root` ❌ (DANGEROUS!)

### Step 2: Create non-root user (SECURE)

Create `Dockerfile.nonroot`:

```dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Create app directory and set ownership
RUN mkdir -p /app && chown -R appuser:appuser /app

# Switch to non-root user
USER appuser

WORKDIR /app

CMD ["whoami"]
```

Build and run:

```bash
docker build -f Dockerfile.nonroot -t test-nonroot .
docker run test-nonroot
```

Output: `appuser` ✅ (SECURE!)

### Step 3: Real application with non-root user

Create `app.py`:

```python
from http.server import HTTPServer, SimpleHTTPRequestHandler
import os

class MyHandler(SimpleHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        uid = os.getuid()
        message = f"Hello! Running as UID: {uid}\n"
        self.wfile.write(message.encode())

if __name__ == '__main__':
    server = HTTPServer(('0.0.0.0', 8080), MyHandler)
    print('Server started on port 8080...')
    server.serve_forever()
```

Create `Dockerfile.secure`:

```dockerfile
FROM python:3.11-slim

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Create and set app directory
WORKDIR /app
COPY --chown=appuser:appuser app.py .

# Switch to non-root user
USER appuser

EXPOSE 8080
CMD ["python", "app.py"]
```

Build and run:

```bash
docker build -f Dockerfile.secure -t secure-app .
docker run -d -p 8080:8080 --name secureapp secure-app
curl http://localhost:8080
```

You'll see the UID is not 0 (not root)!

### Step 4: Cleanup

```bash
docker stop secureapp
docker rm secureapp
```

---

## LAB-11.3: Vulnerability Scanning with Docker Scout <a name="lab3"></a>

Docker Scout is built into Docker Desktop and provides comprehensive vulnerability scanning.

### Step 1: Enable Docker Scout

```bash
# Check if Docker Scout is available
docker scout version

# Enroll (if not already)
docker scout enroll
```

### Step 2: Scan a vulnerable image

```bash
# Pull an older, vulnerable image for testing
docker pull nginx:1.18

# Quick scan
docker scout quickview nginx:1.18

# Detailed CVE report
docker scout cves nginx:1.18
```

You'll see a list of vulnerabilities!

### Step 3: Get recommendations

```bash
# Get recommendations for fixing vulnerabilities
docker scout recommendations nginx:1.18
```

Docker Scout will suggest updating to a newer version!

### Step 4: Compare images

```bash
# Pull latest nginx
docker pull nginx:latest

# Compare old vs new
docker scout compare nginx:1.18 --to nginx:latest
```

You'll see the improvement!

### Step 5: Scan your own image

```bash
# Build a test image
cat > Dockerfile.scanme <<EOF
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y curl openssl
EOF

docker build -f Dockerfile.scanme -t scanme:test .

# Scan it
docker scout cves scanme:test

# Get detailed report
docker scout cves --format sarif --output report.json scanme:test
```

---

## LAB-11.4: Vulnerability Scanning with Trivy <a name="lab4"></a>

Trivy is an open-source security scanner that's very popular in CI/CD pipelines.

### Step 1: Install Trivy

**Linux:**
```bash
# Install Trivy
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
```

**macOS:**
```bash
brew install trivy
```

**Docker (no installation needed):**
```bash
docker run aquasec/trivy --version
```

### Step 2: Scan an image

```bash
# Scan nginx image
trivy image nginx:1.18

# Show only HIGH and CRITICAL vulnerabilities
trivy image --severity HIGH,CRITICAL nginx:1.18

# Output as JSON
trivy image --format json nginx:1.18 > report.json
```

### Step 3: Scan a Dockerfile

Create `Dockerfile.test`:

```dockerfile
FROM ubuntu:18.04
RUN apt-get update && apt-get install -y python2.7 curl
```

Scan it:

```bash
trivy config Dockerfile.test
```

Trivy will warn about using old Ubuntu and Python 2!

### Step 4: Scan with exit code for CI/CD

```bash
# Exit with code 1 if vulnerabilities found
trivy image --exit-code 1 --severity HIGH,CRITICAL nginx:1.18

echo $?  # Will be 1 if vulnerabilities found
```

This is useful in CI/CD to fail the build if vulnerabilities exist!

---

## LAB-11.5: Multi-Stage Builds for Security <a name="lab5"></a>

Multi-stage builds reduce image size and remove build tools from the final image.

### Step 1: Single-stage build (INSECURE)

Create `app.go`:

```go
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello from secure Go app!\n")
}

func main() {
    http.HandleFunc("/", handler)
    fmt.Println("Server starting on :8080")
    http.ListenAndServe(":8080", nil)
}
```

Create `Dockerfile.singlestage`:

```dockerfile
FROM golang:1.21

WORKDIR /app
COPY app.go .

RUN go build -o app app.go

EXPOSE 8080
CMD ["./app"]
```

Build and check size:

```bash
docker build -f Dockerfile.singlestage -t app-single .
docker images app-single
```

Size: **~1GB** (includes entire Go toolchain!)

### Step 2: Multi-stage build (SECURE)

Create `Dockerfile.multistage`:

```dockerfile
# Build stage
FROM golang:1.21 AS builder

WORKDIR /app
COPY app.go .

RUN CGO_ENABLED=0 go build -ldflags="-s -w" -o app app.go

# Final stage - distroless
FROM gcr.io/distroless/static-debian12

COPY --from=builder /app/app /app

EXPOSE 8080
CMD ["/app"]
```

Build and compare:

```bash
docker build -f Dockerfile.multistage -t app-multi .
docker images | grep app-
```

Size: **~2MB** - **500x smaller**!

### Step 3: Test both

```bash
# Single stage
docker run -d -p 8081:8080 --name single app-single
curl http://localhost:8081

# Multi-stage
docker run -d -p 8082:8080 --name multi app-multi
curl http://localhost:8082
```

Both work, but multi-stage is much more secure!

### Step 4: Cleanup

```bash
docker stop single multi
docker rm single multi
```

---

## LAB-11.6: Using Build Secrets Securely <a name="lab6"></a>

**NEVER** include secrets in your Docker images!

### Step 1: Insecure secret handling (DON'T DO THIS!)

```dockerfile
# ❌ INSECURE - Don't do this!
FROM ubuntu:22.04
COPY .env /app/.env          # Secret in image!
COPY api_key.txt /app/       # Secret in image!
```

These secrets will be in the image layers forever!

### Step 2: Using BuildKit secrets (SECURE)

Create `secret.txt`:

```bash
echo "my-secret-api-key" > secret.txt
```

Create `Dockerfile.secrets`:

```dockerfile
# syntax=docker/dockerfile:1.4

FROM alpine:latest

# Use secret during build (not stored in image)
RUN --mount=type=secret,id=mysecret \
    cat /run/secrets/mysecret && \
    echo "Secret used but not stored!"

CMD ["echo", "App running without secrets in image"]
```

Build with secret:

```bash
docker build --secret id=mysecret,src=secret.txt \
  -f Dockerfile.secrets -t app-secrets .
```

### Step 3: Verify secret is not in image

```bash
docker run app-secrets
docker history app-secrets
```

The secret is NOT in the image history!

### Step 4: Use environment variables at runtime instead

```dockerfile
FROM alpine:latest
# Don't COPY secrets
# Use environment variables at runtime
CMD sh -c 'echo "API Key: $API_KEY"'
```

```bash
docker run -e API_KEY=secret-from-runtime myapp
```

---

## LAB-11.7: Security Hardening at Runtime <a name="lab7"></a>

### Step 1: Run with security options

```bash
# Drop all capabilities
docker run --cap-drop=ALL nginx

# Read-only root filesystem
docker run --read-only nginx

# No new privileges
docker run --security-opt=no-new-privileges nginx

# Combined hardening
docker run \
  --read-only \
  --cap-drop=ALL \
  --security-opt=no-new-privileges \
  --user 1000:1000 \
  nginx
```

### Step 2: Resource limits (prevent DoS)

```bash
# Limit memory
docker run -m 512m nginx

# Limit CPU
docker run --cpus=1 nginx

# Combined
docker run -m 512m --cpus=1 nginx
```

### Step 3: Use Docker Compose v2 with security settings

Create `docker-compose.yml`:

```yaml
services:
  web:
    image: nginx:alpine
    read_only: true
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    user: "101:101"
    mem_limit: 512m
    cpus: 1
    ports:
      - "8080:80"
```

Run it:

```bash
docker compose up -d
docker compose ps
docker compose down
```

---

## Security Checklist <a name="checklist"></a>

### Build Time Security ✅

- [ ] Use minimal base images (Alpine, Distroless)
- [ ] Use specific image tags, not `latest`
- [ ] Multi-stage builds to reduce size
- [ ] Run as non-root user
- [ ] Don't include secrets in images
- [ ] Use `.dockerignore` file
- [ ] Scan images for vulnerabilities
- [ ] Update base images regularly
- [ ] Minimize installed packages

### Runtime Security ✅

- [ ] Run containers with `--read-only` when possible
- [ ] Drop unnecessary capabilities
- [ ] Use `--security-opt=no-new-privileges`
- [ ] Set resource limits (memory, CPU)
- [ ] Use secrets management (Docker secrets, Kubernetes secrets)
- [ ] Run as non-root user
- [ ] Use private networks
- [ ] Enable logging and monitoring
- [ ] Regular security audits

### CI/CD Security ✅

- [ ] Automated vulnerability scanning
- [ ] Fail builds on HIGH/CRITICAL CVEs
- [ ] Sign images
- [ ] Use private registries
- [ ] Implement least privilege access
- [ ] Audit Docker daemon logs
- [ ] Keep Docker updated

---

## Summary

You've learned:
- ✅ Using Distroless images (90%+ vulnerability reduction)
- ✅ Running as non-root user
- ✅ Vulnerability scanning (Docker Scout, Trivy)
- ✅ Multi-stage builds for security
- ✅ Secure secret handling
- ✅ Runtime security hardening

**Remember:** Security is not a one-time task—it's an ongoing process!

## References

- [Docker Security Documentation](https://docs.docker.com/engine/security/)
- [Docker Scout](https://docs.docker.com/scout/)
- [Trivy](https://github.com/aquasecurity/trivy)
- [Distroless Images](https://github.com/GoogleContainerTools/distroless)
- [OWASP Docker Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)
