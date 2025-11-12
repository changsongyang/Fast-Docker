# Fast-Docker
This repo aims to cover Docker details (Dockerfile, Image, Container, Commands, Volumes, Docker-Compose, Networks, Security, Multi-Platform Builds) quickly, and possible example usage scenarios (HowTo: LABs) in a nutshell. This guide is updated with the latest Docker features and best practices for 2024/2025.

**Keywords:** Docker-Image, Dockerfile, Containerization, Docker-Compose, Docker-Volume, Docker-Network, Docker-Security, BuildKit, Multi-Platform, Distroless, Container-Security, Healthchecks, Cheatsheet.

> **Note:** This guide has been updated for Docker 2024/2025 including Docker Compose v2, BuildKit, Multi-Platform Builds, and modern security practices.

# Quick Look (HowTo: LABs)

## Basic Docker Concepts
- [LAB-01: Creating First Docker Image and Container using Docker File](https://github.com/omerbsezer/Fast-Docker/blob/main/LAB01-FirstImageFirstContainer.md)
- [LAB-02: Binding Volume to the Different Containers](https://github.com/omerbsezer/Fast-Docker/blob/main/LAB02-DockerVolume.md)
    - [LAB-02.1: Binding Mount to the Container](https://github.com/omerbsezer/Fast-Docker/blob/main/LAB02-DockerVolume.md#app_mount)
- [LAB-03: Docker-Compose v2 - Creating 2 Different Containers: WordPress Container depends on MySql Container](https://github.com/omerbsezer/Fast-Docker/blob/main/LAB03-DockerCompose.md)
- [LAB-06: Transferring Content between Host PC and Docker Container](https://github.com/omerbsezer/Fast-Docker/blob/main/LAB06-DockerTransferringContent.md)

## Advanced Docker Features (NEW 2024/2025)
- [LAB-10: Multi-Platform Builds with Docker BuildKit](https://github.com/omerbsezer/Fast-Docker/blob/main/LAB10-MultiPlatformBuilds.md)
- [LAB-11: Docker Security Best Practices - Distroless Images, Non-Root User, Vulnerability Scanning](https://github.com/omerbsezer/Fast-Docker/blob/main/LAB11-SecurityBestPractices.md)
- [LAB-12: Healthchecks and .dockerignore Best Practices](https://github.com/omerbsezer/Fast-Docker/blob/main/LAB12-HealthchecksDockerignore.md)

## Docker Registry & Configuration
- [LAB-05: Running Docker Free Local Registry, Tagging Image, Pushing Image to the Local Registry, Pulling Image From Local Registry and Deleting Images from Local Registry](https://github.com/omerbsezer/Fast-Docker/blob/main/LAB05-DockerLocalRegistry.md)
- [LAB-09: Docker Configuration (Proxy, Registry)](https://github.com/omerbsezer/Fast-Docker/blob/main/LAB09-DockerConfiguration.md)

## Docker Build Examples
- [LAB-07: Creating Docker Container using Dockerfile to Build C++ on Ubuntu18.04](https://github.com/omerbsezer/Fast-Docker/blob/main/LAB07-DockerfileForLinuxC++Build.md)
- [LAB-08: Creating Docker Container using Dockerfile to Build C++ on Windows](https://github.com/omerbsezer/Fast-Docker/blob/main/LAB08-DockerfileForWindowsC++Build.md)

## Legacy Topics (For Reference)
- [LAB-04: Creating Docker Swarm Cluster (Legacy - Use Kubernetes instead)](https://github.com/omerbsezer/Fast-Docker/blob/main/LAB04-DockerStackService.md)

## Reference
- [Docker Commands Cheatsheet](https://github.com/omerbsezer/Fast-Docker/blob/main/DockerCommandCheatSheet.md)

# Table of Contents
- [Motivation](#motivation)
    - [Needs](#needs)
    - [Benefits](#benefits)
    - [Problems Docker does not solve](#problems)
- [What is Docker?](#whatisdocker)
    - [Architecture](#architecture)
    - [Installation](#installation)
    - [Docker Engine (Daemon, REST API, CLI)](#engine)
    - [Docker Registry and Docker Hub](#registry)
    - [Docker Command Structure](#command)
    - [Docker Container](#container)
    - [Docker Volumes/Bind Mounts](#volume)
    - [Docker Network](#network)
    - [Docker Log](#log)
    - [Docker Stats/Memory-CPU Limitations](#stats)
    - [Docker Environment Variables](#variables)
    - [Docker File](#file)
    - [Docker Image](#image)
    - [Docker Compose v2 (Updated 2024)](#compose)
    - [Docker BuildKit (NEW 2024)](#buildkit)
    - [Multi-Platform Builds (NEW 2024)](#multiplatform)
    - [Docker Security Best Practices (NEW 2024)](#security)
        - [Distroless Images](#distroless)
        - [Non-Root User](#nonroot)
        - [Vulnerability Scanning](#scanning)
    - [Healthchecks (NEW 2024)](#healthchecks)
    - [.dockerignore Best Practices (NEW 2024)](#dockerignore)
    - [Docker Swarm (Legacy)](#swarm)
    - [Docker Stack / Docker Service (Legacy)](#stack)
- [Play With Docker](#playwithdocker)
- [Docker Commands Cheatsheet](#cheatsheet)
- [Container Orchestration: Kubernetes vs Swarm](#orchestration)
- [Other Useful Resources Related Docker](#resource)
- [References](#references)

## Motivation <a name="motivation"></a>
Why should we use Docker? "Docker changed the way applications used to build and ship. It has completely revolutionized the containerization world." (Ref:ItNext)

### Needs <a name="needs"></a>
- Installing all dependencies, setting up a new environment for SW (time-consuming every time to install environment for testing ) 
- We want to run our apps on different platforms (Ubuntu, Windows, Raspberry Pi).
    - Question in our mind: What if, it does not run on a different OS?
- CI/CD Integration Testing: We can handle unit testing, component testing with Jenkins. What if integration testing? 
    - Extending Chain: Jenkins- Docker Image - Docker Container - Automatic testing
- Are our SW products portable to carry on different PC easily? (especially in the development & testing phase)
- Developing, testing, maintenance of code as one Monolithic App could be problematic when the app needs more features/services. It is required to convert one big monolithic app into microservices. 

### Benefits <a name="benefits"></a>

- NOT needed to install dependencies/SWs again & again
- Enables to run on different OS, different platforms
- Enables a consistent environment
- Enables more efficient use of system resources
- Easy to use and maintain
- Efficient use of the system resources
- Isolate SW components
- Enables faster software delivery cycles
- Containers give us instant application portability.
- Enables developers to easily pack, ship, and run any application as a lightweight, portable, self-sufficient container
- Microservice Architecture (Monolithic Apps to MicroService Architecture, e.g. [Cloud Native App](https://docs.microsoft.com/en-us/dotnet/architecture/cloud-native/introduction))

(Ref: Infoworld)

### Problems Docker does not solve<a name="problems"></a>
- Docker does NOT fix your security issues
- Docker does NOT turn applications magically into microservices
- Docker isn’t a substitute for virtual machines

(Ref: Infoworld)

## What is Docker?  <a name="whatisdocker"></a>
- Docker is a tool that reduces the gap between the Development/Deployment phase of a software development cycle.
- Docker is like a VM but it has more features than VMs (no kernel, only small app and file systems, portable)
    - On Linux Kernel (2000s) two features are added (these features support Docker):
        - Namespaces: Isolate process.
        - Control Groups: Resource usage (CPU, Memory) isolation and limitation for each process. 
- Without Docker, each VM consumes 30% of resources (Memory, CPU)

![image](https://user-images.githubusercontent.com/10358317/113183089-ef51fa00-9253-11eb-9ade-771905ce8ebd.png) (Ref: Docker.com)

### Architecture  <a name="architecture"></a>

![image](https://user-images.githubusercontent.com/10358317/113183210-0db7f580-9254-11eb-9716-0de635f3cbdf.png) (Ref: docs.docker.com)


### Installation  <a name="installation"></a>

- Linux: Docker Engine
    - https://docs.docker.com/engine/install/ubuntu/
- Windows: Docker Desktop for Windows 
    - WSL: Windows Subsystem for Linux, 
    - WSL2: virtualization through a highly optimized subset of Hyper-V to run the kernel and distributions, better than WLS.
        - https://docs.docker.com/docker-for-windows/install/
- Mac-OS: Docker Desktop for Mac
    - https://docs.docker.com/docker-for-mac/install/

### Docker Engine (Deamon, REST API, CLI)  <a name="engine"></a>
- There are mainly 3 components in the Docker Engine:
    - Server is the docker daemon named docker daemon. Creates and manages docker images, containers, networks, etc.
    - Rest API instructs docker daemon what to do.
    - Command Line Interface (CLI) is the client used to enter docker commands.

![image](https://user-images.githubusercontent.com/10358317/113183406-45bf3880-9254-11eb-8d13-e68c83f3d349.png) (Ref: Docker.com)

### Docker Registry and Docker Hub  <a name="registry"></a>

- https://hub.docker.com/

![image](https://user-images.githubusercontent.com/10358317/113183434-4eb00a00-9254-11eb-9275-9b1ccf705d5b.png) 

[App: Running Docker Free Local Registry, Tagging Container, Pushing to Local Registry, Pulling From Local Registry and Deleting Images from Local Registry](https://github.com/omerbsezer/Fast-Docker/blob/main/DockerLocalRegistry.md)

### Docker Command Structure  <a name="command"></a>

- docker [ManagementCommand] [Command] 

```
docker container ls -a
docker image ls
docker volume ls
docker network ls
docker container rm -f [containerName or containerID]
```

![image](https://user-images.githubusercontent.com/10358317/113183615-81f29900-9254-11eb-8695-360680931866.png)

![image](https://user-images.githubusercontent.com/10358317/113183721-9fbffe00-9254-11eb-9841-79bf037db34a.png)

### Docker Container  <a name="container"></a>

![image](https://user-images.githubusercontent.com/10358317/113183556-730be680-9254-11eb-8bdd-84c5cf5b86c6.png) (Ref: docker-handbook-borosan)

- When we create the container from the image, in every container, there is an application that is set to run by default app. 
    - When this app runs, the container runs.
    - When this default app finishes/stops, the container stops. 
- There could be more than one app in docker image (such as: sh, ls, basic commands)
- When the Docker container is started, it is allowed that a single application is configured to run automatically.

```
docker container run --name mywebserver -d -p 80:80 -v test:/usr/share/nginx/html nginx
docker container ls -a
docker image pull alpine
docker image push alpine
docker image build -t hello . (run this command where “Dockerfile” is)
(PS: image file name MUST be “Dockerfile”, no extension)
docker save -o hello.tar test/hello
docker load -i <path to docker image tar file>
docker load -i .\hello.tar
```

Goto: [App: Creating First Docker Image and Container using Docker File](https://github.com/omerbsezer/Fast-Docker/blob/main/FirstImageFirstContainer.md)

#### Docker Container: Life Cycle

![image](https://user-images.githubusercontent.com/10358317/113186436-f67b0700-9257-11eb-9b2e-41ccf056e88b.png) (Ref: life-cycle-medium)

```
e.g. [imageName]=alpine, busybox, nginx, ubuntu, etc.
docker image pull [imageName]
docker container run [imageName]
docker container start [containerId or containerName]
docker container stop [containerId or containerName]
docker container pause [containerId or containerName]
docker container unpause [containerId or containerName]
```

#### Docker Container: Union File System  <a name="container-filesystem"></a>
- Images are read only (R/O).
- When containers are created, new read-write (R/W) thin layer is created.

![image](https://user-images.githubusercontent.com/10358317/113183883-d8f86e00-9254-11eb-994b-30c17fe9429b.png) (Ref: docs.docker.com)

### Docker Volumes: Why Volumes needed?
- Containers do not save the changes/logs when erased if there is not any binding to volume/mount. 
- For persistence, volumes/mounts MUST be used. 
- e.g. Creating a log file in the container. When the container is deleted, the log file also deleted with the container. So volumes/binding mounts MUST be used to provide persistence!

![image](https://user-images.githubusercontent.com/10358317/113184189-2d035280-9255-11eb-9409-578ad1f2bd4b.png) (Ref: udemy-course:adan-zye-docker)

### Docker Volumes/Bind Mounts  <a name="volume"></a>

- Volumes and binding mounts must be used for saving logs, output files, and input files.
- When volumes bind to the directory in the container, this directory and volume are synchronized.

```
docker volume create [volumeName]
docker volume create test
docker container run --name [containerName] -v [volumeName]:[pathInContainer] [imageName]
docker container run --name c1 -v test:/app alpine
```
Goto: [App: Binding Volume to the Different Containers](https://github.com/omerbsezer/Fast-Docker/blob/main/DockerVolume.md)

#### Bind Mount
```
docker container run --name [containerName] -v [pathInHost]:[pathInContainer] [imageName]
docker container run --name c1 -v C:\test:/app alpine
```
![image](https://user-images.githubusercontent.com/10358317/113184347-57eda680-9255-11eb-811c-9f55efd11deb.png) (Ref: Docker.com)

Goto: [App: Binding Mount to Container](https://github.com/omerbsezer/Fast-Docker/blob/main/DockerVolume.md#app_mount)
Goto: [App: Transferring Content between Host PC and Docker Container](https://github.com/omerbsezer/Fast-Docker/blob/main/DockerTransferringContent.md)

### Docker Network <a name="network"></a>
- Docker containers work like VMs.
- Every Docker container has network connections
- Docker Network Drivers:
    - None
    - Bridge
    - Host
    - Macvlan
    - Overlay
    
#### Docker Network: Bridge
- Default Network Driver: Bridge (--net bridge)

```
docker network create [networkName]
docker network create bridge1
docker container run --name [containerName] --net [networkName] [imageName] 
docker container run --name c1 --net bridge1 alpine sh
docker network inspect bridge1
docker container run --name c2 --net bridge1 alpine sh
docker network connect bridge1 c2
docker network inspect bridge1
docker network disconnect bridge1 c2
```

- Creating a new network using customized network parameters:

```
docker network create --driver=bridge --subnet=10.10.0.0/16 --ip-range=10.10.10.0/24 --gateway=10.10.10.10 newbridge
```

![image](https://user-images.githubusercontent.com/10358317/113184949-1b6e7a80-9256-11eb-9a0c-fe5c62404a06.png) (Ref: Docker.com)

#### Docker Network: Host
- Containers reach host network interfaces (--net host)

```
docker container run --name [containerName] --net [networkName] [imageName] 
docker container run --name c1 --net host alpine sh
```

![image](https://user-images.githubusercontent.com/10358317/113185061-43f67480-9256-11eb-9b94-83735ce980ce.png) (Ref: Docker.com)

#### Docker Network: MacVlan
- Each Container has its own MAC interface (--net macvlan)

![image](https://user-images.githubusercontent.com/10358317/113185105-52dd2700-9256-11eb-84f2-ef1880eb4f4c.png) (Ref: Docker.com)

#### Docker Network: Overlay 
- Containers that work on different PCs/hosts can work as the same network (--net overlay)

![image](https://user-images.githubusercontent.com/10358317/113185192-6e483200-9256-11eb-8cb4-d8aa170d1a1e.png) (Ref: Docker.com)

#### Port Mapping/Publish:
- Mapping Host PC's port to container port:
 
 ```
-p [hostPort]:[containerPort], --publish [hostPort]:[containerPort] e.g. -p 8080:80, -p 80:80
docker container run --name mywebserver -d -p 80:80 nginx
```

### Docker Log  <a name="log"></a>

- Docker Logs show /dev/stdout, /dev/stderror

```
docker logs --details [containerName]
```

![image](https://user-images.githubusercontent.com/10358317/113289697-d8151a00-92f0-11eb-86e6-6280c4bf2e77.png)


### Docker Stats/Memory-CPU Limitations  <a name="stats"></a>

![image](https://user-images.githubusercontent.com/10358317/113289735-e9f6bd00-92f0-11eb-940b-13113a5a5da2.png)

![image](https://user-images.githubusercontent.com/10358317/113289755-efec9e00-92f0-11eb-9f49-333a4608c523.png)

![image](https://user-images.githubusercontent.com/10358317/113289773-f5e27f00-92f0-11eb-8c4f-2db75f17baeb.png)


### Docker Environment Variables  <a name="variables"></a>

![image](https://user-images.githubusercontent.com/10358317/113289974-40fc9200-92f1-11eb-9f12-1125ec32eabf.png)


### Docker File  <a name="file"></a>

![image](https://user-images.githubusercontent.com/10358317/113185932-54f3b580-9257-11eb-9f50-0d18512a0c40.png)

Goto: [App: Creating Docker Container using Dockerfile to Build C++ on Ubuntu18.04](https://github.com/omerbsezer/Fast-Docker/blob/main/DockerfileForLinuxC%2B%2BBuild.md)

Goto: [App: Creating Docker Container using Dockerfile to Build C++ on Windows](https://github.com/omerbsezer/Fast-Docker/blob/main/DockerfileForWindowsC%2B%2BBuild.md)

#### Sample Docker Files

```
FROM python:alpine3.7
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
EXPOSE 5000
CMD python ./index.py
```

```
FROM ubuntu:18.04
RUN apt-get update -y
RUN apt-get install default-jre -y
WORKDIR /myapp
COPY /myapp .
CMD ["java","hello"]
```

- Multistage Docker File (Creating temporary container):
    - In the example, JDK (Java Development Kit) based temporary image (~440MB) container is created for compilation.
    - Compiled files are copied into JRE (Java Runtime Environment) based image (~145MB). Finally, we have only JRE based image.
```
COPY --from=<stage 1> stage1/src stage2/destination
```
- In the example below, 'compiler' is 'stage1'.
```
FROM mcr.microsoft.com/java/jdk:8-zulu-alpine AS compiler
COPY /myapp /usr/src/myapp
WORKDIR /usr/src/myapp
RUN javac hello.java

FROM mcr.microsoft.com/java/jre:8-zulu-alpine 
WORKDIR /myapp
COPY --from=compiler /usr/src/myapp .
CMD ["java", "hello"]
```

### Docker Image  <a name="image"></a>

- Create Image using Dockerfile
```
docker image build -t hello . (run this command where “Dockerfile” is)
(PS: image file name MUST be “Dockerfile”, no extension)
docker image pull [imageName]
docker image push [imageName]
docker image tag [imageOldName] [imageNewName]
(PS: If you want to push DockerHub, [imageNewName]=[username]/[imageName]:[version])
docker save -o hello.tar test/hello
docker load -i <path to docker image tar file>
docker load -i .\hello.tar
```

![image](https://user-images.githubusercontent.com/10358317/113186047-748ade00-9257-11eb-9c1c-1604d53523e8.png)

Goto: [App: Creating First Docker Image and Container using Docker File](https://github.com/omerbsezer/Fast-Docker/blob/main/FirstImageFirstContainer.md)

### Docker Compose v2 (Updated 2024) <a name="compose"></a>

> **IMPORTANT UPDATE 2024:** Docker Compose v1 (docker-compose) was deprecated in June 2023. Use Docker Compose v2 (docker compose with a space) instead!

- Define and run multi-container applications with Docker.
- Easy to create Docker components using one file: Docker Compose file
- It is a YAML file that defines components:
    - Services,
    - Volumes,
    - Networks,
    - Secrets

#### Key Changes in Compose v2:
- **Command syntax:** `docker compose` (with space) instead of `docker-compose` (with hyphen)
- **Container naming:** Uses hyphen (-) as separator instead of underscore (_)
- **Version field:** The `version:` field is now **obsolete** - start directly with `services:`
- **Better integration:** Integrated into Docker CLI platform
- **Improved performance:** Better build performance with BuildKit

#### Sample "docker-compose.yml" file (Modern 2024 Format):

```yaml
# Note: version field is obsolete in Compose v2
services:
  mydatabase:
    image: mysql:8.0
    restart: always
    volumes:
      - mydata:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    networks:
      - mynet
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      timeout: 20s
      retries: 10

  mywordpress:
    image: wordpress:latest
    depends_on:
      mydatabase:
        condition: service_healthy
    restart: always
    ports:
      - "80:80"
      - "443:443"
    environment:
      WORDPRESS_DB_HOST: mydatabase:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    networks:
      - mynet
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3

volumes:
  mydata: {}

networks:
  mynet:
    driver: bridge
```

#### Commands (Compose v2):

```bash
# Start services
docker compose up -d

# Stop services
docker compose down

# View logs
docker compose logs -f

# List running containers
docker compose ps

# Build or rebuild services
docker compose build

# Pull service images
docker compose pull
```

> **Migration Note:** For most projects, switching to Compose v2 requires no changes to your YAML file. Simply use `docker compose` instead of `docker-compose`.

Goto: [App: Docker-Compose File - Creating 2 Different Containers: WordPress Container depends on MySql Container](https://github.com/omerbsezer/Fast-Docker/blob/main/LAB03-DockerCompose.md)

### Docker BuildKit (NEW 2024) <a name="buildkit"></a>

> **BuildKit is enabled by default since Docker 23.0** - It's the modern build engine for Docker!

BuildKit is Docker's next-generation build system that provides improved performance, better caching, and new features for building container images.

#### Key Features:
- **Parallel build stages:** Build independent stages concurrently
- **Improved caching:** More efficient layer caching and cache reuse
- **Build secrets:** Securely pass secrets without including them in the image
- **SSH forwarding:** Access private repositories during build
- **Advanced Dockerfile syntax:** New instructions and capabilities

#### Enable BuildKit (if using older Docker version):

```bash
# Enable for single build
DOCKER_BUILDKIT=1 docker build .

# Enable permanently (Linux/Mac)
export DOCKER_BUILDKIT=1

# Enable permanently (Windows PowerShell)
$env:DOCKER_BUILDKIT=1
```

#### BuildKit Advanced Features:

```dockerfile
# syntax=docker/dockerfile:1.4

FROM ubuntu:22.04

# Using build secrets (won't be stored in image)
RUN --mount=type=secret,id=mysecret \
    cat /run/secrets/mysecret

# Using SSH forwarding for private repos
RUN --mount=type=ssh \
    git clone git@github.com:private/repo.git

# Cache mount for package managers
RUN --mount=type=cache,target=/var/cache/apt \
    apt-get update && apt-get install -y python3
```

#### Build with secrets:

```bash
docker build --secret id=mysecret,src=./secret.txt .
```

### Multi-Platform Builds (NEW 2024) <a name="multiplatform"></a>

Multi-platform builds allow you to create Docker images that work on different architectures (AMD64, ARM64, etc.) with a single build command.

#### Why Multi-Platform Builds?
- Support ARM-based systems (Apple Silicon M1/M2/M3, Raspberry Pi, AWS Graviton)
- Support AMD64 (Intel/AMD) systems
- Create truly portable container images
- One image works everywhere

#### Setup buildx (included in Docker Desktop):

```bash
# Create a new builder instance
docker buildx create --name mybuilder --use

# Bootstrap the builder
docker buildx inspect --bootstrap

# List available builders
docker buildx ls
```

#### Build for Multiple Platforms:

```bash
# Build for AMD64 and ARM64
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest .

# Build and push to registry
docker buildx build --platform linux/amd64,linux/arm64 \
  -t username/myapp:latest --push .

# Build for specific platform only
docker buildx build --platform linux/arm64 -t myapp:arm64 .
```

#### Multi-Platform Dockerfile Example:

```dockerfile
# syntax=docker/dockerfile:1.4
FROM --platform=$BUILDPLATFORM golang:1.21 AS builder

ARG TARGETPLATFORM
ARG BUILDPLATFORM
ARG TARGETOS
ARG TARGETARCH

WORKDIR /app
COPY . .

# Cross-compile for target platform
RUN CGO_ENABLED=0 GOOS=${TARGETOS} GOARCH=${TARGETARCH} \
    go build -o myapp .

FROM alpine:latest
COPY --from=builder /app/myapp /usr/local/bin/
CMD ["myapp"]
```

#### Available Platform Variables:
- `$BUILDPLATFORM` - Platform where build is running
- `$TARGETPLATFORM` - Platform for which you are building
- `$TARGETOS` - OS component of TARGETPLATFORM
- `$TARGETARCH` - Architecture component of TARGETPLATFORM

Goto: [LAB-10: Multi-Platform Builds with Docker BuildKit](https://github.com/omerbsezer/Fast-Docker/blob/main/LAB10-MultiPlatformBuilds.md)

### Docker Security Best Practices (NEW 2024) <a name="security"></a>

> **CRITICAL:** According to 2024 Docker Security Report, vulnerabilities in container images increased by 25% year-over-year. Security is not optional!

#### <a name="distroless"></a> 1. Use Distroless Images

Distroless images contain only your application and runtime dependencies - no shell, no package managers, no unnecessary tools.

**Benefits:**
- **Smaller attack surface:** 90%+ reduction in potential vulnerabilities
- **Smaller image size:** Often 10-50MB instead of 100-500MB
- **Better security:** No shell means attackers can't execute commands even if they breach the container

**Popular Distroless Base Images:**

```dockerfile
# Static binary (1.9 MB)
FROM gcr.io/distroless/static-debian12

# For apps needing glibc (29.7 MB)
FROM gcr.io/distroless/base-debian12

# For C/C++ apps (32.3 MB)
FROM gcr.io/distroless/cc-debian12

# For Python apps
FROM gcr.io/distroless/python3-debian12

# For Java apps
FROM gcr.io/distroless/java17-debian12

# For Node.js apps
FROM gcr.io/distroless/nodejs20-debian12
```

**Example with Multi-Stage Build:**

```dockerfile
# Build stage
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .

# Production stage - Distroless
FROM gcr.io/distroless/nodejs20-debian12
COPY --from=builder /app /app
WORKDIR /app
CMD ["index.js"]
```

#### <a name="nonroot"></a> 2. Run as Non-Root User

> **WARNING:** 58% of container images run as root (UID 0). This is a major security risk!

**Why it matters:**
- If attacker breaks out of container, they have root on host
- Principle of least privilege
- Industry best practice and compliance requirement

**Implementation:**

```dockerfile
FROM ubuntu:22.04

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

# Install dependencies as root
RUN apt-get update && apt-get install -y python3

# Change ownership
COPY --chown=appuser:appuser . /app

# Switch to non-root user
USER appuser

WORKDIR /app
CMD ["python3", "app.py"]
```

**For Distroless (use numeric UID):**

```dockerfile
FROM gcr.io/distroless/python3-debian12
COPY --chown=65532:65532 /app /app
USER 65532
CMD ["app.py"]
```

#### <a name="scanning"></a> 3. Vulnerability Scanning

Scan your images regularly for known vulnerabilities (CVEs).

**Tools:**

```bash
# Docker Scout (built into Docker Desktop)
docker scout quickview myapp:latest
docker scout cves myapp:latest
docker scout recommendations myapp:latest

# Trivy (open source, very popular)
trivy image myapp:latest

# Grype (from Anchore)
grype myapp:latest

# Snyk
snyk container test myapp:latest
```

**Enable Docker Scout in CI/CD:**

```yaml
# GitHub Actions example
- name: Docker Scout Scan
  uses: docker/scout-action@v1
  with:
    command: cves
    image: myapp:latest
```

#### 4. Additional Security Best Practices:

```dockerfile
# syntax=docker/dockerfile:1.4
FROM ubuntu:22.04

# Update packages to latest security patches
RUN apt-get update && apt-get upgrade -y

# Don't include secrets in image
# BAD:  COPY .env /app/
# GOOD: Use docker build --secret or environment variables at runtime

# Use specific versions, not 'latest'
FROM node:20.11.0-alpine3.19  # Good
# FROM node:latest             # Bad

# Minimize installed packages
RUN apt-get install -y --no-install-recommends \
    python3 \
    && rm -rf /var/lib/apt/lists/*

# Set read-only root filesystem
# docker run --read-only myapp

# Drop capabilities
# docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp
```

Goto: [LAB-11: Docker Security Best Practices](https://github.com/omerbsezer/Fast-Docker/blob/main/LAB11-SecurityBestPractices.md)

### Healthchecks (NEW 2024) <a name="healthchecks"></a>

> **Important:** Without healthchecks, Docker cannot detect if your containerized service is unhealthy, preventing automatic restarts or recovery.

Healthchecks tell Docker how to test if your container is working correctly. A container might be "running" but the application inside might be unresponsive.

#### Dockerfile Healthcheck:

```dockerfile
FROM nginx:alpine

# HTTP healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost/ || exit 1

# Alternative with curl
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/ || exit 1

# For database
HEALTHCHECK --interval=10s --timeout=3s \
  CMD mysqladmin ping -h localhost || exit 1

# For custom app
HEALTHCHECK CMD /app/healthcheck.sh || exit 1
```

#### Parameters:
- `--interval=30s` - Run check every 30 seconds
- `--timeout=3s` - Command must complete within 3 seconds
- `--start-period=5s` - Give container 5 seconds to start before checking
- `--retries=3` - Need 3 consecutive failures to mark unhealthy

#### Docker Compose Healthcheck:

```yaml
services:
  web:
    image: myapp
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  db:
    image: postgres:16
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

#### Check Container Health:

```bash
# View health status
docker ps

# Inspect health details
docker inspect --format='{{json .State.Health}}' container_name | jq

# Depends on healthy service (Compose v2)
services:
  web:
    depends_on:
      db:
        condition: service_healthy
```

### .dockerignore Best Practices (NEW 2024) <a name="dockerignore"></a>

The `.dockerignore` file tells Docker which files to exclude from the build context, improving build speed and security.

#### Why .dockerignore is Critical:
- **Faster builds:** Don't copy unnecessary files
- **Smaller context:** Less data sent to Docker daemon
- **Security:** Don't accidentally include secrets
- **Efficiency:** Better layer caching

#### Comprehensive .dockerignore Example:

```
# .dockerignore

# Git
.git
.gitignore
.gitattributes

# CI/CD
.github
.gitlab-ci.yml
.travis.yml
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

# Dependencies (will be installed in container)
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
*.spec.ts

# Secrets (CRITICAL!)
.env
.env.*
*.pem
*.key
*.crt
secrets/
credentials.json

# Logs
*.log
logs/

# OS
Thumbs.db
.DS_Store

# Large files
*.tar
*.zip
*.gz
*.mp4
*.avi

# Keep specific files (! prefix)
!.env.example
```

#### Testing .dockerignore:

```bash
# See what will be included in build context
docker build --no-cache --progress=plain -t test . 2>&1 | grep "Transferring context"

# Or use docker buildx
docker buildx build --progress=plain .
```

Goto: [LAB-12: Healthchecks and .dockerignore Best Practices](https://github.com/omerbsezer/Fast-Docker/blob/main/LAB12-HealthchecksDockerignore.md)

### Docker Swarm (Legacy) <a name="swarm"></a>

> **NOTE:** Docker Swarm is now considered legacy. For production orchestration, consider **Kubernetes** instead. See [Container Orchestration](#orchestration) section.

One of the Container Orchestration tool: 
- Automating and scheduling the 
    - deployment, 
    - management, 
    - scaling, and
    - networking of containers
- Container Orchestration tools:
    - Docker Swarm,
    - Kubernetes,
    - Mesos

![image](https://user-images.githubusercontent.com/10358317/113186661-3b06a280-9258-11eb-9bb8-3ad38d3c55fb.png) (Ref: udemy-course:adan-zye-docker)

### Docker Stack / Docker Service  <a name="stack"></a>
- With Docker Stack, multiple services can be created with one file.
- It is like a Docker-Compose file but it has more features than a Docker-compose file: update_config, replicas.
- But it is running on when Docker Swarm mode is activated.
- Network must be 'overlay'.

#### Creating, Listing, Inspecting
```
docker service create --name testservice --replicas=5 -p 8080:80 nginx
docker service ps testservice (listing running containers on which nodes)
docker service inspect testservice
```
#### Scaling
```
docker service scale testservice=10 (scaling up the containers to 10 replicas)
```
#### Updating
```
docker service update --detach --update-delay 5s --update-parallelism 2 --image nginx:v2 testservice (previous state: testservice created, now updating)
docker service update --help (to see the parameters of update)
```
#### Rollbacking
```
docker service rollback --detach testservice (rollbacking to previous state)
```

![image](https://user-images.githubusercontent.com/10358317/113303356-4a8df600-9301-11eb-9114-38872ca01f29.png)

Goto: [App: Creating Docker Swarm Cluster With 5 PCs using PlayWithDocker : 3 x WordPress Containers and 1 x MySql Container using Docker-Compose File](https://github.com/omerbsezer/Fast-Docker/blob/main/DockerStackService.md)

## Play With Docker  <a name="playwithdocker"></a>

- https://labs.play-with-docker.com/

![image](https://user-images.githubusercontent.com/10358317/113187037-ae101900-9258-11eb-9789-781ca2f6a464.png)

## Docker Commands Cheatsheet <a name="cheatsheet"></a>

Goto: [Docker Commands Cheatsheet](https://github.com/omerbsezer/Fast-Docker/blob/main/DockerCommandCheatSheet.md)

## Container Orchestration: Kubernetes vs Swarm <a name="orchestration"></a>

> **Industry Standard 2024:** Kubernetes is the de facto standard for container orchestration in production environments.

### Why Kubernetes Over Swarm?

**Docker Swarm Status:**
- Docker Swarm is now considered **legacy technology**
- Limited development and new features since 2020
- Smaller community and ecosystem
- Basic orchestration capabilities
- Good for: Simple deployments, learning, small projects

**Kubernetes Advantages:**
- **Industry standard:** Used by 88% of organizations (CNCF Survey 2024)
- **Massive ecosystem:** Helm charts, operators, monitoring tools, service meshes
- **Cloud native:** Native support in AWS (EKS), Azure (AKS), Google Cloud (GKE)
- **Advanced features:** Auto-scaling, rolling updates, self-healing, advanced networking
- **Enterprise support:** RedHat OpenShift, Rancher, VMware Tanzu
- **Better for:** Production workloads, microservices, multi-cloud, complex applications

### When to Use What?

**Use Docker Compose (Single Host):**
- Development environments
- Simple applications
- Testing
- Single server deployments

**Use Kubernetes (Multi-Host Production):**
- Production microservices
- Applications requiring high availability
- Multi-datacenter deployments
- Complex networking requirements
- Auto-scaling needs

**Use Docker Swarm (Legacy/Learning):**
- Learning container orchestration basics
- Legacy systems already using Swarm
- Very simple multi-host deployments
- Quick prototypes

### Kubernetes Quick Example:

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:latest
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

### Learning Path Recommendation:

1. **Start:** Docker basics (images, containers, volumes, networks)
2. **Next:** Docker Compose for multi-container apps
3. **Then:** Kubernetes fundamentals (if targeting production)
4. **Advanced:** Service meshes (Istio, Linkerd), GitOps (ArgoCD, Flux)

### Kubernetes Resources:
- Official Documentation: https://kubernetes.io/docs/
- Interactive Tutorial: https://kubernetes.io/docs/tutorials/kubernetes-basics/
- Minikube (Local Kubernetes): https://minikube.sigs.k8s.io/
- K3s (Lightweight Kubernetes): https://k3s.io/
- CNCF Landscape: https://landscape.cncf.io/

## Other Useful Resources Related Docker  <a name="resource"></a>

### Official Documentation
- Official Docker Documentation: https://docs.docker.com/
- Docker BuildKit Documentation: https://docs.docker.com/build/buildkit/
- Docker Compose v2 Documentation: https://docs.docker.com/compose/
- Multi-Platform Builds: https://docs.docker.com/build/building/multi-platform/
- Docker Security: https://docs.docker.com/engine/security/

### Security Resources (NEW 2024)
- Docker Scout: https://docs.docker.com/scout/
- Trivy Security Scanner: https://github.com/aquasecurity/trivy
- Distroless Container Images: https://github.com/GoogleContainerTools/distroless
- Docker Security Best Practices: https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html
- Snyk Container Security: https://snyk.io/product/container-vulnerability-management/

### Learning Resources
- Docker Tutorial for Beginners [FULL COURSE in 3 Hours]: https://www.youtube.com/watch?v=3c-iBn73dDE&t=6831s
- Docker and Kubernetes Tutorial | Full Course: https://www.youtube.com/watch?v=bhBSlnQcq2k&t=3088s
- Play with Docker (Interactive Labs): https://labs.play-with-docker.com/
- Docker Workshop: https://dockerlabs.collabnix.com/workshop/docker/

### Tools & Utilities
- Docker Cheatsheet: https://github.com/wsargent/docker-cheat-sheet
- Various Dockerfiles for Different Purposes: https://github.com/jessfraz/dockerfiles
- Dockerfile Best Practices: https://github.com/dnaprawa/dockerfile-best-practices
- All-in-one Docker Image for Deep Learning: https://github.com/floydhub/dl-docker
- Dive (Explore Docker Image Layers): https://github.com/wagoodman/dive
- Hadolint (Dockerfile Linter): https://github.com/hadolint/hadolint

### Blogs & Articles (2024)
- Docker Blog: https://www.docker.com/blog/
- Docker 2024 Highlights: https://www.docker.com/blog/docker-2024-highlights/
- 8 Top Docker Tips & Tricks for 2024: https://www.docker.com/blog/8-top-docker-tips-tricks-for-2024/
- Docker Security in 2025: https://cloudnativenow.com/topics/cloudnativedevelopment/docker/

## References  <a name="references"></a>
- [Docker.com](https://www.docker.com/)
- [docs.docker.com](https://docs.docker.com/get-started/overview/)
- [docker-handbook-borosan](https://borosan.gitbook.io/docker-handbook/docker-images)
- [life-cycle-medium](https://medium.com/future-vision/docker-lifecycle-tutorial-and-quickstart-guide-c5fd5b987e0d)
- [Infoworld](https://www.infoworld.com/article/3310941/why-you-should-use-docker-and-containers.html)
- [ItNext](https://itnext.io/getting-started-with-docker-facts-you-should-know-d000e5815598)
- [udemy-course:adan-zye-docker](https://www.udemy.com/course/adan-zye-docker/)
