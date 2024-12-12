## Chapter 4: Docker Compose

### 4.1 Introduction to Docker Compose
Docker Compose is a tool for defining and running multi-container Docker applications. It uses YAML files to configure application services, making it easier to manage complex applications.

#### Key Benefits
- **Simplified Configuration**: Define your entire application stack in a single file
- **Environment Consistency**: Ensure all developers work with the same setup
- **Service Coordination**: Manage dependencies between containers
- **Development Workflow**: Streamline local development environments

### 4.2 Docker Compose Basics

#### Installation
Docker Compose comes bundled with Docker Desktop. For Linux:
```bash
# Download Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Make it executable
sudo chmod +x /usr/local/bin/docker-compose

# Verify installation
docker-compose --version
```

#### Basic Commands
```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs

# List services
docker-compose ps
```

### 4.3 Writing Docker Compose Files

#### Basic Structure
```yaml
version: "3.8"
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - frontend

  api:
    build: ./api
    environment:
      - DB_HOST=db
    depends_on:
      - db
    networks:
      - frontend
      - backend

  db:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=secret
      - MYSQL_DATABASE=myapp
    volumes:
      - db-data:/var/lib/mysql
    networks:
      - backend

volumes:
  db-data:

networks:
  frontend:
  backend:
```

### 4.4 Advanced Compose Features

#### Environment Variables
```yaml
services:
  web:
    env_file:
      - .env
    environment:
      - API_KEY=${API_KEY}
```

#### Health Checks
```yaml
services:
  web:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
```

#### Service Dependencies
```yaml
services:
  api:
    depends_on:
      db:
        condition: service_healthy
```

### 4.5 Development Workflows

#### Local Development
```yaml
services:
  web:
    build:
      context: .
      target: development
    volumes:
      - .:/app
    command: npm run dev
```

#### Production Configuration
```yaml
services:
  web:
    build:
      context: .
      target: production
    restart: always
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

### 4.6 Hands-on Exercise: Full-Stack Application
Let's create a complete web application with frontend, backend, and database:

```yaml
# docker-compose.yml
version: "3.8"
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://localhost:4000
    depends_on:
      - backend

  backend:
    build: ./backend
    ports:
      - "4000:4000"
    environment:
      - DB_HOST=database
      - DB_USER=root
      - DB_PASSWORD=secret
    depends_on:
      database:
        condition: service_healthy

  database:
    image: postgres:13
    environment:
      - POSTGRES_USER=root
      - POSTGRES_PASSWORD=secret
    volumes:
      - db-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U root"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  db-data:
```

### Summary
In this chapter, you learned:
- Docker Compose fundamentals
- Writing and managing compose files
- Advanced compose features
- Development and production workflows

### Homework
1. Create a compose file for a microservices application
2. Implement development and production configurations
3. Practice using environment variables and secrets
4. Set up a CI/CD pipeline using Docker Compose

### References
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Compose File Reference](https://docs.docker.com/compose/compose-file/)
- [Compose CLI Reference](https://docs.docker.com/compose/reference/)
