# AI DEFRA Search Platform - Developer Documentation

**Generated from analysis of 7 repositories using code-documentation-generator skill**

---

## Table of Contents

- [System Overview](#system-overview)
- [Architecture](#architecture)
- [Services](#services)
- [Setup Instructions](#setup-instructions)
- [API Reference](#api-reference)
- [Data Models](#data-models)
- [Development Workflow](#development-workflow)
- [Testing](#testing)
- [Deployment](#deployment)
- [Troubleshooting](#troubleshooting)

---

## System Overview

**AI DEFRA Search** is a microservices-based AI search platform enabling conversational search over documents using RAG (Retrieval-Augmented Generation).

### Platform Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  Frontend (React)                        │
│              (ai-defra-search-frontend)                 │
│                   Port: 3000                             │
└─────────────────────────┬───────────────────────────────┘
                          │
                    Traefik Proxy
                (frontend.localhost)
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ↓                 ↓                 ↓
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Agent        │  │ Knowledge    │  │ Bedrock Stub │
│ (Chat API)   │  │ (RAG Search) │  │ (Embeddings) │
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
    │ • MongoDB (Conversations, Metadata)     │
    │ • PostgreSQL + pgvector (Embeddings)    │
    │ • Redis (Cache)                         │
    │ • LocalStack/S3 (File Storage)          │
    └─────────────────────────────────────────┘
```

### Key Technologies

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Frontend** | React, TypeScript | Web UI, session management |
| **API Layer** | FastAPI (Python) | Chat API, RAG orchestration |
| **Search** | pgvector, PostgreSQL | Vector storage, similarity search |
| **Metadata** | MongoDB | Conversation history, documents |
| **LLM** | AWS Bedrock | Embeddings, chat completions |
| **Storage** | S3/LocalStack | Document storage |
| **Orchestration** | Docker Compose | Local development |
| **Testing** | Playwright, JMeter | E2E and performance testing |

---

## Architecture

### Service Dependencies

```
Frontend (React)
    ↓ calls
Agent (FastAPI)
    ↓ calls
Knowledge (FastAPI)
    ↓ calls
Bedrock (AWS/Stub)
    
Metadata Storage:
- MongoDB (conversations, knowledge groups)
- PostgreSQL + pgvector (embeddings, chunks)
- Redis (cache)
- S3 (raw documents)
```

### Data Flow: Conversation

```
1. User Input
   └→ Frontend sends to Agent

2. Agent Processing
   └→ Agent calls Bedrock for embeddings
   └→ Agent searches Knowledge for RAG context
   └→ Agent enriches prompt with context
   └→ Agent calls Bedrock for response
   └→ Agent stores conversation in MongoDB

3. Knowledge Service (on demand)
   └→ Receives search query from Agent
   └→ Generates embedding via Bedrock
   └→ Performs pgvector similarity search
   └→ Enriches results with metadata from MongoDB
   └→ Returns ranked chunks to Agent

4. Response
   └→ Agent returns to Frontend
   └→ Frontend displays to user
```

### Data Flow: Document Ingestion

```
1. Upload
   └→ Frontend uploads JSONL to S3

2. Knowledge Service
   └→ Detects file in S3
   └→ Reads pre-chunked JSONL
   └→ For each chunk:
       ├─ Calls Bedrock for embedding
       ├─ Stores vector + metadata in PostgreSQL
       └─ Updates MongoDB with document status

3. Result
   └→ Document ready for search
```

---

## Services

### 1. Frontend (`ai-defra-search-frontend`)

**Purpose:** User interface, session management  
**Language:** JavaScript/React + TypeScript  
**Port:** 3000 (via Traefik: `frontend.localhost`)  
**Key Files:**
- `src/pages/` - Page components
- `src/components/` - Reusable components
- `src/services/` - API client
- `.env.example` - Configuration template

**Environment Variables:**
```
REACT_APP_AGENT_API_URL=http://agent.localhost
REACT_APP_KNOWLEDGE_API_URL=http://knowledge.localhost
REACT_APP_BEDROCK_URL=http://bedrock.localhost
```

**Key Features:**
- Conversational UI for chat
- File upload for documents
- Session management
- Real-time updates

---

### 2. Agent (`ai-defra-search-agent`)

**Purpose:** Chat API, conversation management, LLM orchestration  
**Language:** Python, FastAPI  
**Port:** 8086  
**Key Files:**
- `app/chat/router.py` - Chat endpoints
- `app/chat/service.py` - Chat logic
- `app/bedrock/service.py` - LLM integration
- `app/config.py` - Configuration

**API Endpoints:**

#### POST /chat

```
Request:
{
  "message": "What documents do we have?",
  "conversation_id": "conv-123",
  "user_id": "user-456"
}

Response:
{
  "response": "We have 5 documents about...",
  "conversation_id": "conv-123",
  "sources": [...],
  "metadata": {...}
}
```

#### GET /conversations/{id}

Returns full conversation history.

**Dependencies:**
- AWS Bedrock (or LocalStack stub)
- Knowledge service at `KNOWLEDGE_BASE_URL`
- MongoDB for conversation storage

**Environment Variables:**
```
KNOWLEDGE_BASE_URL=http://knowledge:8085
BEDROCK_ENDPOINT=http://bedrock:4566
MONGO_URI=mongodb://mongo:27017
REDIS_URL=redis://redis:6379
```

---

### 3. Knowledge (`ai-defra-search-knowledge`)

**Purpose:** Document management, vector search, RAG  
**Language:** Python, FastAPI  
**Port:** 8085  
**Database:** PostgreSQL + pgvector, MongoDB  
**Key Files:**
- `app/knowledge_group/` - Document group management
- `app/ingest/` - Document ingestion pipeline
- `app/rag/` - Search endpoints
- `app/ingest/extractors/` - File type handlers (PDF, DOCX, PPTX, JSONL)
- `changelog/` - Liquibase database migrations

**API Endpoints:**

#### POST /documents

```
Request:
{
  "documents": [
    {
      "file_name": "doc.pdf",
      "knowledge_group_id": "group-123",
      "s3_key": "s3://bucket/doc.pdf"
    }
  ]
}

Response: 201 Created
```

#### POST /rag/search

```
Request:
{
  "query": "search term",
  "knowledge_group_ids": ["group-123"],
  "max_results": 5
}

Response:
[
  {
    "content": "...",
    "similarity_score": 0.92,
    "document_id": "doc-123",
    "file_name": "doc.pdf"
  },
  ...
]
```

**Processing Pipeline:**
1. Document upload to S3
2. Extraction (detect format, extract text)
3. Chunking (semantic boundaries)
4. Embedding (Bedrock Titan)
5. Storage (PostgreSQL + pgvector)
6. Metadata enrichment (MongoDB)

**Supported Formats:**
- PDF (PyMuPDF)
- DOCX (python-docx)
- PPTX (python-pptx)
- JSONL (pre-chunked)

---

### 4. Tests - Journey (`ai-defra-search-journey-tests`)

**Purpose:** End-to-end user journey testing  
**Tool:** Playwright  
**Key Test Scenarios:**
- User login and session
- Document upload
- Search and retrieval
- Response generation
- Multi-turn conversation

**Running Tests:**
```bash
npm test
# or with UI
npm run test:ui
```

---

### 5. Tests - Performance (`ai-defra-search-perf-tests`)

**Purpose:** Performance and load testing  
**Tool:** JMeter  
**Test Scenarios:**
- Concurrent searches
- Document upload throughput
- Response time under load
- System stability

**Running Tests:**
```bash
./run-tests-local-docker.sh
```

---

### 6. AWS Bedrock Stub (`ai-defra-search-aws-bedrock-stub`)

**Purpose:** Mock AWS Bedrock for local development  
**Tool:** WireMock  
**Port:** 4566  
**Mocked Endpoints:**
- `/embeddings` - Returns mock embeddings
- `/invoke-model` - Returns mock completions

**Configuration:** `wiremock/mappings/*.json`

---

### 7. Core (`ai-defra-search-core`)

**Purpose:** Orchestration, infrastructure, local development setup  
**Key Files:**
- `compose.yml` - Docker Compose configuration
- `tasks.py` - Orchestration tasks
- `ARCHITECTURE.md` - Architecture documentation
- `AGENTS.md` - Agent specifications

**Services Defined:**
- MongoDB
- PostgreSQL (with pgvector)
- Redis
- LocalStack (S3, SQS)
- Traefik (reverse proxy)

**Network:** `ai-defra-search` bridge network

---

## Setup Instructions

### Prerequisites

```bash
# Required
- Docker (v20.10+)
- Docker Compose (v2.0+)
- Python 3.13+
- Git

# Optional
- Node.js (for direct frontend development)
- uv (Python package manager)
```

### Step 1: Clone Core Repository

```bash
git clone https://github.com/DEFRA/ai-defra-search-core
cd ai-defra-search-core
```

### Step 2: Install Dependencies

```bash
# Using uv (recommended)
uv sync --frozen

# Or using pip
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Step 3: Clone Service Repositories

```bash
# Automated clone
uv run task clone

# Or manual clone
cd services
for repo in \
  ai-defra-search-frontend \
  ai-defra-search-agent \
  ai-defra-search-knowledge \
  ai-defra-search-journey-tests \
  ai-defra-search-perf-tests \
  ai-defra-search-aws-bedrock-stub; do
  git clone https://github.com/DEFRA/$repo
done
```

### Step 4: Start Services

```bash
# Start all services
docker compose up -d

# Wait for services to be healthy
docker compose ps

# View logs
docker compose logs -f
```

### Step 5: Verify Setup

```bash
# Frontend
curl http://frontend.localhost

# Agent API
curl http://localhost:8086/health

# Knowledge API
curl http://knowledge.localhost/health

# Database check
docker compose exec postgres psql -U postgres -c "SELECT version();"
```

**Expected response:**
```
{"status": "healthy"}
```

### Step 6: Access Application

**Frontend:** http://frontend.localhost  
**Agent Swagger:** http://localhost:8086/docs  
**Knowledge Swagger:** http://knowledge.localhost/docs  

---

## API Reference

### Authentication

Requests include `user-id` header:
```
user-id: alice@example.com
```

### Common Response Codes

```
200 OK          - Success
201 Created     - Resource created
400 Bad Request - Invalid input
404 Not Found   - Resource not found
500 Error       - Server error
502 Gateway     - Service unavailable
```

### Agent API

**Base URL:** `http://localhost:8086`

**POST /chat**
- Send chat message
- Request: `{message, conversation_id, user_id}`
- Response: `{response, conversation_id, sources, metadata}`

**GET /conversations/{id}**
- Get conversation history
- Response: Array of messages with timestamps

**GET /health**
- Health check
- Response: `{status: "healthy"}`

### Knowledge API

**Base URL:** `http://knowledge.localhost`

**POST /documents**
- Register document for ingestion
- Request: Array of document objects
- Response: 201 Created

**POST /rag/search**
- Semantic search
- Request: `{query, knowledge_group_ids, max_results}`
- Response: Array of ranked chunks with scores

**GET /upload-status/{upload_id}**
- Check ingestion status
- Response: Array with status, chunk count

**GET /health**
- Health check
- Response: `{status: "healthy"}`

---

## Data Models

### Conversation

```python
{
  "_id": ObjectId,
  "user_id": "user@example.com",
  "conversation_id": "conv-123",
  "messages": [
    {
      "role": "user",
      "content": "What documents exist?",
      "timestamp": "2024-03-20T12:00:00Z"
    },
    {
      "role": "assistant",
      "content": "We have 5 documents...",
      "timestamp": "2024-03-20T12:00:05Z",
      "sources": [...]
    }
  ]
}
```

### Document/Chunk

```python
{
  "id": "chunk-123",
  "content": "Text of document chunk...",
  "embedding": [0.123, 0.456, ...],  # 1024 dimensions
  "metadata": {
    "source": "document.pdf",
    "knowledge_group_id": "group-123",
    "document_id": "doc-123"
  },
  "similarity_score": 0.92  # On search
}
```

### Knowledge Group

```python
{
  "_id": ObjectId,
  "name": "My Documents",
  "created_by": "user@example.com",
  "created_at": "2024-03-20T12:00:00Z"
}
```

---

## Development Workflow

### Making Changes to Frontend

```bash
cd services/ai-defra-search-frontend

# Make code changes
# Auto-reloads on save (with hot reload)

# Run tests
npm test

# Build
npm run build
```

### Making Changes to Agent

```bash
cd services/ai-defra-search-agent

# Make code changes
# API auto-reloads with FastAPI reload

# Run tests
pytest

# Format code
ruff format .
ruff check .
```

### Making Changes to Knowledge

```bash
cd services/ai-defra-search-knowledge

# Make code changes
# API auto-reloads

# Run tests
pytest

# Database migrations managed by Liquibase
# Migrations in: changelog/
```

### Database Migrations

New database changes:
1. Create migration file in `changelog/`
2. Migrations auto-apply on service start
3. Schema tracked in `changelog/db.changelog-master.xml`

---

## Testing

### Unit Tests

```bash
# Agent
cd services/ai-defra-search-agent
pytest

# Knowledge
cd services/ai-defra-search-knowledge
pytest

# Frontend
cd services/ai-defra-search-frontend
npm test
```

### Integration Tests

```bash
cd services/ai-defra-search-knowledge
pytest --integration
```

### End-to-End Tests

```bash
cd services/ai-defra-search-journey-tests
npm test
```

### Performance Tests

```bash
cd services/ai-defra-search-perf-tests
./run-tests-local-docker.sh
```

---

## Deployment

### Docker Images

Each service has a `Dockerfile` for production deployment.

**Build:**
```bash
cd services/ai-defra-search-agent
docker build -t ai-defra-search-agent:latest .
```

### Kubernetes Deployment

Services are containerized and deployable to Kubernetes:

```yaml
# Example: Agent deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: agent
spec:
  replicas: 2
  selector:
    matchLabels:
      app: agent
  template:
    metadata:
      labels:
        app: agent
    spec:
      containers:
      - name: agent
        image: ai-defra-search-agent:latest
        ports:
        - containerPort: 8086
        env:
        - name: KNOWLEDGE_BASE_URL
          value: http://knowledge:8085
        - name: BEDROCK_ENDPOINT
          value: https://bedrock.us-east-1.amazonaws.com
```

### Environment Configuration

**Production variables:**
```
ENVIRONMENT=production
DEBUG=false
LOG_LEVEL=INFO
BEDROCK_ENDPOINT=https://bedrock.us-east-1.amazonaws.com
POSTGRES_SSL=true
MONGO_SSL=true
```

---

## Troubleshooting

### Service Won't Start

```bash
# Check logs
docker compose logs agent
docker compose logs knowledge
docker compose logs postgres

# Verify network
docker network ls | grep ai-defra

# Check port conflicts
lsof -i :8085
lsof -i :8086
lsof -i :3000
```

### Database Connection Issues

```bash
# Check PostgreSQL
docker compose exec postgres psql -U postgres

# Check MongoDB
docker compose exec mongo mongosh

# Check Redis
docker compose exec redis redis-cli ping
```

### API Not Responding

```bash
# Check service health
curl http://localhost:8086/health
curl http://knowledge.localhost/health

# Check containers are running
docker compose ps

# Restart service
docker compose restart agent
docker compose restart knowledge
```

### Search Returns No Results

```bash
# Verify documents uploaded
curl http://knowledge.localhost/documents

# Check document status
curl http://knowledge.localhost/upload-status/{upload_id}

# Verify embeddings created
docker compose exec postgres psql \
  -U postgres -c "SELECT count(*) FROM knowledge_vectors;"

# Check Bedrock connection
curl http://bedrock:4566/
```

### Performance Issues

```bash
# Check system resources
docker stats

# Check slow queries
docker compose logs postgres | grep slow

# Check cache hit rate
docker compose exec redis redis-cli info stats
```

---

## Key Files Reference

| Path | Purpose |
|------|---------|
| `ai-defra-search-core/compose.yml` | Docker Compose configuration |
| `ai-defra-search-agent/app/chat/` | Chat endpoints and logic |
| `ai-defra-search-knowledge/app/rag/` | Search endpoints |
| `ai-defra-search-knowledge/app/ingest/` | Document ingestion |
| `ai-defra-search-frontend/src/` | React components |
| `ai-defra-search-knowledge/changelog/` | Database migrations |

---

## Additional Resources

- **Architecture Details:** `ai-defra-search-core/ARCHITECTURE.md`
- **Agent Specifications:** `ai-defra-search-core/AGENTS.md`
- **API Documentation:** Swagger UI available at service endpoints
- **Contributing:** See LICENCE and README files in each repository

---

**Documentation Generated:** March 20, 2026  
**System Version:** 1.0.0  
**Last Updated:** March 20, 2026

