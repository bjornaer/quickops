## Chapter 9: Docker in Production

### 9.1 Production Readiness
Moving Docker to production requires careful planning and consideration of various factors to ensure reliability, security, and performance.

#### Key Considerations
- **High Availability**: Ensuring service uptime
- **Scalability**: Handling increased load
- **Security**: Protecting applications and data
- **Monitoring**: Tracking system health
- **Backup**: Protecting against data loss

### 9.2 Production Infrastructure

#### Host Configuration
```bash
# System optimization
cat << EOF > /etc/sysctl.d/99-docker-production.conf
net.ipv4.ip_forward=1
net.bridge.bridge-nf-call-iptables=1
net.ipv4.tcp_max_syn_backlog=2048
net.core.somaxconn=1024
EOF

sysctl --system
```

#### Storage Configuration
```bash
# Configure direct-lvm for Docker
cat << EOF > /etc/docker/daemon.json
{
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
EOF
```

### 9.3 High Availability Setup

#### Load Balancer Configuration
```yaml
# docker-compose.yml
version: '3.8'
services:
  haproxy:
    image: haproxy:2.4
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro
    deploy:
      mode: global
      placement:
        constraints:
          - node.role == manager

  app:
    image: myapp:latest
    deploy:
      replicas: 3
      update_config:
        order: start-first
        failure_action: rollback
      restart_policy:
        condition: on-failure
```

#### HAProxy Configuration
```conf
# haproxy.cfg
global
    maxconn 4096

defaults
    mode http
    timeout client 10s
    timeout connect 5s
    timeout server 10s

frontend http_front
    bind *:80
    default_backend http_back

backend http_back
    balance roundrobin
    server app1 app:80 check
    server app2 app:80 check
    server app3 app:80 check
```

### 9.4 Backup and Recovery

#### Volume Backup Strategy
```bash
#!/bin/bash
# backup-volumes.sh

BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d_%H%M%S)

# Backup each volume
for volume in $(docker volume ls -q); do
  echo "Backing up $volume"
  docker run --rm \
    -v $volume:/data \
    -v $BACKUP_DIR:/backup \
    alpine tar czf /backup/$volume-$DATE.tar.gz /data
done
```

#### Database Backup
```yaml
services:
  db-backup:
    image: postgres:13
    volumes:
      - db-data:/var/lib/postgresql/data:ro
      - ./backups:/backups
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    command: |
      bash -c 'pg_dump -U postgres -d myapp > /backups/dump_$$(date +%Y%m%d_%H%M%S).sql'
    secrets:
      - db_password
```
### 9.5 Cost Management and Optimization

#### Resource Cost Analysis
```bash
# Get container resource usage metrics
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}\t{{.BlockIO}}"

# Analyze image sizes
docker images --format "table {{.Repository}}\t{{.Size}}" | sort -k 2 -h
```

#### Cost Optimization Strategies
```yaml
# Resource-optimized deployment
version: '3.8'
services:
  app:
    image: myapp:latest
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.1'
          memory: 128M
    environment:
      NODE_ENV: production
      # Optimize Node.js memory
      NODE_OPTIONS: "--max-old-space-size=384"

  cache:
    image: redis:alpine
    command: redis-server --maxmemory 256mb --maxmemory-policy allkeys-lru
```

#### Cost Monitoring Setup
```yaml
# prometheus-rules.yml
groups:
  - name: CostAlerts
    rules:
      - alert: HighResourceCost
        expr: sum(container_memory_usage_bytes) > 10e9
        for: 12h
        labels:
          severity: warning
        annotations:
          summary: High resource usage detected
          description: Cluster memory usage exceeds 10GB for 12 hours

# grafana-dashboard.json
{
  "panels": [
    {
      "title": "Resource Cost Overview",
      "type": "graph",
      "metrics": [
        "sum(rate(container_cpu_usage_seconds_total[5m])) * 730 * $CPU_COST",
        "sum(container_memory_usage_bytes) / 1024 / 1024 / 1024 * 730 * $MEMORY_COST"
      ]
    }
  ]
}
```

#### Automated Cost Optimization
```python
# cost-optimizer.py
import docker
from datetime import datetime, timedelta

def optimize_containers():
    client = docker.from_env()
    
    # Find idle containers
    for container in client.containers.list():
        stats = container.stats(stream=False)
        cpu_usage = calculate_cpu_usage(stats)
        
        if cpu_usage < 0.1:  # Less than 10% CPU
            print(f"Low usage container found: {container.name}")
            # Scale down or notify
            
def calculate_cpu_usage(stats):
    # CPU usage calculation logic
    return cpu_delta / system_delta * 100

if __name__ == "__main__":
    optimize_containers()
```

#### Budget Management
```yaml
# cost-policies.yaml
policies:
  dev_environment:
    max_monthly_budget: 1000
    alerts:
      - threshold: 80%
        notification: email
    auto_actions:
      - threshold: 90%
        action: scale_down
        exclude:
          - production-critical

  prod_environment:
    max_monthly_budget: 5000
    alerts:
      - threshold: 70%
        notification: slack
      - threshold: 90%
        notification: pagerduty
```
### 9.6 Disaster Recovery

#### Backup Strategies
```yaml
# backup-policy.yaml
version: '3.8'
services:
  backup-manager:
    image: backup-service:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - backup-volume:/backups
    environment:
      BACKUP_RETENTION_DAYS: 30
      BACKUP_SCHEDULE: "0 2 * * *"  # 2 AM daily
      S3_BUCKET: "dr-backups"
      ENCRYPTION_KEY: "${ENCRYPTION_KEY}"

volumes:
  backup-volume:
    driver: rexray/s3fs:latest
    driver_opts:
      bucket: "${BACKUP_BUCKET}"
      region: "us-west-2"
```

