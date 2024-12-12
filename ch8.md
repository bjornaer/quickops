## Chapter 8: Monitoring and Logging Docker Containers

### 8.1 Introduction to Container Monitoring
Effective monitoring and logging are crucial for maintaining healthy containerized applications. This chapter covers tools and practices for monitoring Docker containers.

#### Monitoring Aspects
- **Resource Usage**: CPU, memory, network, and disk metrics
- **Application Health**: Service availability and performance
- **Container Lifecycle**: Start, stop, and restart events
- **Log Management**: Centralized logging and analysis

### 8.2 Built-in Docker Monitoring

#### Basic Monitoring Commands
```bash
# View container stats
docker stats

# Container processes
docker top <container_name>

# View container events
docker events

# Container logs
docker logs -f <container_name>
```

#### Resource Limits Monitoring
```bash
# Run container with resource limits
docker run -d \
  --name monitored-app \
  --memory="512m" \
  --cpus="0.5" \
  nginx

# Monitor specific container
docker stats monitored-app --no-stream
```

### 8.3 Advanced Monitoring Tools

#### Setting up Prometheus and Grafana
```yaml
# docker-compose.yml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana:latest
    depends_on:
      - prometheus
    ports:
      - "3000:3000"

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    ports:
      - "8080:8080"
```

#### Prometheus Configuration
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']
```

### 8.4 Centralized Logging

#### ELK Stack Setup
```yaml
version: '3.8'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.0
    environment:
      - discovery.type=single-node
    ports:
      - "9200:9200"

  logstash:
    image: docker.elastic.co/logstash/logstash:7.15.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:7.15.0
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
```

#### Logstash Configuration
```conf
# logstash.conf
input {
  gelf {
    port => 12201
    type => docker
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
  }
}
```

### 8.5 Application-Level Monitoring

#### Health Checks
```yaml
services:
  app:
    image: myapp:latest
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

#### Metrics Endpoint
```dockerfile
# Dockerfile with Prometheus metrics
FROM node:16-alpine

WORKDIR /app
COPY . .
RUN npm install prom-client

EXPOSE 3000 9090
CMD ["node", "app.js"]
```

### 8.6 Alert Management

#### Alertmanager Setup
```yaml
services:
  alertmanager:
    image: prom/alertmanager:latest
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml

  prometheus:
    volumes:
      - ./alerts.yml:/etc/prometheus/alerts.yml
```

#### Alert Rules
```yaml
# alerts.yml
groups:
  - name: container_alerts
    rules:
      - alert: HighMemoryUsage
        expr: container_memory_usage_bytes > 1e9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: High memory usage detected
```

### 8.7 Hands-on Exercise: Complete Monitoring Stack

Let's set up a comprehensive monitoring solution:

```yaml
# monitoring-stack.yml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus:/etc/prometheus
    ports:
      - "9090:9090"
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    volumes:
      - grafana-storage:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=secret

  alertmanager:
    image: prom/alertmanager:latest
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager:/etc/alertmanager

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    ports:
      - "8080:8080"

volumes:
  grafana-storage:
```

### Summary
In this chapter, you learned:
- Basic and advanced container monitoring
- Setting up centralized logging
- Implementing alerting systems
- Creating comprehensive monitoring solutions

### Homework
1. Set up a complete monitoring stack for a microservices application
2. Create custom Grafana dashboards
3. Implement alert notifications
4. Practice log analysis with ELK stack

### References
- [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
- [Grafana Documentation](https://grafana.com/docs/)
- [ELK Stack Documentation](https://www.elastic.co/guide/index.html)
