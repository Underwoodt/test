# Knowledge API Reference

**Complete API documentation for Search/Knowledge service**

---

## Base URL

**Development:** `http://knowledge.localhost` or `http://localhost:8085`  
**Production:** `https://knowledge.example.com`  

---

## Endpoints

### POST /documents

Register document for processing

**Request:**
```json
{
  "documents": [
    {
      "file_name": "grant.pdf",
      "knowledge_group_id": "group-123",
      "s3_key": "s3://bucket/uploads/grant.pdf"
    }
  ]
}
```

**Response (201 Created):**
```json
{
  "documents": [
    {
      "document_id": "doc-123",
      "status": "processing"
    }
  ]
}
```

---

### POST /rag/search

Semantic search

**Request:**
```json
{
  "query": "grant requirements",
  "knowledge_group_ids": ["group-123", "group-456"],
  "max_results": 5
}
```

**Response (200 OK):**
```json
[
  {
    "chunk_id": "chunk-1",
    "content": "Grant requirements: 1. Must be nonprofit 2. Min 501c3",
    "similarity_score": 0.92,
    "document_id": "doc-123",
    "file_name": "grant.pdf"
  },
  {
    "chunk_id": "chunk-5",
    "content": "Requirements also include financial documentation",
    "similarity_score": 0.88,
    "document_id": "doc-456",
    "file_name": "procedures.pdf"
  }
]
```

---

### GET /upload-status/{id}

Check document ingestion status

**Response (200 OK):**
```json
{
  "upload_id": "up-123",
  "document_id": "doc-123",
  "status": "processing",
  "chunks_processed": 5,
  "total_chunks": 10,
  "created_at": "2024-03-20T12:00:00Z",
  "completed_at": null
}
```

Statuses:
- `queued` - Waiting to process
- `processing` - Currently processing
- `ready` - Ready for search
- `failed` - Processing failed

---

### GET /health

Health check

**Response (200 OK):**
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "database": "connected"
}
```

---

## Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Created |
| 400 | Bad request |
| 404 | Not found |
| 500 | Server error |
| 503 | Service unavailable |

---

## Search Parameters

- **max_results:** 1-100 (default: 5)
- **similarity_threshold:** 0.0-1.0 (default: 0.0, no threshold)

---

## Examples

See [EXAMPLES.md](EXAMPLES.md)

---

**Version:** 1.0.0  
**Last Updated:** March 20, 2026
