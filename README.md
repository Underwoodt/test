# AI DEFRA Search Knowledge - Documentation

Complete developer documentation for the AI DEFRA Search Knowledge RAG platform.

## 📚 Documentation Structure

### Getting Started
- **[README](./getting-started/README.md)** — Quick start guide (5 minutes)
- **[Installation](./getting-started/installation.md)** — Detailed setup instructions
- **[First Steps](./getting-started/first-steps.md)** — Your first search query

### Concepts & Design
- **[Functional Overview](./concepts/functional-overview.md)** — What this app does and why
- **[Architecture](./concepts/architecture.md)** — System design and components
- **[Data Models](./concepts/data-models.md)** — Database schemas and relationships
- **[RAG Pipeline](./concepts/rag-pipeline.md)** — How semantic search works

### API Reference
- **[API Overview](./api/README.md)** — All endpoints with examples
- **[Knowledge Groups API](./api/knowledge-groups.md)** — Manage document collections
- **[Documents API](./api/documents.md)** — Upload and track documents
- **[RAG Search API](./api/rag-search.md)** — Semantic search queries

### For Developers
- **[Development Setup](./development/setup.md)** — Local development environment
- **[Code Structure](./development/code-structure.md)** — Repository organization
- **[Design Patterns](./development/design-patterns.md)** — Architectural patterns used
- **[Testing Guide](./development/testing.md)** — How to test the application
- **[Extending the App](./development/extending.md)** — Add features and file types
- **[Quick Commands](./development/quick-commands.md)** — Common development tasks

### Deployment & Operations
- **[Docker Setup](./deployment/docker.md)** — Container deployment
- **[Production Deployment](./deployment/production.md)** — Deploy to Kubernetes
- **[Configuration](./deployment/configuration.md)** — Environment variables
- **[Monitoring & Logging](./deployment/monitoring.md)** — Observability setup
- **[Troubleshooting](./deployment/troubleshooting.md)** — Common issues and fixes

### Reference
- **[Technology Stack](./reference/technology-stack.md)** — Tools and dependencies
- **[Security](./reference/security.md)** — Access control and data protection
- **[Performance Tuning](./reference/performance.md)** — Optimization tips
- **[FAQ](./reference/faq.md)** — Frequently asked questions

## 🎯 Quick Navigation

### I want to...

| Goal | Start Here |
|------|-----------|
| Get the app running | [Quick Start](./getting-started/README.md) |
| Understand how it works | [Functional Overview](./concepts/functional-overview.md) |
| Use the API | [API Overview](./api/README.md) |
| Start developing | [Development Setup](./development/setup.md) |
| Add a new feature | [Extending the App](./development/extending.md) |
| Deploy to production | [Production Deployment](./deployment/production.md) |
| Fix a problem | [Troubleshooting](./deployment/troubleshooting.md) |
| Learn the architecture | [Architecture](./concepts/architecture.md) |

## 📖 Reading Paths

### For New Users (30 minutes)
1. [Quick Start](./getting-started/README.md) — Get it running
2. [Functional Overview](./concepts/functional-overview.md) — Understand the concept
3. [API Overview](./api/README.md) — See what you can do

### For Developers (2 hours)
1. [Development Setup](./development/setup.md) — Set up your environment
2. [Code Structure](./development/code-structure.md) — Understand the codebase
3. [Design Patterns](./development/design-patterns.md) — Learn how we code
4. [Testing Guide](./development/testing.md) — Write tests

### For DevOps/Platform Teams (1 hour)
1. [Architecture](./concepts/architecture.md) — System design
2. [Docker Setup](./deployment/docker.md) — Containerization
3. [Production Deployment](./deployment/production.md) — Deploy at scale
4. [Configuration](./deployment/configuration.md) — Environment setup

## 🔑 Key Concepts at a Glance

### What Is This?
A **Retrieval-Augmented Generation (RAG) platform** that:
- Ingests documents (PDF, Word, PowerPoint, JSONL)
- Converts them to semantic embeddings (AWS Bedrock)
- Enables fast semantic search across documents
- Returns relevant chunks ranked by similarity

### Why Use It?
- **Grounded Answers** — Search your documents instead of hallucinating
- **User-Scoped** — Each user sees only their own documents
- **Scalable** — Built on PostgreSQL + pgvector for millions of vectors
- **Flexible** — Supports multiple document formats
- **Async** — Non-blocking operations throughout

### Core Stack
```
FastAPI (REST API) + PostgreSQL (vectors) + MongoDB (metadata) + AWS Bedrock (embeddings)
```

## 📊 Documentation Stats

- **Total Pages:** 20+
- **Total Words:** 15,000+
- **Code Examples:** 50+
- **Diagrams:** 10+
- **API Endpoints:** 12+

## 🤝 Contributing to Docs

Improvements welcome! See [Contributing Guide](./CONTRIBUTING.md) for details.

### Before committing doc changes:
- [ ] Spelling and grammar checked
- [ ] Code examples tested
- [ ] Diagrams render correctly in Markdown
- [ ] All links are internal and work
- [ ] Consistent formatting with existing docs

## 📝 License

THIS INFORMATION IS LICENSED UNDER THE CONDITIONS OF THE OPEN GOVERNMENT LICENCE found at:
http://www.nationalarchives.gov.uk/doc/open-government-licence/version/3

Contains public sector information licensed under the Open Government license v3

## 🔗 Quick Links

- **Main Repository:** [ai-defra-search-knowledge](https://github.com/DEFRA/ai-defra-search-knowledge)
- **API Documentation:** [Swagger UI](http://localhost:8085/docs) (when running locally)
- **Issue Tracker:** GitHub Issues
- **Discussions:** GitHub Discussions

---

**Last Updated:** March 19, 2026  
**Documentation Version:** 1.0  
**Repository Version:** 0.1.0
