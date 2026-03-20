# Understanding Logs

**How to read and interpret logs**

---

## Log Levels

| Level | Meaning | Examples |
|-------|---------|----------|
| DEBUG | Detailed info | Variable values, function calls |
| INFO | General info | Service started, request received |
| WARNING | Potential issues | Deprecated API, unusual values |
| ERROR | Problems | Failed requests, exceptions |
| CRITICAL | Severe issues | Service down, data loss |

---

## Log Format

### Standard Format
```
[2024-03-20 12:00:00] [SERVICE] [LEVEL] Message details
```

### FastAPI Format
```
2024-03-20T12:00:00.000Z INFO - "POST /chat HTTP/1.1" 200
```

### Docker Compose Format
```
agent_1  | 2024-03-20 12:00:00 INFO Request from user@example.com
```

---

## Viewing Logs

### View all logs

```bash
docker compose logs
```

### Follow logs (real-time)

```bash
docker compose logs -f
```

### Last 100 lines

```bash
docker compose logs --tail=100
```

### Specific service

```bash
docker compose logs agent
docker compose logs knowledge
docker compose logs postgres
```

### Since timestamp

```bash
docker compose logs --since 2024-03-20T12:00:00
```

---

## Common Log Messages

### Service Starting

```
agent_1 | INFO: Uvicorn running on http://0.0.0.0:8086
```
✅ Service is starting normally

### Database Connected

```
postgres_1 | LOG: database system is ready to accept connections
```
✅ Database initialized

### Search Working

```
knowledge_1 | INFO: Search query processed, found 5 results
```
✅ Search completed

### Error Messages

```
agent_1 | ERROR: Knowledge service unavailable
```
❌ Service connectivity issue

---

## Debugging Log Patterns

### Look for patterns

```bash
# Find all errors
docker compose logs | grep ERROR

# Find slow queries
docker compose logs postgres | grep "duration"

# Find connection issues
docker compose logs | grep -i "connection refused"

# Find timeouts
docker compose logs | grep -i "timeout"
```

---

## Log Analysis

### Count occurrences

```bash
docker compose logs | grep "ERROR" | wc -l
```

### Find slow requests

```bash
docker compose logs agent | grep "duration.*ms" | grep -E "[0-9]{4,}"
```

### Find specific user

```bash
docker compose logs | grep "user@example.com"
```

---

## Storing Logs

### Save to file

```bash
docker compose logs > all-logs.txt
docker compose logs agent > agent-logs.txt
```

### Real-time log recording

```bash
docker compose logs -f > logs.txt &
# Press Ctrl+C to stop
```

---

## Production Logging

### Consider using ELK Stack

- **Elasticsearch** - Log storage
- **Logstash** - Log processing
- **Kibana** - Log visualization

### Configuration

```yaml
services:
  agent:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

---

**Version:** 1.0.0  
**Last Updated:** March 20, 2026
