# Production Deployment

Guide to deploying to production on Kubernetes (CDP environment).

## Prerequisites

- **Kubernetes cluster** running (CDP or similar)
- **Docker registry** for image storage
- **PostgreSQL 14+** with pgvector extension
- **MongoDB 5.0+** with async driver support
- **AWS account** with S3 and Bedrock access
- **VPC networking** to all services

---

## Building Docker Image

### Build for Production

```bash
# Build image
docker build -t ai-defra-search-knowledge:1.0.0 .

# Tag for registry
docker tag ai-defra-search-knowledge:1.0.0 \
  registry.example.com/ai-defra-search-knowledge:1.0.0

# Push to registry
docker push registry.example.com/ai-defra-search-knowledge:1.0.0
```

### Dockerfile Overview

```dockerfile
# Multi-stage build
FROM python:3.13-slim as builder
RUN pip install uv
COPY pyproject.toml uv.lock ./
RUN uv sync --no-dev --no-editable

FROM python:3.13-slim
COPY --from=builder /app/.venv /app/.venv
COPY app /app/app
ENV PATH="/app/.venv/bin:$PATH"
ENTRYPOINT ["python", "-m", "app.main"]
```

---

## Database Setup

### PostgreSQL with pgvector

#### Create Database

```sql
CREATE DATABASE ai_defra_search_knowledge;

-- Connect to new database
\c ai_defra_search_knowledge

-- Install pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Create indexes
CREATE TABLE knowledge_vectors (
    id SERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    embedding vector(1024),
    snapshot_id VARCHAR(255),
    source_id VARCHAR(24),
    metadata JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_embedding ON knowledge_vectors 
    USING ivfflat (embedding vector_cosine_ops)
    WITH (lists = 100);

CREATE INDEX idx_knowledge_group ON knowledge_vectors 
    USING GIN (metadata jsonb_path_ops);
```

#### PostgreSQL Configuration

```bash
# Connection pooling (pgbouncer)
# Set max_connections appropriately
# For production: (CPU_CORES * 2) + effective_cache_size

# Example for 4 cores:
# max_connections = 8 + cache_size

# Enable query logging for debugging
log_statement = 'all'  # or 'slow'
log_min_duration_statement = 5000  # Log queries > 5s
```

#### Run Liquibase Migrations

The database schema is version-controlled using Liquibase:

```bash
# Docker migration container
docker build -f migrator/Dockerfile -t ai-defra-migrator .

# Run migrations
docker run --rm \
  -e POSTGRES_HOST=postgres.prod \
  -e POSTGRES_USER=ai_defra_search_knowledge \
  -e POSTGRES_PASSWORD=<password> \
  ai-defra-migrator
```

Or in Kubernetes as init container:

```yaml
initContainers:
  - name: migrations
    image: ai-defra-migrator:1.0.0
    env:
      - name: POSTGRES_HOST
        valueFrom:
          secretKeyRef:
            name: db-secrets
            key: postgres-host
      # ... other env vars
```

### MongoDB

#### Create Database & Collections

```javascript
use ai-defra-search-knowledge;

// Create collections
db.createCollection("knowledgeGroups");
db.createCollection("documents");

// Create indexes
db.knowledgeGroups.createIndex(
  { "created_by": 1, "name": 1 },
  { unique: true }
);

db.documents.createIndex({ "cdp_upload_id": 1 });
```

#### Replica Set (Recommended)

```bash
# For production, configure MongoDB replica set for high availability
mongod --replSet "prod" --bind_ip_all
```

---

## Kubernetes Deployment

### ConfigMap (Non-sensitive config)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: ai-defra
data:
  PYTHON_ENV: production
  AWS_REGION: eu-west-2
  BEDROCK_EMBEDDING_MODEL_ID: amazon.titan-embed-text-v2:0
  LOG_LEVEL: INFO
```

### Secret (Sensitive config)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
  namespace: ai-defra
type: Opaque
stringData:
  POSTGRES_HOST: postgres.default.svc.cluster.local
  POSTGRES_USER: ai_defra_search_knowledge
  POSTGRES_PASSWORD: <strong-password>
  POSTGRES_DB: ai_defra_search_knowledge
  MONGO_URI: mongodb://mongo-0,mongo-1,mongo-2/ai-defra-search-knowledge
  KNOWLEDGE_UPLOAD_BUCKET_NAME: ai-defra-search-knowledge-prod
  AWS_ACCESS_KEY_ID: <key>
  AWS_SECRET_ACCESS_KEY: <secret>
```

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ai-defra-search-knowledge
  namespace: ai-defra
