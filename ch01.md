## Chapter 1: Docker Fundamentals

### 1.1 Understanding Docker
Docker is a containerization platform that revolutionizes how we package and run applications. In this chapter, we'll explore its core concepts and get hands-on experience with containers.

#### What is Docker?
Docker enables you to package applications and their dependencies into lightweight, portable containers. These containers can run consistently across any environment that has Docker installed, eliminating the infamous "it works on my machine" problem.

#### Key Components
- **Docker Engine**: The runtime that runs and manages containers
- **Docker Images**: Templates containing everything needed to run an application
- **Docker Containers**: Running instances of Docker images
- **Docker Registry**: A repository for storing and sharing images
- **Dockerfile**: A script defining how to build a Docker image

### 1.2 Installing Docker

#### System Requirements
Before installation, ensure your system meets these requirements:
- **Windows**: Windows 10/11 Pro or Enterprise with WSL2 enabled
- **macOS**: macOS 10.15 or later
- **Linux**: A modern kernel (4.x+)

#### Installation Steps

##### Windows and macOS
1. Download Docker Desktop from [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop)
2. Run the installer
3. Follow the setup wizard
4. Start Docker Desktop

##### Linux (Ubuntu)
```bash
# Update package index
sudo apt-get update

# Install Docker
sudo apt-get install docker.io

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker
```

#### Verify Installation
Test your installation by running:
```bash
docker --version
docker run hello-world
```

### 1.3 Docker Basics

#### Running Your First Container
Let's start with a simple example:
```bash
docker run nginx
```

This command:
1. Downloads the nginx image (if not present locally)
2. Creates a container from the image
3. Starts the container running nginx

#### Essential Docker Commands
```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Stop a container
docker stop <container_id>

# Remove a container
docker rm <container_id>

# List images
docker images
```

### 1.4 Best Practices
- Always use specific image tags instead of `latest`
- Clean up unused containers and images regularly
- Follow the principle of least privilege
- Use multi-stage builds for smaller images

### 1.5 Hands-on Exercise
Let's create and run a simple web server:

```bash
# Run nginx with port mapping
docker run -d -p 8080:80 --name my-nginx nginx

# Verify it's running
docker ps

# Access in browser
# Open: http://localhost:8080
```

### Summary
In this chapter, you learned:
- Basic Docker concepts and architecture
- How to install Docker on different platforms
- Essential Docker commands
- How to run your first container

### Homework
1. Run different versions of nginx using specific tags
2. Explore container logs using `docker logs`
3. Try stopping and removing containers
4. Experiment with running containers in interactive mode

### References
- [Docker Get Started Guide](https://docs.docker.com/get-started/)
- [Docker Command Reference](https://docs.docker.com/engine/reference/commandline/cli/)