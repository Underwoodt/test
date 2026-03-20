# Deployment - Getting to Production

**Deploying the system to various environments**

---

## What's in This Section?

| File | Purpose | Audience |
|------|---------|----------|
| **DOCKER.md** | Docker & Docker Compose | DevOps |
| **KUBERNETES.md** | Kubernetes deployment | DevOps, Platform |
| **CHECKLIST.md** | Pre-deployment verification | Everyone |

---

## Deployment Overview

```
Development (Docker Compose)
  ↓
Staging (Kubernetes)
  ↓
Production (Kubernetes)
```

---

## Document Index

### DOCKER.md (Local & Container Deployment)

**Contains:**
- Local development with Docker Compose
- Building Docker images
- Running individual services
- Environment configuration
- Production Docker setup
- Best practices
- Troubleshooting Docker

**Best for:** Understanding Docker and deployment basics

**Covers:**
- Docker commands
- Docker Compose configurations
- Building and pushing images
- Health checks
- Resource limits
- Restart policies

---

### KUBERNETES.md (Production Orchestration)

**Contains:**
- Kubernetes deployment manifests
- Service definitions
- ConfigMap and Secret setup
- Auto-scaling configuration
- Rolling updates
- Rollback procedures

**Best for:** Deploying to production Kubernetes

**Includes:**
- Sample YAML manifests
- Deployment configuration
- Service setup
- HPA (Horizontal Pod Autoscaler)
- Upgrade strategies

---

### CHECKLIST.md (Pre-Deployment Verification)

**Contains:**
- Pre-deployment checklist
- Container preparation steps
- Infrastructure verification
- Configuration checks
- Testing requirements
- Monitoring setup
- Deployment steps
- Post-deployment verification

**Best for:** Making sure everything is ready before deploying

**Sections:**
- Pre-deployment
- Container preparation
- Infrastructure
- Configuration
- Testing
- Monitoring
- Deployment
- Post-deployment

---

## Deployment Paths

### Local Development
**Environment:** Docker Compose  
**Platform:** Personal laptop/workstation  
**See:** DOCKER.md - "Local Development with Docker Compose"

```bash
docker compose up -d
```

---

### Staging Environment
**Environment:** Kubernetes cluster  
**Platform:** Cloud provider (AWS, GCP, Azure)  
**See:** KUBERNETES.md - "Deployment Manifests"

**Steps:**
1. Build and push images
2. Create Kubernetes manifests
3. Apply ConfigMaps and Secrets
4. Deploy services
5. Verify access

---

### Production Environment
**Environment:** Kubernetes cluster  
**Platform:** Cloud provider  
**See:** KUBERNETES.md + CHECKLIST.md

**Before deploying:** Complete CHECKLIST.md

**Steps:**
1. Verify all checklist items
2. Build production images
3. Push to registry
4. Apply manifests
5. Monitor deployment
6. Verify health
7. Complete post-deployment

---

## Quick Reference

### Docker Compose (Local)

```bash
# Start all services
docker compose up -d

# Stop all services
docker compose down

# View logs
docker compose logs -f

# Restart service
docker compose restart agent
```

### Kubernetes

```bash
# Deploy
kubectl apply -f deployment.yaml

# Check status
kubectl get pods

# View logs
kubectl logs <pod>

# Scale
kubectl scale deployment agent --replicas=3

# Update image
kubectl set image deployment/agent agent=registry/agent:v1.0.1

# Rollback
kubectl rollout undo deployment/agent
```

---

## Environment Types

### Development
- **Infrastructure:** Docker Compose on laptop
- **Duration:** Full development cycle
- **Scale:** Single machine
- **Cost:** Free (local resources)

### Staging
- **Infrastructure:** Kubernetes cluster
- **Duration:** Before each release
- **Scale:** Similar to production
- **Cost:** Moderate (shared cluster)

### Production
- **Infrastructure:** Kubernetes cluster
- **Duration:** Always running
- **Scale:** Auto-scaling enabled
- **Cost:** Pay-per-resource

---

## Deployment Components

### Container Images
- Frontend image
- Agent image
- Knowledge image
- (Base images for DB not built, use managed services)

### Kubernetes Resources
- Deployments (for running services)
- Services (for exposing services)
- ConfigMaps (for configuration)
- Secrets (for sensitive data)
- HPA (for auto-scaling)

### External Services
- Managed PostgreSQL (RDS/CloudSQL)
- Managed MongoDB (Atlas/DocumentDB)
- Managed Redis (ElastiCache/Memorystore)
- S3 (or equivalent object storage)

---

## Deployment Flow

```
Code changes
  ↓
Build Docker images
  ↓
Push to registry
  ↓
Update Kubernetes manifests
  ↓
Apply changes (kubectl apply)
  ↓
Monitor rollout
  ↓
Verify health
  ↓
Monitor in production
  ↓
Rollback if needed (kubectl rollout undo)
```

---

## Pre-Deployment Checklist

**Use CHECKLIST.md for:**
- Testing verification
- Security scanning
- Configuration validation
- Monitoring setup
- Documentation review
- Performance validation

**Do not deploy without completing checklist!**

---

## Common Deployment Tasks

### Deploy New Version
**See:** KUBERNETES.md → "Upgrades"

```bash
kubectl set image deployment/agent \
  agent=registry/agent:v1.0.1
```

### Rollback Deployment
**See:** KUBERNETES.md → "Rollback"

```bash
kubectl rollout undo deployment/agent
```

### Scale Service
**See:** KUBERNETES.md → "Scaling"

```bash
kubectl scale deployment agent --replicas=5
```

### Enable Auto-Scaling
**See:** KUBERNETES.md → "Auto-Scaling"

```bash
kubectl apply -f hpa.yaml
```

---

## Environment Variables

### By Environment

**Development:**
- DEBUG=true
- LOG_LEVEL=DEBUG
- Endpoints: localhost

**Staging:**
- DEBUG=false
- LOG_LEVEL=INFO
- Endpoints: staging cluster

**Production:**
- DEBUG=false
- LOG_LEVEL=INFO
- Endpoints: production cluster
- Secrets from vault

---

## Monitoring Deployment

### During Deployment
```bash
# Watch rollout progress
kubectl rollout status deployment/agent

# Check pod status
kubectl get pods -w

# View logs in real-time
kubectl logs -f deployment/agent
```

### After Deployment
- Check error rates
- Monitor latency
- Check resource usage
- Verify user access
- Review logs for warnings

---

## Rollback Procedures

### If Deployment Fails
```bash
kubectl rollout undo deployment/agent
```

### If Issues Found in Production
```bash
kubectl rollout undo deployment/agent
```

### View Rollout History
```bash
kubectl rollout history deployment/agent
```

---

## Next Steps

1. **Prepare to deploy:** Use CHECKLIST.md
2. **Deploy locally:** Use DOCKER.md
3. **Deploy to Kubernetes:** Use KUBERNETES.md
4. **Monitor:** Set up monitoring and logging

---

**Document Version:** 1.0.0  
**Last Updated:** March 20, 2026