spec:
  replicas: 3  # Multiple instances for HA
  selector:
    matchLabels:
      app: ai-defra-search-knowledge
  template:
    metadata:
      labels:
        app: ai-defra-search-knowledge
        version: v1
    spec:
      containers:
      - name: app
        image: registry.example.com/ai-defra-search-knowledge:1.0.0
        imagePullPolicy: Always
        
        ports:
        - name: http
          containerPort: 8085
          protocol: TCP
        
        env:
        - name: PYTHON_ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: PYTHON_ENV
        - name: AWS_REGION
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: AWS_REGION
        - name: POSTGRES_HOST
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: POSTGRES_HOST
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: POSTGRES_USER
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: POSTGRES_PASSWORD
        - name: MONGO_URI
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: MONGO_URI
        - name: AWS_ACCESS_KEY_ID
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: AWS_ACCESS_KEY_ID
        - name: AWS_SECRET_ACCESS_KEY
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: AWS_SECRET_ACCESS_KEY
        
        livenessProbe:
          httpGet:
            path: /health
            port: 8085
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        readinessProbe:
          httpGet:
            path: /health
            port: 8085
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 5
          failureThreshold: 2
        
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"

      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - ai-defra-search-knowledge
              topologyKey: kubernetes.io/hostname
```

### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ai-defra-search-knowledge
  namespace: ai-defra
spec:
  type: ClusterIP
  selector:
    app: ai-defra-search-knowledge
  ports:
  - name: http
    port: 80
    targetPort: 8085
    protocol: TCP
```

### Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ai-defra-search-knowledge
  namespace: ai-defra
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - api.defra.example.com
    secretName: ai-defra-search-knowledge-tls
  rules:
  - host: api.defra.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: ai-defra-search-knowledge
            port:
              number: 80
```

---

## Environment Variables Checklist

**Required:**
- [ ] `POSTGRES_HOST` — PostgreSQL host
- [ ] `POSTGRES_USER` — PostgreSQL user
- [ ] `POSTGRES_PASSWORD` — PostgreSQL password
- [ ] `MONGO_URI` — MongoDB connection string
- [ ] `KNOWLEDGE_UPLOAD_BUCKET_NAME` — S3 bucket name
- [ ] `AWS_REGION` — AWS region
- [ ] `AWS_ACCESS_KEY_ID` — AWS credentials
- [ ] `AWS_SECRET_ACCESS_KEY` — AWS credentials

**Optional:**
- `PYTHON_ENV=production` — Enables production logging
- `BEDROCK_ENDPOINT_URL` — For custom Bedrock endpoint
- `BEDROCK_EMBEDDING_MODEL_ID` — Embedding model (default: Titan v2)
- `ENABLE_METRICS=true` — CloudWatch metrics
- Various timeout values

---

## Health Checks

### Kubernetes Probes

Configured via Deployment:

```yaml
# Liveness: Is the app running?
livenessProbe:
  httpGet:
    path: /health
    port: 8085
  initialDelaySeconds: 30
  periodSeconds: 10

# Readiness: Can it accept traffic?
readinessProbe:
  httpGet:
    path: /health
    port: 8085
  initialDelaySeconds: 10
  periodSeconds: 5
```

### Manual Health Check

```bash
curl https://api.defra.example.com/health
# {"status": "healthy", "timestamp": "2024-03-19T12:00:00Z"}
```

---

## Scaling

### Horizontal Scaling

```bash
# Scale to 5 replicas
kubectl scale deployment ai-defra-search-knowledge --replicas=5 -n ai-defra

# Or use HPA (Horizontal Pod Autoscaler)
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ai-defra-search-knowledge
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ai-defra-search-knowledge
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### Vertical Scaling (PostgreSQL)

For very large datasets (>100M vectors):

1. Add read replicas:
   ```bash
   # Create PostgreSQL read replica
   aws rds create-db-instance-read-replica \
     --db-instance-identifier ai-defra-search-knowledge-read-1
   ```

2. Route searches to read replica, writes to primary

---

## Monitoring & Logging

### Logs to CloudWatch

Logs stream to stdout as JSON:

