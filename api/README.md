# API Overview

Complete reference for all REST API endpoints.

## Base URL

```
http://localhost:8085  # Local development
https://api.example.com # Production
```

## Interactive Documentation

View and test all endpoints:
- **Swagger UI:** `GET /docs`
- **ReDoc:** `GET /redoc`
- **OpenAPI JSON:** `GET /openapi.json`

## Common Headers

All requests (except health check) should include:

```bash
# User identification
-H "user-id: alice@example.com"

# Content type for POST/PUT
-H "Content-Type: application/json"
```

## Response Format

All responses are JSON:

```json
{
  "data": {...},      // Success: response body
  "error": null       // Success: null
}
```

Or error:

```json
{
  "detail": "Knowledge group not found"
}
```

HTTP Status Codes:
- **200** — Success
- **201** — Created
- **400** — Bad request (validation error)
- **404** — Not found or access denied
- **422** — Unprocessable entity (schema error)
- **500** — Server error
- **502** — Upstream failure (Bedrock, Postgres)

---

## Knowledge Groups

Organize documents into logical collections.

### Create Knowledge Group

```bash
POST /knowledge-groups
```

**Request:**
```json
{
  "name": "Customer FAQ"
}
```

**Response:** `201 Created`
```json
{
  "id": "507f1f77bcf86cd799439011",
  "name": "Customer FAQ",
  "created_by": "alice@example.com",
  "created_at": "2024-03-19T12:00:00Z"
}
```

### List Knowledge Groups

```bash
GET /knowledge-groups
```

**Response:** `200 OK`
```json
[
  {
    "id": "507f1f77bcf86cd799439011",
    "name": "Customer FAQ",
    "created_by": "alice@example.com",
    "created_at": "2024-03-19T12:00:00Z"
  },
  {
    "id": "507f1f77bcf86cd799439012",
    "name": "Policies",
    "created_by": "alice@example.com",
    "created_at": "2024-03-19T11:00:00Z"
  }
]
```

### Delete Knowledge Group

```bash
DELETE /knowledge-groups/{group_id}
```

**Response:** `204 No Content`

**Errors:**
- `404` — Group not found or not owned by user

---

## Documents

Upload and manage documents for ingestion.

### Get Supported File Types

```bash
GET /supported-file-types
```

**Response:** `200 OK`
```json
{
  "extensions": [".pdf", ".docx", ".pptx", ".jsonl"]
}
```

### Upload Documents

```bash
POST /documents
```

**Request:** (array of documents)
```json
[
  {
    "file_name": "faq.pdf",
    "knowledge_group_id": "507f1f77bcf86cd799439011",
    "cdp_upload_id": "batch-001",
    "s3_key": "uploads/faq.pdf"
  },
  {
    "file_name": "policies.docx",
    "knowledge_group_id": "507f1f77bcf86cd799439011",
    "cdp_upload_id": "batch-001",
    "s3_key": "uploads/policies.docx"
  }
]
```

**Response:** `201 Created`

**Errors:**
- `404` — Knowledge group not found
- `400` — Invalid file type

**Note:** Ingestion happens asynchronously in the background. Check status via `/upload-status/{upload_id}`.

### Check Upload Status

```bash
GET /upload-status/{upload_id}
```

**Response:** `200 OK`
```json
[
  {
    "id": "507f1f77bcf86cd799439013",
    "file_name": "faq.pdf",
    "status": "ready",  // or "not_started", "in_progress", "failed"
    "chunk_count": 250,
    "created_at": "2024-03-19T12:00:00Z"
  },
  {
    "id": "507f1f77bcf86cd799439014",
    "file_name": "policies.docx",
    "status": "in_progress",
    "chunk_count": null,
    "created_at": "2024-03-19T12:00:00Z"
  }
]
```

**Status values:**
- `not_started` — Queued, waiting to process
- `in_progress` — Extracting, chunking, embedding
- `ready` — Complete, searchable
- `failed` — Error during ingestion (check logs)

### List Documents in Knowledge Group

```bash
GET /documents?knowledge_group_id={group_id}
```

**Response:** `200 OK`
```json
[
  {
    "id": "507f1f77bcf86cd799439013",
    "file_name": "faq.pdf",
    "status": "ready",
    "chunk_count": 250,
    "knowledge_group_id": "507f1f77bcf86cd799439011",
    "created_at": "2024-03-19T12:00:00Z"
  }
]
```

**Errors:**
- `404` — Knowledge group not found or access denied
- `422` — Missing `knowledge_group_id` parameter

---

## RAG Search

Perform semantic search across knowledge groups.

### Search

```bash
POST /rag/search
```

