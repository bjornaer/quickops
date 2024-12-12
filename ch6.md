## Chapter 6: Continuous Integration and Deployment with Docker

### 6.1 Introduction to CI/CD
Continuous Integration and Continuous Deployment (CI/CD) automate the process of building, testing, and deploying applications. Docker plays a crucial role in creating consistent and reliable CI/CD pipelines.

#### Key Concepts
- **Continuous Integration**: Automatically building and testing code changes
- **Continuous Deployment**: Automatically deploying tested code to production
- **Pipeline**: Series of automated steps from code to deployment
- **Artifacts**: Built applications and dependencies

### 6.2 Setting Up a CI/CD Pipeline

#### GitHub Actions Example
```yaml
name: Docker CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build Docker image
        run: docker build -t myapp:test .
      
      - name: Run tests
        run: docker run myapp:test npm test
      
      - name: Push to registry
        if: github.ref == 'refs/heads/main'
        run: |
          docker tag myapp:test registry.example.com/myapp:latest
          docker push registry.example.com/myapp:latest
```

### 6.3 Container Registry Management

#### Private Registry Setup
```bash
# Run private registry
docker run -d \
  -p 5000:5000 \
  --name registry \
  -v registry-data:/var/lib/registry \
  registry:2

# Push to private registry
docker tag myapp:latest localhost:5000/myapp:latest
docker push localhost:5000/myapp:latest
```

#### Registry Authentication
```yaml
# docker-compose.yml for secure registry
version: "3.8"
services:
  registry:
    image: registry:2
    ports:
      - "5000:5000"
    environment:
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
    volumes:
      - ./auth:/auth
      - registry-data:/var/lib/registry
```

### 6.4 Automated Testing

#### Integration Testing
```dockerfile
# Dockerfile.test
FROM node:16-alpine

WORKDIR /app
COPY . .
RUN npm install

CMD ["npm", "run", "test:integration"]
```

#### Test Pipeline
```bash
#!/bin/bash
# test-pipeline.sh

# Build test image
docker build -t myapp:test -f Dockerfile.test .

# Run unit tests
docker run --rm myapp:test npm run test:unit

# Run integration tests
docker run --rm \
  --network test-network \
  myapp:test npm run test:integration

# Clean up
docker rmi myapp:test
```

### 6.5 Deployment Strategies

#### Blue-Green Deployment
```bash
# Deploy new version
docker-compose -f docker-compose.green.yml up -d

# Verify new version
curl http://localhost:8081/health

# Switch traffic
docker-compose -f docker-compose.proxy.yml up -d

# Remove old version
docker-compose -f docker-compose.blue.yml down
```

#### Rolling Updates
```yaml
# docker-compose.yml
services:
  web:
    image: myapp:latest
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
```

### 6.6 Monitoring Deployments

#### Health Checks
```yaml
services:
  app:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

#### Logging and Metrics
```yaml
services:
  app:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
```

### 6.7 Hands-on Exercise: Complete CI/CD Pipeline

Create a full CI/CD pipeline for a web application:

```yaml
# .github/workflows/pipeline.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]

jobs:
  build-test-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to Registry
        uses: docker/login-action@v2
        with:
          registry: registry.example.com
          username: ${{ secrets.REGISTRY_USER }}
          password: ${{ secrets.REGISTRY_PASSWORD }}
      
      - name: Build and Test
        run: |
          docker build -t myapp:test .
          docker run myapp:test npm test
      
      - name: Build and Push Production Image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: registry.example.com/myapp:latest
      
      - name: Deploy to Production
        uses: appleboy/ssh-action@v0.1.10
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.PROD_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /app
            docker-compose pull
            docker-compose up -d
```

### Summary
In this chapter, you learned:
- Setting up CI/CD pipelines with Docker
- Managing container registries
- Implementing automated testing
- Deploying applications safely
- Monitoring deployments

### Homework
1. Create a complete CI/CD pipeline for a microservices application
2. Implement automated testing with different strategies
3. Set up a private Docker registry
4. Practice different deployment patterns

### References
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Docker Registry Documentation](https://docs.docker.com/registry/)
- [CI/CD Best Practices](https://docs.docker.com/develop/dev-best-practices/)