```bash
# Kubernetes captures stdout
kubectl logs -n ai-defra deployment/ai-defra-search-knowledge

# Or CloudWatch
aws logs tail /aws/eks/ai-defra/ai-defra-search-knowledge
```

### Metrics

Enable CloudWatch metrics:

```bash
# Set environment
ENABLE_METRICS=true

# Metrics appear in CloudWatch
aws cloudwatch list-metrics --namespace ai-defra-search-knowledge
```

### Example CloudWatch Dashboard

```json
{
  "widgets": [
    {
      "type": "metric",
      "properties": {
        "metrics": [
          ["ai-defra-search-knowledge", "DocumentsIngested"],
          ["ai-defra-search-knowledge", "SearchLatency"],
          ["ai-defra-search-knowledge", "BedrockApiLatency"]
        ],
        "period": 60,
        "stat": "Average",
        "region": "eu-west-2"
      }
    }
  ]
}
```

---

## Backup & Disaster Recovery

### PostgreSQL Backups

```bash
# AWS RDS automated backups
aws rds modify-db-instance \
  --db-instance-identifier ai-defra-search-knowledge \
  --backup-retention-period 30 \
  --preferred-backup-window "03:00-04:00"

# Restore from backup
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier ai-defra-search-knowledge-restored \
  --db-snapshot-identifier arn:aws:rds:eu-west-2:123456789012:snapshot:...
```

### MongoDB Backups

```bash
# MongoDB backup (mongodump)
mongodump \
  --uri "mongodb+srv://user:pass@mongo.example.com/ai-defra-search-knowledge" \
  --out /backup/ai-defra-search-knowledge

# Restore (mongorestore)
mongorestore \
  --uri "mongodb+srv://user:pass@mongo.example.com" \
  /backup/ai-defra-search-knowledge
```

### S3 Backups

```bash
# Enable versioning
aws s3api put-bucket-versioning \
  --bucket ai-defra-search-knowledge-prod \
  --versioning-configuration Status=Enabled

# Enable MFA delete for protection
aws s3api put-bucket-versioning \
  --bucket ai-defra-search-knowledge-prod \
  --versioning-configuration Status=Enabled,MFADelete=Enabled
```

---

## SSL/TLS

### Self-Signed Cert (Testing)

```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365
```

### Let's Encrypt (Production)

```yaml
# Install cert-manager
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager jetstack/cert-manager

# Create ClusterIssuer
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
```

---

## Upgrade Strategy

### Rolling Update

```bash
# kubectl automatically rolls out new replicas
kubectl set image deployment/ai-defra-search-knowledge \
  app=registry.example.com/ai-defra-search-knowledge:1.1.0 \
  -n ai-defra

# Monitor rollout
kubectl rollout status deployment/ai-defra-search-knowledge -n ai-defra
```

### Blue-Green Deployment

```yaml
# Deploy new version as separate deployment
# Switch traffic via Service selector
# Instant rollback if needed
```

---

## Troubleshooting

### Pod Not Starting

```bash
# Check logs
kubectl logs <pod-name> -n ai-defra

# Check events
kubectl describe pod <pod-name> -n ai-defra

# Check resources
kubectl top nodes
```

### High Latency

```bash
# Check PostgreSQL
# 1. Query slow log
# 2. Check indexes
# 3. Monitor connections

# Check MongoDB
# 1. Check replication lag
# 2. Monitor memory usage

# Check network
# 1. Latency between pods
# 2. CloudWatch VPC Flow Logs
```

---

## Checklist Before Production

- [ ] Docker image built and tested
- [ ] PostgreSQL with pgvector set up
- [ ] MongoDB replica set configured
- [ ] S3 bucket created with versioning + encryption
- [ ] AWS IAM credentials created with minimal permissions
- [ ] Liquibase migrations run successfully
- [ ] Kubernetes manifests created (ConfigMap, Secret, Deployment, Service, Ingress)
- [ ] Health checks configured
- [ ] Monitoring and logging enabled
- [ ] Backup strategy tested
- [ ] SSL/TLS certificate installed
- [ ] Load testing completed
- [ ] Security audit completed

---

## Next Steps

- **Configuration:** See [Configuration](./configuration.md)
- **Monitoring:** See [Monitoring & Logging](./monitoring.md)
- **Troubleshooting:** See [Troubleshooting](./troubleshooting.md)

---

**Last Updated:** March 19, 2026  
**Production Ready:** Yes
