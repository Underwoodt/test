# Technology Stack - AI DEFRA Search Platform

**Tools, frameworks, and technologies used throughout the system**

---

## Overview

AI DEFRA Search uses modern, production-proven technologies across all layers.

---

## Frontend Stack

### Languages & Frameworks
- **React** 18+ - UI framework
- **TypeScript** - Type safety
- **CSS-in-JS** - Styled components

### Key Libraries
- **React Router** - Client-side routing
- **Axios** - HTTP client
- **React Query** - Server state management
- **Jest** - Unit testing
- **React Testing Library** - Component testing

### Build Tools
- **Vite** or **Create React App** - Build bundler
- **ESLint** - Code linting
- **Prettier** - Code formatting

### Performance
- Code splitting
- Lazy loading
- Image optimization

---

## Backend Stack

### Languages & Frameworks
- **Python 3.13+** - Programming language
- **FastAPI** - Web framework
- **Pydantic** - Data validation

### Key Libraries
- **SQLAlchemy** - ORM
- **Motor** - Async MongoDB driver
- **aioredis** - Async Redis client
- **httpx** - Async HTTP client
- **Python-multipart** - File upload
- **python-dotenv** - Environment variables

### Data Extraction
- **PyMuPDF** - PDF extraction
- **python-docx** - DOCX extraction
- **python-pptx** - PPTX extraction

### Development Tools
- **pytest** - Testing
- **ruff** - Code linting
- **black** - Code formatting
- **uv** - Package manager

---

## Database Technologies

### PostgreSQL
- **Version:** 14+
- **Extension:** pgvector
- **Purpose:** Vector storage for semantic search

