## Docker Commands Cheatsheet (Updated 2024)

> **IMPORTANT:** This cheatsheet has been updated for Docker 2024/2025 with Docker Compose v2, BuildKit, Multi-Platform Builds, and Security features.

#### First Try
```bash
docker run hello-world
docker info
docker version
docker --help
```
#### listing (ls=listing)
```
docker images ls
docker container ls -a
docker volume ls
docker network ls
docker container ps -a
docker ps
```
#### image pull from, push to registry
```
docker image pull alpine
docker image push alpine
```
#### creating and entering container (--name=name of new container, -it=interactive mode, sh=shell)
```
docker container run -it --name c1 omerbsezer/sample-java-app sh
```
#### start, stop container (30e=containerID)
```
docker container start 30e    
docker container stop 30e
```
#### entering container (8aa=containerID, webserv=containerName, sh=shell)
```
docker container exec -it webserv sh
docker container run --name c1 -it busybox sh  #create and enter in 
docker exec -it 8aa sh
```
#### local registry running (d=detach mode, p:portBinding)
```
docker container run -d -p 5000:5000 --restart always --name localregistry registry
```
#### port binding, background running ## (d=detach mode, p:portBinding)
```
docker container run --name nginxcontainer -d -p 8080:80 nginx
```
#### volume (v:volume host:container)
```
docker container run --name ng2 -d -p 8080:80 -v test1:/usr/share/nginx/html nginx
```
#### copy container to host (cp:copy)
```
docker cp ng2:/test12 C:\Users\oesezer\Desktop\sampleBindMount
```
#### copying from volume, creating new temp. container
```
docker run -d -v hello:/hello --name c1 busybox
docker cp c1:/hello ./
```
#### container erasing (rm:remove, f:force, prune: delete all)
```
docker container prune
docker container rm -f 123
```
#### creating new bridge (network) (net=network)
```
docker network create bridge1
docker network inspect bridge1
docker container run -it -d --name webserver --net bridge1 omerbsezer/sample-web-php sh
docker container run -it -d --name webdb --net bridge1 omerbsezer/sample-web-mysql sh
docker network inspect bridge1
```
#### creating new bridge (custom subnet, mask and ip range) and connecting
```
docker network create --driver=bridge --subnet=10.10.0.0/16 --ip-range=10.10.10.0/24 --gateway=10.10.10.10 bridge2
docker network inspect bridge2
docker container run -dit --name webserver2 omerbsezer/sample-web-php
docker container run -dit --name webdb2 omerbsezer/sample-web-mysql
docker network connect bridge2 webserver2
docker network connect bridge2 webdb2
docker network inspect bridge2
```
#### image tagging and pushing
```
docker image tag mysql:5 test:latest
docker image push test:latest
```
#### image build (-t: tagging image name)
```
docker image build -t hello . (run this command where “Dockerfile” is)
(PS: image file name MUST be “Dockerfile”, no extension)
```
#### image save and load (t=tag, i=input, o=output)
```
docker save -o <path for created tar file> <image name>
docker save -o [imageName].tar [imageName]
docker load -i <path to docker image tar file>
docker load -i .\[imageName].tar
```
#### env, copy from container to host 
```
docker container run --net host [imageName]
ENV USER="space"
docker container run -d -d p 80:80 --name hd2 -e USER="UserName" [imageName]
docker cp host_path:/usr/src/app .
```
#### multistage image File ##
```
FROM mcr.microsoft.com/java/jdk:8-zulu-alpine AS compiler
COPY --from=compiler /usr/src/app . 
COPY --from=nginx:latest /usr/src/app .
```
#### using ARG (ARG=argument is only used while creating image, difference from ENV: env is reachable from container)
```
ARG VERSION
ADD https://www.python.org/ftp/python/${VERSION}/Python-${VERSION}.tgz .
docker image build -t x1 --build-arg VERSION=3.7.1 .
docker image build -t x1 --build-arg VERSION=3.8.1 .
```
#### creating Image from Container (-c: CMD)
```
docker commit con1 dockerUserName/con1:latest
docker commit -c 'CMD ["java","app"]' con1 dockerUserName/con1:second
docker image inspect dockerUserName/con1:secon
```
#### Docker Compose v2 Commands (UPDATED 2024 - use "docker compose" not "docker-compose")
```bash
# NOTE: Use "docker compose" (with space) instead of "docker-compose" (with hyphen)
# docker-compose v1 was deprecated in June 2023

# Start services
docker compose up -d

# Stop and remove containers
docker compose down

# List containers
docker compose ps

# View logs
docker compose logs
docker compose logs -f                    # Follow logs
docker compose logs service_name          # Logs for specific service

# Execute command in service
docker compose exec websrv ls -al
docker compose exec -it websrv sh         # Interactive shell

# Build or rebuild services
docker compose build
docker compose build --no-cache           # Build without cache

# Validate and view compose file
docker compose config

# View images
docker compose images

# Pull service images
docker compose pull

# Scale services
docker compose up -d --scale web=3        # Scale web service to 3 replicas

# Restart services
docker compose restart

# Pause/Unpause services
docker compose pause
docker compose unpause

# Remove stopped containers
docker compose rm

# View service status
docker compose top
```
#### docker swarm commmunication
```
In server, these ports should be opened:
- TCP port 2377 (Cluster management)
- TCP - UDP port 7946 (Communication between nodes)
- UDP port 4789 (Overlay network) 
```
#### creating docker swarm: binding manager and worker nodes (test it with docker play with creating instances on docker play environment)
```
(PS1: To login PlayWithDocker, you must sign up Docker Hub to use PlayWithDocker)
(PS2: On PlayWithDocker Paste: Shift+Insert, Copy: CTRL+Insert)
docker swarm init --advertise-addr 192.168.0.13 (after choosing manager node, run this command to initialize swarm, IP is the master node’s IP)
	
(after running command above, following writings appear on master PC to add worker PCs)
	Swarm initialized: current node (u4tqju429dcmggxmw29ll8nls) is now a manager.
	To add a worker to this swarm, run the following command:
docker swarm join --token SWMTKN-1 5qbi2uydkawpcuo9rcp4quytr37z1 pzob5ss66o821br09h7x9-3jrqc08scyetlxg9iyhck25u1 192.168.0.13:2377
To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

(run following command on worker nodes)
docker swarm join --token SWMTKN-1 5qbi2uydkawpcuo9rcp4quytr37z1 pzob5ss66o821br09h7x9-3jrqc08scyetlxg9iyhck25u1 192.168.0.13:2377

(if you want to add more manager nodes to swarm cluster, run following command on manager PC to take manager token from master PC)
docker swarm join-token manager

(after running command above, following writings appear on master PC to add manager PCs)
	To add a manager to this swarm, run the following command:
docker swarm join --token SWMTKN-1 5qbi2uydkawpcuo9rcp4quytr37z 1pzob5ss66o821br09h7x9-3bv78968l9qdfza68979np715 192.168.0.13:2377
	This node joined a swarm as a manager

(run following command on manager nodes)
docker swarm join --token SWMTKN-1 5qbi2uydkawpcuo9rcp4quytr37z 1pzob5ss66o821br09h7x9-3bv78968l9qdfza68979np715 192.168.0.13:2377
```
#### list managers and workers on swarm
```
docker node ls 
```
#### docker service, ls
```
(run these commands on manager nodes)
docker service (to see docker service help)
docker service create --name test --replicas=5 -p 8080:80 nginx 
docker service ps test (shows containers, which containers run on which PC)
docker service ls   (list service, shows service detail)
docker service logs test
docker service scale test=3   (desired state 3 replica)
docker service rm test
docker service create --name glb --mode=global nginx
```
#### create overlay network
```
docker network create -d overlay over-net   (creating overlay network)
docker service create --name webserv --network over-net -p 8080:80 --replicas=3 omerbsezer/sample-web-php
docker service create --name db --network over-net omerbsezer/sample-web-mysql
```
#### update service, rollback (update containers)
```
docker service update --help
docker service update -d --update-delay 5s --update-parallelism 2 --image omerbsezer/sample-web-php:v2 websrv (when updating images with version2, updating from 2 parallel branches)
docker service ps websrv
docker service rollback -d websrv (rollback to last status of websrv)
```
#### secret (to work with secrets, swarm mode must be active)
```
notepad or nano username.txt (create username.txt that include username)
notepad or nano password.txt (create password.txt that include password) 
docker secret create username .\username.txt (create username secret object)
docker secret create password .\password.txt
docker secret inspect password
docker secret ls
docker service create -d --name secrettest --secret username --secret password omerbsezer/sample-web-php
docker service ps secrettest
docker ps
docker exec -it c4c sh
(when we are in the container, we can reach and see secrets at “/run/secrets”)
(when new password is needed, new password file must be created again)
echo "password222" | docker secret create password2
docker service update --secret-rm password --secret-add password2 secrettest (password is removed, new password2 object is added)
```
#### Docker Stack (Legacy - Stack is like compose, but runs on swarm)
```bash
# NOTE: Docker Swarm is legacy. Consider Kubernetes for production.
docker stack                                    # See stack help
docker stack deploy -c docker-compose.yml firststack
docker stack ls
docker stack services firststack
docker stack ps firststack
docker stack rm firststack                      # Remove stack
```

