# Development Setup

Complete guide to setting up a local development environment.

## Prerequisites

### Required
- **Python 3.13+** — [Install Python](https://www.python.org/downloads/)
- **Git** — [Install Git](https://git-scm.com/)
- **Docker & Docker Compose** — [Install Docker](https://docs.docker.com/get-docker/)

### Optional
- **VS Code** — [Install VS Code](https://code.visualstudio.com/)
- **Ruff Extension** — Search for "Ruff" in VS Code extensions

### Verify Installation

```bash
python --version          # Should be 3.13+
git --version            # Any recent version
docker --version         # Any recent version
docker compose version   # Should be 2.0+
```

---

## Step 1: Clone Repository

```bash
git clone https://github.com/DEFRA/ai-defra-search-knowledge.git
cd ai-defra-search-knowledge
```

---

## Step 2: Install Python Tools

### Install pipx (if not already installed)

```bash
# macOS
brew install pipx

# Linux
sudo apt-get install pipx

# Windows
python -m pip install --user pipx
```

### Install uv via pipx

```bash
pipx install uv
```

Verify: `uv --version`

---

## Step 3: Create Virtual Environment & Install Dependencies

```bash
# Install dependencies
uv sync

# Activate virtual environment
source .venv/bin/activate      # Linux/macOS
# or
.venv\Scripts\activate         # Windows

# Verify
python --version               # Should show 3.13+
```

---

## Step 4: Install Pre-commit Hooks

Pre-commit hooks automatically run code checks before each commit.

```bash
pre-commit install
```

Test it:
```bash
pre-commit run --all-files
```

---

## Step 5: Create Local Secrets File

Create `compose/secrets.env` (git-ignored):

```bash
# Database passwords
POSTGRES_PASSWORD=dev-password
MONGO_URI=mongodb://mongo:27017

# S3 and Bedrock
KNOWLEDGE_UPLOAD_BUCKET_NAME=ai-defra-search-knowledge-upload-local
AWS_ENDPOINT_URL=http://localstack:4566
BEDROCK_ENDPOINT_URL=http://localstack:4566
```

---

## Step 6: Start Services

### Option A: Use Startup Script (Recommended)

```bash
./scripts/start_dev_server.sh
```

This automatically:
- Starts PostgreSQL with pgvector
- Starts MongoDB
- Starts LocalStack (S3, Bedrock)
- Starts FastAPI with hot-reload

**Wait for:** `Uvicorn running on http://127.0.0.1:8085`

### Option B: Use Docker Compose

```bash
# Start all services in background
docker compose --profile service up -d

# View logs
docker compose logs -f service

# Stop all services
docker compose down
```

---

## Step 7: Verify Setup

### Check API is Running

```bash
curl http://localhost:8085/health
```

Expected: `{"status":"healthy"}`

### Open Swagger UI

Visit: http://localhost:8085/docs

You should see all available endpoints listed.

---

## Development Workflow

### Writing Code

```bash
# 1. Create a feature branch
git checkout -b feature/my-feature

# 2. Write code (IDE automatically formats with Ruff)

# 3. Run tests
uv run pytest

# 4. Check formatting & linting
uv run ruff format . --check
uv run ruff check .

# 5. Commit (pre-commit hooks run automatically)
git commit -m "Add my feature"

# 6. Push
git push origin feature/my-feature
```

### Running Tests

```bash
# All tests
uv run pytest

# Verbose output
uv run pytest -vv

# Specific test file
uv run pytest tests/rag/test_vector_search.py

# Specific test function
uv run pytest tests/rag/test_vector_search.py::test_search_vectors

# With coverage
uv run coverage run -m pytest
uv run coverage html
open coverage/coverage_html/index.html
```

### Code Formatting

```bash
# Format all Python files
uv run ruff format .

# Check (don't change)
uv run ruff format . --check

# Lint with fixes
uv run ruff check . --fix

# Check for linting issues
uv run ruff check .
```

### Hot Reload During Development

The FastAPI server auto-reloads when you change files. You don't need to restart it.

If using Docker Compose, press `w` to enable watch mode:
```bash
docker compose --profile service up
# Press 'w' to enable watch mode
```

---

## IDE Setup (VS Code)

### Install Extensions

1. **Ruff** — For formatting and linting
   - Search for "charliermarsh.ruff" in Extensions
   - Install and reload

2. **Python** — For debugging
   - Search for "ms-python.python" in Extensions

### Configure Settings

Create or edit `.vscode/settings.json`:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "charliermarsh.ruff",
  "editor.codeActionsOnSave": {
    "source.fixAll.ruff": "explicit",
    "source.organizeImports.ruff": "explicit"
  },
  "[python]": {
    "editor.defaultFormatter": "charliermarsh.ruff",
    "editor.formatOnSave": true
  }
}
```

Now code will auto-format on save!

---

## Debugging

### Add a Breakpoint

```python
# In app/rag/router.py
async def search(...):
    import pdb; pdb.set_trace()  # ← Breakpoint here
    # Code execution pauses here
```

### Trigger and Debug

1. Start the server: `./scripts/start_dev_server.sh`
2. Call the endpoint from Swagger UI or curl
3. Server console shows `(Pdb)` prompt
4. Debug commands: `n` (next), `c` (continue), `p variable` (print)

### Debug in VS Code

1. Install Python extension (see above)
2. Click the debug icon (triangle-play) in sidebar
3. Select "Python: FastAPI"
4. Set breakpoints (click line number)
5. Call endpoint
6. VS Code debugger pauses at breakpoint

---

## Database Access

### PostgreSQL

View database directly:

```bash
# Connect to PostgreSQL
docker compose exec postgres psql \
  -U ai_defra_search_knowledge \
  -d ai_defra_search_knowledge

# List tables
\dt

# Query vectors
SELECT count(*) FROM knowledge_vectors;

# Exit
\q
```

### MongoDB

View MongoDB collections:

```bash
# Connect to MongoDB
docker compose exec mongo mongosh

# Switch database
use ai-defra-search-knowledge

# List collections
show collections

# View documents
db.documents.find()

# View knowledge groups
db.knowledgeGroups.find()

# Exit
exit
```

---

## Dependency Management

### Add a Dependency

```bash
# Add regular dependency
uv add package-name

# Add dev dependency
uv add --group dev package-name

# Update dependencies
uv sync --upgrade
```

This updates both `pyproject.toml` and `uv.lock`.

### Check for Outdated Packages

```bash
uv pip list --outdated
```

---

## Environment Variables

### Development Defaults (compose/aws.env)

```bash
PYTHON_ENV=development
AWS_REGION=eu-west-2
LOG_LEVEL=DEBUG
```

### Local Secrets (compose/secrets.env)

Create this file (git-ignored):

```bash
POSTGRES_PASSWORD=<choose-a-password>
MONGO_URI=mongodb://mongo:27017
```

---

## Common Issues

| Issue | Solution |
|-------|----------|
| "Module not found" | Run `uv sync` |
| "Port already in use" | Kill process: `lsof -i :8085 \| tail -1 \| awk '{print $2}' \| xargs kill -9` |
| "Connection refused" (Postgres) | Ensure Docker is running: `docker compose ps` |
| "Permission denied" on script | Run `chmod +x scripts/start_dev_server.sh` |
| Tests timeout | Check services: `docker compose ps` |
| Import errors | Activate venv: `source .venv/bin/activate` |

---

## Project Structure

```
app/
├── main.py                 # FastAPI app setup
├── config.py               # Configuration
├── common/                 # Shared utilities
├── ingest/                 # Document ingestion
├── rag/                    # Search functionality
├── document/               # Document management
├── knowledge_group/        # Knowledge group management
└── health/                 # Health checks

tests/
├── common/                 # Unit tests
├── entrypoints/           # API tests
├── ingest/                # Ingestion tests
└── rag/                   # Search tests
```

See [Code Structure](./code-structure.md) for details.

---

## Next Steps

- **First Query:** Try [First Steps](../getting-started/first-steps.md)
- **Code Patterns:** See [Design Patterns](./design-patterns.md)
- **Write Tests:** See [Testing Guide](./testing.md)
- **Add Features:** See [Extending the App](./extending.md)

---

**Setup Time:** 10-15 minutes  
**Next:** Run your first test with `uv run pytest`
