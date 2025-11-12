## LAB-03: Docker Compose v2 - Creating WordPress Container that Depends on MySQL Container

> **UPDATED 2024:** This LAB has been updated to use Docker Compose v2 with modern best practices including healthchecks and proper dependency management.

This scenario shows how to run multiple Docker containers using the docker-compose.yml file with Docker Compose v2. 
After running containers, it is shown how: 
- to step (enter) into the container, 
- to install binary packages into the container, 
- to see ethernet interfaces
- to send ping packets to see the connection of the containers.

### Steps
- Create an "example" directory on your Desktop.
- Create "docker-compose.yml" in the "example" directory.
- Copy below and paste into "docker-compose.yml" (2 containers are created namely: 'mydatabase', 'mywordpress'. 'mydata' is volume object that binds to the container, environment variables are basically defined, new bridge network is created) 
(Normally, credentials are not defined this way, just example. Docker secret is used to define credentials, safety way).

```yaml
# Note: version field is obsolete in Compose v2
# Modern Docker Compose v2 format with healthchecks

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
    # Healthcheck to ensure database is ready
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-psomewordpress"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  mywordpress:
    image: wordpress:latest
    # Wait for database to be healthy before starting
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
    # Healthcheck for WordPress
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s

volumes:
  mydata: {}

networks:
  mynet:
    driver: bridge
```

- Open a terminal where "docker-compose.yml" is. Run the following command:

> **IMPORTANT:** Use `docker compose` (with space) not `docker-compose` (with hyphen). Docker Compose v1 was deprecated in June 2023.

```bash
docker compose up -d
```
![image](https://user-images.githubusercontent.com/10358317/113313590-ae1d2100-930b-11eb-965b-f8638cef16b6.png)

- Run on the terminal to see a list of containers: 
```
docker container ls -a
```
![image](https://user-images.githubusercontent.com/10358317/113410555-00198180-93b4-11eb-8784-8b561b5afb21.png)


- Open the browser (127.0.0.1) to see the result:

![image](https://user-images.githubusercontent.com/10358317/113315210-58e20f00-930d-11eb-972e-67c6885404bb.png)

- If you want, you can enter one of the containers and ping to another container to see the connection between both containers. They are connected to each other via the bridge "mynet".
- Run on the terminal to see a list of containers: 
```
docker container ls -a
```
- You can see the names of the containers. You should create "docker-compose.yml" under the "example" directory. Docker Compose creates containers and adds names according to the path.

> **NOTE:** In Compose v2, container names use hyphen (-) as separator instead of underscore (_). For example: `example-mywordpress-1`, `example-mydatabase-1`.

- Run on the terminal to enter into the container:
```bash
docker exec -it example-mywordpress-1 sh
# Or use compose exec:
docker compose exec mywordpress sh
```
- Now you are in the container if you see "#". When you run "ifconfig" and "ping", these binaries are not found in the container. But we can update and install these binaries into this container.
- Run on the terminal to update OS in the container:
```
apt-get update
```
- Run on the terminal to install net-tools ('ifconfig') in the container:
```
apt-get install net-tools
```
- Run on the terminal to install iputils-ping ('ping') in the container:
```
apt-get install iputils-ping
```
![image](https://user-images.githubusercontent.com/10358317/113315939-1cfb7980-930e-11eb-8996-f7f23ae87781.png)

- Run on the terminal to see the IP of the current container. It could be "172.19.0.3":
```
ifconfig
```
- Run on the terminal to send ping packets to another container (“example_mydatabase_1”):
```
ping 172.19.0.2
```
![image](https://user-images.githubusercontent.com/10358317/113315708-dad23800-930d-11eb-97a3-4bee7ab1fed1.png)

- Press CTRL+C to stop ping.
- Press CTRL+P+Q to go out of the container to the host terminal.
- Run on the terminal to stop and remove containers:
```bash
docker compose down
```
![image](https://user-images.githubusercontent.com/10358317/113316439-9f843900-930e-11eb-8e34-4fce7460eaae.png)

---

## Key Changes in Docker Compose v2 (2024 Update)

### 1. Command Syntax
- **Old (v1):** `docker-compose up -d`
- **New (v2):** `docker compose up -d` (with space, not hyphen)

### 2. Version Field Obsolete
- The `version:` field at the top of docker-compose.yml is no longer needed
- Modern files start directly with `services:`

### 3. Container Naming
- **Old (v1):** Used underscore: `example_mywordpress_1`
- **New (v2):** Uses hyphen: `example-mywordpress-1`

### 4. Healthchecks and Dependencies
- Added healthchecks to ensure services are actually ready
- Use `depends_on` with `condition: service_healthy` for proper startup order
- WordPress now waits for MySQL to be healthy before starting

### 5. Useful Compose v2 Commands
```bash
# Start services
docker compose up -d

# Stop services
docker compose down

# View logs
docker compose logs -f

# View service status
docker compose ps

# Execute command in service
docker compose exec mywordpress sh

# Restart services
docker compose restart

# Pull latest images
docker compose pull
```

### Benefits of This Setup
- ✅ WordPress waits for database to be ready
- ✅ Healthchecks ensure services are working
- ✅ Modern Compose v2 syntax
- ✅ Proper dependency management
- ✅ MySQL 8.0 with better performance and security

