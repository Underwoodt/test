# Docker Deployment

**Deploying using Docker and Docker Compose**

---

## Local Development with Docker Compose

### Start All Services

```bash
cd ai-defra-search-core
docker compose up -d
```

### Stop All Services

```bash
docker compose down
```

### Clean Everything

```bash
# Remove all containers, networks, volumes
docker compose down -v
```

---

## Building Docker Images

### Build All Images

```bash
docker compose build
```

### Build Specific Service

```bash
docker compose build agent
docker compose build knowledge
docker compose build frontend
```

### Build Without Cache

```bash
docker compose build --no-cache
```

---

## Running Individual Services

### Start Single Service

```bash
docker compose up -d agent
```

### View Logs

```bash
docker compose logs -f agent
```

### Restart Service

```bash
docker compose restart agent
```

---

## Environment Configuration

### Override Configuration

Create `.env` file in project root:

```bash
# .env
ENVIRONMENT=production
DEBUG=false
LOG_LEVEL=INFO
BEDROCK_ENDPOINT=https://bedrock.us-east-1.amazonaws.com
POSTGRES_PASSWORD=secure_password
```

### Use .env file

```bash
docker compose up -d
# Automatically loads variables from .env
```

---

## Production Deployment

### Build Production Image

```bash
# For Agent service
docker build -f services/ai-defra-search-agent/Dockerfile \
  -t my-registry/ai-defra-search-agent:latest \
  services/ai-defra-search-agent/

# Tag version
docker tag my-registry/ai-defra-search-agent:latest \
  my-registry/ai-defra-search-agent:v1.0.0
```

### Push to Registry

```bash
docker push my-registry/ai-defra-search-agent:latest
docker push my-registry/ai-defra-search-agent:v1.0.0
```

### Docker Compose for Production

```yaml
version: '3.8'
services:
  agent:
    image: my-registry/ai-defra-search-agent:v1.0.0
    ports:
      - "8086:8086"
    environment:
      - ENVIRONMENT=production
      - DEBUG=false
    restart: always
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8086/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

---

## Docker Best Practices

### Health Checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD curl -f http://localhost:8086/health || exit 1
```

### Resource Limits

```yaml
services:
  agent:
    resources:
      limits:
        cpus: '1'
        memory: 512M
      reservations:
        cpus: '0.5'
        memory: 256M
```

### Restart Policy

```yaml
services:
  agent:
    restart: unless-stopped
```

---

## Troubleshooting Docker

### View Running Containers

```bash
docker ps

# Include stopped containers
docker ps -a
```

### View Container Logs

```bash
docker logs <container_id>

# Follow logs
docker logs -f <container_id>

# Last 100 lines
docker logs --tail=100 <container_id>
```

### Execute Commands in Container

```bash
docker exec -it <container_id> bash

# Run specific command
docker exec <container_id> curl http://localhost:8086/health
```

### Inspect Container

```bash
docker inspect <container_id>
```

---

**Version:** 1.0.0  
**Last Updated:** March 20, 2026
