# Debugging Guide

**How to debug issues**

---

## Enable Debug Logging

### Python Services

```bash
# In env or docker-compose.yml
LOG_LEVEL=DEBUG

# Then check logs
docker compose logs -f agent
```

### Frontend

```bash
# Open browser console
F12

# Check for errors
# Network tab for API calls
```

---

## Inspect Running Containers

```bash
# Get container ID
docker compose ps

# Execute bash in container
docker exec -it <container_id> bash

# Install debugging tools
apt-get update && apt-get install -y curl jq

# Test connectivity
curl http://agent:8086/health
```

---

## Database Debugging

### PostgreSQL

```bash
# Connect to database
docker compose exec postgres psql -U postgres

# List tables
\dt

# Check chunks table
SELECT count(*) FROM chunks;

# Check specific query
EXPLAIN ANALYZE
SELECT * FROM chunks WHERE embedding <-> '[0.1,0.2,...]' LIMIT 5;
```

### MongoDB

```bash
# Connect to database
docker compose exec mongo mongosh

# List databases
show databases

# Check conversations
db.conversations.find().pretty()

# Count documents
db.documents.countDocuments()
```

---

## Network Debugging

```bash
# Check DNS resolution
docker compose exec <service> nslookup agent

# Test connectivity between services
docker compose exec agent curl http://knowledge:8085/health

# Monitor network traffic
docker exec <container> tcpdump -i eth0 port 8085
```

---

## Performance Debugging

### CPU Profiling

```bash
# Monitor in real-time
docker stats

# Get specific container stats
docker stats <container_id> --no-stream
```

### Memory Debugging

```bash
# Check memory usage
docker inspect <container_id> | grep Memory

# Monitor over time
docker stats --no-stream | grep <service>
```

---

## API Debugging

### cURL with Verbose

```bash
# See full request/response
curl -vv http://localhost:8086/health

# With data
curl -vv -X POST http://localhost:8086/chat \
  -H "Content-Type: application/json" \
  -d '{"message":"hello"}'
```

### Test with timeout

```bash
# Set timeout to catch hanging requests
timeout 5 curl http://localhost:8086/health

# If timeout: request is hanging
```

---

## Log Analysis

### Search logs for errors

```bash
docker compose logs | grep -i error
docker compose logs | grep -i exception
docker compose logs | grep -i failed
```

### Get last N lines

```bash
docker compose logs --tail=100 agent
```

### Search specific container

```bash
docker compose logs knowledge | grep "ERROR"
```

---

## Health Checks

### Manual health checks

```bash
# Agent health
curl http://localhost:8086/health
echo $?  # Should be 0

# Knowledge health
curl http://knowledge.localhost/health
echo $?

# Database health
docker compose exec postgres pg_isready
docker compose exec mongo mongosh --eval "db.adminCommand('ping')"
docker compose exec redis redis-cli ping
```

---

## Inspecting Services

### View environment variables

```bash
docker inspect <container_id> | grep -A 20 "Env"
```

### View volume mounts

```bash
docker inspect <container_id> | grep -A 10 "Mounts"
```

### View networking

```bash
docker inspect <container_id> | grep -A 10 "NetworkSettings"
```

---

## Testing API Endpoints

### Create test script

```bash
#!/bin/bash

# Test endpoints
echo "Testing Agent API..."
curl -w "\n" http://localhost:8086/health

echo "Testing Knowledge API..."
curl -w "\n" http://knowledge.localhost/health

echo "Testing database..."
docker compose exec postgres psql -U postgres -c "SELECT version();"
```

---

**Version:** 1.0.0  
**Last Updated:** March 20, 2026
