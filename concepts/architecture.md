# Architecture

Deep dive into system design and component interactions.

## System Overview

```
┌──────────────────────────────────────────────────────────┐
│                  FastAPI Application                      │
│  (Async, Type-safe, Well-tested)                         │
├──────────────────────────────────────────────────────────┤
│                                                            │
│  ┌────────────────┐  ┌──────────────┐  ┌────────────┐  │
│  │ Knowledge      │  │  Document    │  │    RAG     │  │
│  │ Groups Router  │  │ Ingestion    │  │ Search     │  │
│  │                │  │ Router       │  │ Router     │  │
│  └────────────────┘  └──────────────┘  └────────────┘  │
│         │                   │                 │          │
├─────────┼───────────────────┼─────────────────┼──────────┤
│  Service Layer (Business Logic)                         │
│                                                         │
│  ├─ KnowledgeGroupService                              │
│  ├─ DocumentService / IngestService                    │
│  └─ VectorSearchService                                │
├─────────┼───────────────────┼─────────────────┼──────────┤
│  MongoDB  │  S3 Bucket       │  PostgreSQL+    │ Bedrock │
│  (User   │  (Raw Files)     │  pgvector      │ (Embed) │
│  Metadata)│                 │  (Vectors)     │         │
└──────────┴───────────────────┴────────────────┴──────────┘
```

## Three-Layer Architecture

### Layer 1: API (HTTP Layer)
**Where:** `app/{feature}/router.py`  
**Purpose:** Handle HTTP requests/responses, validation, auth

```python
@router.post("/rag/search")
async def search(body: RagSearchRequest, user_id: str) -> list[RagSearchResult]:
    # Validate user owns knowledge groups
    # Call service layer
    # Return results
```

### Layer 2: Service (Business Logic)
**Where:** `app/{feature}/service.py`, `app/ingest/service.py`, etc.  
**Purpose:** Core business logic, independent of HTTP

```python
async def ingest_document(bucket, s3_key, file_name, ...) -> int:
    # Extract chunks
    # Generate embeddings
    # Store in database
    # Return chunk count
```

### Layer 3: Data (Database Layer)
**Where:** `app/common/{postgres.py, mongo.py}`, `app/ingest/vector_store.py`  
**Purpose:** Database operations, queries

```python
async def insert_vectors(vectors: list) -> None:
    async with get_connection() as conn:
        await conn.execute(insert_sql, vectors)
```

**Key Benefits:**
- ✅ Each layer can be tested independently
- ✅ Reuse service logic in different contexts (API, CLI, events)
- ✅ Easy to mock dependencies in tests
- ✅ Clear separation of concerns

---

## Data Flow: Ingestion Pipeline

```
┌─────────────────────────────────────────────────────────┐
│ USER UPLOADS DOCUMENT                                    │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ POST /documents                                           │
│ • Validate knowledge_group_id                           │
│ • Insert document record in MongoDB                     │
│ • Spawn async ingestion task                            │
│ Return 201 (user can check status anytime)              │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ BACKGROUND TASK (Async)                                  │
│ 1. Update document status → "in_progress"               │
│ 2. Fetch raw file from S3                               │
│    └─ Use extractor based on file type                 │
│       • PDF → PyMuPDF                                   │
│       • DOCX → python-docx                              │
│       • PPTX → python-pptx                              │
│       • JSONL → JSON parser                             │
│ 3. Extract text → get list of chunks                    │
│    └─ Each chunk: {text, source}                        │
│ 4. FOR EACH CHUNK:                                      │
│    a. Generate embedding (AWS Bedrock)                  │
│    b. Create vector record with metadata                │
│ 5. BATCH INSERT vectors → PostgreSQL + pgvector         │
│ 6. Update document status → "ready"                     │
│ 7. Update chunk_count in MongoDB                        │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ DOCUMENT READY FOR SEARCH                                │
│ • Metadata stored in MongoDB                             │
│ • Chunks + vectors stored in PostgreSQL                  │
│ • User can now search                                    │
└─────────────────────────────────────────────────────────┘
```

**Key Design Decisions:**

1. **Async Processing** — Doesn't block API
2. **S3 as Source** — Supports large files, integrates with CDP uploads
3. **Multiple Extractors** — Handles diverse formats
4. **Batch Embedding** — More efficient than per-chunk calls
5. **Status Tracking** — Users can monitor progress

---

## Data Flow: RAG Search Pipeline

