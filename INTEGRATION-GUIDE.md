# AI DEFRA Search - System Integration Guide

**How All Services Work Together**

---

## System Components

### Core Infrastructure
- **Orchestration:** Docker Compose (ai-defra-search-core)
- **Message Queue:** Redis
- **Metadata Storage:** MongoDB
- **Vector Database:** PostgreSQL + pgvector
- **File Storage:** S3 (LocalStack in dev)
- **API Gateway:** Traefik reverse proxy

### Microservices

| Service | Port | Role | Language |
|---------|------|------|----------|
| **Frontend** | 3000 | Web UI, sessions | React/TypeScript |
| **Agent** | 8086 | Chat, orchestration | Python/FastAPI |
| **Knowledge** | 8085 | Search, ingestion | Python/FastAPI |
| **Bedrock Stub** | 4566 | Embeddings (dev) | WireMock |

---

## Request Flows

### Flow 1: User Asks a Question

```
1. Frontend (React)
   └─ User types: "What is the process?"
   └─ Sends to Agent API
   
2. Agent Service (FastAPI)
   └─ Receives chat message
   └─ Extracts intent and context
   
3. Call Bedrock for Embeddings
   └─ Converts query to vector
   └─ Calls AWS Bedrock (or local stub)
   
4. Call Knowledge Service
   └─ POST /rag/search
   └─ Passes: query embedding, knowledge groups
   
5. Knowledge Service
   └─ Searches pgvector database
   └─ Finds similar chunks
   └─ Enriches with MongoDB metadata
   └─ Returns ranked results
   
6. Agent Processes Results
   └─ Selects top chunks for context
   └─ Formats context for LLM
   └─ Calls Bedrock for response
   └─ Stores conversation in MongoDB
   
7. Return to Frontend
   └─ Response displayed to user
   └─ Sources shown for verification
```

### Flow 2: User Uploads a Document

```
1. Frontend (React)
   └─ User selects file
   └─ Uploads to Knowledge Service
   
2. Knowledge Service
   └─ Receives file from S3
   └─ Detects file type (PDF/DOCX/PPTX/JSONL)
   └─ Updates status: "in_progress" in MongoDB
   
3. Background Task
   a) Extract text
      └─ Uses appropriate extractor (PyMuPDF, python-docx, etc.)
   
   b) Split into chunks
      └─ Semantic chunking (sentence boundaries)
      └─ 200-500 tokens per chunk
   
   c) Generate embeddings
      └─ For each chunk, call Bedrock
      └─ Get 1024-dim vector
   
   d) Store in PostgreSQL
      └─ Insert vectors + content
      └─ Add metadata (doc_id, chunk_id, source)
   
   e) Update metadata in MongoDB
      └─ Set status: "ready"
      └─ Record chunk count
      └─ Set completion timestamp
   
4. Document Ready for Search
   └─ User can now search across this document
```

### Flow 3: Multi-Turn Conversation

```
Conversation history:
User:       "Tell me about grants"
Assistant:  "[Returns grant info from documents]"
User:       "What's the deadline?" 
Assistant:  "[Uses conversation context + new search]"
User:       "For which programs?"
Assistant:  "[Uses full conversation history for context]"

Agent Service:
- Stores each turn in MongoDB
- Maintains conversation_id
- Passes full history to LLM with new question
- LLM can reference previous turns
- User sees threaded conversation
```

---

## Service Dependencies

### Agent → Knowledge Service

**When Agent needs to search:**
```python
# Agent calls Knowledge
GET http://knowledge:8085/rag/search
{
  "query": "user question",
  "knowledge_group_ids": ["group1", "group2"],
  "max_results": 5
}

# Returns ranked chunks
[
  {
    "content": "...",
    "similarity_score": 0.95,
    "document_id": "doc-123"
  },
  ...
]
```

### Agent → Bedrock

