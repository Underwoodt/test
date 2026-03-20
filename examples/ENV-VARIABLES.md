# Environment Variables

**Configuration examples**

---

## Development (.env)

```bash
# Environment
ENVIRONMENT=development
DEBUG=true
LOG_LEVEL=DEBUG

# Frontend
REACT_APP_AGENT_URL=http://localhost:8086
REACT_APP_KNOWLEDGE_URL=http://knowledge.localhost

# Agent
KNOWLEDGE_BASE_URL=http://knowledge:8085
BEDROCK_ENDPOINT=http://bedrock:4566

# Databases
DATABASE_URL=postgresql://postgres:password@postgres:5432/ai_defra
MONGO_URI=mongodb://admin:password@mongo:27017/ai_defra
REDIS_URL=redis://redis:6379

# Features
ENABLE_CACHE=true
ENABLE_LOGGING=true
ENABLE_METRICS=false
```

---

## Production (.env.production)

```bash
# Environment
ENVIRONMENT=production
DEBUG=false
LOG_LEVEL=INFO

# Frontend
REACT_APP_AGENT_URL=https://agent.example.com
REACT_APP_KNOWLEDGE_URL=https://knowledge.example.com

# Agent
KNOWLEDGE_BASE_URL=https://knowledge.example.com
BEDROCK_ENDPOINT=https://bedrock.us-east-1.amazonaws.com
BEDROCK_API_KEY=${BEDROCK_API_KEY}

# Databases
DATABASE_URL=postgresql://${DB_USER}:${DB_PASSWORD}@rds.example.com:5432/ai_defra
MONGO_URI=mongodb+srv://${MONGO_USER}:${MONGO_PASSWORD}@mongodb.example.com/ai_defra
REDIS_URL=redis://redis.example.com:6379

# Security
SSL_CERT_PATH=/etc/ssl/certs/cert.pem
SSL_KEY_PATH=/etc/ssl/certs/key.pem

# Features
ENABLE_CACHE=true
ENABLE_LOGGING=true
ENABLE_METRICS=true

# Limits
MAX_REQUEST_SIZE=100MB
MAX_FILE_SIZE=50MB
MAX_TIMEOUT=30
```

---

## Kubernetes Secrets

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: ai-defra-secrets
type: Opaque
stringData:
  DATABASE_URL: postgresql://user:pass@postgres:5432/db
  MONGO_URI: mongodb://user:pass@mongo:27017/db
  REDIS_URL: redis://redis:6379
  BEDROCK_API_KEY: your-api-key
  BEDROCK_ENDPOINT: https://bedrock.us-east-1.amazonaws.com
  JWT_SECRET: your-secret-key
```

---

## Docker Compose Override

Create `.env.docker`:

```bash
# Override default variables
POSTGRES_PASSWORD=my-secure-password
MONGO_INITDB_ROOT_PASSWORD=my-secure-password
DATABASE_URL=postgresql://postgres:my-secure-password@postgres:5432/ai_defra
```

Use:
```bash
docker compose --env-file .env.docker up
```

---

## Available Variables

### Environment

```
ENVIRONMENT
- development / staging / production

DEBUG
- true / false

LOG_LEVEL
- DEBUG / INFO / WARNING / ERROR / CRITICAL
```

### Service URLs

```
KNOWLEDGE_BASE_URL
- URL for Knowledge service

BEDROCK_ENDPOINT
- AWS Bedrock endpoint or stub
```

### Database

```
DATABASE_URL
- PostgreSQL connection string

MONGO_URI
- MongoDB connection string

REDIS_URL
- Redis connection string
```

### Features

```
ENABLE_CACHE
- true / false

ENABLE_LOGGING
- true / false

ENABLE_METRICS
- true / false
```

### Limits

```
MAX_REQUEST_SIZE
- Max request body size

MAX_FILE_SIZE
- Max upload file size

MAX_TIMEOUT
- Request timeout in seconds
```

---

**Version:** 1.0.0  
**Last Updated:** March 20, 2026