**Request:**
```json
{
  "query": "How do I reset my password?",
  "knowledge_group_ids": ["507f1f77bcf86cd799439011"],
  "max_results": 5
}
```

**Response:** `200 OK`
```json
[
  {
    "content": "To reset your password, visit the account settings page and click 'Reset Password'...",
    "similarity_score": 0.92,
    "document_id": "507f1f77bcf86cd799439013",
    "file_name": "faq.pdf",
    "s3_key": "s3://bucket/faq.pdf"
  },
  {
    "content": "Password resets typically process within 24 hours...",
    "similarity_score": 0.87,
    "document_id": "507f1f77bcf86cd799439013",
    "file_name": "faq.pdf",
    "s3_key": "s3://bucket/faq.pdf"
  }
]
```

**Fields:**
- `content` — Text of the chunk
- `similarity_score` — 0 to 1 (higher = more relevant)
- `document_id` — ID of source document
- `file_name` — Original filename
- `s3_key` — S3 location of document

**Errors:**
- `404` — Knowledge group not found or not owned by user
- `502` — Bedrock embedding or Postgres search failed
- `422` — Invalid request (missing fields, invalid types)

**Performance:**
- Typical latency: 100-500ms
- Supported knowledge groups: Unlimited
- Max results: Configurable, default 10

---

## Health & Status

### Health Check

```bash
GET /health
```

**Response:** `200 OK`
```json
{
  "status": "healthy",
  "timestamp": "2024-03-19T12:00:00Z"
}
```

**Use for:** Kubernetes probes, load balancer checks.

---

## Example Workflows

### Complete Document Upload & Search Flow

```bash
# 1. Create knowledge group
GROUP_ID=$(curl -s -X POST http://localhost:8085/knowledge-groups \
  -H "user-id: alice" \
  -H "Content-Type: application/json" \
  -d '{"name": "FAQ"}' | jq -r '.id')

# 2. Upload documents
UPLOAD_ID="batch-001"
curl -X POST http://localhost:8085/documents \
  -H "Content-Type: application/json" \
  -d "[{
    \"file_name\": \"faq.pdf\",
    \"knowledge_group_id\": \"$GROUP_ID\",
    \"cdp_upload_id\": \"$UPLOAD_ID\",
    \"s3_key\": \"uploads/faq.pdf\"
  }]"

# 3. Poll status (wait for "ready")
curl http://localhost:8085/upload-status/$UPLOAD_ID \
  | jq '.[0].status'

# 4. Search
curl -X POST http://localhost:8085/rag/search \
  -H "user-id: alice" \
  -H "Content-Type: application/json" \
  -d "{
    \"query\": \"How do I reset my password?\",
    \"knowledge_group_ids\": [\"$GROUP_ID\"],
    \"max_results\": 5
  }"
```

### Searching Multiple Knowledge Groups

```bash
curl -X POST http://localhost:8085/rag/search \
  -H "user-id: alice" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "payment methods",
    "knowledge_group_ids": [
      "507f1f77bcf86cd799439011",  // FAQ
      "507f1f77bcf86cd799439012"   // Policies
    ],
    "max_results": 10
  }'
```

### Using in Python

```python
import httpx
import json

async with httpx.AsyncClient() as client:
    # Search
    response = await client.post(
        "http://localhost:8085/rag/search",
        json={
            "query": "How do I...?",
            "knowledge_group_ids": ["group-123"],
            "max_results": 5
        },
        headers={"user-id": "alice"}
    )
    
    results = response.json()
    for chunk in results:
        print(f"Score: {chunk['similarity_score']}")
        print(f"Content: {chunk['content']}\n")
```

---

## Rate Limiting & Quotas

| Resource | Limit | Reset |
|----------|-------|-------|
| Requests per minute | Unlimited (implement via gateway) | N/A |
| Document size | 500 MB | Per upload |
| Knowledge groups | Unlimited | Per user |
| Chunks per document | 10,000 max | Per document |

---

## Deprecation Policy

API changes follow semantic versioning:
- **Major:** Breaking changes (v2.0.0)
- **Minor:** New features, backward compatible (v1.1.0)
- **Patch:** Fixes, no changes (v1.0.1)

Deprecated endpoints flagged in API docs 1 version before removal.

---

## See Also

- **[Knowledge Groups API](./knowledge-groups.md)** — Detailed reference
- **[Documents API](./documents.md)** — Detailed reference
- **[RAG Search API](./rag-search.md)** — Detailed reference
- **[Functional Overview](../concepts/functional-overview.md)** — How it all works

---

**Last Updated:** March 19, 2026  
**API Version:** 1.0.0
