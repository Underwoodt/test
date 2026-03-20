# Agent API Reference

**Complete API documentation for Chat/Agent service**

---

## Base URL

**Development:** `http://localhost:8086`  
**Production:** `https://agent.example.com`  

---

## Authentication

All requests require:
```
Headers:
  user-id: user@example.com
```

---

## Endpoints

### POST /chat

Send a chat message

**Request:**
```json
{
  "message": "What are grant requirements?",
  "conversation_id": "conv-123",
  "user_id": "user@example.com"
}
```

**Response (200 OK):**
```json
{
  "response": "Grant requirements are...",
  "conversation_id": "conv-123",
  "sources": [
    {
      "document_id": "doc-123",
      "file_name": "grant.pdf",
      "similarity_score": 0.92
    }
  ],
  "metadata": {
    "tokens_used": 150,
    "search_results": 5,
    "latency_ms": 1500
  }
}
```

**Error (503 Service Unavailable):**
```json
{
  "error": "Service unavailable",
  "message": "Knowledge service is down"
}
```

---

### GET /conversations/{id}

Get conversation history

**Parameters:**
- `id` (string, path) - Conversation ID

**Response (200 OK):**
```json
{
  "conversation_id": "conv-123",
  "user_id": "user@example.com",
  "created_at": "2024-03-20T12:00:00Z",
  "messages": [
    {
      "role": "user",
      "content": "What are grant requirements?",
      "timestamp": "2024-03-20T12:00:00Z"
    },
    {
      "role": "assistant",
      "content": "Grant requirements are...",
      "timestamp": "2024-03-20T12:00:05Z",
      "sources": [...]
    }
  ]
}
```

---

### GET /health

Health check

**Response (200 OK):**
```json
{
  "status": "healthy",
  "version": "1.0.0"
}
```

---

## Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 400 | Bad request |
| 401 | Unauthorized (missing user-id) |
| 404 | Not found |
| 500 | Server error |
| 503 | Service unavailable |

---

## Rate Limiting

- **Per minute:** 60 requests
- **Per hour:** 1000 requests
- **Per user:** Based on plan

---

## Timeouts

- **Response timeout:** 30 seconds
- **Read timeout:** 60 seconds

---

## Examples

See [EXAMPLES.md](EXAMPLES.md)

---

**Version:** 1.0.0  
**Last Updated:** March 20, 2026