---

## NEW 2024 Features

#### Docker BuildKit (Enabled by Default since Docker 23.0)
```bash
# BuildKit is enabled by default. For older versions:
export DOCKER_BUILDKIT=1                        # Linux/Mac
$env:DOCKER_BUILDKIT=1                          # Windows PowerShell

# Build with BuildKit features
docker build --secret id=mysecret,src=./secret.txt .
docker build --ssh default .

# View build progress
docker build --progress=plain .

# Build with cache mount
docker build --build-arg BUILDKIT_INLINE_CACHE=1 .
```

#### Multi-Platform Builds (NEW 2024)
```bash
# Create and use buildx builder
docker buildx create --name mybuilder --use
docker buildx inspect --bootstrap
docker buildx ls

# Build for multiple platforms
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest .

# Build and push to registry
docker buildx build --platform linux/amd64,linux/arm64 \
  -t username/myapp:latest --push .

# Build for specific platform
docker buildx build --platform linux/arm64 -t myapp:arm64 .

# Build for ARM (Apple Silicon, Raspberry Pi, AWS Graviton)
docker buildx build --platform linux/arm64 -t myapp:arm .

# List available platforms
docker buildx inspect --bootstrap

# Remove builder
docker buildx rm mybuilder
```

#### Docker Security - Vulnerability Scanning (NEW 2024)
```bash
# Docker Scout (built into Docker Desktop)
docker scout quickview myapp:latest
docker scout cves myapp:latest
docker scout recommendations myapp:latest
docker scout compare myapp:latest --to myapp:previous

# Enable Docker Scout
docker scout enroll

# Trivy - Open Source Security Scanner
trivy image myapp:latest
trivy image --severity HIGH,CRITICAL myapp:latest
trivy image --format json myapp:latest

# Scan Dockerfile
trivy config Dockerfile

# Grype - Vulnerability Scanner
grype myapp:latest

# Snyk Container Security
snyk container test myapp:latest
snyk container monitor myapp:latest
```

