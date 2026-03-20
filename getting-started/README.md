# Getting Started

## Quick Start (5 Minutes)

Get the AI DEFRA Search Knowledge application running locally in 5 minutes.

### Prerequisites

- **Python 3.13+**
- **Docker & Docker Compose** (for local services)
- **pipx** (Python package installer)

### Step 1: Clone and Install

```bash
# Clone the repository
git clone https://github.com/DEFRA/ai-defra-search-knowledge.git
cd ai-defra-search-knowledge

# Install uv
pipx install uv

# Install dependencies
uv sync

# Activate virtual environment
source .venv/bin/activate  # Linux/Mac
# or
.venv\Scripts\activate     # Windows

# Install pre-commit hooks
pre-commit install
```

### Step 2: Start the Application

```bash
# Use the startup script (recommended)
./scripts/start_dev_server.sh
```

This will:
- ✅ Start PostgreSQL + pgvector
- ✅ Start MongoDB
- ✅ Start LocalStack (S3, Bedrock)
- ✅ Start FastAPI server with hot-reload

**Wait for:** `Uvicorn running on http://127.0.0.1:8085`

### Step 3: Verify Installation

1. **Open API Docs:** http://localhost:8085/docs
2. **Health Check:** `curl http://localhost:8085/health`
3. **Try an Example:** Click any endpoint in Swagger UI

### What's Next?

- **First Query:** See [First Steps](./first-steps.md)
- **API Reference:** See [API Overview](../api/README.md)
- **Development:** See [Development Setup](../development/setup.md)

---

## What You Just Installed

| Service | Port | Purpose |
|---------|------|---------|
| FastAPI App | 8085 | REST API |
| PostgreSQL | 5432 | Vector database |
| MongoDB | 27017 | Metadata database |
| LocalStack | 4566 | AWS services (local) |

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Port 8085 in use" | Kill process: `lsof -i :8085 \| grep -v PID \| awk '{print $2}' \| xargs kill -9` |
| "Docker not running" | Start Docker Desktop or daemon |
| "Module not found" | Run `uv sync` again |
| "Permission denied on script" | Run `chmod +x scripts/start_dev_server.sh` |

## Next Steps

- ✅ [First Steps](./first-steps.md) — Create a knowledge group and search
- 📖 [Installation](./installation.md) — Detailed setup guide
- 🎯 [Functional Overview](../concepts/functional-overview.md) — Understand what it does

---

**Total Time:** ~5 minutes  
**Next Up:** 10 minutes to your first search query
