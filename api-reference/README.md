# API Reference - Complete API Documentation

**All endpoints and examples for the AI DEFRA Search platform**

---

## What's in This Section?

| File | Purpose | Audience |
|------|---------|----------|
| **AGENT-API.md** | Chat API endpoints | Developers |
| **KNOWLEDGE-API.md** | Search API endpoints | Developers |
| **EXAMPLES.md** | Code examples | Everyone |

---

## Quick Reference

### Chat API
**Base URL:** `http://localhost:8086`  
**Main endpoint:** `POST /chat`

```bash
curl -X POST http://localhost:8086/chat \
  -H "user-id: user@example.com" \
  -d '{"message":"Hello","conversation_id":"conv-1"}'
```

### Search API
**Base URL:** `http://knowledge.localhost` or `http://localhost:8085`  
**Main endpoint:** `POST /rag/search`

```bash
curl -X POST http://knowledge.localhost/rag/search \
  -d '{"query":"requirements","max_results":5}'
```

---

## Document Index

### AGENT-API.md (Chat Service)

**Endpoints:**
- `POST /chat` - Send message
- `GET /conversations/{id}` - Get conversation history
- `GET /health` - Health check

**Response format:**
```json
{
  "response": "...",
  "conversation_id": "conv-1",
  "sources": [...],
  "metadata": {...}
}
```

**Use when:**
- Sending chat messages
- Getting conversation history
- Checking chat service health

---

### KNOWLEDGE-API.md (Search Service)

**Endpoints:**
- `POST /documents` - Upload documents
- `POST /rag/search` - Semantic search
- `GET /upload-status/{id}` - Check upload progress
- `GET /health` - Health check

**Response format:**
```json
[
  {
    "chunk_id": "chunk-1",
    "content": "...",
    "similarity_score": 0.92,
    "document_id": "doc-1",
    "file_name": "doc.pdf"
  }
]
```

**Use when:**
- Uploading documents
- Searching documents
- Checking upload status

---

### EXAMPLES.md (Code Examples)

**Includes:**
- cURL examples (all endpoints)
- Python examples (simple & advanced)
- JavaScript examples (browser & Node.js)
- Complete workflows
- Error handling
- Response examples

**Languages covered:**
- Bash/cURL
- Python
- JavaScript

---

## API Overview

```
Your Application
  ↓
Chat API (Agent)
  ├─ POST /chat
  ├─ GET /conversations/{id}
  └─ GET /health

Search API (Knowledge)
  ├─ POST /documents
  ├─ POST /rag/search
  ├─ GET /upload-status/{id}
  └─ GET /health
```

---

## Common Tasks

### Send a Chat Message

**See:** AGENT-API.md → POST /chat  
**Example:** EXAMPLES.md → Chat API Examples

```bash
curl -X POST http://localhost:8086/chat \
  -H "user-id: user@example.com" \
  -d '{
    "message":"What are grant requirements?",
    "conversation_id":"conv-001",
    "user_id":"user@example.com"
  }'
```

### Search Documents

**See:** KNOWLEDGE-API.md → POST /rag/search  
**Example:** EXAMPLES.md → Search API Examples

```bash
curl -X POST http://knowledge.localhost/rag/search \
  -d '{
    "query":"grant requirements",
    "knowledge_group_ids":[],
    "max_results":5
  }'
```

### Upload Document

**See:** KNOWLEDGE-API.md → POST /documents  
**Example:** EXAMPLES.md → Upload Document

```bash
curl -X POST http://knowledge.localhost/documents \
  -d '{
    "documents":[{
      "file_name":"grant.pdf",
      "knowledge_group_id":"group-1",
      "s3_key":"s3://bucket/grant.pdf"
    }]
  }'
```

### Check Upload Status

**See:** KNOWLEDGE-API.md → GET /upload-status/{id}

```bash
curl http://knowledge.localhost/upload-status/up-123
```

### Get Conversation History

**See:** AGENT-API.md → GET /conversations/{id}

```bash
curl http://localhost:8086/conversations/conv-001 \
  -H "user-id: user@example.com"
```

---

## Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Created (POST successful) |
| 400 | Bad request (invalid parameters) |
| 401 | Unauthorized (missing auth) |
| 404 | Not found (resource doesn't exist) |
| 500 | Server error |
| 503 | Service unavailable (temporary issue) |

---

## Authentication

All requests require:
```
Headers:
  user-id: user@example.com
```

---

## Rate Limiting

**Default limits:**
- 60 requests per minute per user
- 1000 requests per hour per user

**Headers returned:**
```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1711000000
```

---

## Timeouts

| Operation | Timeout |
|-----------|---------|
| Chat response | 30 seconds |
| Search | 10 seconds |
| Upload status | 10 seconds |

---

## Error Response Format

```json
{
  "error": "Error message",
  "status": 400,
  "details": {
    "field": "message",
    "reason": "required"
  }
}
```

---

## Testing APIs

### Using Swagger UI

**Agent:** http://localhost:8086/docs  
**Knowledge:** http://knowledge.localhost/docs

Interactive documentation with "Try it out" button

### Using cURL

See EXAMPLES.md for cURL commands

### Using Python

See EXAMPLES.md → Python Examples

### Using JavaScript

See EXAMPLES.md → JavaScript Examples

---

## API Development Workflow

1. **Read the docs:**
   - AGENT-API.md or KNOWLEDGE-API.md

2. **Check examples:**
   - EXAMPLES.md for your language

3. **Test in Swagger UI:**
   - /docs endpoint

4. **Implement in your code:**
   - Use provided examples as template

5. **Debug if needed:**
   - See [troubleshooting/](../troubleshooting/)

---

## Next Steps

- **Use examples:** Go to EXAMPLES.md
- **Deploy:** Go to [deployment/](../deployment/)
- **Troubleshoot:** Go to [troubleshooting/](../troubleshooting/)

---

**Document Version:** 1.0.0  
**Last Updated:** March 20, 2026
