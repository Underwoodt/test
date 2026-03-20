# Setup - Getting Started

**Getting the system up and running locally**

---

## What's in This Section?

| File | Purpose | Time |
|------|---------|------|
| **PREREQUISITES.md** | What you need installed | 10 min |
| **INSTALLATION.md** | Step-by-step setup | 20 min |
| **VERIFICATION.md** | Verify everything works | 10 min |

---

## Quick Start

```bash
# 1. Check prerequisites
# Read: PREREQUISITES.md (10 min)

# 2. Clone and install
git clone https://github.com/DEFRA/ai-defra-search-core
cd ai-defra-search-core
uv sync  # Install dependencies

# 3. Clone services
uv run task clone  # Or manual clone (see INSTALLATION.md)

# 4. Start services
docker compose up -d

# 5. Verify
# Open browser: http://frontend.localhost
```

**Total time:** ~30-40 minutes

---

## Step-by-Step Sections

### PREREQUISITES.md (Do First!)

**What you need before starting:**
- Operating system requirements
- Required software (Docker, Python, Git)
- System resources (CPU, RAM, disk)
- Network requirements
- Optional development tools

**Action:** Install anything missing from this list

---

### INSTALLATION.md (Then Do This)

**Step-by-step installation guide:**

**Steps:**
1. Clone core repository
2. Install Python dependencies
3. Clone service repositories
4. Start Docker services
5. Wait for initialization
6. Access the application

**Estimated time:** 20-30 minutes

**What you'll have after:**
- Core repo cloned
- All services running
- Database initialized
- Application accessible

---

### VERIFICATION.md (Finally Do This)

**Confirm everything is working:**

**Checks:**
- Docker containers running
- Frontend accessible
- APIs responding
- Databases connected
- Network connectivity
- Health checks passing

**Expected results:**
- All containers running
- All API endpoints responding
- All databases accessible
- No errors in logs

---

## Common Scenarios

### I want to get started quickly
1. Skip prerequisites (assume you know what you're doing)
2. Run INSTALLATION.md (20 min)
3. Skip VERIFICATION.md and just try using it

**Time:** 20 minutes

### I'm new to this stack
1. Read PREREQUISITES.md carefully (15 min)
2. Install missing software (10 min)
3. Follow INSTALLATION.md step-by-step (20 min)
4. Run VERIFICATION.md (10 min)

**Time:** 55 minutes

### I keep getting errors
1. Check PREREQUISITES.md for missing software
2. Follow INSTALLATION.md carefully
3. Run VERIFICATION.md to diagnose issues
4. See troubleshooting/ for help

---

## Installation Overview

```
Start
  ↓
Check PREREQUISITES.md
  ├─ Have Docker? YES → continue
  └─ Have Docker? NO → install it
  ↓
Follow INSTALLATION.md
  ├─ Step 1: Clone core
  ├─ Step 2: Install Python deps
  ├─ Step 3: Clone services
  ├─ Step 4: Start Docker
  └─ Step 5: Wait for init
  ↓
Run VERIFICATION.md
  ├─ Check containers
  ├─ Check APIs
  ├─ Check databases
  └─ Check health
  ↓
Done! → Go to frontend.localhost
```

---

## What Gets Started?

After following INSTALLATION.md, you'll have:

| Service | Port | Status |
|---------|------|--------|
| Frontend | 3000 | Running |
| Agent API | 8086 | Running |
| Knowledge API | 8085 | Running |
| PostgreSQL | 5432 | Running |
| MongoDB | 27017 | Running |
| Redis | 6379 | Running |
| Bedrock Stub | 4566 | Running |
| Traefik | 8080 | Running |

---

## Access Points

After setup:

**Frontend:**
- URL: http://frontend.localhost
- Or: http://localhost:3000

**Agent API Documentation:**
- URL: http://localhost:8086/docs
- Swagger interactive docs

**Knowledge API Documentation:**
- URL: http://knowledge.localhost/docs
- Swagger interactive docs

**Infrastructure Dashboard:**
- URL: http://localhost:8080
- Traefik dashboard showing all services

---

## Troubleshooting Setup

### Installation fails at step X
→ See [troubleshooting/](../troubleshooting/COMMON-ISSUES.md)

### Docker won't start
→ Check PREREQUISITES.md for Docker installation

### Port already in use
→ See [troubleshooting/COMMON-ISSUES.md](../troubleshooting/COMMON-ISSUES.md)

### Can't access frontend
→ Run VERIFICATION.md to diagnose

---

## Next Steps After Setup

1. **Learn the system:** Go to [docs/](../docs/)
2. **Use the APIs:** Go to [api-reference/](../api-reference/)
3. **Deploy:** Go to [deployment/](../deployment/)
4. **Get help:** Go to [troubleshooting/](../troubleshooting/)

---

## Document References

- PREREQUISITES.md - What to install
- INSTALLATION.md - How to install
- VERIFICATION.md - How to verify

---

**Document Version:** 1.0.0  
**Last Updated:** March 20, 2026