#### Healthchecks (NEW 2024)
```bash
# View container health status
docker ps
docker inspect --format='{{json .State.Health}}' container_name

# Test healthcheck manually
docker exec container_name curl -f http://localhost/health

# View healthcheck logs
docker inspect --format='{{range .State.Health.Log}}{{.Output}}{{end}}' container_name
```

#### Docker System Management (NEW 2024)
```bash
# System information
docker system df                                # Show docker disk usage
docker system df -v                             # Verbose output

# Clean up
docker system prune                             # Remove unused data
docker system prune -a                          # Remove all unused images
docker system prune --volumes                   # Include volumes
docker system prune -a --volumes                # Remove everything unused

# Events monitoring
docker system events
docker system events --filter 'type=container'

# Real-time resource usage
docker stats
docker stats --no-stream                        # Single snapshot
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

#### Container Resource Limits (Best Practice 2024)
```bash
# CPU limits
docker run --cpus=2 myapp                       # Limit to 2 CPUs
docker run --cpu-shares=512 myapp               # CPU shares (relative weight)

# Memory limits
docker run -m 512m myapp                        # Limit memory to 512MB
docker run -m 512m --memory-swap 1g myapp       # Memory + swap limit

# Combined resource limits
docker run -m 512m --cpus=1 --name myapp nginx
```

#### Security Best Practices (NEW 2024)
```bash
# Run as non-root user
docker run --user 1000:1000 myapp

# Read-only root filesystem
docker run --read-only myapp

# Drop all capabilities and add only needed ones
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp

# Use security options
docker run --security-opt=no-new-privileges myapp

# Scan image before running
docker scout cves myapp:latest && docker run myapp:latest
```

#### Image Analysis Tools (NEW 2024)
```bash
# Dive - Explore image layers
dive myapp:latest

# Hadolint - Dockerfile linter
hadolint Dockerfile

# View image history with sizes
docker history myapp:latest
docker history --no-trunc myapp:latest

# Inspect image
docker image inspect myapp:latest
docker image inspect --format='{{.Size}}' myapp:latest
```

#### Docker Init (NEW 2024) - Quick Project Setup
```bash
# Initialize Docker assets for a project
docker init

# This creates:
# - Dockerfile
# - .dockerignore
# - docker-compose.yml
# Supports: Node.js, Python, Go, Rust, ASP.NET
```

#### Docker Context (Multiple Docker Hosts)
```bash
# List contexts
docker context ls

# Create new context
docker context create mycontext --docker "host=ssh://user@remote-host"

# Use context
docker context use mycontext

# Return to default
docker context use default
```

---

## Quick Reference

### Most Used Commands (2024)
```bash
# Images
docker build -t myapp .
docker pull nginx:latest
docker push username/myapp:latest
docker images
docker rmi myapp:latest

# Containers
docker run -d -p 8080:80 --name web nginx
docker ps
docker ps -a
docker stop web
docker rm web
docker logs -f web
docker exec -it web sh

# Compose v2 (NEW)
docker compose up -d
docker compose down
docker compose logs -f

# Multi-platform (NEW)
docker buildx build --platform linux/amd64,linux/arm64 -t myapp .

# Security (NEW)
docker scout cves myapp:latest

# Cleanup
docker system prune -a --volumes
```

### Image Size Optimization (Best Practice 2024)
```bash
# Use specific tags, not latest
FROM node:20.11.0-alpine3.19

# Multi-stage builds
FROM node:20 AS builder
# ... build steps
FROM gcr.io/distroless/nodejs20-debian12
COPY --from=builder /app /app

# Use .dockerignore file (create it!)
# Minimize layers (combine RUN commands)
```
