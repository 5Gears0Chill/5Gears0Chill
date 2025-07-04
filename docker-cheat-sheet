# Docker CLI Cheat Sheet for Dummies

## Basic Container Commands

### Running Containers
```bash
# Run a container from an image
docker run [image_name]

# Run container in background (detached mode)
docker run -d [image_name]

# Run container with a custom name
docker run --name my-container [image_name]

# Run container and map ports (host:container)
docker run -p 8080:80 [image_name]

# Run container with environment variables
docker run -e VAR_NAME=value [image_name]

# Run container and remove it when it stops
docker run --rm [image_name]

# Run container interactively with terminal
docker run -it [image_name] /bin/bash
```

### Managing Running Containers
```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Stop a running container
docker stop [container_id_or_name]

# Start a stopped container
docker start [container_id_or_name]

# Restart a container
docker restart [container_id_or_name]

# Remove a container
docker rm [container_id_or_name]

# Remove all stopped containers
docker container prune
```

### Accessing Containers
```bash
# Execute command in running container
docker exec [container_id] [command]

# Open interactive shell in running container
docker exec -it [container_id] /bin/bash

# View container logs
docker logs [container_id]

# Follow container logs (like tail -f)
docker logs -f [container_id]

# Copy files from container to host
docker cp [container_id]:/path/to/file /host/path

# Copy files from host to container
docker cp /host/path [container_id]:/path/to/file
```

## Image Commands

### Managing Images
```bash
# List all images
docker images

# Pull an image from registry
docker pull [image_name]

# Remove an image
docker rmi [image_name]

# Remove unused images
docker image prune

# Build image from Dockerfile
docker build -t [image_name] .

# Build image with custom Dockerfile
docker build -f [dockerfile_path] -t [image_name] .
```

### Image Information
```bash
# Show image details
docker inspect [image_name]

# Show image history/layers
docker history [image_name]

# Search for images on Docker Hub
docker search [search_term]
```

## Volume Commands

### Managing Volumes
```bash
# List volumes
docker volume ls

# Create a volume
docker volume create [volume_name]

# Remove a volume
docker volume rm [volume_name]

# Remove unused volumes
docker volume prune

# Mount volume to container
docker run -v [volume_name]:/path/in/container [image_name]

# Mount host directory to container
docker run -v /host/path:/container/path [image_name]
```

## Network Commands

### Managing Networks
```bash
# List networks
docker network ls

# Create a network
docker network create [network_name]

# Remove a network
docker network rm [network_name]

# Connect container to network
docker network connect [network_name] [container_name]

# Disconnect container from network
docker network disconnect [network_name] [container_name]
```

## System Commands

### System Information
```bash
# Show Docker system information
docker info

# Show Docker version
docker version

# Show disk usage
docker system df

# Remove unused data (containers, networks, images)
docker system prune

# Remove everything (including volumes)
docker system prune -a --volumes
```

### Monitoring
```bash
# Show running processes in container
docker top [container_id]

# Show resource usage statistics
docker stats

# Show resource usage for specific container
docker stats [container_id]
```

## Docker Compose (Bonus)

### Basic Compose Commands
```bash
# Start services defined in docker-compose.yml
docker-compose up

# Start services in background
docker-compose up -d

# Stop services
docker-compose down

# View logs for all services
docker-compose logs

# Build services
docker-compose build

# Pull latest images
docker-compose pull
```

## Common Examples

### Example 1: Run a Web Server
```bash
# Run nginx web server on port 8080
docker run -d -p 8080:80 --name my-nginx nginx

# Check if it's running
docker ps

# View logs
docker logs my-nginx

# Stop and remove
docker stop my-nginx
docker rm my-nginx
```

### Example 2: Run a Database
```bash
# Run MySQL database
docker run -d \
  --name my-mysql \
  -e MYSQL_ROOT_PASSWORD=mypassword \
  -e MYSQL_DATABASE=mydb \
  -p 3306:3306 \
  mysql:8.0

# Connect to database
docker exec -it my-mysql mysql -u root -p
```

### Example 3: Development Environment
```bash
# Run Node.js app with mounted source code
docker run -d \
  --name my-app \
  -p 3000:3000 \
  -v $(pwd):/app \
  -w /app \
  node:16 \
  npm start
```

## Quick Tips

- Use `docker ps -q` to get only container IDs
- Use `$(docker ps -q)` to reference all running containers
- Container names must be unique
- Use `--help` with any command to see options: `docker run --help`
- Images are read-only templates, containers are running instances
- Always clean up unused containers and images to save disk space
- Use `.dockerignore` file to exclude files when building images

## Common Flags Explained

- `-d` = detached (run in background)
- `-it` = interactive terminal
- `-p` = port mapping
- `-v` = volume/mount
- `-e` = environment variable
- `--name` = give container a name
- `--rm` = remove container when it stops
- `-f` = follow (for logs)
- `-a` = all (show stopped containers too)
