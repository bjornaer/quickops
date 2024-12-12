## Chapter 2: Diving into Docker Images

### 2.1 Understanding Docker Images
Docker images are the foundation of containers. They are read-only templates containing an application's code, runtime, libraries, and dependencies. In this chapter, we'll learn how to create, manage, and optimize Docker images.

#### Image Architecture
- **Layers**: Images are built in layers, each representing a command in the Dockerfile
- **Base Images**: Minimal images that serve as a starting point
- **Cache**: Docker caches layers to speed up builds
- **Tags**: Labels that help version and identify images

### 2.2 Working with Docker Images

#### Pulling Images
```bash
# Pull a specific image
docker pull nginx:1.21

# Pull multiple versions
docker pull ubuntu:20.04
docker pull ubuntu:22.04
```

#### Listing and Managing Images
```bash
# List all images
docker images

# Remove an image
docker rmi nginx:1.21

# Remove unused images
docker image prune
```

### 2.3 Creating Custom Images

#### Writing Dockerfiles
A basic Dockerfile example:
```dockerfile
# Use an official Python runtime as base
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Copy requirements file
COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Command to run the application
CMD ["python", "app.py"]
```

#### Building Images
```bash
# Build an image
docker build -t myapp:1.0 .

# Build with different tags
docker build -t myapp:1.0 -t myapp:latest .
```

### 2.4 Multi-stage Builds
Multi-stage builds help create smaller, more secure images:

```dockerfile
# Build stage
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 2.5 Image Best Practices

#### Security
- Use official base images
- Regularly update base images
- Scan images for vulnerabilities
- Avoid running containers as root

#### Performance
- Minimize layer count
- Use .dockerignore files
- Clean up in the same layer where files are added
- Leverage build cache effectively

#### Example .dockerignore
```plaintext
node_modules
npm-debug.log
Dockerfile
.git
.env
*.md
```

### 2.6 Hands-on Exercise
Let's create a custom web application image:

```bash
# Create a simple web application
mkdir web-app && cd web-app

# Create index.html
echo "<h1>Hello Docker!</h1>" > index.html

# Create Dockerfile
cat << EOF > Dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
EXPOSE 80
EOF

# Build and run
docker build -t webapp:1.0 .
docker run -d -p 8080:80 webapp:1.0
```

### Summary
In this chapter, you learned:
- How Docker images work
- Creating and managing custom images
- Multi-stage build optimization
- Best practices for image creation

### Homework
1. Create a multi-stage Dockerfile for a Node.js application
2. Experiment with different base images
3. Practice optimizing image size
4. Build and push an image to Docker Hub

### References
- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [Docker Build Guide](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Multi-stage Builds](https://docs.docker.com/develop/develop-images/multistage-build/)
