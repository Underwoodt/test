# Docker Compose Examples

**Common Docker Compose configurations**

---

## Basic Development

```yaml
version: '3.8'

services:
  frontend:
    image: ai-defra-search-frontend:latest
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_AGENT_URL=http://localhost:8086
      - REACT_APP_KNOWLEDGE_URL=http://knowledge.localhost

  agent:
    image: ai-defra-search-agent:latest
    ports:
      - "8086:8086"
    environment:
      - KNOWLEDGE_BASE_URL=http://knowledge:8085
      - DATABASE_URL=postgresql://postgres:password@postgres:5432/ai_defra
    depends_on:
      - postgres
      - mongo
      - redis

  knowledge:
    image: ai-defra-search-knowledge:latest
    ports:
      - "8085:8085"
    environment:
      - DATABASE_URL=postgresql://postgres:password@postgres:5432/ai_defra
    depends_on:
      - postgres
      - mongo

  postgres:
    image: pgvector/pgvector:latest
    environment:
      POSTGRES_PASSWORD: password
      POSTGRES_DB: ai_defra
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  mongo:
    image: mongo:latest
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password
    volumes:
      - mongo_data:/data/db
    ports:
      - "27017:27017"

  redis:
    image: redis:latest
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  mongo_data:
  redis_data:

networks:
  default:
    name: ai-defra-search
```

---

## With Health Checks

```yaml
services:
  agent:
    image: ai-defra-search-agent:latest
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8086/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  postgres:
    image: pgvector/pgvector:latest
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  mongo:
    image: mongo:latest
    healthcheck:
      test: echo 'db.runCommand("ping").ok'
      interval: 10s
      timeout: 5s
      retries: 5
```

---

## Production with Resources

```yaml
services:
  agent:
    image: my-registry/ai-defra-search-agent:v1.0.0
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
    restart: unless-stopped
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  knowledge:
    image: my-registry/ai-defra-search-knowledge:v1.0.0
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
        reservations:
          cpus: '1'
          memory: 1G
    restart: unless-stopped
```

---

**Version:** 1.0.0  
**Last Updated:** March 20, 2026
