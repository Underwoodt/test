# Service Interactions - AI DEFRA Search Platform

**How services communicate, dependencies, and interaction patterns**

---

## Table of Contents

- [Service Map](#service-map)
- [Communication Patterns](#communication-patterns)
- [Dependency Graph](#dependency-graph)
- [API Contracts](#api-contracts)
- [Synchronous Interactions](#synchronous-interactions)
- [Asynchronous Interactions](#asynchronous-interactions)
- [Error Handling Between Services](#error-handling-between-services)
- [Resilience Patterns](#resilience-patterns)

---

## Service Map

### All Services and Their Roles

```
┌─────────────────┐
│    Frontend     │  React Web UI
│  (Port: 3000)   │  - User interface
└────────┬────────┘  - Session management
         │           - File upload
         │ HTTP REST
         │
    ┌────▼────────────────────────────────────┐
    │                                         │
    ↓                                         ↓
┌──────────────┐                    ┌──────────────┐
│ Agent        │◄──────────HTTP────►│ Knowledge    │
│ (Port: 8086) │    /rag/search     │ (Port: 8085) │
│              │                    │              │
│ FastAPI      │                    │ FastAPI      │
│ - Chat API   │    ┌──────────┐    │ - Ingestion  │
│ - Orchestr.  │    │          │    │ - Search     │
└──┬───────────┘    │ Bedrock  │    │ - Metadata   │
   │                │ (4566)   │    └──┬───────────┘
   │                │          │       │
   │                │ AWS/Stub │       │
   └────────┬───────┴──────────┴───────┘
            │ Embeddings
            │ Responses
            │
    ┌───────▼────────────────┐
    │  External Services     │
    ├────────────────────────┤
    │ • PostgreSQL + pgvector│
    │ • MongoDB              │
    │ • Redis                │
    │ • S3/LocalStack        │
    └────────────────────────┘
```

---

## Communication Patterns

### Pattern 1: Request-Response (Synchronous)

**Used for:** Real-time operations that need immediate response

```
Service A                      Service B

POST /endpoint        ──────→  
                      ↓ Process request
                 ◄────────    
            Response received

Time: ~100ms - 2s (depends on operation)
Used by: Frontend ↔ Agent, Agent ↔ Knowledge, Agent ↔ Bedrock
```

### Pattern 2: Fire-and-Forget (Asynchronous)

**Used for:** Long-running operations that don't need immediate response

```
Service A                      Service B

POST /task       ──────→       Queue task
                               ↓ Process in background
Immediately returns            Update status in DB

Time: Return immediately, task completes later
Used by: Document ingestion (upload → background processing)
```

### Pattern 3: Database Query (Direct)

**Used for:** Services directly accessing shared data

```
Service A              Database

SELECT * FROM         ──────→  
  chunks WHERE               ↓ Query
  embedding ~~> vector  ◄────────  
                      Results
```

---

## Dependency Graph

### Who Depends on Whom?

```
Frontend
  ├─ Depends on: Agent (chat API)
  ├─ Depends on: Knowledge (upload endpoint)
  └─ Depends on: Redis (sessions)

Agent
  ├─ Depends on: Knowledge (/rag/search)
  ├─ Depends on: Bedrock (embeddings, responses)
  ├─ Depends on: MongoDB (conversation storage)
  └─ Depends on: Redis (caching)

Knowledge
  ├─ Depends on: PostgreSQL (vector storage)
  ├─ Depends on: MongoDB (metadata)
  ├─ Depends on: Bedrock (embeddings)
  ├─ Depends on: S3 (document storage)
  └─ Depends on: Redis (caching)

Bedrock
  └─ No internal dependencies (external service)

PostgreSQL
  └─ No dependencies

MongoDB
  └─ No dependencies

Redis
  └─ No dependencies

S3/LocalStack
  └─ No dependencies
```

### Criticality Level

```
CRITICAL (system down if missing):
  ├─ Frontend (users can't access UI)
  ├─ Agent (chat not available)
  ├─ PostgreSQL (search unavailable)
  └─ Bedrock (embeddings/responses unavailable)

HIGH (some features unavailable):
  ├─ Knowledge (search unavailable, but chat works)
  └─ MongoDB (conversations lost, but chat works)

MEDIUM (performance degraded):
  ├─ Redis (cache misses, slower performance)
  └─ S3 (can't upload new documents)
```

---

## API Contracts

### Frontend ↔ Agent

**Endpoint:** `POST http://agent:8086/chat`

**Request:**
```json
{
  "message": "What are grant requirements?",
  "conversation_id": "conv-abc-123",
  "user_id": "user@example.com"
}
```

**Response:**
```json
{
  "response": "Grant requirements include...",
  "conversation_id": "conv-abc-123",
  "sources": ["doc-123", "doc-456"],
  "metadata": {
    "tokens_used": 150,
    "search_results": 5
  }
}
```

**Error Response:**
```json
{
  "error": "Service unavailable",
  "status": 503,
  "message": "Knowledge service is temporarily down"
}
```

---

### Frontend ↔ Knowledge

**Endpoint:** `POST http://knowledge:8085/documents`

**Request:**
```json
{
  "s3_key": "uploads/user-123/grant.pdf",
  "file_name": "grant.pdf",
  "knowledge_group_id": "group-abc"
}
```

**Response:**
```json
{
  "document_id": "doc-789",
  "status": "processing",
  "message": "Document queued for processing"
}
```

---

### Agent ↔ Knowledge

**Endpoint:** `POST http://knowledge:8085/rag/search`

**Request:**
```json
{
  "query": "grant requirements",
  "knowledge_group_ids": ["group-1", "group-2"],
  "max_results": 5
}
```

**Response:**
```json
[
  {
    "content": "Requirements: 1...",
    "similarity_score": 0.92,
    "document_id": "doc-123",
    "file_name": "grant.pdf"
  },
  {
    "content": "Eligibility: ...",
    "similarity_score": 0.88,
    "document_id": "doc-456",
    "file_name": "procedures.pdf"
  }
]
```

---

### Agent ↔ Bedrock

**Endpoint:** `POST http://bedrock:4566/embeddings` (local) or AWS endpoint (prod)

**Request:**
```json
{
  "model": "amazon.titan-embed-text-v2:0",
  "text": "What are grant requirements?"
}
```

**Response:**
```json
{
  "embedding": [0.123, 0.456, ..., 0.789],
  "input_tokens": 8
}
```

---

## Synchronous Interactions

### When Frontend Calls Agent

```
Frontend                          Agent
   │                              │
   ├─ HTTP POST /chat             │
   │  Headers:                     │
   │  ├─ Content-Type: application/json
   │  └─ Authorization: Bearer token
   │  Body:                        │
   │  ├─ message                   │
   │  ├─ conversation_id           │
   │  └─ user_id                   │
   │                               │
   │◄──────────────────────────────┤
   │  Response: 200 OK             │
   │  Body:                        │
   │  ├─ response                  │
   │  ├─ conversation_id           │
   │  └─ sources                   │
   │                               │
   Timeout after: 30 seconds
   Retry after: 3 attempts
```

### When Agent Calls Knowledge

```
Agent                            Knowledge
   │                              │
   ├─ HTTP POST /rag/search       │
   │  Headers:                     │
   │  └─ Content-Type: application/json
   │  Body:                        │
   │  ├─ query                     │
   │  ├─ knowledge_group_ids       │
   │  └─ max_results               │
   │                               │
   │◄──────────────────────────────┤
   │  Response: 200 OK             │
   │  Body: Array of chunks        │
   │  (JSON array)                 │
   │                               │
   Timeout after: 5 seconds
   Retry after: 2 attempts
   Fall back to: Cached results
```

### When Agent Calls Bedrock

```
Agent                            Bedrock
   │                              │
   ├─ HTTP POST /embeddings       │
   │  or /invoke-model            │
   │                               │
   │◄──────────────────────────────┤
   │  Response: 200 OK             │
   │                               │
   Timeout after: 10 seconds
   Retry after: 2 attempts
   Fall back to: Cached embeddings
```

---

## Asynchronous Interactions

### Document Upload Processing

```
Frontend                Knowledge (Sync)        Knowledge (Background)
   │                            │                       │
   ├─ POST /documents           │                       │
   │  ├─ s3_key                 │                       │
   │  ├─ file_name              │                       │
   │  └─ knowledge_group_id     │                       │
   │                             │                       │
   │◄────────────────────────────┤                       │
   │ Response: 202 Accepted       │                       │
   │ {                           │                       │
   │   document_id: "doc-123",   │                       │
   │   status: "processing"      │   ├─ Extract text     │
   │ }                           │   ├─ Split chunks     │
   │                             │   ├─ Generate embeds  │
   │                             │   ├─ Store in DB      │
   │ User doesn't wait           │   └─ Update status    │
   │ for processing              │                       │
   │                             │   MongoDB updated:    │
   │                             │   status = "ready"    │
```

### Polling for Status

```
Frontend                 Knowledge
   │                       │
   ├─ GET /upload-status  │
   │  {                   │
   │    upload_id: "up-1" │
   │  }                   │
   │                       │
   │◄──────────────────────┤
   │ Response: 200 OK      │
   │ {                     │
   │   status: "processing",
   │   chunks_done: 5,     │
   │   total_chunks: 10    │
   │ }                     │
   │                       │
   | (Poll every 2 seconds)
   │                       │
   ├─ GET /upload-status  │
   │                       │
   │◄──────────────────────┤
   │ Response: 200 OK      │
   │ {                     │
   │   status: "ready",    │
   │   chunks_done: 10,    │
   │   total_chunks: 10    │
   │ }                     │
```

---

## Error Handling Between Services

### When Knowledge Service is Down

```
Frontend                       Agent                      Knowledge
   │                           │                          │
   ├──POST /chat              │                           │
   │  (includes search)        │                           │
   │                           ├──POST /rag/search        │
   │                           │                          ✗ No response
   │                           │                          (Timeout)
   │                           │                          │
   │                           ├─ Retry (2x)              │
   │                           │                          ✗ Still down
   │                           │                          │
   │                           ├─ Check Redis cache       │
   │                           │  For recent searches     │
   │                           │                          │
   │◄───────────────────────────<─ Return cached or error │
   │ Response: 200 OK          │                          │
   │ {                         │                          │
   │   response: "ChatGPT...", │                          │
   │   note: "No fresh search" │                          │
   │ }                         │                          │
```

### When Bedrock is Unavailable

```
Agent                          Bedrock
   │                            │
   ├─ POST /embeddings          │
   │  (query embedding)         │
   │                            │
   │◄────────────────────────────┤ ✗ Error: 503
   │ Retry (2x)                 │
   │                            │
   │◄────────────────────────────┤ ✗ Still down
   │                            │
   ├─ Check Redis cache          │
   │  For similar embedding     │
   │                            │
   ├─ If found: Use cached      │
   │ If not found: Return error │
```

### Circuit Breaker Pattern

```
State: CLOSED (healthy)
  All requests pass through
  ├─ Error rate acceptable
  └─ Circuit stays CLOSED

State: OPEN (failing)
  Too many errors detected
  ├─ Requests fail fast
  ├─ Don't retry external service
  └─ Wait for recovery

State: HALF_OPEN (testing)
  Try one request to external service
  ├─ If succeeds → Back to CLOSED
  └─ If fails → Back to OPEN
```

---

## Resilience Patterns

### Retry Strategy

```
Service Call
  │
  ├─ Try 1: Call service
  │  ├─ Success? → Return result
  │  └─ Fail? → Continue
  │
  ├─ Wait: 100ms (exponential backoff)
  ├─ Try 2: Call service again
  │  ├─ Success? → Return result
  │  └─ Fail? → Continue
  │
  ├─ Wait: 200ms (exponential backoff)
  ├─ Try 3: Call service again
  │  ├─ Success? → Return result
  │  └─ Fail? → Fallback
  │
  └─ Fallback: Cache or error
```

### Timeout Strategy

```
Service Call
  │
  ├─ Send request
  ├─ Start timer: 5 seconds (Knowledge)
  │                 30 seconds (Agent)
  │                 10 seconds (Bedrock)
  │
  ├─ Wait for response
  │  ├─ Response received? → Use it
  │  └─ Timeout? → Treat as failure
  │
  └─ Proceed with retry/fallback
```

### Caching Strategy

```
Request → Cache check
  │
  ├─ Cache HIT (valid, recent)
  │  └─ Return cached result
  │  └─ Skip external call
  │
  └─ Cache MISS (not found, expired)
     ├─ Call external service
     ├─ Cache result for future
     └─ Return fresh result
```

### Fallback Strategy

```
Primary path fails
  │
  ├─ Fallback 1: Redis cache
  │  ├─ If found → Return cached
  │  └─ If not → Continue
  │
  ├─ Fallback 2: Default result
  │  ├─ Return error message
  │  └─ Suggest retry later
  │
  └─ Log error for investigation
```

---

## Service Communication Summary

### Request Counts During User Interaction

```
One user asks a question:
  Frontend → Agent:              1 request
  Agent → Knowledge:             1 request
  Agent → Bedrock (embed query): 1 request
  Knowledge → PostgreSQL:        1 query
  Knowledge → MongoDB:           1 query (metadata enrichment)
  Agent → Bedrock (response):    1 request
  Agent → MongoDB (store):       1 write
  ─────────────────────────
  Total: 8 service calls

Total time: ~2-3 seconds
```

### Concurrent Request Handling

```
Multiple users asking questions:

User 1 → Agent → Knowledge → PostgreSQL ┐
                                        ├─ Parallel execution
User 2 → Agent → Knowledge → PostgreSQL ┤
                                        ├─ Can run at same time
User 3 → Agent → Knowledge → PostgreSQL ┘

Load balancer distributes:
  50% to Agent replica 1
  50% to Agent replica 2

Each Knowledge query parallelized:
  Search + Metadata enrichment (concurrent)
```

---

## Best Practices for Service Interaction

### Do's ✅
- Add request IDs for tracing
- Include user context (user_id)
- Set appropriate timeouts
- Implement retry logic
- Cache when possible
- Log all interactions
- Monitor error rates
- Use circuit breakers

### Don'ts ❌
- Call services synchronously in a loop
- Forget error handling
- Set timeouts too high
- Retry forever
- Cache sensitive data
- Ignore error responses
- Forget rate limiting
- Make blocking calls in UI

---

**Document Version:** 1.0.0  
**Last Updated:** March 20, 2026  
**Status:** Production Ready

