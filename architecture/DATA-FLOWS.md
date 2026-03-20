# Data Flows - AI DEFRA Search Platform

**Complete data flow documentation showing how data moves through the system**

---

## Table of Contents

- [Main Data Flows](#main-data-flows)
- [Request Flow: User Asks Question](#request-flow-user-asks-question)
- [Request Flow: User Uploads Document](#request-flow-user-uploads-document)
- [Multi-Turn Conversation](#multi-turn-conversation)
- [Search Deep Dive](#search-deep-dive)
- [Ingestion Deep Dive](#ingestion-deep-dive)
- [Error Flows](#error-flows)

---

## Main Data Flows

There are three major data flows in the system:

1. **User Ask Flow** - User asks a question → System returns answer
2. **Document Upload Flow** - User uploads document → System indexes it
3. **Conversation Flow** - Multi-turn conversation with context

---

## Request Flow: User Asks Question

### Step-by-Step

**Complete flow from user question to response:**

```
Step 1: User Types Message
┌──────────────────────────┐
│ Frontend (React)         │
│ User Input: "What are    │
│ the grant requirements?" │
└────────────┬─────────────┘
             │
             ↓ HTTP POST /chat
             │
Step 2: Message to Agent
┌──────────────────────────┐
│ Agent Service (FastAPI)  │
│ Receives:                │
│ - message (user input)   │
│ - conversation_id        │
│ - user_id                │
└────────────┬─────────────┘
             │
             ↓ Internal Processing
             │
Step 3: Generate Query Embedding
┌──────────────────────────┐
│ Agent → Bedrock          │
│ Request: Embed query     │
│ Input: "What are the    │
│ grant requirements?"      │
└────────────┬─────────────┘
             │
             ↓ Returns: [0.123, 0.456, ..., 0.789]  (1024 dims)
             │
Step 4: Search for Context
┌──────────────────────────┐
│ Agent → Knowledge        │
│ POST /rag/search         │
│ {                        │
│   "query": "...",        │
│   "embedding": [...],    │
│   "knowledge_groups": [] │
│ }                        │
└────────────┬─────────────┘
             │
             ↓ Search Processing
             │
Step 5: Knowledge Searches Database
┌──────────────────────────────────────┐
│ Knowledge Service                    │
│                                      │
│ 1. Receives query embedding          │
│ 2. Queries PostgreSQL:               │
│    SELECT * FROM chunks WHERE       │
│    embedding <-> query_embedding    │
│    ORDER BY distance LIMIT 5         │
│                                      │
│ Result: Top 5 similar chunks         │
└────────────┬─────────────────────────┘
             │
             ↓ Get Metadata
             │
Step 6: Enrich with Metadata
┌──────────────────────────────────────┐
│ Knowledge Service                    │
│                                      │
│ 1. Gets chunk IDs from search        │
│ 2. Queries MongoDB for metadata      │
│    - Document name                   │
│    - Document ID                     │
│    - Chunk position                  │
│                                      │
│ Returns: Chunks with metadata        │
└────────────┬─────────────────────────┘
             │
             ↓ Return Results
             │
Step 7: Results Back to Agent
┌──────────────────────────────────────┐
│ Agent Service                        │
│                                      │
│ Receives: [                          │
│   {                                  │
│     "content": "Grant applications...",
│     "document_id": "doc-123",        │
│     "similarity_score": 0.92         │
│   },                                 │
│   ...                                │
│ ]                                    │
└────────────┬─────────────────────────┘
             │
             ↓ Prepare Context
             │
Step 8: Build LLM Prompt
┌──────────────────────────────────────┐
│ Agent Service                        │
│                                      │
│ Creates prompt:                      │
│ "Based on these documents:           │
│ [chunk 1]                            │
│ [chunk 2]                            │
│ ...                                  │
│ Answer: What are grant requirements?"│
└────────────┬─────────────────────────┘
             │
             ↓ Call Bedrock
             │
Step 9: Generate Response
┌──────────────────────────────────────┐
│ Agent → Bedrock                      │
│                                      │
│ POST /invoke-model                   │
│ Model: Claude                        │
│ Prompt: [prepared context + question]│
└────────────┬─────────────────────────┘
             │
             ↓ AI Processing
             │
Step 10: Bedrock Generates Response
┌──────────────────────────────────────┐
│ Bedrock / AWS                        │
│                                      │
│ Generates:                           │
│ "Grant requirements are:             │
│  1. Minimum 3 years experience       │
│  2. Non-profit status                │
│  3. Financial documentation          │
│  4. Project proposal"                │
└────────────┬─────────────────────────┘
             │
             ↓ Return Response
             │
Step 11: Store Conversation
┌──────────────────────────────────────┐
│ Agent Service                        │
│                                      │
│ Stores in MongoDB:                   │
│ {                                    │
│   "conversation_id": "conv-123",     │
│   "messages": [                      │
│     {                                │
│       "role": "user",                │
│       "content": "What are...",      │
│       "timestamp": "..."             │
│     },                               │
│     {                                │
│       "role": "assistant",           │
│       "content": "Grant req...",     │
│       "sources": ["doc-123", ...],   │
│       "timestamp": "..."             │
│     }                                │
│   ]                                  │
│ }                                    │
└────────────┬─────────────────────────┘
             │
             ↓ Return to Frontend
             │
Step 12: Display to User
┌──────────────────────────────────────┐
│ Frontend (React)                     │
│                                      │
│ Display:                             │
│ Q: "What are grant requirements?"   │
│ A: "Grant requirements are: 1..."   │
│ Sources: doc-123.pdf, doc-456.pdf   │
└──────────────────────────────────────┘
```

### Total Flow Time

```
Step 3 (Embed):        500-1000ms
Step 5 (Search):       20-50ms
Step 6 (Metadata):     10-20ms
Step 9 (Generate):     1000-2000ms
Step 11 (Store):       20-50ms
                       ────────────
TOTAL:                 ~2.5 seconds
```

---

## Request Flow: User Uploads Document

### Step-by-Step

```
Step 1: User Selects File
┌──────────────────────────┐
│ Frontend (React)         │
│ User selects: grant.pdf  │
│ (1 MB file)              │
└────────────┬─────────────┘
             │
             ↓ Upload to S3
             │
Step 2: File to S3
┌──────────────────────────────────────┐
│ LocalStack/AWS S3                    │
│                                      │
│ Stores file at:                      │
│ s3://uploads/user-123/grant.pdf      │
│                                      │
│ Returns: Upload complete ✓           │
└────────────┬─────────────────────────┘
             │
             ↓ Frontend confirms
             │
Step 3: Frontend Notifies Knowledge
┌──────────────────────────────────────┐
│ Frontend → Knowledge                 │
│                                      │
│ POST /documents                      │
│ {                                    │
│   "s3_key": "user-123/grant.pdf",   │
│   "file_name": "grant.pdf",         │
│   "knowledge_group_id": "group-1"   │
│ }                                    │
└────────────┬─────────────────────────┘
             │
             ↓ Queued for processing
             │
Step 4: Knowledge Detects File
┌──────────────────────────────────────┐
│ Knowledge Service (Background)       │
│                                      │
│ 1. Detects new file in S3            │
│ 2. Updates status: "in_progress"     │
│    in MongoDB                        │
└────────────┬─────────────────────────┘
             │
             ↓ Begin processing
             │
Step 5: Extract Text
┌──────────────────────────────────────┐
│ Knowledge Service                    │
│                                      │
│ 1. Reads PDF from S3                 │
│ 2. Uses PyMuPDF to extract text      │
│ 3. Result: 5000 characters of text   │
└────────────┬─────────────────────────┘
             │
             ↓ Chunk text
             │
Step 6: Split into Chunks
┌──────────────────────────────────────┐
│ Knowledge Service                    │
│                                      │
│ Text → Semantic chunking             │
│ Result: 10 chunks                    │
│ [                                    │
│   "Introduction to grants...",       │
│   "Eligibility criteria...",         │
│   "Application process...",          │
│   ...                                │
│ ]                                    │
└────────────┬─────────────────────────┘
             │
             ↓ Generate embeddings
             │
Step 7: For Each Chunk
┌──────────────────────────────────────┐
│ Knowledge Service Loop               │
│                                      │
│ FOR EACH chunk:                      │
│   1. Call Bedrock: embed chunk       │
│   2. Get: [0.123, 0.456, ...]        │
│   3. Store in PostgreSQL:            │
│      INSERT INTO chunks (            │
│        content,                      │
│        embedding,                    │
│        document_id,                  │
│        metadata                      │
│      )                               │
│   4. Update MongoDB chunk count      │
│                                      │
│ Timing: ~1 second per chunk          │
│ Total: ~10 seconds for all           │
└────────────┬─────────────────────────┘
             │
             ↓ Finalize
             │
Step 8: Mark Complete
┌──────────────────────────────────────┐
│ Knowledge Service                    │
│                                      │
│ Updates MongoDB:                     │
│ {                                    │
│   "document_id": "doc-123",          │
│   "status": "ready",                 │
│   "chunk_count": 10,                 │
│   "completed_at": "2024-03-20..."    │
│ }                                    │
└────────────┬─────────────────────────┘
             │
             ↓ Done
             │
Step 9: Document Ready
┌──────────────────────────────────────┐
│ Document can now be searched!        │
└──────────────────────────────────────┘
```

### Background Processing

All document processing is **asynchronous**:

```
Frontend ────→ Knowledge (API)
                ↓
           Background Worker
           ├─ Extract text
           ├─ Split chunks
           ├─ Generate embeddings
           ├─ Store in DB
           └─ Update status
           
User can immediately upload another file
Frontend doesn't wait for processing
```

---

## Multi-Turn Conversation

### How Context is Maintained

```
Turn 1:
User: "What are grant requirements?"
Agent: "Grant requirements are..."
MongoDB stores:
{
  "conversation_id": "conv-123",
  "messages": [user msg, assistant response]
}

Turn 2:
User: "What's the deadline?"
Agent receives:
- New message: "What's the deadline?"
- Conversation history from MongoDB (all previous turns)
- Uses full history to understand context
Agent: "The deadline is..."
MongoDB stores:
{
  "conversation_id": "conv-123",
  "messages": [user msg 1, response 1, user msg 2, response 2]
}

Turn 3:
User: "For which programs?"
Agent receives:
- Full conversation history (all previous turns)
- Can reference earlier messages
Agent: "The programs include..."
```

### Context Window

The system passes full conversation history to Bedrock:

```
Full Context Passed to Bedrock:
"Turn 1: User asked about grant requirements
Response: Grant requirements are...
Turn 2: User asked about deadline
Response: Deadline is...
Turn 3: User asks: For which programs?

Based on this context, answer:"
```

---

## Search Deep Dive

### Vector Similarity Search

```
Query: "How do I apply for grants?"
    ↓
Step 1: Generate Query Embedding
  Bedrock: embed query
  Result: query_vector = [0.1, 0.2, ..., 0.9]
    ↓
Step 2: Vector Search in PostgreSQL
  SELECT chunk_id, content, distance
  FROM chunks
  WHERE embedding <-> query_vector < 1  -- cosine distance < 1
  ORDER BY embedding <-> query_vector
  LIMIT 5
    ↓
Step 3: Results (Sorted by Similarity)
  [
    {chunk_id: 1, content: "...", distance: 0.15},
    {chunk_id: 5, content: "...", distance: 0.22},
    {chunk_id: 12, content: "...", distance: 0.31},
    ...
  ]
    ↓
Step 4: Similarity Score
  Score = 1 - distance
  Distance 0.15 → Score 0.85 (85% similar)
  Distance 0.22 → Score 0.78 (78% similar)
```

### Why Vector Search Works

```
Traditional Keyword Search:
Query: "How do I apply?"
Results:
  ✓ Found: "Apply for grants"
  ✗ Missing: "Submission procedures" (no "apply" keyword)
  ✗ Missing: "Grant application process" (slightly different wording)

Vector/Semantic Search:
Query: "How do I apply?"
Results:
  ✓ Found: "Apply for grants" (exact match)
  ✓ Found: "Submission procedures" (semantic match)
  ✓ Found: "Grant application process" (semantic match)
  ✓ Found: "Requesting funds procedures" (related concept)
```

---

## Ingestion Deep Dive

### Chunking Strategy

**Semantic Chunking (not just by size):**

```
Document Text:
"Grant Overview. 
Grants are...

Eligibility Criteria.
To be eligible...

Application Process.
1. Prepare documents..."

Result:
Chunk 1: "Grant Overview. Grants are..."
Chunk 2: "Eligibility Criteria. To be eligible..."
Chunk 3: "Application Process. 1. Prepare..."

NOT:
Chunk 1: "Grant Overview. Grants are to be eligible..."
Chunk 2: "...Eligibility Criteria. To be eligible..."
```

### Embedding Process

```
For Each Chunk:
    ↓
Bedrock Request:
{
  "model": "amazon.titan-embed-text-v2:0",
  "text": "Grant Overview. Grants are..."
}
    ↓
Bedrock Response:
{
  "embedding": [0.123, 0.456, ..., 0.789],  // 1024 dims
  "input_tokens": 25
}
    ↓
Store in PostgreSQL:
INSERT INTO chunks (
  id, content, embedding, document_id, metadata
) VALUES (
  'chunk-1',
  'Grant Overview. Grants are...',
  '[0.123, 0.456, ..., 0.789]',
  'doc-123',
  '{"chunk_num": 1, "total_chunks": 3}'
);
```

### Metadata Storage

```
PostgreSQL (Vector & Content):
└─ Chunk vectors
└─ Chunk text
└─ Embedding vectors
└─ Chunk IDs

MongoDB (Metadata):
└─ Document info
   ├─ Document name
   ├─ Upload date
   ├─ Knowledge group
   └─ Chunk count
└─ Ingestion status
   ├─ In progress
   ├─ Complete
   └─ Errors (if any)
```

---

## Error Flows

### Network Error During Upload

```
User uploads → Network fails → Upload incomplete
    ↓
Frontend detects error
    ↓
User can retry
    ↓
If retry successful → Continue normal flow
```

### Bedrock Timeout During Search

```
Agent calls Bedrock → Timeout (no response)
    ↓
Agent → Redis (check cache)
    ↓
Cache hit? 
  YES → Return cached embedding
  NO → Return error to user
```

### Database Connection Error

```
Agent → PostgreSQL search → Connection error
    ↓
Fallback strategy:
  1. Retry connection (3x)
  2. Try read replica
  3. Return error
    ↓
User gets: "Search temporarily unavailable"
```

---

## Data Volume Examples

### Small System (Development)
```
Documents:       100
Chunks:          1,000
Embeddings:      1,000 x 1024 floats = 4 MB
PostgreSQL Size: ~10 MB
MongoDB Size:    ~20 MB
Total:           ~30 MB
```

### Medium System (100 staff, 6 months)
```
Documents:       5,000
Chunks:          50,000
Embeddings:      50,000 x 1024 = 200 MB
PostgreSQL Size: ~500 MB
MongoDB Size:    ~200 MB
Total:           ~700 MB
```

### Large System (1000 staff, 2 years)
```
Documents:       50,000
Chunks:          500,000
Embeddings:      500,000 x 1024 = 2 GB
PostgreSQL Size: ~5 GB
MongoDB Size:    ~2 GB
Total:           ~7 GB
```

---

**Document Version:** 1.0.0  
**Last Updated:** March 20, 2026  
**Status:** Production Ready

