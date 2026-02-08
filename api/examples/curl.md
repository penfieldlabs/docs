# cURL Examples

Complete cURL examples for the Penfield API.

---

## Authentication

### Get JWT Token

```bash
curl -X POST https://api.penfield.app/api/v2/auth/token \
-H "Authorization: Bearer tm_your_tenant_ak_your_key" \
-H "Content-Type: application/json"
```

### Set Token Variable

```bash
# Get token and save to variable (using python3 for JSON parsing)
export JWT_TOKEN=$(curl -s -X POST https://api.penfield.app/api/v2/auth/token \
-H "Authorization: Bearer tm_your_tenant_ak_your_key" \
-H "Content-Type: application/json" | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['access_token'])")

echo "Token: ${JWT_TOKEN:0:20}..."

# Alternative with jq (if installed)
# export JWT_TOKEN=$(curl -s ... | jq -r '.data.access_token')
```

### Verify Token

```bash
curl -X GET https://api.penfield.app/api/v2/auth/verify \
-H "Authorization: Bearer $JWT_TOKEN"
```

---

## Memories

### Create Memory

```bash
curl -X POST https://api.penfield.app/api/v2/memories \
-H "Authorization: Bearer $JWT_TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "content": "Python asyncio provides async/await syntax for concurrent programming",
  "memory_type": "fact",
  "importance": 0.8,
  "tags": ["python", "async", "concurrency"]
}'
```

### Get Memory

```bash
curl -X GET https://api.penfield.app/api/v2/memories/550e8400-e29b-41d4-a716-446655440000 \
-H "Authorization: Bearer $JWT_TOKEN"
```

### List Memories

```bash
# Basic list
curl -X GET "https://api.penfield.app/api/v2/memories" \
-H "Authorization: Bearer $JWT_TOKEN"

# With filters
curl -X GET "https://api.penfield.app/api/v2/memories?memory_type=fact&importance_threshold=0.7&tags=python&sort=-importance" \
-H "Authorization: Bearer $JWT_TOKEN"

# Pagination
curl -X GET "https://api.penfield.app/api/v2/memories?page=2&per_page=50" \
-H "Authorization: Bearer $JWT_TOKEN"
```

### Update Memory

```bash
curl -X PUT https://api.penfield.app/api/v2/memories/550e8400-e29b-41d4-a716-446655440000 \
-H "Authorization: Bearer $JWT_TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "importance": 0.9,
  "metadata": {"verified": true}
}'
```

### Delete Memory

```bash
curl -X DELETE https://api.penfield.app/api/v2/memories/550e8400-e29b-41d4-a716-446655440000 \
-H "Authorization: Bearer $JWT_TOKEN"
```

---

## Search

### Hybrid Search

```bash
curl -X POST https://api.penfield.app/api/v2/search/hybrid \
-H "Authorization: Bearer $JWT_TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "query": "Python async programming patterns",
  "limit": 10,
  "bm25_weight": 0.4,
  "vector_weight": 0.4,
  "graph_weight": 0.2
}'
```

### Search with Filters

```bash
curl -X POST https://api.penfield.app/api/v2/search/hybrid \
-H "Authorization: Bearer $JWT_TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "query": "machine learning",
  "limit": 20,
  "memory_types": ["fact", "insight"],
  "importance_threshold": 0.6,
  "enable_graph_expansion": true,
  "graph_max_depth": 2
}'
```

### Search Stats

```bash
curl -X GET https://api.penfield.app/api/v2/search/stats \
-H "Authorization: Bearer $JWT_TOKEN"
```

---

## Relationships

### Create Relationship

```bash
curl -X POST https://api.penfield.app/api/v2/relationships \
-H "Authorization: Bearer $JWT_TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "from_id": "550e8400-e29b-41d4-a716-446655440000",
  "to_id": "660f9500-f30c-52e5-b827-557766551111",
  "relationship_type": "supports",
  "strength": 0.8
}'
```

