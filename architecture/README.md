# Architecture Documentation

System design, data flows, and service interactions.

## Files

- **SYSTEM-DESIGN.md** - Overall system architecture
- **DATA-FLOWS.md** - How data flows through the system
- **SERVICE-INTERACTIONS.md** - How services communicate

## Architecture Overview

```
Frontend (React) 
    ↓
Agent (Chat API)
    ├→ Knowledge (Search)
    ├→ Bedrock (LLM)
    └→ MongoDB (History)
         ↓
    PostgreSQL + pgvector (Vectors)
```

## Key Concepts

- Microservices architecture
- Semantic search using vector embeddings
- RAG (Retrieval-Augmented Generation)
- Document ingestion pipeline

---

See FULL-DOCUMENTATION.md → "Architecture" for complete details
