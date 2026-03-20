# Examples - Code and Configuration Examples

**Ready-to-use examples for common tasks**

---

## What's in This Section?

| File | Purpose | Audience |
|------|---------|----------|
| **API-REQUESTS.md** | API request examples | Developers |
| **DOCKER-COMPOSE.md** | Docker Compose configurations | DevOps |
| **ENV-VARIABLES.md** | Environment variable examples | Everyone |

---

## Document Index

### API-REQUESTS.md (How to Call APIs)

**Contains:**
- Chat API examples
- Search API examples
- Python examples (simple & advanced)
- JavaScript examples (browser & Node.js)
- Complete workflows
- Error handling patterns
- Response examples

**Languages:**
- cURL
- Python
- JavaScript

**Examples included:**
- Simple message
- Multi-turn conversation
- Search queries
- With error handling
- Complete workflows

---

### DOCKER-COMPOSE.md (Docker Configurations)

**Contains:**
- Basic development setup
- With health checks
- Production configuration
- With resource limits
- Restart policies
- Logging configuration

**Use these as templates for:**
- Local development
- Staging environment
- Production deployment

---

### ENV-VARIABLES.md (Environment Configuration)

**Contains:**
- Development environment variables
- Production environment variables
- Kubernetes secrets
- Docker Compose override files
- All available variables documented
- What each variable does

**Examples for:**
- Development setup
- Production setup
- Kubernetes deployment

---

## Quick Start by Task

### I need to call the Chat API

→ Go to **API-REQUESTS.md** → Chat API Examples

```bash
curl -X POST http://localhost:8086/chat \
  -H "user-id: user@example.com" \
  -d '{"message":"Hello"}'
```

### I need to call the Search API

→ Go to **API-REQUESTS.md** → Search API Examples

```bash
curl -X POST http://knowledge.localhost/rag/search \
  -d '{"query":"requirements","max_results":5}'
```

### I need Python code to call APIs

→ Go to **API-REQUESTS.md** → Python Examples

```python
import requests
response = requests.post('http://localhost:8086/chat', ...)
```

### I need JavaScript code

→ Go to **API-REQUESTS.md** → JavaScript Examples

```javascript
fetch('http://localhost:8086/chat', {...})
```

### I need a Docker Compose file

→ Go to **DOCKER-COMPOSE.md**

```yaml
version: '3.8'
services:
  agent:
    image: ai-defra-search-agent:latest
    ports:
      - "8086:8086"
```

### I need environment variables

→ Go to **ENV-VARIABLES.md**

```bash
ENVIRONMENT=development
DEBUG=true
LOG_LEVEL=DEBUG
```

---

## Copy-Paste Ready

All examples in this section are copy-paste ready:
- Just copy and use
- Modify as needed for your situation
- Test before deploying

---

## Languages Covered

| Language | Files |
|----------|-------|
| Bash/cURL | API-REQUESTS.md |
| Python | API-REQUESTS.md |
| JavaScript | API-REQUESTS.md |
| YAML | DOCKER-COMPOSE.md |
| Shell | ENV-VARIABLES.md |

---

## By Task Type

### Call APIs
**File:** API-REQUESTS.md

- Send message
- Get conversation
- Search documents
- Upload documents
- Check status
- Error handling

### Configure Services
**File:** DOCKER-COMPOSE.md

- Development setup
- Health checks
- Production config
- Resource limits

### Set Configuration
**File:** ENV-VARIABLES.md

- Development vars
- Production vars
- Kubernetes secrets
- Docker overrides

---

## Complete Example Workflows

### Python Workflow

See: API-REQUESTS.md → Python Examples → Multi-request Workflow

```python
# 1. Send message
# 2. Get conversation
# 3. Search documents
```

### cURL Workflow

See: API-REQUESTS.md → Multi-Turn Conversation

```bash
# Message 1
# Message 2 (same conversation)
# Get conversation
```

---

## Testing Examples

### Test in Swagger UI
1. Go to http://localhost:8086/docs
2. Click "Try it out"
3. Fill in parameters
4. Click "Execute"

### Test with cURL
Copy example from API-REQUESTS.md and run

### Test with Python
Copy code from API-REQUESTS.md and run

### Test with JavaScript
Copy code from API-REQUESTS.md and run

---

## Customizing Examples

### For Your API URL
Replace:
- `http://localhost:8086` → your Agent URL
- `http://knowledge.localhost` → your Knowledge URL

### For Your User
Replace:
- `user@example.com` → your user email

### For Your Data
Replace:
- `"Hello"` → your message
- `"requirements"` → your search query

---

## Validating Examples

After copying an example:

1. ✅ Check syntax
2. ✅ Update URLs
3. ✅ Update user IDs
4. ✅ Run command
5. ✅ Verify response
6. ✅ Check for errors

---

## Next Steps

1. **Copy an example** from the appropriate file
2. **Customize** for your needs
3. **Test** to verify it works
4. **Integrate** into your application

---

**Document Version:** 1.0.0  
**Last Updated:** March 20, 2026