#### Multi-Region Deployment
```yaml
# docker-compose.dr.yml
version: '3.8'
services:
  app:
    image: myapp:latest
    deploy:
      replicas: 3
      placement:
        constraints:
          - node.labels.region==${DEPLOY_REGION}
    environment:
      DATABASE_URL: "${DB_REPLICA_URL}"
      CACHE_ENDPOINTS: "${CACHE_ENDPOINTS}"

  db-sync:
    image: db-replication:latest
    environment:
      PRIMARY_DB: "${PRIMARY_DB_URL}"
      REPLICA_DB: "${REPLICA_DB_URL}"
      SYNC_INTERVAL: "5m"
```

#### Automated Failover
```bash
#!/bin/bash
# failover.sh

# Check primary region health
check_region_health() {
    local region=$1
    local endpoints=(${PRIMARY_ENDPOINTS})
    
    for endpoint in "${endpoints[@]}"; do
        if ! curl -sf "$endpoint/health" > /dev/null; then
            return 1
        fi
    done
    return 0
}

# Initiate failover
perform_failover() {
    echo "Initiating failover to secondary region..."
    
    # Update DNS
    aws route53 change-resource-record-sets \
        --hosted-zone-id ${HOSTED_ZONE} \
        --change-batch file://dns-failover.json
        
    # Promote replica database
    docker exec db-replica pg_ctl promote
    
    # Scale up secondary region
    docker stack deploy -c docker-compose.dr.yml app_stack
}

# Monitor and execute
while true; do
    if ! check_region_health "primary"; then
        perform_failover
        break
    fi
    sleep 30
done
```

#### Data Synchronization
```yaml
# sync-config.yaml
sync:
  databases:
    - name: main_db
      type: postgres
      primary:
        host: primary-db.example.com
        port: 5432
      replicas:
        - host: replica-db-1.example.com
          port: 5432
          region: us-west-2
        - host: replica-db-2.example.com
          port: 5432
          region: eu-west-1
      
  storage:
    - type: s3
      buckets:
        primary: primary-storage
        replicas:
          - name: backup-storage-1
            region: us-west-2
          - name: backup-storage-2
            region: eu-west-1
      sync_schedule: "*/15 * * * *"
```

#### Recovery Testing
```yaml
# recovery-test.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: dr-test
spec:
  template:
    spec:
      containers:
      - name: dr-test
        image: dr-test:latest
        env:
        - name: BACKUP_LOCATION
          value: "s3://dr-backups/latest"
        - name: TEST_REGION
          value: "us-west-2"
        volumeMounts:
        - name: test-results
          mountPath: /results
      volumes:
      - name: test-results
        persistentVolumeClaim:
          claimName: test-results-pvc
```

#### Recovery Runbook
```yaml
# recovery-procedures.yaml
procedures:
  database_recovery:
    steps:
      - verify_backup_integrity:
          command: "check_backup.sh ${BACKUP_ID}"
          timeout: 5m
      - restore_database:
          command: "restore_db.sh ${BACKUP_ID}"
          timeout: 30m
      - verify_data:
          command: "verify_data.sh"
          timeout: 10m
          
  application_recovery:
    steps:
      - deploy_infrastructure:
          command: "terraform apply -auto-approve"
          timeout: 15m
      - deploy_applications:
          command: "docker stack deploy -c docker-compose.dr.yml app"
          timeout: 10m
      - health_check:
          command: "check_health.sh"
          timeout: 5m
          
  network_recovery:
    steps:
      - update_dns:
          command: "update_dns.sh ${NEW_REGION}"
          timeout: 5m
      - verify_routing:
          command: "verify_routes.sh"
          timeout: 5m
```

### 9.7 Production Security

#### Security Checklist
```yaml
# security-checklist.yml
version: '3.8'
services:
  app:
    image: myapp:latest
    security_opt:
      - no-new-privileges:true
    read_only: true
    tmpfs:
      - /tmp
    capabilities:
      drop:
        - ALL
      add:
        - NET_BIND_SERVICE
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

#### Network Security
```yaml
services:
  frontend:
    networks:
      - frontend
  backend:
    networks:
      - frontend
      - backend
  database:
    networks:
      - backend

networks:
  frontend:
    driver: overlay
    encryption: true
  backend:
    driver: overlay
    internal: true
    encryption: true
```

### 9.8 Scaling Strategies

#### Horizontal Scaling
```bash
# Scale service
docker service scale myapp=5

# Configure auto-scaling
docker service update \
  --update-parallelism 2 \
  --update-delay 10s \
  myapp
```

#### Resource Management
```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

### 9.9 Hands-on Exercise: Production Deployment

Create a complete production-ready deployment:

```yaml
# production-stack.yml
version: '3.8'
services:
  loadbalancer:
    image: haproxy:2.4
    ports:
      - "80:80"
      - "443:443"
    configs:
      - source: haproxy_config
        target: /usr/local/etc/haproxy/haproxy.cfg
    deploy:
      mode: global
      placement:
        constraints:
          - node.role == manager

  app:
    image: myapp:latest
    secrets:
      - app_secret
    deploy:
      replicas: 3
      update_config:
        order: start-first
        failure_action: rollback
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
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
  haproxy_config:
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
- Production infrastructure setup
- High availability configuration
- Backup and recovery strategies
- Security best practices
- Scaling and resource management

### Homework
1. Set up a production environment with high availability
2. Implement automated backup solutions
3. Create a disaster recovery plan
4. Practice scaling and failover scenarios

### References
- [Docker Production Guide](https://docs.docker.com/config/containers/production/)
- [Security Best Practices](https://docs.docker.com/engine/security/security/)
- [High Availability Guide](https://docs.docker.com/config/containers/ha/)
