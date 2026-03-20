# Functional Overview

## What Is AI DEFRA Search Knowledge?

A **Retrieval-Augmented Generation (RAG) platform** that lets users:

1. **Upload documents** (PDF, Word, PowerPoint, JSONL)
2. **Search semantically** across those documents
3. **Get grounded answers** from actual document content

Instead of an LLM hallucinating, it retrieves real document chunks and returns them ranked by relevance.

## Why Does It Matter?

### The Problem
> Large Language Models sometimes make up answers (hallucinate) when they don't know something.

### The Solution
> Search real documents first, then use those documents to augment the LLM prompt.

**Result:** Factual, verifiable answers with sources.

## How It Works: The RAG Pipeline

```
User Query
    ↓ [Convert to vector via AWS Bedrock]
Vector Embedding
    ↓ [Search PostgreSQL with pgvector]
Top-K Similar Chunks
    ↓ [Enrich with metadata from MongoDB]
Ranked Results with Source References
    ↓
User gets grounded answer
```

## Core Components

### 1. Knowledge Groups
**What:** Organize documents by user

```
User "alice" owns:
├── Knowledge Group "FAQ" (5 documents, 1,250 chunks)
└── Knowledge Group "Policies" (3 documents, 890 chunks)

User "bob" owns:
└── Knowledge Group "Training" (2 documents, 450 chunks)
```

**Key Feature:** Users only see their own groups (data isolation)

### 2. Document Ingestion
**What:** Extract, chunk, and embed documents

```
Upload file.pdf
    ↓
Extract text from PDF
    ↓
Split into 200-token chunks (semantic boundaries)
    ↓
Generate embeddings (AWS Bedrock Titan)
    ↓
Store chunks + vectors in PostgreSQL
    ↓
Ready for search!
```

**Supported formats:**
- **PDF** — Text extraction via PyMuPDF
- **DOCX** — Word document parsing
- **PPTX** — PowerPoint slide extraction
- **JSONL** — Pre-chunked JSON lines (fastest)

### 3. Semantic Search
**What:** Find relevant chunks using vector similarity

```
Query: "How do I reset my password?"
    ↓
Embed query using same Bedrock model
    ↓
Calculate cosine distance to all chunks
    ↓
Return top-5 closest matches (sorted by similarity)
    ↓
Enrich with document names and links
```

**Why it works:**
- Similar meanings = similar vectors
- No keyword matching needed
- Handles synonyms and paraphrasing

## Data Model

### Knowledge Group
```json
{
  "_id": "ObjectID",
  "name": "FAQ Documents",
  "created_by": "user@example.com",
  "created_at": "2024-03-19T12:00:00Z"
}
```

### Document
```json
{
  "_id": "ObjectID",
  "file_name": "faq.pdf",
  "knowledge_group_id": "...",
  "status": "ready",
  "chunk_count": 250,
  "created_at": "2024-03-19T12:00:00Z"
}
```

### Knowledge Vector (Chunk)
```json
{
  "content": "To reset your password, visit...",
  "embedding": [0.123, 0.456, ..., 0.789],  // 1024 dimensions
  "similarity_score": 0.92,                  // On search
  "metadata": {
    "source": "faq.pdf",
    "knowledge_group_id": "..."
  }
}
```

## Technology Stack

| Layer | Technology | Why? |
|-------|-----------|------|
| **API Framework** | FastAPI | Async, auto-documented, fast |
| **Metadata DB** | MongoDB | Flexible, async driver, scalable |
| **Vector DB** | PostgreSQL + pgvector | ACID, cost-effective, reliable |
| **Embeddings** | AWS Bedrock (Titan) | Managed, high-quality, enterprise |
| **File Processing** | PyMuPDF, python-docx, pptx | Robust text extraction |
| **Orchestration** | Docker Compose | Local dev, reproducible environments |

## Key Features

### ✅ User Scoping
All operations filtered by `user-id` header. Users only see their own data.

```python
# Only return documents for this user's knowledge groups
db.documents.find({"knowledge_group_id": user_group_id})
```

### ✅ Async Processing
Document ingestion runs in background (non-blocking).

```
POST /documents → HTTP 201 (immediately)
                 + async task starts
                 + user can check status anytime
```

### ✅ Type Safety
Pydantic models validate all requests/responses.

```python
class RagSearchRequest(BaseModel):
    query: str
    knowledge_group_ids: list[str]
    max_results: int = 10
```

