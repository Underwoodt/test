# Common Issues & Solutions

**Troubleshooting guide for common problems**

---

## Services Won't Start

### Symptom
```
docker compose up -d
# Nothing starts or immediate exit
```

### Solution

```bash
# Check logs
docker compose logs

# Full restart
docker compose down
docker compose up -d --force-recreate

# Wait longer (2-3 minutes)
sleep 180

# Verify
docker compose ps
```

---

## Port Already in Use

### Symptom
```
docker: Error response from daemon: driver failed programming external connectivity on endpoint 
Bind for 0.0.0.0:3000 failed: port is already allocated
```

### Solution

```bash
# Find process using port
lsof -i :3000

# Kill process
kill -9 <PID>

# Or change port in docker-compose.yml
# ports:
#   - "3001:3000"

# Restart
docker compose up -d
```

---

## Can't Access Frontend

### Symptom
```
curl http://frontend.localhost
# Connection refused
```

### Solution

**Update /etc/hosts (macOS/Linux):**
```bash
127.0.0.1 frontend.localhost
127.0.0.1 knowledge.localhost
```

**Or test without .localhost:**
```bash
curl http://localhost:3000
```

---

## API Not Responding

### Symptom
```
curl http://localhost:8086/health
# Connection refused
```

### Solution

```bash
# Check if container is running
docker compose ps | grep agent

# View logs
docker compose logs agent

# Restart
docker compose restart agent

# Give it time to start
sleep 10

# Test health
curl http://localhost:8086/health
```

---

## Database Connection Error

### Symptom
```
ERROR: Could not connect to PostgreSQL
Error: connect ECONNREFUSED 127.0.0.1:5432
```

### Solution

```bash
# Check PostgreSQL is running
docker compose ps | grep postgres

# Check logs
docker compose logs postgres

# Verify network
docker network inspect ai-defra-search

# Restart database
docker compose restart postgres

# Wait for startup
sleep 10

# Test connection
docker compose exec postgres psql -U postgres -c "SELECT version();"
```

---

## Out of Disk Space

### Symptom
```
Docker: no space left on device
```

### Solution

```bash
# Check disk usage
docker system df

# Clean unused data
docker system prune

# Remove unused images
docker image prune -a

# Remove unused volumes
docker volume prune

# Check available space
df -h
```

---

## Memory Issues

### Symptom
```
Container killed due to memory limit
OOMKilled
```

### Solution

```bash
# Check memory usage
docker stats

# Increase limits in docker-compose.yml
# resources:
#   limits:
#     memory: 1G

# Restart
docker compose down
docker compose up -d
```

---

## Document Upload Fails

### Symptom
```
Upload fails silently or returns error
```

### Solution

```bash
# Check Knowledge service logs
docker compose logs knowledge

# Verify S3 connectivity
docker compose exec knowledge aws s3 ls \
  --endpoint-url http://localhost:4566

# Check file format is supported
# PDF, DOCX, PPTX, or JSONL expected

# Try with a different file
```

---

## Search Returns No Results

### Symptom
```
Search query returns []
```

### Solution

```bash
# Verify documents are uploaded
docker compose exec mongo mongosh
db.documents.find()

# Check vectors exist
docker compose exec postgres psql -U postgres
SELECT count(*) FROM chunks;

# Regenerate embeddings if needed
# Delete document and re-upload
```

---

## High Memory Usage

### Symptom
```
Services using too much memory
```

### Solution

```bash
# Check what's using memory
docker stats

# Restart memory-heavy service
docker compose restart knowledge

# Check for memory leaks in code
# Look at logs for patterns

# Increase Redis timeout
# Reduce cache TTL
```

---

## Network Issues

### Symptom
```
Services can't communicate
curl: (7) Failed to connect
```

### Solution

```bash
# Check docker network
docker network inspect ai-defra-search

# Verify service names
docker compose ps

# Try hostname instead of localhost
curl http://agent:8086/health

# Restart networking
docker network disconnect ai-defra-search <container>
docker network connect ai-defra-search <container>
```

---

## Performance Issues

### Symptom
```
Slow responses
High latency
```

### Solution

```bash
# Check resource usage
docker stats

# Check logs for errors
docker compose logs -f agent | grep -i error

# Monitor database queries
docker compose logs postgres | grep slow

# Restart services
docker compose restart

# Check network latency
ping localhost
```

---

## Duplicate Messages

### Symptom
```
Same message appears twice in chat
```

### Solution

```bash
# Clear conversation history
docker compose exec mongo mongosh
db.conversations.deleteMany({})

# Check for duplicate IDs in code
# Verify unique constraint on database

# Restart services
docker compose restart
```

---

**Version:** 1.0.0  
**Last Updated:** March 20, 2026
