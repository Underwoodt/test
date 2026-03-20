# API Examples

**Code examples for all API endpoints**

---

## Chat API Examples

### Using cURL

**Send a message:**
```bash
curl -X POST http://localhost:8086/chat \
  -H "Content-Type: application/json" \
  -H "user-id: user@example.com" \
  -d '{
    "message": "What are grant requirements?",
    "conversation_id": "conv-123",
    "user_id": "user@example.com"
  }'
```

**Get conversation:**
```bash
curl http://localhost:8086/conversations/conv-123 \
  -H "user-id: user@example.com"
```

### Using Python

```python
import requests

headers = {
    "Content-Type": "application/json",
    "user-id": "user@example.com"
}

data = {
    "message": "What are grant requirements?",
    "conversation_id": "conv-123",
    "user_id": "user@example.com"
}

response = requests.post(
    "http://localhost:8086/chat",
    json=data,
    headers=headers
)

print(response.json())
```

### Using JavaScript

```javascript
fetch('http://localhost:8086/chat', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'user-id': 'user@example.com'
  },
  body: JSON.stringify({
    message: "What are grant requirements?",
    conversation_id: "conv-123",
    user_id: "user@example.com"
  })
})
.then(r => r.json())
.then(data => console.log(data));
```

---

## Search API Examples

### Using cURL

**Semantic search:**
```bash
curl -X POST http://knowledge.localhost/rag/search \
  -H "Content-Type: application/json" \
  -d '{
    "query": "grant requirements",
    "knowledge_group_ids": ["group-123"],
    "max_results": 5
  }'
```

**Check upload status:**
```bash
curl http://knowledge.localhost/upload-status/up-123
```

### Using Python

```python
import requests

data = {
    "query": "grant requirements",
    "knowledge_group_ids": ["group-123"],
    "max_results": 5
}

response = requests.post(
    "http://knowledge.localhost/rag/search",
    json=data
)

results = response.json()
for chunk in results:
    print(f"Score: {chunk['similarity_score']}")
    print(f"Content: {chunk['content']}")
    print("---")
```

### Using JavaScript

```javascript
const response = await fetch('http://knowledge.localhost/rag/search', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    query: "grant requirements",
    knowledge_group_ids: ["group-123"],
    max_results: 5
  })
});

const results = await response.json();
results.forEach(chunk => {
  console.log(`Score: ${chunk.similarity_score}`);
  console.log(`Content: ${chunk.content}`);
});
```

---

## Complete Workflow Example

### Python

```python
import requests
import json

BASE_AGENT = "http://localhost:8086"
BASE_KNOWLEDGE = "http://knowledge.localhost"
USER_ID = "user@example.com"

# 1. Send a chat message
chat_response = requests.post(
    f"{BASE_AGENT}/chat",
    headers={"user-id": USER_ID},
    json={
        "message": "What are the requirements?",
        "conversation_id": "conv-1",
        "user_id": USER_ID
    }
)
print("Chat response:", chat_response.json())

# 2. Get conversation history
conv_response = requests.get(
    f"{BASE_AGENT}/conversations/conv-1",
    headers={"user-id": USER_ID}
)
print("Conversation:", conv_response.json())

# 3. Search for documents
search_response = requests.post(
    f"{BASE_KNOWLEDGE}/rag/search",
    json={
        "query": "requirements",
        "knowledge_group_ids": [],
        "max_results": 5
    }
)
print("Search results:", search_response.json())
```

---

## Error Handling

### Python with Error Handling

```python
import requests
from requests.exceptions import RequestException

def make_chat_request(message, conv_id):
    try:
        response = requests.post(
            "http://localhost:8086/chat",
            headers={"user-id": "user@example.com"},
            json={
                "message": message,
                "conversation_id": conv_id,
                "user_id": "user@example.com"
            },
            timeout=30
        )
        response.raise_for_status()
        return response.json()
    except requests.exceptions.ConnectionError:
        print("Cannot connect to Chat API")
    except requests.exceptions.Timeout:
        print("Chat API timeout")
    except requests.exceptions.HTTPError as e:
        print(f"HTTP error: {e.response.status_code}")
    except Exception as e:
        print(f"Unexpected error: {e}")

result = make_chat_request("Hello", "conv-123")
```

---

## Pagination Example

### Search with Pagination

```bash
# First page
curl "http://knowledge.localhost/rag/search" \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "query": "requirements",
    "knowledge_group_ids": ["group-123"],
    "max_results": 10,
    "offset": 0
  }'

# Next page
curl "http://knowledge.localhost/rag/search" \
  -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "query": "requirements",
    "knowledge_group_ids": ["group-123"],
    "max_results": 10,
    "offset": 10
  }'
```

---

**Version:** 1.0.0  
**Last Updated:** March 20, 2026
