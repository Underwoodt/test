# Verification - Confirm Setup Works

**Checklist to verify all systems operational**

---

## Quick Health Check

### 1. Docker Containers Running

```bash
docker compose ps
```

Expected output - all running:
```
CONTAINER ID        IMAGE                STATUS              PORTS
...
ai-defra-search-frontend        Up 2 minutes
ai-defra-search-agent           Up 2 minutes
ai-defra-search-knowledge       Up 2 minutes
postgres                        Up 2 minutes
mongo                          Up 2 minutes
redis                          Up 2 minutes
localstack                     Up 2 minutes
traefik                        Up 2 minutes
```

### 2. Frontend Accessible

```bash
curl http://frontend.localhost -I
```

Expected response:
```
HTTP/1.1 200 OK
Content-Type: text/html
```

### 3. Agent API Healthy

```bash
curl http://localhost:8086/health
```

Expected response:
```json
{"status":"healthy"}
```

### 4. Knowledge API Healthy

```bash
curl http://knowledge.localhost/health
```

Expected response:
```json
{"status":"healthy"}
```

---

## Full Verification Checklist

### ✅ Services Section

```bash
# Frontend health
curl -s http://frontend.localhost | grep -c "html" && echo "✓ Frontend"

# Agent health
curl -s http://localhost:8086/health | jq . && echo "✓ Agent"

# Knowledge health
curl -s http://knowledge.localhost/health | jq . && echo "✓ Knowledge"

# Bedrock stub
curl -s http://bedrock:4566 && echo "✓ Bedrock"
```

### ✅ Database Section

```bash
# PostgreSQL
docker compose exec postgres psql -U postgres -c "SELECT version();" && echo "✓ PostgreSQL"

# MongoDB
docker compose exec mongo mongosh --eval "db.version()" && echo "✓ MongoDB"

# Redis
docker compose exec redis redis-cli ping | grep PONG && echo "✓ Redis"

# pgvector extension
docker compose exec postgres psql -U postgres -c "CREATE EXTENSION IF NOT EXISTS vector; SELECT * FROM pg_extension WHERE extname = 'vector';" && echo "✓ pgvector"
```

### ✅ Network Section

```bash
# DNS resolution
nslookup frontend.localhost

# Port accessibility
netstat -an | grep -E "3000|8085|8086|5432|27017|6379" && echo "✓ All ports open"
```

### ✅ Storage Section

```bash
# S3 bucket creation (LocalStack)
aws s3 ls --endpoint-url http://localhost:4566 && echo "✓ S3 available"
```

---

## API Testing

### Test Chat API

```bash
# Make a test request to Agent
curl -X POST http://localhost:8086/chat \
  -H "Content-Type: application/json" \
  -H "user-id: test@example.com" \
  -d '{
    "message": "Hello",
    "conversation_id": "test-conv-1",
    "user_id": "test@example.com"
  }'
```

Expected response:
```json
{
  "response": "...",
  "conversation_id": "test-conv-1",
  "sources": []
}
```

### Test Search API

```bash
# Make a test search request
curl -X POST http://knowledge.localhost/rag/search \
  -H "Content-Type: application/json" \
  -d '{
    "query": "test query",
    "knowledge_group_ids": [],
    "max_results": 5
  }'
```

Expected response:
```json
[]
```
(Empty array because no documents uploaded yet)

---

## Browser Verification

Open these URLs in your browser to verify:

1. **Frontend:** http://frontend.localhost
   - ✅ See chat interface
   - ✅ See upload button
   - ✅ No errors in console

2. **Agent Swagger:** http://localhost:8086/docs
   - ✅ See interactive API documentation
   - ✅ Can expand endpoints
   - ✅ Can see request/response schemas

3. **Knowledge Swagger:** http://knowledge.localhost/docs
   - ✅ See search and document endpoints
   - ✅ Can expand POST /rag/search
   - ✅ Can see parameters

4. **Traefik Dashboard:** http://localhost:8080
   - ✅ See services listed
   - ✅ See routes configured
   - ✅ All green/healthy

---

## Log Verification

### View Startup Logs

```bash
# See full startup sequence
docker compose logs

# Check for errors
docker compose logs | grep -i error

# Check specific service
docker compose logs agent | tail -50
docker compose logs knowledge | tail -50
```

### Look for Success Messages

```bash
# Agent startup complete
docker compose logs agent | grep "Application startup complete"

# Knowledge startup complete
docker compose logs knowledge | grep "Application startup complete"

# Database connections established
docker compose logs postgres | grep "ready to accept connections"
```

---

## End-to-End Test

### Upload a Test Document

1. Go to http://frontend.localhost
2. Click "Upload Document"
3. Select any PDF, DOCX, or text file
4. See "Upload complete" message

### Make a Search

1. After upload completes, type in chat: "search my document"
2. Should get a response

---

## Status Summary

Create a status report:

```bash
#!/bin/bash

echo "=== AI DEFRA Search System Status ==="
echo ""
echo "Services:"
docker compose ps | grep -E "Up|Down|Exit"

echo ""
echo "Container count:"
docker ps -q | wc -l

echo ""
echo "Disk usage:"
docker system df

echo ""
echo "Network connectivity:"
curl -s http://frontend.localhost > /dev/null && echo "✓ Frontend" || echo "✗ Frontend"
curl -s http://localhost:8086/health | jq . > /dev/null && echo "✓ Agent" || echo "✗ Agent"
curl -s http://knowledge.localhost/health | jq . > /dev/null && echo "✓ Knowledge" || echo "✗ Knowledge"

echo ""
echo "=== Setup Verified ==="
```

---

## Troubleshooting Verification Issues

### Service not responding
```bash
# Check logs
docker compose logs <service_name>

# Restart service
docker compose restart <service_name>

# Full restart
docker compose down
docker compose up -d
```

### Database not initialized
```bash
# Check database logs
docker compose logs postgres
docker compose logs mongo

# Reset and restart
docker compose down -v
docker compose up -d

# Wait 2-3 minutes for initialization
```

### Port conflicts
```bash
# Find using process
lsof -i :<port>

# Kill process
kill -9 <PID>

# Restart service
docker compose restart <service>
```

---

## Next Steps

Once verified:
1. Start using the system
2. Read [Development Workflow](../setup/)
3. Explore [API Reference](../api-reference/)
4. Review [Architecture](../architecture/)

---

**Document Version:** 1.0.0  
**Last Updated:** March 20, 2026
