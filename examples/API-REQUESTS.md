# API Request Examples

**Ready-to-use API request examples**

---

## Chat API Examples

### Simple Message

```bash
curl -X POST http://localhost:8086/chat \
  -H "Content-Type: application/json" \
  -H "user-id: user@example.com" \
  -d '{
    "message": "Hello!",
    "conversation_id": "conv-001",
    "user_id": "user@example.com"
  }'
```

### Multi-Turn Conversation

```bash
# Message 1
curl -X POST http://localhost:8086/chat \
  -H "Content-Type: application/json" \
  -H "user-id: user@example.com" \
  -d '{
    "message": "What are grant requirements?",
    "conversation_id": "conv-001",
    "user_id": "user@example.com"
  }'

# Message 2 (same conversation)
curl -X POST http://localhost:8086/chat \
  -H "Content-Type: application/json" \
  -H "user-id: user@example.com" \
  -d '{
    "message": "What is the deadline?",
    "conversation_id": "conv-001",
    "user_id": "user@example.com"
  }'
```

### Get Conversation

```bash
curl http://localhost:8086/conversations/conv-001 \
  -H "user-id: user@example.com"
```

---

## Search API Examples

### Basic Search

```bash
curl -X POST http://knowledge.localhost/rag/search \
  -H "Content-Type: application/json" \
  -d '{
    "query": "grant requirements",
    "knowledge_group_ids": [],
    "max_results": 5
  }'
```

### Search with Specific Groups

```bash
curl -X POST http://knowledge.localhost/rag/search \
  -H "Content-Type: application/json" \
  -d '{
    "query": "eligibility criteria",
    "knowledge_group_ids": ["group-123", "group-456"],
    "max_results": 10
  }'
```

### Upload Document

```bash
curl -X POST http://knowledge.localhost/documents \
  -H "Content-Type: application/json" \
  -d '{
    "documents": [
      {
        "file_name": "grant.pdf",
        "knowledge_group_id": "group-123",
        "s3_key": "s3://bucket/grant.pdf"
      }
    ]
  }'
```

### Check Upload Status

```bash
curl http://knowledge.localhost/upload-status/up-123
```

---

## Python Examples

### Simple Chat

```python
import requests

response = requests.post(
    'http://localhost:8086/chat',
    headers={'user-id': 'user@example.com'},
    json={
        'message': 'What are grant requirements?',
        'conversation_id': 'conv-001',
        'user_id': 'user@example.com'
    }
)
print(response.json())
```

### Search with Error Handling

```python
import requests

try:
    response = requests.post(
        'http://knowledge.localhost/rag/search',
        json={
            'query': 'requirements',
            'knowledge_group_ids': [],
            'max_results': 5
        },
        timeout=10
    )
    response.raise_for_status()
    results = response.json()
    for result in results:
        print(f"Score: {result['similarity_score']}")
        print(f"Content: {result['content'][:100]}...")
except requests.exceptions.RequestException as e:
    print(f"Error: {e}")
```

### Multi-request Workflow

```python
import requests
import json

BASE_AGENT = 'http://localhost:8086'
BASE_KNOWLEDGE = 'http://knowledge.localhost'
USER_ID = 'user@example.com'

# 1. Send message
chat = requests.post(
    f'{BASE_AGENT}/chat',
    headers={'user-id': USER_ID},
    json={
        'message': 'Requirements?',
        'conversation_id': 'conv-001',
        'user_id': USER_ID
    }
).json()
print(json.dumps(chat, indent=2))

# 2. Search documents
search = requests.post(
    f'{BASE_KNOWLEDGE}/rag/search',
    json={
        'query': 'requirements',
        'knowledge_group_ids': [],
        'max_results': 5
    }
).json()
print(f"Found {len(search)} results")
```

---

## JavaScript Examples

### Basic Fetch

```javascript
const response = await fetch('http://localhost:8086/chat', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'user-id': 'user@example.com'
  },
  body: JSON.stringify({
    message: 'What are requirements?',
    conversation_id: 'conv-001',
    user_id: 'user@example.com'
  })
});

const data = await response.json();
console.log(data);
```

### With Error Handling

```javascript
async function makeRequest(url, options) {
  try {
    const response = await fetch(url, options);
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    console.error('Request failed:', error);
    throw error;
  }
}

// Usage
const result = await makeRequest('http://localhost:8086/chat', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'user-id': 'user@example.com'
  },
  body: JSON.stringify({
    message: 'Hello',
    conversation_id: 'conv-001',
    user_id: 'user@example.com'
  })
});
```

---

## Response Examples

### Chat Response

```json
{
  "response": "Grant requirements include: 1. Must be nonprofit organization, 2. Minimum 3 years operating history, 3. Annual revenue under $2M",
  "conversation_id": "conv-001",
  "sources": [
    {
      "document_id": "doc-123",
      "file_name": "grant-guidelines.pdf",
      "similarity_score": 0.94
    },
    {
      "document_id": "doc-456",
      "file_name": "eligibility-criteria.pdf",
      "similarity_score": 0.89
    }
  ],
  "metadata": {
    "tokens_used": 150,
    "search_results": 2,
    "latency_ms": 1234
  }
}
```

### Search Response

```json
[
  {
    "chunk_id": "chunk-789",
    "content": "Eligibility requirements: Nonprofit status verified by IRS exemption letter",
    "similarity_score": 0.92,
    "document_id": "doc-123",
    "file_name": "grant-guidelines.pdf"
  },
  {
    "chunk_id": "chunk-790",
    "content": "Minimum 3 years of established operations required",
    "similarity_score": 0.88,
    "document_id": "doc-456",
    "file_name": "eligibility-criteria.pdf"
  }
]
```

---

**Version:** 1.0.0  
**Last Updated:** March 20, 2026
