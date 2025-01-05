## Chapter 5: Docker Security and Best Practices

### Quick Security Checklist
- [ ] Keep Docker and host system updated
- [ ] Use rootless Docker when possible
- [ ] Implement least privilege principle
- [ ] Scan images for vulnerabilities
- [ ] Use multi-stage builds
- [ ] Enable Docker Content Trust
- [ ] Configure resource limits
- [ ] Implement network segmentation
- [ ] Regular security audits
- [ ] Monitor container activities

### 5.1 Docker Security Fundamentals
Security is crucial when running containers in production. This chapter covers essential security practices and tools for Docker environments.

#### Security Layers
- **Host Security**: Protecting the Docker host system through updates, firewall configuration, and access controls
- **Docker Daemon Security**: Securing the Docker runtime through TLS encryption, user namespace remapping, and proper configuration
- **Container Security**: Isolating containers using cgroups, namespaces, and security options while limiting capabilities
- **Image Security**: Ensuring trusted base images, scanning for vulnerabilities, and implementing proper signing mechanisms

### 5.2 Host and Daemon Security

#### Host System Hardening
```bash
# Update system packages
sudo apt-get update && sudo apt-get upgrade

# Enable firewall
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow 2376/tcp  # Docker daemon TLS
```

#### Docker Daemon Security
```json
{
  "userns-remap": "default",
  "live-restore": true,
  "no-new-privileges": true,
  "seccomp-profile": "/etc/docker/seccomp.json"
}
```

### 5.3 Container Security

#### Running Containers Securely
```bash
# Run container with limited capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx

# Set memory limits
docker run --memory="512m" --memory-swap="1g" nginx

# Run as non-root user
docker run -u 1000:1000 nginx
```

#### Security Scanning
```bash
# Scan image for vulnerabilities
docker scan nginx:latest

# Use Trivy for detailed scanning
trivy image nginx:latest
```

### 5.4 Image Security Best Practices

#### Dockerfile Security
```dockerfile
# Use specific version tags
FROM nginx:1.21-alpine

# Create non-root user
RUN adduser -D myuser
USER myuser

# Set proper permissions
COPY --chown=myuser:myuser . /app

# Use multi-stage builds to reduce attack surface
FROM node:16 AS builder
# ... build steps ...

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
```

#### Image Signing and Trust
```bash
# Enable Docker Content Trust
export DOCKER_CONTENT_TRUST=1

# Sign and push image
docker trust sign myregistry/myimage:latest
docker push myregistry/myimage:latest
```

### 5.5 Network Security

#### Network Isolation
```yaml
# docker-compose.yml with network isolation
version: "3.8"
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
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No external access
```

#### TLS Configuration
```bash
# Generate TLS certificates
openssl req -newkey rsa:4096 -nodes -sha256 \
  -keyout domain.key -x509 -days 365 -out domain.crt

# Configure nginx with TLS
docker run -d \
  -v $(pwd)/domain.crt:/etc/nginx/ssl/domain.crt \
  -v $(pwd)/domain.key:/etc/nginx/ssl/domain.key \
  nginx
```

### 5.6 Monitoring and Auditing

#### Container Monitoring
```bash
# View container resource usage
docker stats

# Enable logging
docker run --log-driver=journald nginx

# Audit container events
docker events --filter 'type=container'
```

#### Security Auditing Tools
```bash
# Install Docker Bench Security
git clone https://github.com/docker/docker-bench-security.git
cd docker-bench-security
sudo sh docker-bench-security.sh
```

### 5.7 Hands-on Exercise: Securing a Production Application

```yaml
# docker-compose.yml with security features
version: "3.8"
services:
  app:
    build: 
      context: .
      dockerfile: Dockerfile
    user: "1000:1000"
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

### 5.8 Compliance and Governance

#### Basic Compliance Requirements
```yaml
# compliance-basics.yaml
compliance:
  container_security:
    # Basic container security requirements
    non_root_user: required
    privileged_containers: forbidden
    read_only_root: preferred
    
  access_control:
    # Access control requirements
    rbac_enabled: required
    authentication_required: required
    audit_logging: required
    
  data_protection:
    # Data protection requirements
    encryption_at_rest: required
    secure_communications: required
    backup_encryption: required
```

#### Regulatory Compliance Implementation
```yaml
# compliance-config.yaml
compliance:
  gdpr:
    data_retention:
      logs: 30d
      user_data: 90d
    data_encryption:
      at_rest: true
      in_transit: true
    data_access:
      audit_logging: true
      role_based: true
  
  hipaa:
    phi_protection:
      encryption_standard: "AES-256"
      backup_encryption: true
    access_controls:
      authentication: "2FA"
      session_timeout: 15m

  pci:
    card_data:
      encryption: true
      masking: true
    network_security:
      segmentation: true
      firewall_rules: true
```

#### Audit Trail Configuration
```yaml
# audit-policy.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: RequestResponse
  resources:
  - group: ""
    resources: ["secrets", "configmaps"]
  
