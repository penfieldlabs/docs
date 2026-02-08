# Quick Start Guide

Get from zero to your first stored memory in under 5 minutes.

---

## Prerequisites

- A Penfield account ([sign up](https://portal.penfield.app))
- An API key (from Portal → Settings → API Keys)

---

## Step 1: Get Your JWT Token

Exchange your API key for a JWT token.

> **Note:** This uses API Key Exchange, the simplest auth method. It returns both access and refresh tokens. For other auth options, see [Authentication](../endpoints/authentication.md).

```bash
# Replace with your actual API key
API_KEY="tm_yourteam_ak_yourkey"

# Get JWT token
TOKEN=$(curl -s -X POST https://api.penfield.app/api/v2/auth/token \
-H "Authorization: Bearer $API_KEY" \
-H "Content-Type: application/json" | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['access_token'])")

echo "Got token: ${TOKEN:0:20}..."
```

---

## Step 2: Store Your First Memory

```bash
curl -X POST https://api.penfield.app/api/v2/memories \
-H "Authorization: Bearer $TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "content": "Penfield is an AI memory layer that provides persistent memory across sessions",
  "memory_type": "fact",
  "importance": 0.8,
  "tags": ["penfield", "intro"]
}'
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "content": "Penfield is an AI memory layer...",
    "memory_type": "fact",
    "importance": 0.8,
    "tags": ["penfield", "intro"],
    "created_at": "2025-01-06T12:00:00Z"
  }
}
```

---

## Step 3: Search Your Memories

```bash
curl -X POST https://api.penfield.app/api/v2/search/hybrid \
-H "Authorization: Bearer $TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "query": "memory layer",
  "limit": 10
}'
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "items": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "content": "Penfield is an AI memory layer...",
        "score": 0.95,
        "memory_type": "fact"
      }
    ],
    "total": 1
  }
}
```

---

## Step 4: Connect Related Memories

Store another memory and connect them:

```bash
# Store a second memory
MEMORY2=$(curl -s -X POST https://api.penfield.app/api/v2/memories \
-H "Authorization: Bearer $TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "content": "Penfield uses hybrid search combining BM25, vector, and graph traversal",
  "memory_type": "fact",
  "importance": 0.7,
  "tags": ["penfield", "search"]
}' | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['id'])")

# Connect the memories
curl -X POST https://api.penfield.app/api/v2/relationships \
-H "Authorization: Bearer $TOKEN" \
-H "Content-Type: application/json" \
-d "{
  \"from_id\": \"$MEMORY2\",
  \"to_id\": \"550e8400-e29b-41d4-a716-446655440000\",
  \"relationship_type\": \"supports\",
  \"strength\": 0.8
}"
```

---

## What's Next?

Now that you have the basics:

1. **Explore Memory Types** - Learn about the [11 memory types](./memory-types.md) and when to use each
2. **Build Knowledge Graphs** - See all [24 relationship types](./relationship-types.md)
3. **Handle Errors** - Review [error handling patterns](./error-handling.md)
4. **Best Practices** - Follow our [best practices guide](./best-practices.md)

---

## Quick Reference

### Authentication
```bash
# Get token
curl -X POST https://api.penfield.app/api/v2/auth/token \
-H "Authorization: Bearer $API_KEY"

# Refresh token
curl -X POST https://api.penfield.app/api/v2/auth/refresh \
-d '{"refresh_token": "YOUR_REFRESH_TOKEN"}'
```

### Core Operations
```bash
# Create memory
POST /api/v2/memories

# Search
POST /api/v2/search/hybrid

# Get memory
GET /api/v2/memories/{id}

# Update memory
PUT /api/v2/memories/{id}

# Create relationship
POST /api/v2/relationships

# Traverse graph
POST /api/v2/relationships/traverse
```

### Token Lifetimes
- Access token: 3 days
- Refresh token: 30 days

Check `expires_in` in the response for exact values.

---

## Common Issues

### "Token expired"
Refresh your token or get a new one:
```bash
curl -X POST https://api.penfield.app/api/v2/auth/refresh \
-H "Content-Type: application/json" \
-d '{"refresh_token": "YOUR_REFRESH_TOKEN"}'
```

### "Validation failed"
Check your request body. Common issues:
- `content` must be 1-10,000 characters
- `importance` must be 0.0-1.0
- `memory_type` must be one of the valid types

### No search results
- Check spelling in your query
- Try broader search terms
- Verify memories were created successfully