```
┌─────────────────────────────────────────────────────────┐
│ USER QUERY                                               │
│ {query: "How do I...?", knowledge_group_ids: [...]}     │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ POST /rag/search                                          │
│ 1. Validate user_id header (required)                   │
│ 2. FOR EACH knowledge_group_id:                         │
│    a. Look up in MongoDB                                │
│    b. Verify created_by matches user_id                 │
│    c. Return 404 if not found or not owned              │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ GENERATE QUERY EMBEDDING                                 │
│ • Call AWS Bedrock with same model as ingestion         │
│ • Get 1024-dimension vector                             │
│ • Return embedding                                       │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ VECTOR SIMILARITY SEARCH (PostgreSQL)                    │
│ • SQL: SELECT * FROM knowledge_vectors                  │
│        WHERE metadata->>'knowledge_group_id' IN (...)   │
│        ORDER BY embedding <=> :query_vector             │
│        LIMIT :max_results                               │
│                                                          │
│ • Uses pgvector cosine distance operator (<=>)          │
│ • Returns content, source_id, similarity_score          │
│ • O(log n) with indexes, fast even for millions         │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ ENRICH WITH METADATA (MongoDB)                           │
│ • Extract document_ids from search results              │
│ • Batch query MongoDB for file_name, s3_key             │
│ • Map metadata back to results                          │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│ RETURN RANKED RESULTS                                    │
│ [{                                                       │
│   content: "...",                                        │
│   similarity_score: 0.92,                               │
│   file_name: "faq.pdf",                                 │
│   s3_key: "s3://...",                                   │
│   document_id: "..."                                    │
│ }, ...]                                                  │
└─────────────────────────────────────────────────────────┘
```

**Performance Characteristics:**

| Operation | Latency | Scaling |
|-----------|---------|---------|
| Validate groups (MongoDB) | 10-50ms | Linear with # groups |
| Embed query (Bedrock) | 500-2000ms | Depends on AWS latency |
| Vector search (PostgreSQL) | 50-200ms | O(log n) with indexes |
| Enrich metadata (MongoDB) | 10-50ms | Parallel with results |
| **Total** | **600-2300ms** | **Scales to millions** |

---

## Database Design

### PostgreSQL (Vector Storage)

**Table: `knowledge_vectors`**

```sql
CREATE TABLE knowledge_vectors (
    id SERIAL PRIMARY KEY,
    content TEXT NOT NULL,              -- Chunk text
    embedding vector(1024),             -- Bedrock embedding
    snapshot_id VARCHAR(255),           -- doc:key identifier
    source_id VARCHAR(24),              -- MongoDB document ID
    metadata JSONB,                     -- {source, knowledge_group_id}
    created_at TIMESTAMP DEFAULT NOW()
);

-- Indexes for performance
CREATE INDEX idx_embedding ON knowledge_vectors USING ivfflat (embedding vector_cosine_ops);
CREATE INDEX idx_knowledge_group ON knowledge_vectors USING GIN (metadata);
CREATE INDEX idx_source_id ON knowledge_vectors (source_id);
```

**Why PostgreSQL?**
- ACID compliance
- pgvector extension mature and stable
- Cost-effective for MVP scale (10M-100M vectors)
- JSONB metadata is flexible
- Can scale horizontally with read replicas

### MongoDB (Metadata)

**Collection: `knowledgeGroups`**

```javascript
{
  _id: ObjectId,
  name: String,
  created_by: String (user ID),
  created_at: ISODate,
  
  // Unique index: (created_by, name)
  // Ensures users can't have duplicate group names
}
```

**Collection: `documents`**

```javascript
{
  _id: ObjectId,
  file_name: String,
  status: String,              // not_started, in_progress, ready, failed
  knowledge_group_id: String,
  cdp_upload_id: String,
  s3_key: String,
  created_at: ISODate,
  chunk_count: Integer         // Set after ingestion
}
```

**Why MongoDB?**
- Flexible schema for document metadata
- Native async driver (motor)
- Schemaless = easier future evolution
- Good separation from vector storage

---

## Component Details

### Ingest Pipeline Components

#### 1. Extractors (`app/ingest/extractors/`)

Each file type has an extractor:

```python
class Extractor(ABC):
    @abstractmethod
    def extract(self, data: bytes, file_name: str) -> list[dict]:
        """Return list of {text, source}"""
```

**Implementations:**
- `PDFExtractor` — Uses PyMuPDF for text extraction
- `DocxExtractor` — Uses python-docx for Word parsing
- `PptxExtractor` — Uses python-pptx for slides
- `JsonlChunkExtractor` — Line-by-line JSON parser

#### 2. Chunking (`app/ingest/extractors/chunking.py`)

Splits text into semantic chunks:

```python
def chunk_text(text: str, max_tokens: int = 512) -> list[str]:
    # 1. Split on sentence boundaries
    # 2. Merge sentences until approaching max_tokens
    # 3. Respect paragraph breaks when possible
    # 4. Preserve context
```

**Why semantic?**
- Better quality embeddings
- Sentence boundaries preserve meaning
- Token limits prevent truncation

#### 3. Embedding Service (`app/common/bedrock.py`)

Generates embeddings:

```python
class BedrockEmbeddingService:
    def generate_embeddings(self, text: str) -> list[float]:
        # Call AWS Bedrock API
        # Model: amazon.titan-embed-text-v2:0
        # Returns: 1024-dim vector
```

**Why Bedrock?**
- Managed service (no infrastructure)
- High quality embeddings
- Low latency
- Enterprise support

