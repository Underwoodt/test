# Installation - Setup Guide

**Step-by-step guide to get system running locally**

---

## Overview

Setup takes about 15-30 minutes depending on your internet speed.

---

## Step 1: Clone Core Repository

```bash
# Clone the main repository
git clone https://github.com/DEFRA/ai-defra-search-core
cd ai-defra-search-core

# Verify you're in the right directory
ls
# You should see: README.md, compose.yml, pyproject.toml, etc.
```

---

## Step 2: Install Python Dependencies

**Option A: Using uv (Recommended)**

```bash
# Install uv if you haven't already
curl -LsSf https://astral.sh/uv/install.sh | sh

# Sync dependencies
uv sync --frozen

# Verify installation
uv --version
```

**Option B: Using pip**

```bash
# Create virtual environment
python3 -m venv .venv

# Activate virtual environment
source .venv/bin/activate  # macOS/Linux
# or
.venv\Scripts\activate  # Windows

# Install dependencies
pip install -r requirements.txt
```

---

## Step 3: Clone Service Repositories

**Automated (Recommended):**

```bash
# Clone all service repositories automatically
uv run task clone

# This creates:
# services/
#   ├─ ai-defra-search-frontend
#   ├─ ai-defra-search-agent
#   ├─ ai-defra-search-knowledge
#   ├─ ai-defra-search-journey-tests
#   ├─ ai-defra-search-perf-tests
#   └─ ai-defra-search-aws-bedrock-stub
```

**Manual (Alternative):**

```bash
mkdir -p services
cd services

git clone https://github.com/DEFRA/ai-defra-search-frontend.git
git clone https://github.com/DEFRA/ai-defra-search-agent.git
git clone https://github.com/DEFRA/ai-defra-search-knowledge.git
git clone https://github.com/DEFRA/ai-defra-search-journey-tests.git
git clone https://github.com/DEFRA/ai-defra-search-perf-tests.git
git clone https://github.com/DEFRA/ai-defra-search-aws-bedrock-stub.git

cd ..  # Back to core directory
```

---

## Step 4: Start Docker Services

```bash
# Start all services in background
docker compose up -d

# Verify services are running
docker compose ps

# You should see these containers:
# - ai-defra-search-frontend (running)
# - ai-defra-search-agent (running)
# - ai-defra-search-knowledge (running)
# - postgres (running)
# - mongo (running)
# - redis (running)
# - localstack (running)
# - traefik (running)
```

---

## Step 5: Wait for Services to Initialize

```bash
# Watch logs to see initialization
docker compose logs -f

# Look for these messages:
# - postgres: "ready to accept connections"
# - mongo: "server started on port 27017"
# - redis: "Ready to accept connections"
# - agent: "Application startup complete"
# - knowledge: "Application startup complete"

# Press Ctrl+C to exit logs
```

First startup typically takes 1-2 minutes as services initialize.

---

## Step 6: Verify Setup

```bash
# Check Frontend
curl http://frontend.localhost

# Check Agent API health
curl http://localhost:8086/health
# Should return: {"status":"healthy"}

# Check Knowledge API health
curl http://knowledge.localhost/health
# Should return: {"status":"healthy"}

# Check databases
docker compose exec postgres psql -U postgres -c "SELECT version();"
docker compose exec mongo mongosh --eval "db.version()"
docker compose exec redis redis-cli ping
```

---

## Step 7: Access the Application

Open your browser and navigate to:

### Frontend
- URL: http://frontend.localhost
- You should see the chat interface

### API Documentation

**Agent API:**
- URL: http://localhost:8086/docs
- Interactive Swagger documentation

**Knowledge API:**
- URL: http://knowledge.localhost/docs
- Interactive Swagger documentation

### Infrastructure Monitoring

**Traefik Dashboard:**
- URL: http://localhost:8080
- View all routes and services

---

## Troubleshooting Installation

### Docker won't start
```bash
# Restart Docker daemon
docker daemon

# Or restart Docker desktop (Mac/Windows)
```

### Port conflicts
```bash
# Find what's using port 3000
lsof -i :3000

# Kill the process
kill -9 <PID>
```

### Services failing to start
```bash
# Check logs
docker compose logs agent
docker compose logs knowledge

# Rebuild images
docker compose build --no-cache

# Restart
docker compose restart
```

### Database errors
```bash
# Clean up volumes
docker compose down -v

# Start fresh
docker compose up -d

# Wait longer (2-3 minutes)
```

---

## Installation Complete!

You're ready to start developing. Continue to:
- [VERIFICATION.md](VERIFICATION.md) - Verify everything works
- [Setup Guide](../setup/) - More configuration options
- [Architecture](../architecture/) - Understand system design

---

**Document Version:** 1.0.0  
**Last Updated:** March 20, 2026
