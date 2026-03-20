# Prerequisites - System Requirements

**What you need before getting started**

---

## Operating System

✅ **macOS** 11+ (Intel or Apple Silicon)  
✅ **Linux** (Ubuntu 20.04+, Debian 11+, CentOS 8+)  
✅ **Windows** 10/11 with WSL2  

---

## Required Software

### Docker & Docker Compose

**Docker** - Container runtime
- Version: 20.10+
- Download: https://docs.docker.com/get-docker/

**Docker Compose** - Orchestration
- Version: 2.0+
- Included with Docker Desktop

**Verify installation:**
```bash
docker --version
docker compose --version
```

### Python

**Version:** 3.13+

**Installation:**
```bash
# macOS (using Homebrew)
brew install python@3.13

# Ubuntu/Debian
sudo apt-get install python3.13

# Verify
python3 --version
```

### Git

**Version:** Any recent version

**Installation:**
```bash
# macOS
brew install git

# Ubuntu/Debian
sudo apt-get install git

# Verify
git --version
```

### Optional: uv (Python Package Manager)

**Recommended** for faster dependency installation

**Installation:**
```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Verify
uv --version
```

---

## System Resources

### Minimum Requirements

| Resource | Amount |
|----------|--------|
| **CPU** | 2 cores |
| **RAM** | 4 GB |
| **Disk** | 10 GB |

### Recommended Requirements

| Resource | Amount |
|----------|--------|
| **CPU** | 4 cores |
| **RAM** | 8 GB |
| **Disk** | 20 GB |

### For Running All Services

If running all services locally:
- PostgreSQL + pgvector (500 MB)
- MongoDB (300 MB)
- Redis (100 MB)
- Frontend (React dependencies: 500 MB)
- Agent (Python dependencies: 300 MB)
- Knowledge (Python dependencies: 300 MB)

**Total disk needed:** 2-3 GB for dependencies

---

## Network Requirements

### Open Ports

| Service | Port | Protocol |
|---------|------|----------|
| Frontend | 3000 | HTTP/TCP |
| Agent | 8086 | HTTP/TCP |
| Knowledge | 8085 | HTTP/TCP |
| Traefik | 8080 | HTTP/TCP |
| Bedrock Stub | 4566 | HTTP/TCP |
| PostgreSQL | 5432 | TCP |
| MongoDB | 27017 | TCP |
| Redis | 6379 | TCP |

### DNS Requirements

**For local development:** Update `/etc/hosts` or use `.localhost` domains

```
127.0.0.1 frontend.localhost
127.0.0.1 knowledge.localhost
```

---

## Development Tools (Optional)

### Code Editor

Recommended editors:
- **VS Code** (https://code.visualstudio.com/)
- **PyCharm** (https://www.jetbrains.com/pycharm/)
- **IntelliJ IDEA** (https://www.jetbrains.com/idea/)

### VS Code Extensions (if using VS Code)

```
ms-vscode.remote-containers
ms-vscode.remote-wsl
python
prettier
eslint
thunder-client
```

### Command Line Tools

**macOS:**
```bash
brew install jq     # JSON processor
brew install curl   # HTTP client
```

**Ubuntu/Debian:**
```bash
sudo apt-get install jq curl
```

---

## GitHub Access

### SSH Key Setup (Recommended)

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your-email@example.com"

# Add to SSH agent
ssh-add ~/.ssh/id_ed25519

# Copy key to GitHub
cat ~/.ssh/id_ed25519.pub
# Visit: https://github.com/settings/ssh/new
```

### HTTPS Alternative

```bash
# Install git credential manager
brew install git-credential-osxkeychain  # macOS
sudo apt-get install git-credential-pass # Linux
```

---

## Disk Space Check

### macOS
```bash
df -h
```

### Linux
```bash
df -h
```

### Windows (PowerShell)
```powershell
Get-Volume
```

**Ensure at least 10-20 GB free**

---

## Verification Checklist

Run these commands to verify your setup:

```bash
# Docker
docker run hello-world

# Docker Compose
docker compose --version

# Python
python3 --version
python3 -m pip --version

# Git
git --version

# Internet connectivity
curl https://api.github.com

# DNS resolution
nslookup github.com
```

---

## Troubleshooting

### Docker not found
```bash
# Restart Docker daemon
docker --version

# If fails, reinstall from:
# https://docs.docker.com/get-docker/
```

### Python not found
```bash
# Verify installation
python3 --version

# Add to PATH if needed (macOS/Linux)
export PATH="/usr/local/opt/python@3.13/bin:$PATH"
```

### Port already in use
```bash
# Find what's using port 3000
lsof -i :3000

# Kill the process
kill -9 <PID>
```

### Disk space low
```bash
# Clean up Docker
docker system prune

# Remove unused images
docker image prune
```

---

## Next Steps

Once prerequisites are installed, proceed to [INSTALLATION.md](INSTALLATION.md)

---

**Document Version:** 1.0.0  
**Last Updated:** March 20, 2026