#### 4. Vector Store (`app/ingest/vector_store.py`)

Persists vectors:

```python
async def insert_vectors(vectors: list) -> None:
    # UPSERT INTO knowledge_vectors
    # Duplicate chunks overwrite (same snapshot_id)
```

### RAG Search Components

#### 1. Vector Search (`app/rag/vector_search.py`)

Similarity search:

```python
async def search_vectors(embedding, group_ids, max_results) -> list[dict]:
    # SELECT * FROM knowledge_vectors
    # WHERE metadata->>'knowledge_group_id' IN (group_ids)
    # ORDER BY embedding <=> :embedding  -- cosine distance
    # LIMIT max_results
```

#### 2. Search Router (`app/rag/router.py`)

Orchestrates search:

```python
async def search(body, user_id):
    # 1. Validate groups
    # 2. Generate embedding
    # 3. Vector search
    # 4. Enrich metadata
    # 5. Return results
```

---

## Async Design Pattern

The entire app uses async/await:

```python
# API Route
@router.post("/documents")
async def create_documents(body: list[DocumentCreate]):
    # Spawn async task (fire and forget)
    for doc in body:
        asyncio.create_task(ingest_document(doc))
    
    return 201  # Don't wait for tasks!

# Background Task
async def ingest_document(doc):
    # All I/O is async
    data = await to_thread(fetch_from_s3, ...)  # S3 call
    chunks = extractor.extract(data, ...)        # CPU work
    embeddings = await to_thread(bedrock_call, ...)  # Bedrock call
    await insert_vectors(embeddings)  # PostgreSQL call
```

**Benefits:**
- Hundreds of concurrent requests without blocking
- Long operations (S3, Bedrock) don't block API
- Efficient resource use

**Tradeoff:**
- Requires async-aware libraries (motor, SQLAlchemy async, httpx)
- Steeper learning curve than sync code

---

## Configuration & Dependency Injection

### Configuration (`app/config.py`)

```python
class AppConfig(BaseSettings):
    # Reads from environment variables
    postgres_host: str = "postgres"
    mongo_uri: str | None = None
    bedrock_model_id: str = "amazon.titan-embed-text-v2:0"
```

**Why Pydantic Settings?**
- Type safety
- Validation
- No .env files in production
- Works in Docker, K8s, local dev

### Dependency Injection (FastAPI `Depends`)

```python
@router.post("/rag/search")
async def search(
    body: RagSearchRequest,
    user_id: Annotated[str, Header(...)],
    db: Annotated[AsyncDatabase, Depends(get_db)],  # ← Injected
) -> list[RagSearchResult]:
    # db is provided by FastAPI
    # Testable: inject mock db in tests
```

**Why DI?**
- Testable without actual database
- Easy to mock/stub dependencies
- Single responsibility principle

---

## Error Handling

### Request Validation

Pydantic validates all requests:

```python
class RagSearchRequest(BaseModel):
    query: str
    knowledge_group_ids: list[str]
    max_results: int = 10
    
    # Validated on receive
    # Returns 422 if invalid
```

### Service Errors

Services raise exceptions, routers handle them:

```python
# Service raises
if not found:
    raise HTTPException(status_code=404, detail="Not found")

# Router returns HTTP response
# Status code + JSON body sent to client
```

### Logging

All errors logged:

```python
try:
    result = await search_vectors(...)
except Exception:
    logger.error("Vector search failed", exc_info=True)
    raise HTTPException(502, "Search failed")
```

---

## Security Model

### User Scoping

Every operation checks user ownership:

```python
# Find knowledge group
doc = await db.knowledge_groups.find_one({
    "_id": object_id,
    "created_by": user_id  # ← Filter by user
})

# If not found, return 404
# User can't see/modify other users' data
```

### No Authentication

This app assumes authentication happens upstream:
- API Gateway
- Service mesh
- Proxy
- The `user-id` header is assumed trusted

---

## Testing Architecture

### Unit Tests

Test individual functions:

```python
def test_chunk_text():
    result = chunk_text("Long text...", max_size=100)
    assert len(result) > 0
```

### Integration Tests

Test with mocked dependencies:

```python
@pytest.mark.asyncio
async def test_search(mocker):
    mock_db = mocker.MagicMock()
    mocker.patch("app.rag.vector_search.search_vectors", return_value=[...])
    result = await search(RagSearchRequest(...), db=mock_db)
    assert result is not None
```

### API Tests

Test HTTP layer:

```python
def test_search_endpoint(client):
    response = client.post(
        "/rag/search",
        json={"query": "...", ...},
        headers={"user-id": "alice"}
    )
    assert response.status_code == 200
```

---

## Next Steps

- **Design Patterns:** See [Design Patterns](./design-patterns.md)
- **Code Structure:** See [Code Structure](./code-structure.md)
- **Deployment:** See [Production Deployment](../deployment/production.md)

---

**Architecture Version:** 1.0  
**Last Updated:** March 19, 2026
