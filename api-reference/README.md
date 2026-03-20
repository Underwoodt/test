# API Reference

Complete API documentation for all services.

## Files

- **AGENT-API.md** - Chat API endpoints
- **KNOWLEDGE-API.md** - Search and ingestion API
- **EXAMPLES.md** - Code examples and cURL requests

## Quick Reference

### Agent API (Port 8086)
- POST /chat - Send a chat message
- GET /conversations/{id} - Get conversation history
- GET /health - Health check

### Knowledge API (Port 8085)
- POST /documents - Upload documents
- POST /rag/search - Semantic search
- GET /upload-status/{id} - Check ingestion status
- GET /health - Health check

## Examples

See EXAMPLES.md for:
- cURL commands
- Python examples
- JavaScript examples
- Request/response formats

---

See FULL-DOCUMENTATION.md → "API Reference" for complete details
