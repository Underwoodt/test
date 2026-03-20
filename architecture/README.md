# Architecture - System Design Deep Dive

**In-depth documentation of system architecture and design**

---

## What's in This Section?

| File | Purpose | Audience |
|------|---------|----------|
| **SYSTEM-DESIGN.md** | Complete system design | Architects |
| **DATA-FLOWS.md** | How data moves | Developers |
| **SERVICE-INTERACTIONS.md** | Service communication | Developers, DevOps |

---

## Document Index

### SYSTEM-DESIGN.md (Comprehensive Overview)

**Contains:**
- High-level system architecture with diagrams
- All 4 services explained in detail
- Data layer components (PostgreSQL, MongoDB, Redis, S3)
- 5 key design principles
- Technology decisions with justification
- Scaling strategy (horizontal & vertical)
- Performance characteristics
- Resilience & fault tolerance
- Security architecture

**Best for:** Understanding why the system is designed this way

**Key diagrams:**
- System architecture diagram
- Service responsibilities matrix
- Data flow overview

---

### DATA-FLOWS.md (How Data Moves)

**Contains:**
- Complete user ask flow (12 steps with timing)
- Complete document upload flow (9 steps)
- Multi-turn conversation handling
- Vector similarity search explained
- Document ingestion pipeline
- Chunking strategy
- Embedding process
- Error flows
- Data volume examples (dev, medium, large)

**Best for:** Understanding data movement through the system

**Includes:**
- Step-by-step flow diagrams
- Timing information
- Database queries shown
- API calls documented

---

### SERVICE-INTERACTIONS.md (Service Communication)

**Contains:**
- Service map showing all connections
- 3 communication patterns (sync, async, direct DB)
- Complete dependency graph
- All API contracts documented
- Synchronous interaction details
- Asynchronous patterns
- Error handling between services
- Resilience patterns (retry, timeout, cache, fallback)
- Best practices for service interaction

**Best for:** Understanding how services talk to each other

**Includes:**
- Service dependency diagrams
- API request/response examples
- Error handling code patterns
- Resilience strategies

---

## Reading Paths

### For System Architects
1. Read: SYSTEM-DESIGN.md (complete) - 15 min
2. Read: DATA-FLOWS.md (skim) - 5 min
3. Read: SERVICE-INTERACTIONS.md (skim) - 5 min
4. Reference: As needed

**Time:** 25 minutes

### For Backend Developers
1. Read: DATA-FLOWS.md - 10 min
2. Read: SERVICE-INTERACTIONS.md - 10 min
3. Skim: SYSTEM-DESIGN.md (scaling section) - 3 min
4. Reference: As you implement

**Time:** 23 minutes

### For DevOps Engineers
1. Read: SYSTEM-DESIGN.md (scaling & resilience) - 5 min
2. Read: SERVICE-INTERACTIONS.md (error handling) - 5 min
3. Skim: DATA-FLOWS.md - 3 min
4. Reference: During operations

**Time:** 13 minutes

---

## Key Concepts Explained

### Microservices Architecture
- System split into 4 independent services
- Each has one responsibility
- Communicate via REST APIs
- Deploy independently
- Scale independently

### Vector Embeddings
- Text converted to 1024-dimensional vectors
- Uses AWS Bedrock (Titan Embed V2)
- Enables semantic/meaning-based search
- ~500-1000ms per chunk

### Semantic Search (Vector Similarity)
- Compare vectors using cosine distance
- Find most similar chunks
- Return ranked by similarity
- Much better than keyword search

### Synchronous vs Asynchronous
**Synchronous:** Real-time operations (chat, search)
**Asynchronous:** Long operations (document ingestion)

### Stateless Services
- All state in external databases
- Services can restart anytime
- Easy to scale horizontally
- No local session storage

---

## System Layers

