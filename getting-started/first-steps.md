# First Steps: Your First Search Query

Get your first semantic search working in 10 minutes.

## Prerequisites

✅ Application is running on http://localhost:8085  
✅ PostgreSQL, MongoDB, LocalStack are healthy

Not set up? Go to [Quick Start](./README.md) first.

---

## Step 1: Create a Knowledge Group (2 min)

A **Knowledge Group** is a collection of documents owned by a user.

### Via Swagger UI

1. Open http://localhost:8085/docs
2. Find **POST /knowledge-groups** endpoint
3. Click "Try it out"
4. Enter request body:
   ```json
   {
     "name": "My First Knowledge Group"
   }
   ```
5. Enter header `user-id: alice` (username)
6. Click "Execute"

### Response

```json
{
  "id": "507f1f77bcf86cd799439011",
  "name": "My First Knowledge Group",
  "created_by": "alice",
  "created_at": "2024-03-19T12:00:00Z"
}
```

**Save the `id` — you'll need it next!**

### Via curl

```bash
curl -X POST http://localhost:8085/knowledge-groups \
  -H "user-id: alice" \
  -H "Content-Type: application/json" \
  -d '{"name": "My First Knowledge Group"}'
```

---

## Step 2: Get Sample Content (3 min)

For demo purposes, we'll create a sample JSON file with pre-chunked content.

### Create sample-content.jsonl

Create a file `sample-content.jsonl`:

```jsonl
{"text": "To reset your password, visit the account settings page. Click the 'Reset Password' button and follow the email verification link.", "source": "faq.pdf"}
{"text": "Password resets typically process within 24 hours. You'll receive a confirmation email when complete.", "source": "faq.pdf"}
{"text": "For security, passwords must be at least 12 characters long and contain uppercase, lowercase, numbers, and symbols.", "source": "security-guide.pdf"}
{"text": "Two-factor authentication adds an extra security layer. Enable it in your security settings.", "source": "security-guide.pdf"}
{"text": "Lost access to your 2FA device? Contact support@example.com with your account details for recovery options.", "source": "support.txt"}
```

### Upload to LocalStack S3

```bash
# Set up AWS CLI for LocalStack
aws configure --profile local
# Access Key: test
# Secret Key: test
# Region: eu-west-2

# Upload file
aws s3 --endpoint-url=http://localhost:4566 --profile local \
  cp sample-content.jsonl \
  s3://ai-defra-search-knowledge-upload-local/sample.jsonl
```

---

## Step 3: Register Document for Ingestion (2 min)

Tell the system to ingest the document from S3.

### Via Swagger UI

1. Find **POST /documents** endpoint
2. Click "Try it out"
3. Enter request body:
   ```json
   [
     {
       "file_name": "sample-content.jsonl",
       "knowledge_group_id": "507f1f77bcf86cd799439011",
       "cdp_upload_id": "batch-001",
       "s3_key": "s3://ai-defra-search-knowledge-upload-local/sample.jsonl"
     }
   ]
   ```
4. Click "Execute"

### Response

```
201 Created
```

Document is now being ingested asynchronously!

### Via curl

```bash
curl -X POST http://localhost:8085/documents \
  -H "Content-Type: application/json" \
  -d '[{
    "file_name": "sample-content.jsonl",
    "knowledge_group_id": "507f1f77bcf86cd799439011",
    "cdp_upload_id": "batch-001",
    "s3_key": "s3://ai-defra-search-knowledge-upload-local/sample.jsonl"
  }]'
```

---

## Step 4: Check Ingestion Status (2 min)

Wait for the document to be chunked and embedded.

### Via Swagger UI

1. Find **GET /upload-status/{upload_id}** endpoint
2. Click "Try it out"
3. Enter `upload_id`: `batch-001`
4. Click "Execute"

### Response

```json
[
  {
    "id": "507f1f77bcf86cd799439012",
    "file_name": "sample-content.jsonl",
    "status": "ready",
    "chunk_count": 5,
    "created_at": "2024-03-19T12:00:00Z"
  }
]
```

**Status = "ready"?** ✅ Document is searchable!

### Via curl

```bash
curl http://localhost:8085/upload-status/batch-001
```

---

## Step 5: Perform Semantic Search (1 min)

Search for information by meaning, not keywords.

### Via Swagger UI

