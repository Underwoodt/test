# Kubernetes Deployment

**Deploying to Kubernetes**

---

## Prerequisites

- Kubernetes cluster (1.24+)
- kubectl configured
- Docker images pushed to registry

---

## Deployment Manifests

### Agent Deployment

```yaml
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
        image: my-registry/ai-defra-search-agent:latest
        ports:
        - containerPort: 8086
        env:
        - name: KNOWLEDGE_BASE_URL
          value: http://knowledge:8085
        - name: BEDROCK_ENDPOINT
          valueFrom:
            secretKeyRef:
              name: bedrock-secret
              key: endpoint
        resources:
          requests:
            cpu: 500m
            memory: 512Mi
          limits:
            cpu: 1000m
            memory: 1Gi
        livenessProbe:
          httpGet:
            path: /health
            port: 8086
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 8086
          initialDelaySeconds: 20
          periodSeconds: 5
```

### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: agent
spec:
  selector:
    app: agent
  type: ClusterIP
  ports:
  - port: 8086
    targetPort: 8086
```

### ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: agent-config
data:
  LOG_LEVEL: "INFO"
  ENVIRONMENT: "production"
```

### Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: bedrock-secret
type: Opaque
stringData:
  endpoint: https://bedrock.us-east-1.amazonaws.com
  key: <your-key>
```

---

## Deploy to Kubernetes

### Deploy

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
```

### Check Status

```bash
kubectl get deployments
kubectl get pods
kubectl get services
```

### View Logs

```bash
kubectl logs <pod_name>
kubectl logs -f <pod_name>
```

---

## Scaling

### Manual Scaling

```bash
kubectl scale deployment agent --replicas=3
```

### Auto-Scaling

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: agent-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: agent
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

## Upgrades

### Rolling Update

```bash
kubectl set image deployment/agent \
  agent=my-registry/ai-defra-search-agent:v1.0.1

# Check status
kubectl rollout status deployment/agent
```

### Rollback

```bash
kubectl rollout undo deployment/agent
```

---

**Version:** 1.0.0  
**Last Updated:** March 20, 2026
