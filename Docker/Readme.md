# Docker Commands

```shell
# Start and enable Docker daemon (Linux systemd-based OS)
systemctl start docker && systemctl enable docker

#------------------------------------------------------------------------------#


# Build image with tag from Dockerfile in current directory
docker build -t <image_name> .

# Build image with build-time argument
docker build --build-arg MY_VAR=value -t <image_name> .

# Build image using specific Dockerfile
docker build -f /path/to/Dockerfile-name -t <your-image-name> .

#------------------------------------------------------------------------------#

# Tag an existing image with a new name and version
docker tag <source-image> username/<new-image>:version

# Push image to Docker Hub
docker push <image_name>

# Pull image from Docker Hub with optional tag
docker pull <image-name>:<tag>

# Remove image
docker rmi <image-id>

# Login to Docker Hub
docker login

# Scan image for vulnerabilities (Docker Scout)
docker scan <Image_name>

# Prune unused images
docker image prune

#------------------------------------------------------------------------------#

# Create container (without starting it)
docker create --name <container_name> <image>

# Run container (foreground)
docker run --name <name> <image_ID>

# Run container (detached/background)
docker run -d --name <name> <image>

# Run with interactive terminal
docker run -it --name <name> <image> /bin/bash

# Run with port mapping (host:container)
docker run -d -p 8080:80 --name <name> <image>

# Run with environment variable
docker run -d --name <container name> -e <key>=<value> <image>

# Run with resource limits (CPU and memory)
docker run --cpu=0.5 --memory=1g <image>

#------------------------------------------------------------------------------#

# Create container (without starting it)
docker create --name <container_name> <image>

# Run container (foreground)
docker run --name <name> <image_ID>

# Run container (detached/background)
docker run -d --name <name> <image>

# Run with interactive terminal
docker run -it --name <name> <image> /bin/bash

# Run with port mapping (host:container)
docker run -d -p 8080:80 --name <name> <image>

# Run with environment variable
docker run -d --name <container name> -e <key>=<value> <image>

# Run with resource limits (CPU and memory)
docker run --cpu=0.5 --memory=1g <image>

#------------------------------------------------------------------------------#

# Start a stopped container
docker start <container-id>

# Stop a running container
docker stop <container-id>

# Restart a container
docker restart <container-id>

# Remove a container
docker rm <container-id>

# Start all stopped containers
docker start $(docker ps -a -q)

# Attach to running container (interactive session)
docker attach <Container_ID>

# Commit changes in container as a new image
docker commit <container_name> <new_image_name>

#------------------------------------------------------------------------------#

# List running containers
docker ps

# List all containers (running + stopped)
docker ps -a

# List all container IDs only
docker ps -a -q

# Get detailed container/image info in JSON
docker inspect <id> | less

# Show running processes in a container
docker top <c-id>

# Display real-time resource usage stats
docker stats <container-id>

# Show Docker system disk usage
docker system df

# Show container filesystem differences
docker diff <container-id>

#------------------------------------------------------------------------------#

# Copy from container to host
docker cp <container-id>:<container-path> <file-path-on-host>

# Copy from host to container
docker cp <file-path-on-host> <container-id>:<container-path>

#------------------------------------------------------------------------------#

# Remove all unused containers, networks, images, and build cache
docker system prune

# Remove all unused volumes
docker volume prune

#------------------------------------------------------------------------------#

# Run command inside an already running container
docker exec -it <running-container-id> <command>

#------------------------------------------------------------------------------#

# Use/share volume from another container
docker run -it --name <container_name1> --privileged=true --volume-from <container2> <base_image> /bin/bash

# Mount host volume to container
docker run -it --name <container_name> -v <host_dir>:<container_dir> <image_name> /bin/bash

# Mount volume to specific path in container
docker run -it --name <container-name> -v /<volume-path> <image> /bin/bash


```