1. Find **POST /rag/search** endpoint
2. Click "Try it out"
3. Enter header: `user-id: alice`
4. Enter request body:
   ```json
   {
     "query": "How do I reset my password?",
     "knowledge_group_ids": ["507f1f77bcf86cd799439011"],
     "max_results": 3
   }
   ```
5. Click "Execute"

### Response

```json
[
  {
    "content": "To reset your password, visit the account settings page. Click the 'Reset Password' button and follow the email verification link.",
    "similarity_score": 0.95,
    "document_id": "507f1f77bcf86cd799439012",
    "file_name": "sample-content.jsonl",
    "s3_key": "s3://ai-defra-search-knowledge-upload-local/sample.jsonl"
  },
  {
    "content": "Password resets typically process within 24 hours. You'll receive a confirmation email when complete.",
    "similarity_score": 0.91,
    "document_id": "507f1f77bcf86cd799439012",
    "file_name": "sample-content.jsonl",
    "s3_key": "s3://ai-defra-search-knowledge-upload-local/sample.jsonl"
  },
  {
    "content": "For security, passwords must be at least 12 characters long and contain uppercase, lowercase, numbers, and symbols.",
    "similarity_score": 0.73,
    "document_id": "507f1f77bcf86cd799439012",
    "file_name": "sample-content.jsonl",
    "s3_key": "s3://ai-defra-search-knowledge-upload-local/sample.jsonl"
  }
]
```

🎉 **You have your first search results!**

### Via curl

```bash
curl -X POST http://localhost:8085/rag/search \
  -H "user-id: alice" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "How do I reset my password?",
    "knowledge_group_ids": ["507f1f77bcf86cd799439011"],
    "max_results": 3
  }'
```

---

## Try More Queries

### Query 2: Security Question

```json
{
  "query": "How do I enable two-factor authentication?",
  "knowledge_group_ids": ["507f1f77bcf86cd799439011"],
  "max_results": 3
}
```

### Query 3: Lost Device

```json
{
  "query": "I lost my 2FA device, what do I do?",
  "knowledge_group_ids": ["507f1f77bcf86cd799439011"],
  "max_results": 3
}
```

### Query 4: Password Requirements

```json
{
  "query": "What are the password rules?",
  "knowledge_group_ids": ["507f1f77bcf86cd799439011"],
  "max_results": 3
}
```

Notice how the system understands synonyms:
- "rules" matches "must be"
- "enable" matches "adds"
- "lost device" matches "Lost access"

---

## Understand What Happened

### 1. Document Ingestion

```
sample-content.jsonl (5 chunks)
        ↓
Embedded as vectors (1024 dimensions each)
        ↓
Stored in PostgreSQL + metadata in MongoDB
```

### 2. Search Query

```
"How do I reset my password?"
        ↓
Converted to embedding (same Bedrock model)
        ↓
Compared to all chunks using cosine similarity
        ↓
Top 3 results returned, ranked by score
```

### 3. Score Interpretation

```
0.95 = Very similar (99% confidence)
0.91 = Similar (91% confidence)
0.73 = Somewhat related (73% confidence)
0.0  = Not related at all
```

---

## Next Steps

- **Upload Real Documents:** Use your own PDF/DOCX/PPTX files
- **Multiple Knowledge Groups:** Create another group and search both
- **Integration:** See how to use results in an LLM prompt
- **Development:** Read [Development Setup](../development/setup.md)

---

## Quick Troubleshooting

| Problem | Solution |
|---------|----------|
| Status still "in_progress" | Wait another 10 seconds, then check again |
| 0 results returned | Check knowledge_group_id matches upload |
| 404 error on search | Verify user-id header is set |
| Results don't match query | Try rephrasing the query |

---

## Try It in Python

```python
import httpx

async def search_documents():
    async with httpx.AsyncClient() as client:
        # Search
        response = await client.post(
            "http://localhost:8085/rag/search",
            json={
                "query": "How do I reset my password?",
                "knowledge_group_ids": ["507f1f77bcf86cd799439011"],
                "max_results": 3
            },
            headers={"user-id": "alice"}
        )
        
        results = response.json()
        
        # Print results
        for i, chunk in enumerate(results, 1):
            print(f"\n#{i} (Score: {chunk['similarity_score']:.2f})")
            print(f"Source: {chunk['file_name']}")
            print(f"Content: {chunk['content'][:100]}...")

# Run
asyncio.run(search_documents())
```

---

**Total Time:** 10 minutes  
**Next:** Try with your own documents!