### List Relationships

```bash
# All relationships for a memory
curl -X GET "https://api.penfield.app/api/v2/relationships?memory_id=550e8400-e29b-41d4-a716-446655440000" \
-H "Authorization: Bearer $JWT_TOKEN"

# Filter by type
curl -X GET "https://api.penfield.app/api/v2/relationships?relationship_type=supports" \
-H "Authorization: Bearer $JWT_TOKEN"
```

### Traverse Graph

```bash
curl -X POST https://api.penfield.app/api/v2/relationships/traverse \
-H "Authorization: Bearer $JWT_TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "start_memory_id": "550e8400-e29b-41d4-a716-446655440000",
  "max_depth": 3,
  "direction": "OUTBOUND"
}'
```

### Delete Relationship

```bash
# By relationship ID
curl -X DELETE https://api.penfield.app/api/v2/relationships/550e8400-e29b-41d4-a716-446655440000 \
-H "Authorization: Bearer $JWT_TOKEN"

# Between two memories
curl -X DELETE "https://api.penfield.app/api/v2/relationships/between?from_id=uuid1&to_id=uuid2" \
-H "Authorization: Bearer $JWT_TOKEN"
```

---

## Documents

### Upload Document

```bash
curl -X POST https://api.penfield.app/api/v2/documents/upload \
-H "Authorization: Bearer $JWT_TOKEN" \
-F "file=@/path/to/document.pdf" \
-F 'metadata={"project": "research"}'
```

### List Documents

```bash
curl -X GET "https://api.penfield.app/api/v2/documents" \
-H "Authorization: Bearer $JWT_TOKEN"
```

### Get Document

```bash
# Metadata only
curl -X GET https://api.penfield.app/api/v2/documents/550e8400-e29b-41d4-a716-446655440000 \
-H "Authorization: Bearer $JWT_TOKEN"

# With chunks
curl -X GET "https://api.penfield.app/api/v2/documents/550e8400-e29b-41d4-a716-446655440000?include_chunks=true" \
-H "Authorization: Bearer $JWT_TOKEN"
```

### Download Document

```bash
curl -X GET https://api.penfield.app/api/v2/documents/550e8400-e29b-41d4-a716-446655440000/download \
-H "Authorization: Bearer $JWT_TOKEN" \
-o downloaded-file.pdf
```

### Delete Document

```bash
curl -X DELETE https://api.penfield.app/api/v2/documents/550e8400-e29b-41d4-a716-446655440000 \
-H "Authorization: Bearer $JWT_TOKEN"
```

---

## Artifacts

### Save Artifact

```bash
curl -X POST https://api.penfield.app/api/v2/artifacts \
-H "Authorization: Bearer $JWT_TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "path": "/notes/python-tips.md",
  "content": "# Python Tips\n\n- Use list comprehensions\n- Prefer generators"
}'
```

### Get Artifact

```bash
curl -X GET "https://api.penfield.app/api/v2/artifacts?path=/notes/python-tips.md" \
-H "Authorization: Bearer $JWT_TOKEN"
```

### List Artifacts

```bash
# Root directory
curl -X GET "https://api.penfield.app/api/v2/artifacts/list" \
-H "Authorization: Bearer $JWT_TOKEN"

# Specific directory
curl -X GET "https://api.penfield.app/api/v2/artifacts/list?prefix=/notes" \
-H "Authorization: Bearer $JWT_TOKEN"
```

### Delete Artifact

```bash
curl -X DELETE "https://api.penfield.app/api/v2/artifacts?path=/notes/python-tips.md" \
-H "Authorization: Bearer $JWT_TOKEN"
```

---

## Tags

### Add Tags

```bash
curl -X POST https://api.penfield.app/api/v2/memories/550e8400-e29b-41d4-a716-446655440000/tags \
-H "Authorization: Bearer $JWT_TOKEN" \
-H "Content-Type: application/json" \
-d '{"tags": ["python", "important"]}'
```