**For embeddings and chat completions:**
```
POST http://bedrock:4566/embeddings
{
  "model": "amazon.titan-embed-text-v2:0",
  "text": "user query"
}

Returns: [0.123, 0.456, ..., 0.789]  # 1024 dimensions
```

### Knowledge → Bedrock

**For generating embeddings:**
```
Same as above, called for each document chunk
```

### Frontend → Agent

**For chat API:**
```
POST http://agent:8086/chat
{
  "message": "user message",
  "conversation_id": "conv-123",
  "user_id": "user@example.com"
}

Returns:
{
  "response": "assistant response",
  "conversation_id": "conv-123",
  "sources": [...]
}
```

### All → Databases

**MongoDB (Conversations & Metadata)**
```
- Document collections
- Conversation history
- Knowledge groups
- User sessions
```

**PostgreSQL + pgvector (Vectors)**
```
- Document chunks
- Embeddings (1024 dims each)
- Metadata as JSONB
- Full-text search indexes
```

**Redis (Cache)**
```
- Session tokens
- Cached embeddings
- Rate limiting counters
```

---

## Data Flow Diagrams

### Complete Request Flow

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  1. User asks in Frontend                          │
│     ↓                                               │
│  2. Frontend → Agent API (POST /chat)              │
│     ↓                                               │
│  3. Agent → Bedrock (embed query)                  │
│     ↓                                               │
│  4. Agent → Knowledge (POST /rag/search)           │
│     ↓                                               │
│  5. Knowledge → PostgreSQL (find similar chunks)   │
│     ↓                                               │
│  6. Knowledge → MongoDB (get metadata)             │
│     ↓                                               │
│  7. Knowledge → Agent (return results)             │
│     ↓                                               │
│  8. Agent → Bedrock (generate response)            │
│     ↓                                               │
│  9. Agent → MongoDB (store conversation)           │
│     ↓                                               │
│ 10. Agent → Frontend (return response)             │
│     ↓                                               │
│ 11. Frontend displays to user                      │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Document Processing Flow

```
┌──────────────────────────────────────────────────┐
│                                                  │
│  1. User uploads file in Frontend               │
│     ↓                                            │
│  2. File sent to S3 (via LocalStack in dev)    │
│     ↓                                            │
│  3. Knowledge detects new file                  │
│     ↓                                            │
│  4. Knowledge → MongoDB (status: in_progress)  │
│     ↓                                            │
│  5. Extract text (format-specific)              │
│     ↓                                            │
│  6. Chunk text (semantic boundaries)            │
│     ↓                                            │
│  7. For each chunk:                             │
│       ├─ Bedrock (embed)                        │
│       ├─ PostgreSQL (store vector + content)   │
│       └─ MongoDB (update chunk count)           │
│     ↓                                            │
│  8. MongoDB (status: ready)                     │
│     ↓                                            │
│  9. Document ready for search                   │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## Concurrency & Performance

### Parallel Processing

**Document Ingestion:**
- Multiple workers process documents in parallel
- Each chunk embedded independently
- Batch inserts to PostgreSQL for efficiency
- Status updates to MongoDB as chunks complete

**Search:**
- Vector search uses PostgreSQL indexes (pgvector IVFFLAT)
- Similarity search is O(log n) complexity
- Results fetched from MongoDB in parallel
- Response time: <500ms for typical queries

### Caching Strategy

**Cached in Redis:**
- User sessions (expiry: 24 hours)
- Recent search results (expiry: 1 hour)
- Embedding cache (expiry: 7 days)
- Rate limiting counters (expiry: 1 minute)

**Benefits:**
- Reduced Bedrock API calls
- Faster response times
- Lower operational costs
- Improved user experience

---

## Scalability Architecture

### Horizontal Scaling

**Current Setup (Development):**
```
Single instance of each service
Suitable for: Development, testing, small deployments
```

**Production Scaling (Recommended):**
```
Frontend:
  - 3-5 replicas behind load balancer
  - Stateless (sessions in shared storage)

