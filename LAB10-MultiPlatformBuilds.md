# LAB-10: Multi-Platform Builds with Docker BuildKit

> **NEW 2024:** Build Docker images that work on multiple architectures (AMD64, ARM64) with a single command!

## Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Why Multi-Platform Builds?](#why)
- [LAB-10.1: Setup Docker Buildx](#lab1)
- [LAB-10.2: Build Simple Multi-Platform Image](#lab2)
- [LAB-10.3: Multi-Platform Go Application](#lab3)
- [LAB-10.4: Multi-Platform Node.js Application](#lab4)
- [LAB-10.5: Build and Push to Registry](#lab5)
- [Real-World Use Cases](#usecases)

## Overview <a name="overview"></a>

Multi-platform builds allow you to create Docker images that work across different CPU architectures with a single build command. This is essential in 2024 as ARM-based systems (Apple Silicon, AWS Graviton, Raspberry Pi) become increasingly common.

## Prerequisites <a name="prerequisites"></a>

- Docker Desktop 4.0+ (includes buildx) OR Docker Engine with buildx plugin
- Basic understanding of Docker and Dockerfiles
- Internet connection (for pulling base images)

## Why Multi-Platform Builds? <a name="why"></a>

**Common Platforms in 2024:**
- `linux/amd64` - Intel/AMD processors (traditional servers, most PCs)
- `linux/arm64` - ARM processors (Apple M1/M2/M3, AWS Graviton, Raspberry Pi)
- `linux/arm/v7` - 32-bit ARM (older Raspberry Pi models)

**Benefits:**
- One image works everywhere
- No need for separate builds per architecture
- Better developer experience
- Cloud cost savings (ARM instances are cheaper)
- Support for Apple Silicon Macs

## LAB-10.1: Setup Docker Buildx <a name="lab1"></a>

### Step 1: Check if buildx is available

```bash
docker buildx version
```

Expected output:
```
github.com/docker/buildx v0.12.0
```

### Step 2: List existing builders

```bash
docker buildx ls
```

You should see the default builder.

### Step 3: Create a new builder instance

```bash
# Create and use a new builder
docker buildx create --name multiplatform --use

# Bootstrap the builder (download necessary components)
docker buildx inspect --bootstrap
```

### Step 4: Verify available platforms

```bash
docker buildx inspect multiplatform
```

Look for the "Platforms" line - you should see multiple platforms like:
```
Platforms: linux/amd64, linux/arm64, linux/arm/v7, ...
```

**Result:** You now have a multi-platform builder ready!

---

## LAB-10.2: Build Simple Multi-Platform Image <a name="lab2"></a>

### Step 1: Create a simple web application

```bash
mkdir multiplatform-test
cd multiplatform-test
```

Create `index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Multi-Platform Docker Demo</title>
</head>
<body>
    <h1>Hello from Multi-Platform Docker!</h1>
    <p>This image works on AMD64 and ARM64!</p>
</body>
</html>
```

### Step 2: Create Dockerfile

Create `Dockerfile`:

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

### Step 3: Build for multiple platforms

```bash
# Build for both AMD64 and ARM64
docker buildx build --platform linux/amd64,linux/arm64 \
  -t multiplatform-nginx:latest \
  --load .
```

**Note:** `--load` loads the image for your current platform. For multi-platform, you'll need to push to a registry (see LAB-10.5).

### Step 4: Build for your current platform and test

```bash
# Build for current platform only
docker buildx build --platform linux/amd64 \
  -t multiplatform-nginx:latest \
  --load .

# Run the container
docker run -d -p 8080:80 --name testweb multiplatform-nginx:latest

# Test it
curl http://localhost:8080
```

### Step 5: Cleanup

```bash
docker stop testweb
docker rm testweb
```

---

## LAB-10.3: Multi-Platform Go Application <a name="lab3"></a>

Go is perfect for multi-platform builds with cross-compilation!

### Step 1: Create Go application

Create `main.go`:

```go
package main

import (
    "fmt"
    "net/http"
    "runtime"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello from Multi-Platform Docker!\n")
    fmt.Fprintf(w, "OS: %s\n", runtime.GOOS)
    fmt.Fprintf(w, "Architecture: %s\n", runtime.GOARCH)
}

func main() {
    http.HandleFunc("/", handler)
    fmt.Println("Server starting on :8080")
    http.ListenAndServe(":8080", nil)
}
```

### Step 2: Create multi-platform Dockerfile

Create `Dockerfile`:

```dockerfile
# syntax=docker/dockerfile:1.4

# Build stage - use build platform
FROM --platform=$BUILDPLATFORM golang:1.21-alpine AS builder

# Build arguments
ARG TARGETPLATFORM
ARG BUILDPLATFORM
ARG TARGETOS
ARG TARGETARCH

WORKDIR /app
COPY main.go .

# Cross-compile for target platform
RUN CGO_ENABLED=0 GOOS=${TARGETOS} GOARCH=${TARGETARCH} \
    go build -ldflags="-s -w" -o app main.go

# Final stage - minimal image
FROM alpine:latest

RUN apk --no-cache add ca-certificates

WORKDIR /app
COPY --from=builder /app/app .

EXPOSE 8080
CMD ["./app"]
```

### Step 3: Build for multiple platforms

```bash
docker buildx build --platform linux/amd64,linux/arm64 \
  -t multiplatform-go:latest \
  -f Dockerfile .
```

### Step 4: Test on your platform

```bash
# Build and load for current platform
docker buildx build --platform linux/amd64 \
  -t multiplatform-go:latest \
  --load .

# Run and test
docker run -d -p 8080:8080 --name goapp multiplatform-go:latest

# Check the output
curl http://localhost:8080
```

You should see output showing the OS and architecture!

### Step 5: Cleanup

```bash
docker stop goapp
docker rm goapp
```

---

## LAB-10.4: Multi-Platform Node.js Application <a name="lab4"></a>

### Step 1: Create Node.js application

Create `package.json`:

```json
{
  "name": "multiplatform-node",
  "version": "1.0.0",
  "main": "server.js",
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

Create `server.js`:

```javascript
const express = require('express');
const os = require('os');
const app = express();

app.get('/', (req, res) => {
  res.json({
    message: 'Hello from Multi-Platform Node.js!',
    platform: process.platform,
    arch: process.arch,
    hostname: os.hostname()
  });
});

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### Step 2: Create multi-platform Dockerfile

Create `Dockerfile`:

```dockerfile
# syntax=docker/dockerfile:1.4

FROM --platform=$BUILDPLATFORM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY server.js ./

# Final stage
FROM node:20-alpine

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copy from builder
COPY --from=builder --chown=nodejs:nodejs /app .

USER nodejs

EXPOSE 3000
CMD ["node", "server.js"]
```

### Step 3: Create .dockerignore

Create `.dockerignore`:

```
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
```

### Step 4: Build multi-platform

```bash
docker buildx build --platform linux/amd64,linux/arm64 \
  -t multiplatform-node:latest \
  .
```

---

## LAB-10.5: Build and Push to Registry <a name="lab5"></a>

To use multi-platform images, you need to push them to a registry.

### Step 1: Login to Docker Hub (or your registry)

```bash
docker login
```

### Step 2: Build and push multi-platform image

```bash
# Replace 'username' with your Docker Hub username
docker buildx build --platform linux/amd64,linux/arm64 \
  -t username/multiplatform-demo:latest \
  --push .
```

**Important:** The `--push` flag automatically pushes after building. You cannot use `--load` with multiple platforms.

### Step 3: Verify the manifest

```bash
# Inspect the image manifest
docker buildx imagetools inspect username/multiplatform-demo:latest
```

You should see multiple platform entries in the output!

### Step 4: Pull and run on any platform

```bash
# This will pull the correct image for your platform
docker pull username/multiplatform-demo:latest
docker run -d -p 8080:80 username/multiplatform-demo:latest
```

---

## Real-World Use Cases <a name="usecases"></a>

### Use Case 1: Development on Apple Silicon, Deploy on Linux AMD64

**Problem:** You develop on M1/M2 Mac (ARM64) but deploy to AWS EC2 (AMD64)

**Solution:**
```bash
# Build for both platforms
docker buildx build --platform linux/amd64,linux/arm64 \
  -t myapp:latest --push .

# On M1 Mac: Uses ARM64 variant
docker run myapp:latest

# On AWS EC2: Uses AMD64 variant
docker run myapp:latest
```

### Use Case 2: Cost Optimization with AWS Graviton

**Problem:** Want to use cheaper ARM-based AWS Graviton instances

**Solution:**
```bash
# Support both traditional and Graviton instances
docker buildx build --platform linux/amd64,linux/arm64 \
  -t myapp:latest --push .

# Deploy to mixed fleet
# - Graviton (ARM64): 40% cheaper
# - Regular EC2 (AMD64): When ARM not available
```

### Use Case 3: IoT and Edge Computing

**Problem:** Deploy to Raspberry Pi (ARM) and regular servers (AMD64)

**Solution:**
```bash
# Build for all ARM variants and AMD64
docker buildx build \
  --platform linux/amd64,linux/arm64,linux/arm/v7 \
  -t iot-app:latest --push .

# Works on:
# - Raspberry Pi 4 (arm64)
# - Raspberry Pi 3 (arm/v7)
# - Cloud servers (amd64)
```

---

## Best Practices

1. **Use Multi-Stage Builds:** Smaller final images
2. **Leverage Build Arguments:** `TARGETPLATFORM`, `TARGETOS`, `TARGETARCH`
3. **Test on Multiple Platforms:** Use QEMU or real hardware
4. **Push to Registry:** Can't load multi-platform locally
5. **Use Cross-Compilation:** Faster than QEMU emulation
6. **Cache Efficiently:** Use BuildKit cache mounts

## Troubleshooting

### Issue: "exec format error"
**Cause:** Wrong architecture image
**Solution:** Build with correct `--platform` flag

### Issue: Build very slow
**Cause:** Using QEMU emulation
**Solution:** Use cross-compilation (see Go example)

### Issue: Cannot load multi-platform
**Cause:** `--load` only works with single platform
**Solution:** Use `--push` to registry or build single platform

## Summary

You've learned:
- ✅ Setting up Docker Buildx
- ✅ Building multi-platform images
- ✅ Cross-compilation techniques
- ✅ Pushing to registries
- ✅ Real-world use cases

Multi-platform builds are essential in 2024 for modern cloud-native applications!

## References

- [Docker Multi-Platform Documentation](https://docs.docker.com/build/building/multi-platform/)
- [BuildKit Documentation](https://docs.docker.com/build/buildkit/)
- [Faster Multi-Platform Builds Guide](https://www.docker.com/blog/faster-multi-platform-builds-dockerfile-cross-compilation-guide/)