- level: Metadata
  resources:
  - group: ""
    resources: ["pods", "services"]

# fluentd-config.yaml
<match kubernetes.audit>
  @type elasticsearch
  host elasticsearch
  port 9200
  index_name audit-logs
  type_name audit
  logstash_format true
  logstash_prefix audit
  <buffer>
    @type file
    path /var/log/audit-buffer
    flush_interval 5s
  </buffer>
</match>
```

#### Policy Enforcement
```yaml
# opa-policies.yaml
package kubernetes.admission

deny[msg] {
    input.request.kind.kind == "Pod"
    not input.request.object.metadata.labels.owner
    msg := "Pod must have owner label"
}

deny[msg] {
    input.request.kind.kind == "Deployment"
    not input.request.object.spec.template.spec.securityContext.runAsNonRoot
    msg := "Containers must not run as root"
}

# gatekeeper-constraint.yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: require-owner-label
spec:
  match:
    kinds:
    - apiGroups: [""]
      kinds: ["Pod"]
  parameters:
    labels: ["owner"]
```

#### Compliance Monitoring
```yaml
# compliance-monitor.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: compliance-rules
data:
  rules.yaml: |
    groups:
    - name: ComplianceAlerts
      rules:
      - alert: UnencryptedSecret
        expr: |
          sum(secrets_encryption_status{encrypted="false"}) > 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: Unencrypted secrets detected
          
      - alert: UnapprovedImage
        expr: |
          count(container_images{registry!="approved.registry.com"}) > 0
        for: 5m
        labels:
          severity: high
```

#### Automated Compliance Checks
```python
#!/usr/bin/env python3
# compliance-check.py

import kubernetes
import yaml
import logging
from datetime import datetime

def check_compliance():
    try:
        # Initialize Kubernetes client
        kubernetes.config.load_kube_config()
        v1 = kubernetes.client.CoreV1Api()
        
        # Check Pod Security
        pods = v1.list_pod_for_all_namespaces()
        for pod in pods.items:
            check_pod_security(pod)
            
        # Check Network Policies
        check_network_segregation()
        
        # Check Encryption
        check_data_encryption()
        
    except Exception as e:
        logging.error(f"Compliance check failed: {str(e)}")
        
def check_pod_security(pod):
    security_context = pod.spec.security_context
    if not security_context or not security_context.run_as_non_root:
        logging.warning(f"Pod {pod.metadata.name} not configured to run as non-root")
        
def check_network_segregation():
    # Verify network policies exist
    networking_v1 = kubernetes.client.NetworkingV1Api()
    policies = networking_v1.list_network_policy_for_all_namespaces()
    if not policies.items:
        logging.error("No network policies found")
        
def check_data_encryption():
    # Check secrets encryption
    secrets = v1.list_secret_for_all_namespaces()
    for secret in secrets.items:
        if not secret.metadata.annotations.get("encryption"):
            logging.warning(f"Secret {secret.metadata.name} may not be encrypted")

if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO)
    check_compliance()
```

#### Audit Configuration
```yaml
# audit-config.yaml
audit:
  # Container events to monitor
  events:
    - container_create
    - container_start
    - container_stop
    - image_pull
    - volume_mount
    
  # Retention settings
  retention:
    log_files: 30
    audit_records: 90
    
  # Alert conditions
  alerts:
    - condition: privileged_container_started
      severity: high
    - condition: unauthorized_image_used
      severity: medium
    - condition: container_drift_detected
      severity: high
```

#### Compliance Reporting
```yaml
# compliance-report.yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: compliance-report-generator
spec:
  schedule: "0 0 * * 0"  # Weekly
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: report-generator
            image: compliance-reporter:latest
            env:
            - name: REPORT_FORMAT
              value: "pdf"
            - name: REPORT_DESTINATION
              value: "s3://compliance-reports/"
            - name: COMPLIANCE_STANDARDS
              value: "gdpr,hipaa,pci"
          restartPolicy: OnFailure
```

### 5.9 Security Troubleshooting

#### Common Issues and Solutions
- Permission denied errors
- Container escape vulnerabilities
- Resource exhaustion
- Network isolation problems
- Image vulnerability alerts

#### Debugging Security Issues
```bash
# Check container capabilities
docker inspect --format '{{.HostConfig.CapAdd}}' container_name

# Review container processes
docker top container_name

# Inspect container mounts
docker inspect --format '{{.Mounts}}' container_name

# Check container network settings
docker inspect --format '{{.NetworkSettings}}' container_name
```

### Summary
In this chapter, you learned:
- Docker security fundamentals
- Container and image hardening techniques
- Network security configuration
- Monitoring and auditing practices
- Compliance and governance best practices

### Homework
1. Implement a secure multi-container application
2. Set up vulnerability scanning in a CI pipeline
3. Create security policies for container deployment
4. Practice incident response scenarios

### References
- [Docker Security Documentation](https://docs.docker.com/engine/security/)
- [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker)
- [Docker Security Best Practices](https://snyk.io/blog/10-docker-image-security-best-practices/)