**pgvector Features:**
- Vector data type (1024 dimensions)
- Distance operators (<->, <#>, <+>)
- IVF indexes for fast search
- HNSW indexes (newer option)

**Usage:**
```sql
CREATE TABLE chunks (
  id UUID PRIMARY KEY,
  content TEXT,
  embedding vector(1024),
  metadata JSONB
);
CREATE INDEX ON chunks USING IVFFLAT (embedding vector_cosine_ops);
```

### MongoDB
- **Version:** 5.0+
- **Purpose:** Flexible document storage
- **Features:**
  - Document-oriented
  - Schema flexibility
  - Horizontal scaling
  - Transactions

**Collections:**
- conversations
- knowledge_groups
- documents
- ingestion_jobs

### Redis
- **Version:** 7.0+
- **Purpose:** Caching & sessions
- **Data Types:**
  - Strings (sessions)
  - Hashes (metadata)
  - Lists (queues)
  - Sorted sets (rankings)

**TTLs:**
- Sessions: 24 hours
- Search results: 1 hour
- Embeddings: 7 days

---

## AI/ML Technologies

### AWS Bedrock
- **Models Used:**
  - Titan Embed V2 (embeddings)
  - Claude (chat)
- **API:** RESTful HTTP
- **Features:**
  - Managed service
  - No model hosting needed
  - Pay-per-use pricing
  - Multiple model options

### Vector Embeddings
- **Dimension:** 1024
- **Model:** Titan Embed V2
- **Cost:** Per token
- **Latency:** 500-1000ms per chunk

### Semantic Search
- **Algorithm:** Vector similarity (cosine distance)
- **Complexity:** O(log n) with IVFFLAT index
- **Accuracy:** 92% relevant in top 5

---

## Container & Orchestration

### Docker
- **Purpose:** Containerize applications
- **Files:** Dockerfile in each service
- **Registry:** Docker Hub / Private registry

**Docker Features Used:**
- Multi-stage builds
- Health checks
- Volume mounting
- Environment variables

### Docker Compose
- **Purpose:** Local development orchestration
- **File:** `compose.yml`
- **Services:** 10+ services in one file
- **Networks:** ai-defra-search bridge

### Kubernetes (Production)
- **Version:** 1.24+
- **Features:**
  - Auto-scaling
  - Service discovery
  - Rolling updates
  - Health checks
  - Resource limits

**Kubernetes Resources:**
- Deployments
- Services
- ConfigMaps
- Secrets
- PersistentVolumes

---

## Storage

### S3 / LocalStack
- **Purpose:** File storage for documents
- **Formats:** PDF, DOCX, PPTX, JSONL
- **Local:** LocalStack in development
- **Production:** AWS S3

**Operations:**
- Upload
- Download
- Delete
- List (with pagination)

---

## Networking

### Traefik
- **Purpose:** Reverse proxy & load balancer
- **Port:** 8080 (dashboard)
- **Features:**
  - Automatic service discovery
  - SSL/TLS termination
  - Load balancing
  - Middleware support

**Routes:**
- `frontend.localhost` → Frontend (3000)
- `knowledge.localhost` → Knowledge (8085)
- Port forwarding for Agent (8086)

---

## Testing Tools

### Unit Testing
- **pytest** - Python testing framework
- **Jest** - JavaScript testing

### Integration Testing
- **Testcontainers** - Docker containers for tests
- **@testing-library/react** - React component testing

### End-to-End Testing
- **Playwright** - Browser automation
- **test scenarios:** Complete user journeys

### Performance Testing
- **JMeter** - Load testing
- **Metrics:** Response time, throughput, error rate

### Code Quality
- **SonarQube** - Code analysis
- **ruff** - Python linting
- **ESLint** - JavaScript linting

---

## Logging & Monitoring

### Logging
- **Python:** Built-in logging module
- **JavaScript:** Console API
- **Format:** Structured JSON (recommended)
- **Level:** DEBUG, INFO, WARNING, ERROR, CRITICAL

### Monitoring
- **Prometheus** - Metrics collection
- **Grafana** - Visualization
- **ELK Stack** - Log aggregation (optional)

**Metrics:**
- Response time
- Error rate
- Request volume
- Resource usage

---

## Development Tools

### Version Control
- **Git** - Source control
- **GitHub** - Repository hosting

### Package Managers
- **Python:** uv (recommended)
- **Node.js:** npm / yarn

### Code Formatters
- **Python:** black, ruff
- **JavaScript:** Prettier, ESLint

### CI/CD
- **GitHub Actions** (recommended)
- **Jenkins** (alternative)
- **GitLab CI** (alternative)

---

## Security

### Authentication
- **OAuth 2.0** (recommended)
- **SSO** (Single Sign-On)
- **JWT** (JSON Web Tokens)

### Encryption
- **Transit:** HTTPS/TLS 1.3
- **At Rest:** Database-level encryption (TBD)

### Secrets Management
- **Environment variables** (.env files)
- **Vault** (for production)
- **AWS Secrets Manager** (for production)

---

## Performance Optimization

### Caching
- **Redis** - Application cache
- **HTTP cache** - Browser cache

### Database
- **Indexes** - pgvector IVFFLAT
- **Connection pooling** - psycopg2-pool
- **Query optimization** - Explain plans

### API
- **Gzip compression** - HTTP responses
- **JSON serialization** - Fast serialization
- **Pagination** - Large result sets

---

## Production Deployment

### Infrastructure
- **Cloud:** AWS (recommended)
- **Containerization:** Docker
- **Orchestration:** Kubernetes
- **Load balancing:** ALB / ELB

### Database
- **PostgreSQL RDS** - Managed PostgreSQL
- **DocumentDB/MongoDB Atlas** - Managed MongoDB
- **ElastiCache** - Managed Redis

### Storage
- **S3** - Document storage
- **CloudFront** - CDN (if needed)

### Monitoring
- **CloudWatch** - AWS monitoring
- **X-Ray** - Distributed tracing

---

## Technology Decisions

### Why PostgreSQL + pgvector?
✅ Vector search native support  
✅ ACID transactions  
✅ Proven at scale  
✅ SQL familiarity  

### Why MongoDB?
✅ Flexible schema  
✅ Good for chat data  
✅ Horizontal scaling  

### Why FastAPI?
✅ Modern async  
✅ Type safety  
✅ Auto OpenAPI docs  
✅ High performance  

### Why React?
✅ Component-based  
✅ Large ecosystem  
✅ Good performance  

### Why Docker?
✅ Consistency  
✅ Portability  
✅ Isolation  

---

## Version Constraints

```
Python: >= 3.13
FastAPI: >= 0.100
PostgreSQL: >= 14
MongoDB: >= 5.0
Redis: >= 7.0
Docker: >= 20.10
Docker Compose: >= 2.0
React: >= 18
Node.js: >= 18 (for frontend dev)
```

---

**Document Version:** 1.0.0  
**Last Updated:** March 20, 2026
