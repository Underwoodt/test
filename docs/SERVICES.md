# Services - AI DEFRA Search Platform

**Detailed description of all 7 services**

---

## Service Overview

| Service | Type | Language | Port | Role |
|---------|------|----------|------|------|
| Frontend | UI | React/TS | 3000 | Web interface |
| Agent | API | FastAPI | 8086 | Chat orchestration |
| Knowledge | API | FastAPI | 8085 | Search & ingestion |
| Journey Tests | Testing | Playwright | N/A | E2E tests |
| Perf Tests | Testing | JMeter | N/A | Performance tests |
| Bedrock Stub | Mock | WireMock | 4566 | LLM mock |
| Core | Infrastructure | Docker | N/A | Orchestration |

---

## 1. Frontend Service

**Technology:** React + TypeScript  
**Port:** 3000 (via Traefik proxy)  
**Repository:** `ai-defra-search-frontend`

### Responsibilities
- Web UI for users
- Session management
- File upload
- Chat interface
- Real-time updates

### Key Features
- Chat conversation display
- Document upload with progress
- Session persistence
- Error handling
- Responsive design

### Architecture
```
src/
├── components/     # Reusable UI components
├── pages/         # Page components
├── services/      # API client
├── hooks/         # Custom React hooks
├── utils/         # Utilities
└── types/         # TypeScript types
```

### API Integration
- Connects to Agent API (8086)
- Calls Knowledge API (8085) for uploads
- Session management

### Testing
- Jest for unit tests
- React Testing Library
- Snapshot tests

---

## 2. Agent Service (Chat API)

**Technology:** FastAPI + Python  
**Port:** 8086  
**Repository:** `ai-defra-search-agent`

### Responsibilities
- Chat endpoint
- Conversation management
- LLM orchestration
- RAG coordination
- User authentication

### Key Features
- Multi-turn conversations
- Context awareness
- Error recovery
- Response generation
- Conversation storage

### API Endpoints
```
POST /chat              # Send message
GET /conversations/{id} # Get history
GET /health            # Health check
```

### Architecture
```
app/
├── chat/          # Chat endpoints & logic
├── bedrock/       # LLM integration
├── config.py      # Configuration
├── main.py        # FastAPI app
└── models/        # Data models
```

### Dependencies
- Calls Knowledge service (8085)
- Calls Bedrock (4566)
- Uses MongoDB
- Uses Redis for caching

### Testing
- pytest for unit tests
- Mock Bedrock for integration tests
- Load testing with JMeter

---

## 3. Knowledge Service (RAG)

**Technology:** FastAPI + Python  
**Port:** 8085  
**Repository:** `ai-defra-search-knowledge`

### Responsibilities
- Document ingestion
- Semantic search
- Vector management
- Metadata storage
- Chunk management

### Key Features
- Multiple file format support (PDF, DOCX, PPTX, JSONL)
- Semantic chunking
- Vector embedding
- Metadata enrichment
- Status tracking

### API Endpoints
```
POST /documents         # Upload documents
POST /rag/search       # Semantic search
GET /upload-status/{id} # Check ingestion
GET /health            # Health check
```

### Architecture
```
app/
├── ingest/            # Document ingestion
├── rag/              # Search endpoints
├── knowledge_group/   # Group management
├── extractors/        # Format-specific extractors
├── config.py          # Configuration
└── main.py           # FastAPI app
```

### Dependencies
- PostgreSQL + pgvector (vectors)
- MongoDB (metadata)
- Bedrock (embeddings)
- S3/LocalStack (files)
- Redis (caching)

### Processing Pipeline
1. Receive file
2. Extract text (format-specific)
3. Split into chunks
4. Generate embeddings
5. Store in PostgreSQL
6. Index metadata in MongoDB

---

## 4. Journey Tests (E2E)

**Technology:** Playwright + JavaScript  
**Repository:** `ai-defra-search-journey-tests`

### Responsibilities
- End-to-end user journey testing
- Integration testing
- User flow validation
- Error scenario testing

### Test Scenarios
- User login
- Document upload
- Search queries
- Multi-turn conversations
- Error handling

### Architecture
```
tests/
├── login/           # Authentication tests
├── upload/          # File upload tests
├── search/          # Search functionality
└── conversation/    # Multi-turn tests
```

### Running Tests
```bash
npm test           # Run all tests
npm run test:ui    # UI mode
npm run test:debug # Debug mode
```

---

## 5. Performance Tests (Load)

**Technology:** JMeter + Docker  
**Repository:** `ai-defra-search-perf-tests`

### Responsibilities
- Load testing
- Performance monitoring
- Stress testing
- Capacity planning

### Test Scenarios
- Concurrent user load
- Sustained traffic
- Spike tests
- Document upload throughput

### Metrics Collected
- Response time
- Throughput (requests/sec)
- Error rate
- Resource utilization

### Running Tests
```bash
./run-tests-local.sh        # Local
./run-tests-local-docker.sh # Docker
```

---

## 6. Bedrock Stub (Mock LLM)

**Technology:** WireMock + Docker  
**Port:** 4566  
**Repository:** `ai-defra-search-aws-bedrock-stub`

### Responsibilities
- Mock AWS Bedrock for local development
- Simulate embeddings
- Simulate chat completions
- Manage responses

### Mocked Endpoints
```
/embeddings     # Text to vector
/invoke-model   # Chat completions
```

### Configuration
- Response mappings in `wiremock/mappings/`
- Configurable delay simulation
- Error scenarios

### Using in Development
```bash
# Bedrock Stub starts automatically with Docker Compose
# In production, use real AWS Bedrock
```

---

## 7. Core Infrastructure

**Technology:** Docker Compose + Python  
**Repository:** `ai-defra-search-core`

### Responsibilities
- Local development orchestration
- Service coordination
- Network management
- Infrastructure configuration

### Key Files
- `compose.yml` - Docker Compose config
- `tasks.py` - Orchestration tasks
- `ARCHITECTURE.md` - Architecture docs

### Services Orchestrated
- Frontend
- Agent
- Knowledge
- PostgreSQL
- MongoDB
- Redis
- LocalStack (S3)
- Traefik (proxy)
- Bedrock Stub

### Commands
```bash
docker compose up -d    # Start all
docker compose down     # Stop all
docker compose logs -f  # View logs
docker compose ps       # Status
```

---

## Service Communication

### Synchronous (Real-time)

```
Frontend ←→ Agent ←→ Knowledge
Agent ←→ Bedrock
Knowledge ←→ PostgreSQL
```

### Asynchronous (Background)

```
Frontend → Knowledge (upload)
         ↓
    Background job
    (extract, embed, store)
```

---

## Service Dependencies

```
Frontend depends on:
  - Agent (chat API)
  - Knowledge (uploads)
  - Redis (sessions)

Agent depends on:
  - Knowledge (search)
  - Bedrock (embeddings/responses)
  - MongoDB (storage)
  - Redis (cache)

Knowledge depends on:
  - PostgreSQL (vectors)
  - MongoDB (metadata)
  - Bedrock (embeddings)
  - S3 (files)
  - Redis (cache)

Bedrock depends on:
  - Nothing (external service)

Core depends on:
  - All services
  - All databases
```

---

## Deployment Strategy

### Local Development
- Docker Compose
- All services in one compose file
- LocalStack for AWS services

### Production
- Docker images for each service
- Kubernetes orchestration
- Real AWS services
- Auto-scaling
- High availability

---

**Document Version:** 1.0.0  
**Last Updated:** March 20, 2026

