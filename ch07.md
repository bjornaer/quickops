## Chapter 7: Container Orchestration with Docker Swarm

### 7.1 Introduction to Container Orchestration
Container orchestration automates the deployment, scaling, and management of containerized applications. Docker Swarm is Docker's native orchestration solution.

#### Key Concepts
- **Cluster**: A group of Docker hosts working together
- **Nodes**: Individual machines in the cluster
- **Services**: Distributed applications running on nodes
- **Tasks**: Individual container instances of a service
- **Load Balancing**: Distributing requests across containers

### 7.2 Setting Up a Swarm Cluster

#### Initialize Swarm
```bash
# Initialize on first node (manager)
docker swarm init --advertise-addr <MANAGER-IP>

# Join additional nodes as workers
docker swarm join --token <WORKER-TOKEN> <MANAGER-IP>:2377

# List nodes in the swarm
docker node ls
```

#### Node Management
```bash
# Promote a worker to manager
docker node promote <NODE-NAME>

# Demote a manager to worker
docker node demote <NODE-NAME>

# Remove a node
docker node rm <NODE-NAME>
```

### 7.3 Deploying Services

#### Basic Service Deployment
```bash
# Create a service
docker service create \
  --name webapp \
  --replicas 3 \
  --publish 80:80 \
  nginx:latest

# List services
docker service ls

# Inspect service
docker service inspect webapp

# View service logs
docker service logs webapp
```

#### Service Updates
```bash
# Update service image
docker service update \
  --image nginx:1.21 \
  --update-parallelism 1 \
  --update-delay 30s \
  webapp
```

### 7.4 Stack Deployment

#### Creating a Stack File
```yaml
# docker-compose.stack.yml
version: '3.8'
services:
  web:
    image: nginx:latest
    deploy:
      replicas: 3
      placement:
        constraints:
          - node.role == worker
      update_config:
        parallelism: 1
        delay: 10s
    ports:
      - "80:80"

  api:
    image: api:latest
    deploy:
      replicas: 2
      placement:
        max_replicas_per_node: 1
    secrets:
      - api_key

  db:
    image: postgres:13
    deploy:
      placement:
        constraints:
          - node.labels.db == true
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:

secrets:
  api_key:
    external: true
```

#### Deploy Stack
```bash
# Deploy stack
docker stack deploy -c docker-compose.stack.yml myapp

# List stacks
docker stack ls

# List stack services
docker stack services myapp

# Remove stack
docker stack rm myapp
```

### 7.5 Swarm Networking

#### Overlay Networks
```bash
# Create overlay network
docker network create \
  --driver overlay \
  --attachable \
  frontend-network

# Create service with network
docker service create \
  --name webapp \
  --network frontend-network \
  nginx
```

#### Service Discovery
```yaml
services:
  web:
    image: nginx
    networks:
      - frontend
  api:
    image: api
    networks:
      - frontend
      - backend
    environment:
      - DB_HOST=db
  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
    driver: overlay
  backend:
    driver: overlay
    internal: true
```

### 7.6 Swarm Security

#### Secret Management
```bash
# Create secret
echo "my-secret-data" | docker secret create api_key -

# Use secret in service
docker service create \
  --name api \
  --secret api_key \
  myapi:latest
```

#### Node Labels and Constraints
```bash
# Add label to node
docker node update --label-add db=true node-1

# Use constraint in service
docker service create \
  --name db \
  --constraint 'node.labels.db == true' \
  postgres
```

### 7.7 Hands-on Exercise: Production Stack Deployment

Let's deploy a complete production stack:

```yaml
# production-stack.yml
version: '3.8'
services:
  loadbalancer:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    deploy:
      replicas: 2
      placement:
        constraints:
          - node.role == worker
    configs:
      - source: nginx_conf
        target: /etc/nginx/nginx.conf

  app:
    image: myapp:latest
    deploy:
      replicas: 4
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
    secrets:
      - source: app_secret
        target: /app/config/secret
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:13
    volumes:
      - db-data:/var/lib/postgresql/data
    secrets:
      - db_password
    deploy:
      placement:
        constraints:
          - node.labels.db == true
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  db-data:

configs:
  nginx_conf:
    external: true
  db_config:
    external: true

secrets:
  app_secret:
    external: true
  db_password:
    external: true
```

### Summary
In this chapter, you learned:
- Docker Swarm fundamentals
- Setting up and managing a swarm cluster
- Deploying services and stacks
- Implementing security and networking

### Homework
1. Set up a multi-node swarm cluster
2. Deploy a microservices application using stacks
3. Implement rolling updates and rollbacks
4. Configure secure secret management

### References
- [Docker Swarm Documentation](https://docs.docker.com/engine/swarm/)
- [Swarm Mode Tutorial](https://docs.docker.com/engine/swarm/swarm-tutorial/)
- [Docker Configs](https://docs.docker.com/engine/swarm/configs/)
