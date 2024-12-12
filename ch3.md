## Chapter 3: Docker Networking and Storage

### 3.1 Docker Networking Fundamentals
Docker networking enables containers to communicate with each other and the outside world. Understanding networking is crucial for building distributed applications.

#### Network Types
- **bridge**: Default network for container communication
- **host**: Shares the host's network stack
- **none**: Completely isolated network
- **overlay**: Multi-host networking
- **macvlan**: Assigns MAC addresses to containers

### 3.2 Working with Docker Networks

#### Managing Networks
```bash
# List networks
docker network ls

# Create a network
docker network create my-network

# Inspect network
docker network inspect my-network

# Remove network
docker network rm my-network
```

#### Connecting Containers
```bash
# Run container with network
docker run -d --network my-network --name app1 nginx

# Connect existing container
docker network connect my-network app2
```

### 3.3 Docker Storage

#### Storage Types
- **Volumes**: Managed by Docker
- **Bind Mounts**: Direct host filesystem mapping
- **tmpfs**: Temporary filesystem in memory

#### Managing Volumes
```bash
# Create volume
docker volume create my-data

# List volumes
docker volume ls

# Remove volume
docker volume rm my-data
```

#### Using Storage in Containers
```bash
# Run container with volume
docker run -d \
  --name db \
  -v my-data:/var/lib/mysql \
  mysql:8.0

# Use bind mount
docker run -d \
  --name web \
  -v $(pwd)/html:/usr/share/nginx/html \
  nginx
```

### 3.4 Network and Storage Best Practices

#### Networking
- Use custom networks for container isolation
- Implement network segmentation
- Use DNS for service discovery
- Monitor network traffic

#### Storage
- Use volumes for persistent data
- Regular backup of important volumes
- Clean up unused volumes
- Use appropriate storage drivers

### 3.5 Hands-on Exercise: Multi-Container Application
Let's create a web application with a database:

```bash
# Create network
docker network create web-app-net

# Start MySQL container
docker run -d \
  --name mysql \
  --network web-app-net \
  -v mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=myapp \
  mysql:8.0

# Start web application
docker run -d \
  --name webapp \
  --network web-app-net \
  -p 8080:80 \
  -e DB_HOST=mysql \
  webapp:1.0
```

### 3.6 Troubleshooting

#### Network Troubleshooting
```bash
# Test container connectivity
docker exec webapp ping mysql

# View container logs
docker logs webapp

# Inspect container networking
docker inspect webapp
```

#### Storage Troubleshooting
```bash
# Check volume mounts
docker inspect -f '{{ .Mounts }}' webapp

# Verify volume contents
docker run --rm -v my-data:/data alpine ls /data
```

### Summary
In this chapter, you learned:
- Docker networking concepts and types
- Storage options and management
- Best practices for networking and storage
- Multi-container application deployment

### Homework
1. Create a three-tier application using custom networks
2. Implement data persistence using volumes
3. Practice network troubleshooting
4. Experiment with different storage drivers

### References
- [Docker Networking Guide](https://docs.docker.com/network/)
- [Docker Storage Guide](https://docs.docker.com/storage/)
- [Docker Compose Networking](https://docs.docker.com/compose/networking/)
