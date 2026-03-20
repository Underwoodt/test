# Troubleshooting - Solving Problems

**Help when something goes wrong**

---

## What's in This Section?

| File | Purpose | Best For |
|------|---------|----------|
| **COMMON-ISSUES.md** | Solutions to known problems | Quick fixes |
| **DEBUGGING.md** | How to debug issues | Diagnosing problems |
| **LOGS.md** | Understanding logs | Finding root cause |

---

## Quick Troubleshooting

### Problem → Solution Quick Lookup

| Problem | See |
|---------|-----|
| Docker won't start | COMMON-ISSUES.md |
| Port already in use | COMMON-ISSUES.md |
| API not responding | COMMON-ISSUES.md |
| Database error | COMMON-ISSUES.md |
| Out of disk space | COMMON-ISSUES.md |
| Memory issues | COMMON-ISSUES.md |
| Slow performance | COMMON-ISSUES.md |
| Search returns no results | COMMON-ISSUES.md |
| Can't figure it out | DEBUGGING.md |
| Seeing errors in logs | LOGS.md |

---

## Document Index

### COMMON-ISSUES.md (Known Problems & Solutions)

**Contains:**
- 12+ common issues with solutions
- Symptoms you'll see
- Step-by-step solutions
- Commands to run
- What to check next

**Issues covered:**
- Services won't start
- Port already in use
- Can't access frontend
- API not responding
- Database connection errors
- Out of disk space
- Memory issues
- Document upload fails
- Search returns no results
- High memory usage
- Network issues
- Performance issues
- Duplicate messages

**Best for:** "I have this problem, fix it now"

---

### DEBUGGING.md (How to Debug)

**Contains:**
- How to enable debug logging
- Inspecting running containers
- Database debugging (PostgreSQL, MongoDB)
- Network debugging
- Performance debugging (CPU, memory)
- API debugging with cURL
- Log analysis techniques
- Health check procedures
- Service inspection
- Test script examples

**Best for:** "I need to figure out what's wrong"

**Techniques:**
- Enable debug logging
- Connect to containers
- Query databases
- Test endpoints
- Monitor performance
- Analyze logs

---

### LOGS.md (Understanding Logs)

**Contains:**
- Log levels explained
- Log format examples
- How to view logs
- How to follow logs in real-time
- Common log messages explained
- Log pattern analysis
- Debugging by log pattern
- Log storage for production

**Best for:** "What do these log messages mean?"

**Covers:**
- Log levels (DEBUG, INFO, WARNING, ERROR, CRITICAL)
- Reading log formats
- Viewing logs
- Following logs
- Interpreting messages
- Finding patterns
- Storing logs long-term

---

## Troubleshooting Flowchart

```
Something's wrong
  ↓
Can you describe the problem?
  ├─ YES → Go to COMMON-ISSUES.md
  │        Find matching symptom
  │        Follow solution steps
  │
  └─ NO → Go to DEBUGGING.md
           Enable debug logging
           Inspect containers
           Check health
           Look at logs

Still stuck?
  ↓
Go to LOGS.md
  ├─ Find log patterns
  ├─ Understand error messages
  ├─ Look for root cause
  └─ Try solutions
```

---

## Troubleshooting Workflow

### Step 1: Describe the Problem
- What were you doing?
- What error did you see?
- What were you expecting?

### Step 2: Check COMMON-ISSUES.md
- Look for matching symptom
- Follow solution steps
- Try suggested fixes

### Step 3: Still Stuck? → Debugging
- Enable debug logging
- Inspect containers
- Run health checks
- Check network connectivity

### Step 4: Read the Logs
- Find error messages
- Look for patterns
- Understand what's happening
- Identify root cause

### Step 5: Fix and Test
- Apply fix
- Run verification
- Check logs confirm success
- Verify system works

---

## Most Common Issues (Priority Order)

1. **Docker won't start** → COMMON-ISSUES.md
2. **Port already in use** → COMMON-ISSUES.md
3. **Database connection error** → COMMON-ISSUES.md
4. **Out of disk space** → COMMON-ISSUES.md
5. **API not responding** → COMMON-ISSUES.md

---

## Debugging Techniques

### Enable Debug Logging
```bash
# Set LOG_LEVEL
export LOG_LEVEL=DEBUG

# Check logs
docker compose logs -f
```

### Connect to Container
```bash
docker exec -it <container> bash

# Test connectivity
curl http://service:port/health
```

### Query Database
```bash
# PostgreSQL
docker compose exec postgres psql -U postgres

# MongoDB
docker compose exec mongo mongosh

# Redis
docker compose exec redis redis-cli
```

### Test API
```bash
# Verbose cURL
curl -vv http://localhost:8086/health

# With timeout
timeout 5 curl http://localhost:8086/health
```

---

## Emergency Procedures

### If Everything is Broken
```bash
# Hard reset
docker compose down -v
docker compose up -d --force-recreate

# Wait for initialization (2-3 minutes)
sleep 180

# Verify
docker compose ps
curl http://localhost:8086/health
```

### If Out of Disk Space
```bash
# Clean up
docker system prune
docker image prune -a
docker volume prune

# Check space
df -h
```

### If Running Out of Memory
```bash
# Check usage
docker stats

# Stop heavy service
docker compose stop knowledge

# Increase memory in docker-compose.yml
# Restart
docker compose up -d
```

---

## Getting Help

### Before Asking for Help

1. ✅ Read COMMON-ISSUES.md
2. ✅ Enable debug logging
3. ✅ Check logs with LOGS.md
4. ✅ Run verification commands
5. ✅ Save all logs and commands you ran

### What to Share

- Docker version: `docker --version`
- Docker Compose version: `docker compose --version`
- Python version: `python3 --version`
- All logs from: `docker compose logs`
- Error messages: Copy/paste exactly
- Steps to reproduce: Exact commands you ran

---

## Preventive Measures

### Regular Maintenance

```bash
# Weekly
docker system prune
docker image prune -a

# Monthly
docker volume prune
Review logs for warnings

# Before deployment
Run full verification checklist
Check all health endpoints
```

---

## Reference

- **COMMON-ISSUES.md** - Known issues & fixes
- **DEBUGGING.md** - How to debug
- **LOGS.md** - Understanding logs

---

**Document Version:** 1.0.0  
**Last Updated:** March 20, 2026