Agent:
  - 2-4 replicas behind load balancer
  - Stateless (state in MongoDB)

Knowledge:
  - 1-2 primary replicas (read-intensive)
  - Read replicas for scaling search

Databases:
  - PostgreSQL: primary + read replicas
  - MongoDB: replica set (3 nodes minimum)
  - Redis: cluster for distributed cache
```

### Load Distribution

```
Traffic → Load Balancer (Nginx/HAProxy)
  ├─ 30% Frontend requests (static assets, low load)
  ├─ 50% Agent requests (chat, CPU-intensive)
  └─ 20% Knowledge requests (search, I/O intensive)

Each service scales independently based on demand
```

---

## Error Handling & Resilience

### Service Failures

**If Bedrock is down:**
- Agent returns cached responses
- User sees: "Response from cache, may be outdated"
- System remains operational

**If Knowledge is down:**
- Agent can't search documents
- Returns: "Search unavailable, try again later"
- Chat still works (no document context)

**If MongoDB is down:**
- Conversations can't be stored
- User can still chat
- History lost after session ends
- Warning shown to user

**If PostgreSQL is down:**
- Vector search unavailable
- Returns: "Document search unavailable"
- Chat via Bedrock alone still works

### Graceful Degradation

```
Full Functionality:
  ✓ Chat + Search + Documents

If Bedrock Down:
  ✓ Chat (cached) + Search (cached)

If Knowledge Down:
  ✓ Chat (via Bedrock only)

If MongoDB Down:
  ✓ Chat (single session only)

If PostgreSQL Down:
  ✓ Chat (single session only)
```

---

## Monitoring & Observability

### Key Metrics to Monitor

**Performance:**
- API response time (target: <500ms)
- Database query time (target: <100ms)
- Bedrock API latency (typical: 500-2000ms)

**Reliability:**
- Service uptime (target: 99.5%)
- Error rate (target: <0.1%)
- Database connection pool usage

**Usage:**
- Requests per minute
- Active users
- Documents processed
- Average search results returned

**Business:**
- User adoption rate
- Average session duration
- Search success rate
- Cost per search

### Logging Strategy

```
Frontend:
  - Client-side errors
  - User interactions
  - API calls

Agent:
  - Chat requests
  - Search calls
  - Bedrock API usage
  - Conversation storage

Knowledge:
  - Document uploads
  - Search queries
  - Database operations
  - Embedding generation
```

---

## Testing Strategy

### Unit Tests
Each service has unit tests for business logic

### Integration Tests
Test service interactions:
- Agent → Knowledge
- Agent → Bedrock
- Knowledge → PostgreSQL

### End-to-End Tests
Test full user journeys:
- Upload document → Search → Find answer
- Multi-turn conversation
- Error scenarios

### Performance Tests
Load testing with:
- 100+ concurrent users
- 1000+ documents
- Large query volumes

---

## Deployment Checklist

- [ ] All services containerized (Docker images built)
- [ ] Docker Compose configuration tested
- [ ] Environment variables configured
- [ ] Database migrations applied
- [ ] Sample data loaded
- [ ] API endpoints verified
- [ ] Tests passing (unit + integration)
- [ ] Performance tests acceptable
- [ ] Security review completed
- [ ] Monitoring configured
- [ ] Backup strategy in place
- [ ] Disaster recovery plan written

---

## Support & Operations

### Daily Operations
- Monitor service health
- Check error logs
- Verify backups
- Monitor resource usage

### Weekly Maintenance
- Review performance metrics
- Check for failed jobs
- Update dependencies
- Review security alerts

### Monthly Tasks
- Database optimization
- Cache cleanup
- User feedback review
- Plan improvements

### Quarterly Reviews
- Architecture review
- Performance analysis
- Security audit
- Capacity planning

---

**Integration Guide Version:** 1.0.0  
**Last Updated:** March 20, 2026  
**Generated by:** code-documentation-generator skill