### Remove Tags

```bash
curl -X DELETE https://api.penfield.app/api/v2/memories/550e8400-e29b-41d4-a716-446655440000/tags \
-H "Authorization: Bearer $JWT_TOKEN" \
-H "Content-Type: application/json" \
-d '{"tags": ["deprecated"]}'
```

### List All Tags

```bash
curl -X GET https://api.penfield.app/api/v2/tags \
-H "Authorization: Bearer $JWT_TOKEN"
```

---

## Analysis

### Reflect

```bash
curl -X POST https://api.penfield.app/api/v2/analysis/reflect \
-H "Authorization: Bearer $JWT_TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "time_window": "week",
  "include_documents": false
}'
```

---

## Utility Scripts

### Complete Workflow Script

```bash
#!/bin/bash
# penfield-demo.sh - Complete API workflow

API_KEY="tm_your_tenant_ak_your_key"
BASE_URL="https://api.penfield.app"

# Helper function to parse JSON (works without jq)
json_get() {
  python3 -c "import sys,json; print(json.load(sys.stdin)$1)"
}

# Get token
echo "Authenticating..."
RESPONSE=$(curl -s -X POST "$BASE_URL/api/v2/auth/token" \
-H "Authorization: Bearer $API_KEY" \
-H "Content-Type: application/json")
TOKEN=$(echo "$RESPONSE" | json_get "['data']['access_token']")

if [ -z "$TOKEN" ] || [ "$TOKEN" == "None" ]; then
  echo "Authentication failed"
  echo "$RESPONSE"
  exit 1
fi
echo "Authenticated successfully"

# Create memory
echo "Creating memory..."
MEMORY=$(curl -s -X POST "$BASE_URL/api/v2/memories" \
-H "Authorization: Bearer $TOKEN" \
-H "Content-Type: application/json" \
-d '{"content": "Test memory from script", "memory_type": "fact"}')
MEMORY_ID=$(echo "$MEMORY" | json_get "['data']['id']")
echo "Created memory: $MEMORY_ID"

# Search
echo "Searching..."
RESULTS=$(curl -s -X POST "$BASE_URL/api/v2/search/hybrid" \
-H "Authorization: Bearer $TOKEN" \
-H "Content-Type: application/json" \
-d '{"query": "test", "limit": 5}')
COUNT=$(echo "$RESULTS" | json_get "len(['data']['items'])")
echo "Found $COUNT results"

# Cleanup
echo "Cleaning up..."
curl -s -X DELETE "$BASE_URL/api/v2/memories/$MEMORY_ID" \
-H "Authorization: Bearer $TOKEN"
echo "Done"
```

### Token Refresh Script

```bash
#!/bin/bash
# refresh-token.sh - Refresh expired token
# Note: Requires a valid refresh_token from OAuth flow with offline_access scope

REFRESH_TOKEN="your_refresh_token"
BASE_URL="https://api.penfield.app"

RESPONSE=$(curl -s -X POST "$BASE_URL/api/v2/auth/refresh" \
-H "Content-Type: application/json" \
-d "{\"refresh_token\": \"$REFRESH_TOKEN\"}")

NEW_TOKEN=$(echo "$RESPONSE" | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['access_token'])")
NEW_REFRESH=$(echo "$RESPONSE" | python3 -c "import sys,json; print(json.load(sys.stdin)['data'].get('refresh_token', ''))")

echo "New access token: ${NEW_TOKEN:0:20}..."
export JWT_TOKEN=$NEW_TOKEN

# Important: Save the new refresh token (tokens rotate per RFC 9700)
if [ -n "$NEW_REFRESH" ]; then
  echo "New refresh token received (old one is now invalid)"
  export REFRESH_TOKEN=$NEW_REFRESH
fi
```