### ✅ Error Handling
Graceful errors with clear messages.

```
404 → Knowledge group not found or access denied
502 → Bedrock embedding or Postgres failure
422 → Invalid request body
```

## Workflow Example

### Typical User Journey

**Step 1: Create Knowledge Group**
```bash
curl -X POST http://localhost:8085/knowledge-groups \
  -H "user-id: alice" \
  -H "Content-Type: application/json" \
  -d '{"name": "Customer FAQ"}'
```

**Step 2: Upload Documents**
```bash
curl -X POST http://localhost:8085/documents \
  -H "Content-Type: application/json" \
  -d '[{
    "file_name": "faq.pdf",
    "knowledge_group_id": "group-123",
    "cdp_upload_id": "batch-456",
    "s3_key": "uploads/faq.pdf"
  }]'
```
→ Document is fetched, chunked, embedded, stored (2-10 seconds)

**Step 3: Search Knowledge Base**
```bash
curl -X POST http://localhost:8085/rag/search \
  -H "user-id: alice" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "How do I reset my password?",
    "knowledge_group_ids": ["group-123"],
    "max_results": 5
  }'
```

**Response:**
```json
[
  {
    "content": "To reset your password, visit...",
    "similarity_score": 0.92,
    "file_name": "faq.pdf",
    "document_id": "doc-789"
  },
  {
    "content": "Password reset typically takes 24 hours...",
    "similarity_score": 0.87,
    "file_name": "faq.pdf",
    "document_id": "doc-789"
  }
]
```

**Step 4: Use in LLM**
```python
# Take top search results and augment prompt
context = "\n".join([r["content"] for r in search_results])

prompt = f"""
Using this context:
{context}

Answer the user's question:
{user_query}
"""

response = llm.generate(prompt)  # Now grounded!
```

## Architecture Diagram

```
┌──────────────────────────────────────────────────┐
│             FastAPI Application                   │
│  (Async, Type-hinted, Well-tested)               │
├──────────────────────────────────────────────────┤
│                                                   │
│  Knowledge  │   Document    │      RAG           │
│  Groups     │   Ingestion   │    Search          │
│             │               │                    │
├──────────────────────────────────────────────────┤
│       ↓            ↓              ↓              │
│    MongoDB       S3 Bucket   PostgreSQL+pgvector
│   (Metadata)   (Raw Files)   (Vectors & Chunks)
│                     ↓
│              AWS Bedrock
│          (Embedding Generation)
└──────────────────────────────────────────────────┘
```

## Security & Access Control

### User Isolation
- All queries filtered by `user-id` header
- Knowledge groups scoped to user via unique index: `(created_by, name)`
- Document access validated before search

### Error Masking
- Returns 404 for both "not found" and "access denied"
- Prevents information leakage

### Data Protection
- TLS for all database connections (required in production)
- Optional MongoDB encryption at rest
- S3 bucket encryption configurable

## Performance Characteristics

| Operation | Latency | Scaling |
|-----------|---------|---------|
| Create group | <50ms | Instant |
| Ingest PDF (100 pages) | 5-10s | Async, non-blocking |
| Embed 250 chunks | 3-5s | Parallel via Bedrock |
| Search query | 100-500ms | O(log n) with indexes |
| Return top-10 | <50ms | Constant time |

## Limitations & Tradeoffs

| Aspect | Current | Reason |
|--------|---------|--------|
| **Max vectors** | 100M per instance | Suitable MVP size; shard if larger |
| **Chunk size** | Token-based (dynamic) | Balances richness vs. quality |
| **Embedding model** | Titan v2 only | AWS-native, good quality |
| **Search distance** | Cosine only | Industry standard, normalized |
| **Scaling** | Horizontal (API instances) | Multiple servers behind LB |

## What's NOT Included

- ❌ LLM response generation (bring your own)
- ❌ User authentication (use header-based or API gateway)
- ❌ Web UI (API-first, build what you need)
- ❌ Reranking (future enhancement)
- ❌ Hybrid search (BM25 + vector, future)

## Next Steps

- **See it in action:** [First Steps](./first-steps.md)
- **Understand the API:** [API Overview](../api/README.md)
- **Learn the architecture:** [Architecture](./architecture.md)
- **Start developing:** [Development Setup](../development/setup.md)

---

**Read Time:** 10 minutes  
**Next:** Try [First Steps](./first-steps.md)
