# Documentation - System Overview

**Complete documentation about the AI DEFRA Search system**

---

## What's in This Section?

| File | Purpose | Audience |
|------|---------|----------|
| **ARCHITECTURE.md** | System design & components | Architects, Tech Leads |
| **SERVICES.md** | Detailed service descriptions | Developers, DevOps |
| **TECHNOLOGIES.md** | Tech stack & decisions | All technical staff |

---

## Quick Navigation

### I want to understand...

- **System design** → Read [ARCHITECTURE.md](ARCHITECTURE.md)
- **Individual services** → Read [SERVICES.md](SERVICES.md)
- **Technology choices** → Read [TECHNOLOGIES.md](TECHNOLOGIES.md)

---

## Document Index

### ARCHITECTURE.md (5 min read)

**What:** System architecture overview  
**Covers:**
- High-level design diagram
- Microservices pattern
- Component roles
- Data flow overview
- Scaling considerations
- Fault tolerance
- Security architecture

**Best for:** Understanding how the system works at a high level

### SERVICES.md (8 min read)

**What:** Detailed description of each service  
**Covers:**
- Frontend (React UI)
- Agent (Chat API)
- Knowledge (Search & RAG)
- Testing services
- Infrastructure service

**Best for:** Understanding what each service does and why

### TECHNOLOGIES.md (10 min read)

**What:** Technology stack and decisions  
**Covers:**
- Frontend stack (React, TypeScript)
- Backend stack (FastAPI, Python)
- Databases (PostgreSQL, MongoDB, Redis)
- AI/ML (Bedrock, embeddings)
- Container & orchestration
- Testing tools
- Why each technology was chosen

**Best for:** Understanding technical decisions and learning what tools are used

---

## Reading Paths

### For Developers
1. Read: ARCHITECTURE.md (5 min)
2. Read: SERVICES.md (8 min)
3. Read: TECHNOLOGIES.md (10 min)
4. Navigate to: [setup/](../setup/) for getting started
5. Navigate to: [api-reference/](../api-reference/) for API details

**Total time:** ~30 minutes

### For DevOps/SRE
1. Read: ARCHITECTURE.md - focus on "Scaling" section (3 min)
2. Read: TECHNOLOGIES.md - focus on "Container & Orchestration" (5 min)
3. Read: SERVICES.md - skim (3 min)
4. Navigate to: [deployment/](../deployment/) for deployment guides

**Total time:** ~15 minutes

### For Architects
1. Read: ARCHITECTURE.md (complete) (5 min)
2. Read: TECHNOLOGIES.md (complete) (10 min)
3. Skim: SERVICES.md (3 min)
4. Navigate to: [architecture/](../architecture/) for detailed design docs

**Total time:** ~20 minutes

---

## Key Concepts

### Microservices Architecture
System broken into independent services:
- Each service has one responsibility
- Services communicate via REST APIs
- Each service can be deployed independently
- Each service can be scaled independently

### Vector/Semantic Search
Finds relevant information using AI embeddings:
- Convert text to vectors (1024 dimensions)
- Compare vectors using similarity
- Return most similar results

### RAG (Retrieval-Augmented Generation)
Uses AI to answer questions grounded in documents:
1. User asks question
2. System searches for relevant documents
3. AI generates answer based on documents
4. Answer includes source attribution

---

## System Layers

```
┌─────────────────────────────────────┐
│    Frontend Layer (React UI)        │
├─────────────────────────────────────┤
│    API Layer (FastAPI services)     │
│  - Chat/Agent                       │
│  - Search/Knowledge                 │
├─────────────────────────────────────┤
│    Data Layer (Databases)           │
│  - PostgreSQL (vectors)             │
│  - MongoDB (metadata)               │
│  - Redis (cache)                    │
│  - S3 (files)                       │
├─────────────────────────────────────┤
│    AI Layer (AWS Bedrock)           │
│  - Embeddings                       │
│  - Chat completions                 │
└─────────────────────────────────────┘
```

---

## Service Communication

```
Frontend
  ↓
Agent API
  ├─ Calls Knowledge (search)
  ├─ Calls Bedrock (embeddings & chat)
  └─ Stores in MongoDB
  
Knowledge API
  ├─ Searches PostgreSQL (vectors)
  ├─ Enriches from MongoDB (metadata)
  ├─ Calls Bedrock (embeddings)
  └─ Stores in PostgreSQL + MongoDB
```

---

## Technology Stack at a Glance

| Layer | Technologies |
|-------|--------------|
| **Frontend** | React, TypeScript, Vite |
| **Backend** | FastAPI, Python 3.13+, Pydantic |
| **Search** | PostgreSQL, pgvector, Cosine similarity |
| **Storage** | MongoDB, Redis, S3 |
| **AI** | AWS Bedrock, Titan Embed, Claude |
| **Container** | Docker, Docker Compose |
| **Orchestration** | Kubernetes (production) |
| **Testing** | pytest, Jest, Playwright, JMeter |

---

## Performance Targets

| Operation | Target |
|-----------|--------|
| Chat response | < 2 seconds |
| Vector search | < 100 ms |
| API latency | < 50 ms |
| Document upload | Immediate |

---

## Next Steps

1. **Getting Started:** Go to [setup/QUICK-START.md](../setup/README.md)
2. **API Details:** Go to [api-reference/](../api-reference/)
3. **Deployment:** Go to [deployment/](../deployment/)
4. **Troubleshooting:** Go to [troubleshooting/](../troubleshooting/)

---

**Document Version:** 1.0.0  
**Last Updated:** March 20, 2026

