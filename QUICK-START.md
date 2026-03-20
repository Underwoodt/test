# Quick Start Guide - AI DEFRA Search

**Get up to speed in 5 minutes**

---

## What is AI DEFRA Search?

An AI-powered document search platform that helps you find information using natural language questions instead of keywords.

**Example:** Instead of searching "grant application", you ask "How do I apply for grants?" and get relevant answers.

---

## System at a Glance

```
User's Computer
     ↓
[React Frontend] (your browser)
     ↓
[FastAPI Agent] (chat service)
     ↓
[FastAPI Knowledge] (search service)
     ↓
[PostgreSQL + pgvector] (vector search)
[MongoDB] (data storage)
[AWS Bedrock] (AI models)
```

---

## Key Components

| Component | Role | Port |
|-----------|------|------|
| **Frontend** | Web UI for users | 3000 |
| **Agent** | Chat API, conversation management | 8086 |
| **Knowledge** | Document search, ingestion | 8085 |
| **Databases** | Data storage (Postgres, Mongo, Redis) | Various |
| **Bedrock** | AI embeddings & chat completions | 4566 |

---

## 7 Repositories You Need

1. **ai-defra-search-core** - Orchestration & infrastructure
2. **ai-defra-search-frontend** - React web interface
3. **ai-defra-search-agent** - Chat API (FastAPI)
4. **ai-defra-search-knowledge** - Search & ingestion (FastAPI)
5. **ai-defra-search-journey-tests** - End-to-end tests
6. **ai-defra-search-perf-tests** - Performance tests
7. **ai-defra-search-aws-bedrock-stub** - Bedrock mock for local dev

---

## What Each Service Does

### Frontend (React)
- User interface
- File upload
- Chat interface
- Session management

### Agent (FastAPI)
- Chat endpoint
- Conversation management
- Calls Knowledge for search
- Calls Bedrock for AI

### Knowledge (FastAPI)
- Document ingestion
- Vector search
- RAG (Retrieval-Augmented Generation)
- Metadata management

### Databases
- **PostgreSQL + pgvector:** Stores document vectors
- **MongoDB:** Stores conversations & metadata
- **Redis:** Caching & sessions
- **S3:** Document storage

---

## How It Works: Step by Step

### When User Asks a Question

```
1. User types: "What's the grant deadline?"
   ↓
2. Frontend sends to Agent API
   ↓
3. Agent converts question to AI embedding
   ↓
4. Agent calls Knowledge: "Search for grant deadline info"
   ↓
5. Knowledge searches PostgreSQL for similar documents
   ↓
6. Knowledge returns top matches with relevance scores
   ↓
7. Agent calls Bedrock: "Generate response based on these documents"
   ↓
8. Bedrock returns AI-generated answer
   ↓
9. Agent stores conversation in MongoDB
   ↓
10. Frontend displays answer with sources
```

### When User Uploads a Document

```
1. User uploads PDF
   ↓
2. File goes to S3 storage
   ↓
3. Knowledge Service detects file
   ↓
4. Extract text from PDF
   ↓
5. Split into chunks (semantic boundaries)
   ↓
6. For each chunk:
   - Generate embedding via Bedrock
   - Store in PostgreSQL + pgvector
   - Index for fast search
   ↓
7. Update MongoDB with status
   ↓
8. Document ready for search
```

---

## Technology Stack

### Languages & Frameworks
- **Frontend:** React + TypeScript
- **Backend:** FastAPI (Python)
- **Testing:** Playwright, JMeter
- **Containers:** Docker, Docker Compose

### Databases
- **PostgreSQL:** Relational DB + pgvector (vectors)
- **MongoDB:** NoSQL for flexible data
- **Redis:** In-memory cache

### AI/ML
- **AWS Bedrock:** Embeddings & LLM
- **pgvector:** Vector similarity search

### DevOps
- **Docker:** Containerization
- **Docker Compose:** Local orchestration
- **Kubernetes:** Production scaling

---

## Setup Overview

### Prerequisites
- Docker & Docker Compose
- Python 3.13+ (recommended: use uv)
- Git

### Installation (3 steps)

**Step 1: Clone core repo**
```bash
git clone https://github.com/DEFRA/ai-defra-search-core
cd ai-defra-search-core
```

**Step 2: Clone all services**
```bash
uv run task clone
# This clones all 7 repos into services/
```

**Step 3: Start Docker Compose**
```bash
docker compose up -d
```

**Done!** Access at: http://frontend.localhost

---

## First Steps

### 1. Verify Everything Works
```bash
# Check all services are running
docker compose ps

# Check Agent API health
curl http://localhost:8086/health

# Check Knowledge API health
curl http://knowledge.localhost/health
```

### 2. Access the UI
- Open browser: http://frontend.localhost
- You should see the chat interface

