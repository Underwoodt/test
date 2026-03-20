# Architecture - AI DEFRA Search

**System architecture, components, and design overview**

---

## Quick Overview

**AI DEFRA Search** uses a microservices architecture with clear separation between:
- **Frontend** - User interface
- **Agent** - Chat orchestration
- **Knowledge** - Search & ingestion
- **Data layer** - Storage & search

---

## High-Level Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Frontend (React)                    │
│              Web UI on Port 3000                     │
└─────────────────────────┬───────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ↓                 ↓                 ↓
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Agent        │  │ Knowledge    │  │ Bedrock      │
│ (Chat API)   │  │ (RAG Search) │  │ (LLM)        │
│ Port: 8086   │  │ Port: 8085   │  │ Port: 4566   │
└──────┬───────┘  └──────┬───────┘  └──────────────┘
       │                 │
       └─────────────────┼─────────────────┐
                         │                 │
        ┌────────────────┼────────────────┐│
        │                │                ││
        ↓                ↓                ↓↓
    ┌──────────────────────────────────────────┐
    │        Shared Data Infrastructure        │
    ├──────────────────────────────────────────┤
    │ • PostgreSQL + pgvector (vectors)        │
    │ • MongoDB (conversations)                │
    │ • Redis (cache & sessions)               │
    │ • S3 / LocalStack (files)                │
    └──────────────────────────────────────────┘
```

---

## Architectural Pattern

**Microservices Architecture** - Each service has:
- Single responsibility
- REST API interface
- Independent deployment
- Independent scaling

**Benefits:**
- Team independence
- Technology flexibility
- Fault isolation
- Easy to test

---

## Key Design Decisions

### 1. Frontend Separation
**Why:** Decouples UI from business logic
- React handles all UI state
- API-first design
- Can deploy frontend independently

### 2. FastAPI for Services
**Why:** Modern, fast, with automatic documentation
- Type safety with Python hints
- AsyncIO for high concurrency
- Auto-generated OpenAPI docs
- Growing ecosystem

### 3. Vector Database (pgvector)
**Why:** Native vector support for semantic search
- SQL interface (familiar)
- ACID transactions
- Scales to millions of vectors
- Proven at scale

### 4. MongoDB for Metadata
**Why:** Flexible document storage
- Schema flexibility
- Good for chat data (nested arrays)
- Horizontal scaling
- Rich query language

### 5. Stateless Services
**Why:** Enable easy horizontal scaling
- All state in external databases
- Services can restart anytime
- Load balanced traffic distribution

---

## Component Details

### Frontend Service
**Technology:** React + TypeScript
**Port:** 3000
**Responsibilities:**
- Chat interface
- File uploads
- Session management
- Real-time updates

### Agent Service
**Technology:** FastAPI + Python
**Port:** 8086
**Responsibilities:**
- Chat endpoint
- Conversation management
- LLM orchestration
- Knowledge integration

### Knowledge Service
**Technology:** FastAPI + Python
**Port:** 8085
**Responsibilities:**
- Document ingestion
- Semantic search
- Embedding generation
- Metadata management

### Bedrock Service
**Technology:** AWS Bedrock / WireMock
**Port:** 4566 (stub) / AWS endpoint (prod)
**Responsibilities:**
- Text embeddings
- Chat completions
- Model management

---

## Data Flow

### When User Asks a Question

```
1. User types in Frontend
2. Frontend → Agent API (POST /chat)
3. Agent generates query embedding via Bedrock
4. Agent calls Knowledge (POST /rag/search)
5. Knowledge searches PostgreSQL
6. Knowledge enriches with MongoDB metadata
7. Knowledge returns results to Agent
8. Agent calls Bedrock to generate response
9. Agent stores in MongoDB
10. Response sent to Frontend
11. Frontend displays to user
```

**Total time:** ~2-3 seconds

### When User Uploads Document

```
1. Frontend uploads file to S3
2. Frontend notifies Knowledge service
3. Knowledge detects file (background job)
4. Extracts text (format-specific)
5. Splits into chunks (semantic)
6. For each chunk:
   - Generates embedding via Bedrock
   - Stores in PostgreSQL + pgvector
7. Updates metadata in MongoDB
8. Document ready for search
```

**Time:** Background processing (async)

---

## Scaling Considerations

### Horizontal Scaling (Add More Servers)

**Frontend:**
- Multiple React instances behind load balancer
- Stateless (sessions in Redis)

**Agent:**
- Multiple FastAPI instances
- Stateless (state in MongoDB)

**Knowledge:**
- Multiple instances for reading
- Primary for writing
- Read replicas for scaling

**Databases:**
- PostgreSQL: Master + read replicas
- MongoDB: Replica set
- Redis: Cluster

### Vertical Scaling (Bigger Servers)

- Upgrade CPU/memory
- Use faster storage
- Optimize queries

---

## Fault Tolerance

### Service Fails

**If Agent down:** Chat unavailable, Knowledge still works
**If Knowledge down:** Search unavailable, chat via LLM only
**If Bedrock down:** Use cached embeddings, fallback responses
**If PostgreSQL down:** Use read replica, cache results
**If MongoDB down:** Conversations lost after session

### Graceful Degradation

System continues operating with reduced functionality:
- Cache results when services down
- Return errors with explanations
- Maintain user sessions
- Log errors for investigation

---

## Security Architecture

### Authentication
- User ID header on requests
- SSO/OAuth recommended (not in scope)
- Session tokens in Redis

### Authorization
- Each user isolated data
- Documents private by default
- Access control via MongoDB

### Data Protection
- HTTPS/TLS for transit
- Encryption at rest (configurable)
- Database access restricted

### Audit Trail
- All searches logged
- Uploads tracked
- Access logs maintained

---

## Technology Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Frontend** | React + TypeScript | Web UI |
| **API** | FastAPI + Python | Backend services |
| **Search** | PostgreSQL + pgvector | Vector storage |
| **Metadata** | MongoDB | Document storage |
| **Cache** | Redis | Session & cache |
| **Storage** | S3 / LocalStack | File storage |
| **LLM** | AWS Bedrock | Embeddings & chat |
| **Container** | Docker | Deployment |
| **Orchestration** | Docker Compose (dev) / Kubernetes (prod) | Runtime |
| **Testing** | Playwright / JMeter | Quality assurance |

---

## Performance Targets

| Operation | Target | Typical |
|-----------|--------|---------|
| Chat response | <2s | 500ms-1.5s |
| Vector search | <100ms | 20-50ms |
| Upload | Immediate | <100ms |
| API latency | <50ms | 10-20ms |

---

## Future Improvements

### Near Term
- Rate limiting
- Advanced filters
- Batch processing

### Medium Term
- Real-time collaboration
- Custom embeddings
- Advanced analytics

### Long Term
- Multi-region deployment
- Distributed architecture
- Enterprise features

---

**Document Version:** 1.0.0  
**Last Updated:** March 20, 2026

