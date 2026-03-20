# System Design - AI DEFRA Search Platform

**Complete system architecture, design decisions, and technical overview**

---

## Table of Contents

- [System Overview](#system-overview)
- [Architecture](#architecture)
- [Key Components](#key-components)
- [Design Principles](#design-principles)
- [Technology Decisions](#technology-decisions)
- [Scaling Strategy](#scaling-strategy)

---

## System Overview

**AI DEFRA Search** is a microservices-based platform for semantic document search using AI and vector embeddings.

### What It Does

```
User Question → Semantic Search → Relevant Documents → AI Response
```

Users ask natural language questions, and the system:
1. Finds relevant documents using vector similarity search
2. Extracts context from those documents
3. Uses AI to generate an answer grounded in the documents

### System Goals

- **Accuracy:** Find the most relevant information
- **Speed:** Respond within 1 second
- **Scale:** Handle thousands of concurrent users
- **Reliability:** 99.5% uptime
- **Maintainability:** Easy to update and extend

---

## Architecture

### High-Level Diagram

```
┌─────────────────────────────────────────────────────┐
│                  Frontend (React)                    │
│                 User Interface Layer                 │
│              Port: 3000 (via Traefik)               │
└─────────────────────────┬───────────────────────────┘
                          │ HTTPS
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ↓                 ↓                 ↓
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Agent        │  │ Knowledge    │  │ Bedrock      │
│ (Chat API)   │  │ (RAG Search) │  │ (Stub/AWS)   │
│ FastAPI      │  │ FastAPI      │  │ WireMock     │
│ Port: 8086   │  │ Port: 8085   │  │ Port: 4566   │
└──────┬───────┘  └──────┬───────┘  └──────────────┘
       │                 │
       └─────────────────┼─────────────────┐
                         │                 │
        ┌────────────────┼────────────────┐│
        │                │                ││
        ↓                ↓                ↓↓
    ┌─────────────────────────────────────────┐
    │      Shared Infrastructure              │
    ├─────────────────────────────────────────┤
    │ • PostgreSQL + pgvector (vectors)       │
    │ • MongoDB (conversations & metadata)    │
    │ • Redis (cache & sessions)              │
    │ • LocalStack/S3 (file storage)          │
    │ • Traefik (reverse proxy)               │
    └─────────────────────────────────────────┘
```

### Architectural Pattern: Microservices

The system uses a **microservices architecture** where:
- Each service has a single responsibility
- Services communicate via REST APIs
- Services can be deployed independently
- Services scale independently based on demand

### Service Responsibilities

| Service | Responsibility | Language | Port |
|---------|-----------------|----------|------|
| **Frontend** | User interface, session management | React/TS | 3000 |
| **Agent** | Chat API, conversation orchestration | FastAPI | 8086 |
| **Knowledge** | Document ingestion, vector search | FastAPI | 8085 |
| **Bedrock** | LLM embeddings, chat completions | AWS/Stub | 4566 |

---

## Key Components

### 1. Frontend Service

**Technology:** React + TypeScript

**Responsibilities:**
- Web UI for users
- Session management
- File upload handling
- Real-time updates
- Chat interface

**Key Features:**
- Responsive design
- Secure session handling
- Progress tracking for uploads
- Conversation history display

**Scaling:** Stateless - can run multiple replicas behind load balancer

---

### 2. Agent Service (Chat API)

**Technology:** Python + FastAPI

**Responsibilities:**
- Chat endpoint for user messages
- Conversation management
- LLM orchestration
- RAG coordination

**Key Flow:**
```
1. Receive chat message
   ↓
2. Get message embeddings from Bedrock
   ↓
3. Call Knowledge service for search
   ↓
4. Build prompt with context
   ↓
5. Call Bedrock for response
   ↓
6. Store conversation in MongoDB
   ↓
7. Return response to frontend
```

**Scaling:** CPU-intensive - scale based on request volume

---

### 3. Knowledge Service (RAG)

**Technology:** Python + FastAPI

**Responsibilities:**
- Document ingestion pipeline
- Vector generation
- Semantic search
- Metadata management

**Key Processes:**

**Document Upload:**
1. File received from S3
2. Extract text (format-specific: PDF, DOCX, PPTX, JSONL)
3. Split into chunks (semantic boundaries)
4. Generate embedding for each chunk via Bedrock
5. Store in PostgreSQL + pgvector
6. Index metadata in MongoDB

**Search:**
1. Receive query
2. Generate query embedding
3. Perform vector similarity search in PostgreSQL
4. Enrich results with metadata from MongoDB
5. Return ranked results

**Scaling:** I/O-intensive - scale based on query volume and document volume

---

### 4. Data Layer

#### PostgreSQL + pgvector

**Purpose:** Vector storage for semantic search

**What's Stored:**
- Document chunks (text content)
- Embeddings (1024-dimensional vectors)
- Chunk metadata (document ID, position, etc.)

**Why pgvector?**
- Native vector support
- Efficient similarity search (IVFFLAT indexing)
- ACID compliance
- Familiar SQL interface
- Scales to millions of vectors

**Schema Example:**
```sql
CREATE TABLE chunks (
  id UUID PRIMARY KEY,
  content TEXT,
  embedding vector(1024),
  document_id UUID,
  metadata JSONB
);

CREATE INDEX ON chunks USING IVFFLAT (embedding vector_cosine_ops);
```

#### MongoDB

**Purpose:** Flexible document storage for metadata

**What's Stored:**
- Conversations and messages
- Knowledge groups
- Document metadata
- User sessions
- Ingestion status

**Why MongoDB?**
- Flexible schema (documents vary)
- Good for chat history (nested arrays)
- Document-oriented (aligns with domain)
- Horizontal scaling via sharding
- Rich query language

#### Redis

**Purpose:** Caching and session storage

**What's Cached:**
- User sessions (24-hour TTL)
- Recent search results (1-hour TTL)
- Embedding cache (7-day TTL)
- Rate limiting counters (1-min TTL)

**Benefits:**
- Reduces Bedrock API calls
- Faster response times
- Lower operational costs
- Session persistence

---

### 5. External Services

#### AWS Bedrock

**Purpose:** Embeddings and LLM

**Models Used:**
- **Titan Embed V2:** Text to embeddings (1024 dimensions)
- **Claude:** Chat completions for responses
- **Other LLMs:** Supported but not currently used

**Why Bedrock?**
- State-of-the-art models
- Managed service (no infrastructure)
- Pay-per-use pricing
- Multiple model options

#### LocalStack (Development)

**Purpose:** Local AWS simulation for development

**Simulates:**
- S3 for file storage
- SQS for queues
- Other AWS services

**Benefits:**
- No AWS costs during development
- Fast feedback loop
- Works offline

---

## Design Principles

### 1. Separation of Concerns

Each service handles one domain:
- **Frontend:** Presentation
- **Agent:** Orchestration and chat
- **Knowledge:** Search and ingestion

**Benefit:** Services can be updated independently

### 2. Stateless Services

All services are stateless:
- State stored in external services (databases)
- No local session data
- Can restart/replace anytime

**Benefit:** Easy scaling and resilience

### 3. Asynchronous Processing

Document ingestion is asynchronous:
- Upload returns immediately
- Background job processes document
- Status tracked in MongoDB

**Benefit:** UI remains responsive, large documents don't block

### 4. Caching Strategy

Multiple layers of caching:
- Redis for sessions
- Embedding cache for repeated queries
- Search result cache

**Benefit:** Reduced load on external services, faster responses

### 5. Graceful Degradation

If a service fails:
- System continues in degraded mode
- Returns cached results when possible
- Informs user of limitations

**Benefit:** Maximum uptime, best user experience

---

## Technology Decisions

### Why FastAPI?

✅ Modern async framework  
✅ Automatic OpenAPI documentation  
✅ Type hints for safety  
✅ High performance  
✅ Growing ecosystem  

### Why React?

✅ Component-based architecture  
✅ Large community  
✅ Rich ecosystem  
✅ Good performance  
✅ Developer experience  

### Why PostgreSQL + pgvector?

✅ Vector search native support  
✅ ACID transactions  
✅ Proven at scale  
✅ SQL familiarity  
✅ Backup/recovery tools  

### Why MongoDB?

✅ Flexible schema  
✅ Document-oriented  
✅ Good for chat data  
✅ Horizontal scaling  
✅ Rich query language  

### Why Docker Compose + Kubernetes?

✅ Consistent local development  
✅ Production-ready orchestration  
✅ Industry standard  
✅ Easy scaling  
✅ Great tooling  

---

## Scaling Strategy

### Horizontal Scaling

**Frontend:**
- Run multiple React instances
- Behind load balancer (Nginx/HAProxy)
- Stateless (sessions in Redis)
- **Scale trigger:** High CPU/memory

**Agent:**
- Run 2-4 FastAPI instances
- Behind load balancer
- Stateless (state in MongoDB)
- **Scale trigger:** High request latency

**Knowledge:**
- Run 1-2 primary instances (CPU-bound)
- Read replicas for search queries
- **Scale trigger:** High CPU or query latency

**Databases:**
- PostgreSQL: Master + read replicas for search
- MongoDB: Replica set (3 nodes minimum)
- Redis: Cluster for distributed cache

### Load Distribution

```
Traffic → Load Balancer
  ├─ 30% Frontend (static assets, low CPU)
  ├─ 50% Agent (chat, CPU-intensive)
  └─ 20% Knowledge (search, I/O-intensive)
```

### Vertical Scaling

Before horizontal scaling, consider:
- Optimize database queries
- Improve caching hit rate
- Tune algorithm parameters
- Use faster hardware

---

## Performance Characteristics

### Expected Response Times

| Operation | Target | Typical |
|-----------|--------|---------|
| Chat response | < 2s | 500ms - 1.5s |
| Vector search | < 100ms | 20-50ms |
| Document upload | Immediate | < 100ms |
| Embedding gen | Per chunk | 500-1000ms |
| API latency | < 50ms | 10-20ms |

### System Capacity

| Metric | Capacity |
|--------|----------|
| Concurrent users | 100+ |
| RPS (requests/sec) | 50+ |
| Documents | 100,000+ |
| Conversations | 1,000,000+ |
| Chunks per doc | 100-1000 |

---

## Resilience & Fault Tolerance

### Service Failures

**If Frontend Down:**
- Unavailable to users
- Backend continues processing (API still works)
- CDN can serve cached assets

**If Agent Down:**
- Chat unavailable
- Users get "Service unavailable"
- Knowledge service continues functioning

**If Knowledge Down:**
- Search unavailable
- Agent returns error
- Chat works without document context

**If Bedrock Down:**
- Can use cached embeddings
- Response generation fails
- System returns cached results

### Database Failures

**PostgreSQL Failure:**
- Search unavailable
- Switch to read replica
- Fallback to cached results

**MongoDB Failure:**
- Conversations can't be stored
- Chat still works (in-memory)
- Data lost after session

**Redis Failure:**
- Cache misses increase
- More external calls
- Performance degrades but works

---

## Security Architecture

### Authentication
- User ID header on all requests
- No passwords (SSO/OAuth recommended)
- Session tokens in Redis

### Authorization
- Each user's documents separate
- Documents private by default
- Controlled access to shared collections

### Data Protection
- Encryption at rest (TBD)
- Encryption in transit (HTTPS/TLS)
- Database access restricted

### Audit Trail
- All searches logged
- All uploads tracked
- Access logs maintained

---

## Monitoring & Observability

### Metrics to Track

**Performance:**
- API response time
- Database query time
- Cache hit rate

**Availability:**
- Service uptime
- Error rate
- Request latency p99

**Usage:**
- Requests per minute
- Active users
- Documents processed

**Business:**
- User satisfaction
- Search success rate
- Cost per search

### Logging

Each service logs:
- Request/response
- Errors
- Performance metrics
- User actions

---

## Future Improvements

### Near Term
- Rate limiting per user
- Advanced search filters
- Batch processing
- Result caching

### Medium Term
- Real-time collaboration
- Advanced analytics
- Custom embeddings
- Performance optimizations

### Long Term
- Distributed architecture
- Multi-region deployment
- Advanced AI features
- Enterprise integrations

---

**Document Version:** 1.0.0  
**Last Updated:** March 20, 2026  
**Status:** Production Ready