### 3. Upload a Document
- Click "Upload Document"
- Choose a PDF or Word file
- Wait for processing

### 4. Ask a Question
- Type a natural language question
- System searches and returns answers

### 5. Check the Logs
```bash
# View all logs
docker compose logs -f

# View specific service
docker compose logs -f agent
docker compose logs -f knowledge
```

---

## Directory Structure After Clone

```
ai-defra-search-core/
├── compose.yml              ← Start with: docker compose up
├── services/
│   ├── ai-defra-search-frontend/
│   ├── ai-defra-search-agent/
│   ├── ai-defra-search-knowledge/
│   ├── ai-defra-search-journey-tests/
│   ├── ai-defra-search-perf-tests/
│   └── ai-defra-search-aws-bedrock-stub/
├── ARCHITECTURE.md          ← Read this for design
└── README.md               ← Original readme
```

---

## Key Concepts

### Vector Search
Converting text to numbers (embeddings) so we can find similar documents mathematically.

### RAG (Retrieval-Augmented Generation)
1. Find relevant documents (retrieval)
2. Use them as context for AI response (augmentation)
3. Generate response using both context and AI (generation)

### Microservices
Multiple independent services that communicate via APIs:
- Frontend service
- Chat service (Agent)
- Search service (Knowledge)
- Each can be updated/scaled independently

### PostgreSQL + pgvector
PostgreSQL database with pgvector extension for vector similarity search:
- Store document chunks as vectors (1024 dimensions)
- Fast similarity search using cosine distance
- Scales to millions of documents

---

## Common Tasks

### Change Code and Restart
```bash
cd services/ai-defra-search-agent
# Make code changes
docker compose restart agent
```

### Check Logs for Errors
```bash
docker compose logs agent | grep -i error
```

### Run Tests
```bash
cd services/ai-defra-search-agent
pytest

# Or E2E tests
cd services/ai-defra-search-journey-tests
npm test
```

### Access Database
```bash
# PostgreSQL
docker compose exec postgres psql -U postgres

# MongoDB
docker compose exec mongo mongosh
```

### Stop Everything
```bash
docker compose down
```

---

## Next Steps

### Read Next
1. [setup/INSTALLATION.md](../setup/INSTALLATION.md) - Detailed setup guide
2. [docs/ARCHITECTURE.md](../docs/ARCHITECTURE.md) - System architecture
3. [api-reference/AGENT-API.md](../api-reference/AGENT-API.md) - API reference

### Explore
- Look at `services/ai-defra-search-frontend/src/` for React code
- Look at `services/ai-defra-search-agent/app/` for API code
- Look at `services/ai-defra-search-knowledge/app/` for search code

### Make Changes
- Edit code in services/
- Code reloads automatically (with hot reload for React)
- Run tests to verify changes

---

## Troubleshooting Quick Fix

### Services won't start
```bash
# Check logs
docker compose logs

# Clean up and restart
docker compose down
docker compose up -d
```

### Can't access http://frontend.localhost
```bash
# Check Traefik is running
docker compose ps | grep traefik

# Verify DNS
nslookup frontend.localhost
# or edit /etc/hosts: 127.0.0.1 frontend.localhost
```

### Port already in use
```bash
# Find what's using port 3000
lsof -i :3000

# Stop the other process or use different port
```

---

## Key URLs (Local Development)

| Service | URL |
|---------|-----|
| **Frontend** | http://frontend.localhost |
| **Agent Swagger** | http://localhost:8086/docs |
| **Knowledge Swagger** | http://knowledge.localhost/docs |
| **MongoDB Express** | http://localhost:8081 |
| **Traefik Dashboard** | http://localhost:8080 |

---

## What to Learn Next

### For Developers
1. Architecture design
2. API endpoints
3. Data models
4. Code structure

### For DevOps
1. Docker setup
2. Environment config
3. Database setup
4. Monitoring

### For Architects
1. System design
2. Data flows
3. Service interactions
4. Scalability

---

## Summary

✅ **AI DEFRA Search** = AI-powered document search platform  
✅ **7 services** working together in microservices architecture  
✅ **3 main APIs** = Frontend, Agent, Knowledge  
✅ **Quick to setup** = 15 minutes from clone to running  
✅ **Easy to develop** = Hot reload, automatic testing  

---

## Need More Detail?

- **Setup:** See [setup/INSTALLATION.md](../setup/INSTALLATION.md)
- **Architecture:** See [docs/ARCHITECTURE.md](../docs/ARCHITECTURE.md)
- **APIs:** See [api-reference/](../api-reference/)
- **Deployment:** See [deployment/](../deployment/)
- **Troubleshooting:** See [troubleshooting/COMMON-ISSUES.md](../troubleshooting/COMMON-ISSUES.md)

---

**Generated:** March 20, 2026  
**Status:** ✅ Ready to use  