```
┌────────────────────────────────────────┐
│  Presentation Layer (React Frontend)   │
├────────────────────────────────────────┤
│  API Layer (FastAPI Services)          │
│  ├─ Agent (Chat orchestration)         │
│  └─ Knowledge (Search & ingestion)     │
├────────────────────────────────────────┤
│  AI Layer (AWS Bedrock)                │
│  ├─ Embeddings (Titan Embed V2)        │
│  └─ Chat (Claude)                      │
├────────────────────────────────────────┤
│  Data Layer (Databases)                │
│  ├─ PostgreSQL + pgvector (vectors)    │
│  ├─ MongoDB (metadata & conversations) │
│  ├─ Redis (cache & sessions)           │
│  └─ S3 (document storage)              │
└────────────────────────────────────────┘
```

---

## Data Movement Flows

### User Asks Question
```
Frontend → Agent API → Knowledge API
                         ↓
                    PostgreSQL (search)
                         ↓
                    MongoDB (metadata)
                         ↓
                    Agent (builds prompt)
                         ↓
                    Bedrock (generates response)
                         ↓
                    MongoDB (store conversation)
                         ↓
                    Frontend (display)
```

**Total time:** 2-3 seconds

### User Uploads Document
```
Frontend → S3 (upload)
                ↓
        Knowledge API (detects)
                ↓
        Background job
           ├─ Extract text
           ├─ Split chunks
           ├─ Generate embeddings (via Bedrock)
           ├─ Store in PostgreSQL
           └─ Store metadata in MongoDB
```

**Time:** Background processing (async)

---

## Service Interaction Patterns

### Pattern 1: Request-Response (Synchronous)
Used for real-time operations:
- Chat API calls
- Search queries
- Health checks

**Timeout:** 30 seconds for chat, 5-10 seconds for search

### Pattern 2: Fire-and-Forget (Asynchronous)
Used for long-running operations:
- Document ingestion
- Large file processing
- Background jobs

**Returns immediately,** processes in background

### Pattern 3: Direct Database Access
Used for data-heavy operations:
- Vector search in PostgreSQL
- Metadata lookups in MongoDB
- Cache gets/sets in Redis

---

## Design Decisions & Trade-offs

### Why PostgreSQL + pgvector?
**Pros:** Vector search native, ACID, proven at scale  
**Cons:** Not as flexible as NoSQL

### Why MongoDB for metadata?
**Pros:** Flexible schema, good for chat data  
**Cons:** Eventually consistent, not ACID

### Why stateless services?
**Pros:** Easy scaling, resilience  
**Cons:** More database load, slightly more latency

### Why async for ingestion?
**Pros:** UI responsive, can process large files  
**Cons:** Can't show immediate progress

---

## Performance Considerations

| Operation | Target | Actual | Notes |
|-----------|--------|--------|-------|
| Chat response | <2s | 1-2s | Includes network |
| Vector search | <100ms | 20-50ms | Fast with index |
| Embedding gen | N/A | 500-1000ms per chunk | Bedrock latency |
| Upload | Immediate | <100ms | Queued async |

---

## Scaling Strategy

### Horizontal Scaling (Add servers)
- Frontend: 2-10 instances
- Agent: 2-4 instances  
- Knowledge: 1-2 instances
- Databases: Replicas

### Vertical Scaling (Bigger servers)
- Before horizontal, optimize code
- Improve database indexes
- Tune query performance

### Load Distribution
- 30% frontend
- 50% agent (CPU-intensive)
- 20% knowledge (I/O-intensive)

---

## Resilience Patterns

### Circuit Breaker
Prevents cascading failures when service is down

### Retry with Backoff
Tries request multiple times with increasing delays

### Caching
Redis cache for results, embeddings, sessions

### Graceful Degradation
Continue with reduced functionality if parts fail

---

## Next Steps

1. **Implement:** Use DATA-FLOWS.md to understand how to implement features
2. **Debug:** Use SERVICE-INTERACTIONS.md for troubleshooting
3. **Scale:** Use SYSTEM-DESIGN.md for scaling strategies
4. **Optimize:** Reference documents as you optimize

---

**Document Version:** 1.0.0  
**Last Updated:** March 20, 2026
