# Troubleshooting Guide

Solutions to common issues when using the Penfield API.

---

## Authentication Issues

### "Invalid API key" or "Invalid token" (401)

**Symptom:**
```json
{
  "status": "error",
  "error": {
    "code": "AUTH_UNAUTHORIZED",
    "message": "Invalid API key"
  }
}
```

The message tells the cases apart:

| Message | Cause |
|---------|-------|
| `Invalid API key` | The key sent to `/auth/token` has the wrong format or was revoked |
| `Invalid token` | The bearer token is not a valid access token. You get this when an API key is sent to an endpoint that expects the JWT |
| `No access token provided (expected Authorization header or cookie)` | The `Authorization` header is missing |

**Solutions:**
```bash
# Check API key format (should be tm_xxx_ak_xxx)
echo $API_KEY | grep -E '^tm_.*_ak_.*$'

# Get a fresh JWT token
curl -X POST https://api.penfield.app/api/v2/auth/token \
-H "Authorization: Bearer $API_KEY" \
-H "Content-Type: application/json"
```

### Token expired (401)

**Symptom:** a token that worked before now gets a 401. Access tokens last 24 hours.

**Solution:**
```bash
# Refresh using refresh token
curl -X POST https://api.penfield.app/api/v2/auth/refresh \
-H "Content-Type: application/json" \
-d '{"refresh_token": "YOUR_REFRESH_TOKEN"}'
```

**Prevention:**
- Store token expiry time: `expires_at = time.time() + expires_in`
- Refresh proactively when `time.time() > expires_at - 300`

### "Insufficient permissions" (403)

**Symptom:**
```json
{
  "status": "error",
  "error": {
    "code": "AUTH_INSUFFICIENT_PERMISSIONS",
    "message": "Insufficient permissions. Required scope(s): memories:write, write, admin"
  }
}
```

The message lists the scopes that would allow the call.

**Solution:**
Use a token (or an API key) that has the required scope:
- `read` - For reading/searching
- `write` - For creating/updating
- `delete` - For deleting memories, relationships, documents and artifacts
- `profile` - For profile access
- `offline_access` - For refresh tokens

Check available scopes via discovery:
```bash
curl -s https://api.penfield.app/.well-known/oauth-authorization-server | \
  python3 -c "import sys,json; print(json.load(sys.stdin)['scopes_supported'])"
```

---

## Validation Errors

### "Request validation failed" (422)

**Symptom:**
```json
{
  "status": "error",
  "error": {
    "code": "VAL_VALIDATION_FAILED",
    "message": "Request validation failed",
    "details": null,
    "field_errors": {
      "content": ["Value error, Memory content is 10,001 characters, exceeds the 10,000 character limit. ..."]
    }
  }
}
```

`field_errors` names each field that failed and why.

**Common causes and limits:**

| Field | Limit | Solution |
|-------|-------|----------|
| `content` | 10,000 chars | Truncate, split into multiple memories, upload as a document, or store as an artifact with a summary memory linking to it |
| `importance` | 0.0-1.0 | Ensure value is in range |
| `tags` | 10 max | Use fewer, broader tags |
| `memory_type` | 11 valid types | Check [Memory Types Guide](./memory-types.md) |

### Invalid UUID (422)

**Symptom:**
```json
{
  "error": {
    "code": "VAL_VALIDATION_FAILED",
    "message": "Request validation failed",
    "field_errors": {
      "memory_id": ["Input should be a valid UUID, ..."]
    }
  }
}
```

**Solution:**
UUIDs must be in standard format: `550e8400-e29b-41d4-a716-446655440000`

```python
import uuid

# Validate before sending
try:
    uuid.UUID(memory_id)
except ValueError:
    raise ValueError(f"Invalid UUID: {memory_id}")
```

### Invalid relationship type (422)

**Symptom:** `VAL_VALIDATION_FAILED` with `field_errors.relationship_type`.

**Solution:**
Use one of the 24 valid relationship types:
```
supersedes, updates, evolution_of, supports, contradicts,
disputes, parent_of, child_of, sibling_of, causes,
influenced_by, prerequisite_for, implements, documents,
example_of, tests, responds_to, references, inspired_by,
follows, precedes, depends_on, composed_of, part_of
```

See [Relationship Types Guide](./relationship-types.md).

---

## Search Issues

### No Results Returned

**Possible causes:**

1. **Query too specific**
   - Try broader search terms
   - Remove filters temporarily

2. **Memories not indexed yet**
   - Wait a few seconds after creating
   - Embeddings are generated asynchronously

3. **Wrong source filter**
   ```json
   // MCP recall: this only searches documents, not memories
   {"query": "my note", "source_type": "document"}

   // REST search: the same filter
   {"query": "my note", "source_types": ["document_upload"]}

   // Search all
   {"query": "my note"}
   ```

4. **Date filter too narrow**
   ```json
   // Make sure your memory was created in this range
   {"query": "test", "start_date": "2026-01-01", "end_date": "2026-01-07"}
   ```

### Low Relevance Scores

**Symptom:** Results return but with low scores (< 0.1)

**Solutions:**
1. Include more keywords in your memories
2. Use more descriptive content
3. Check if your query matches the language in your memories

### Results Not in Expected Order

The hybrid search combines:
- BM25 (keyword matching)
- Vector (semantic similarity)
- Graph (relationship traversal)

If important results appear lower:
1. Include relevant keywords in their content
2. Add more relationships to them
3. Adjust the search weights (`bm25_weight`, `vector_weight`, `graph_weight`) toward the method that should count most

The `importance` score does not change the ranking. To leave out low-importance memories, filter with `importance_threshold`.

---

## Rate Limiting

### "Rate limit exceeded" (429)

**Symptom:**
```json
{
  "status": "error",
  "error": {
    "code": "BIZ_RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded"
  }
}
```

**Response headers** (the values are examples; the limit depends on the endpoint):
```
x-ratelimit-limit: 35
x-ratelimit-remaining: 0
x-ratelimit-reset: 1704542400
retry-after: 3
```

**Solutions:**

1. **Wait and retry:**
```python
import time

if response.status_code == 429:
    retry_after = int(response.headers.get("Retry-After", 60))
    time.sleep(retry_after)
    # Retry request
```

2. **Implement exponential backoff:**
```python
for attempt in range(5):
    response = make_request()
    if response.status_code != 429:
        break
    time.sleep(2 ** attempt)
```

3. **Batch operations** to reduce request count

4. **Upgrade tier** for higher limits

---

## Connection/Network Issues

### Timeouts

**Symptom:** Request hangs or times out

**Solutions:**

1. **Set reasonable timeouts:**
```python
response = requests.post(url, json=data, timeout=30)
```

2. **Use retry logic:**
```python
from tenacity import retry, stop_after_attempt

@retry(stop=stop_after_attempt(3))
def api_call():
    return requests.post(url, json=data, timeout=30)
```

### SSL Certificate Errors

**Symptom:** `SSL: CERTIFICATE_VERIFY_FAILED`

**Solutions:**
1. Update your CA certificates
2. Check system time is correct
3. Don't disable SSL verification in production

---

## Memory/Relationship Issues

### "Memory not found" (404)

**Causes:**
1. Memory was deleted
2. Wrong UUID
3. Memory belongs to different tenant

**Solution:**
```bash
# Verify memory exists
curl -X GET "https://api.penfield.app/api/v2/memories/$MEMORY_ID" \
-H "Authorization: Bearer $TOKEN"
```

### "Relationship already exists" (400)

**Symptom:** `VAL_VALIDATION_FAILED`, message `Relationship validation failed`, with the reason in `details.errors`:

```json
"details": {
  "errors": ["Relationship supports already exists between these memories"]
}
```

**Cause:** Trying to create a duplicate relationship

**Solutions:**
1. Check if relationship exists first
2. Use a different relationship type
3. Update existing relationship instead

### Circular Relationship Warning

Creating A → B → C → A can cause infinite traversal loops.

**Prevention:**
- Design relationships as directed acyclic graphs when possible
- Set `max_depth` limits on traversal operations

---

## Artifact Issues

### "Invalid path" (400)

**Path rules:**
- Must start with `/`
- Must include filename (not just directory)
- No `..` (path traversal)
- No hidden files starting with `.`
- Max depth: 10 folders (the file name does not count)
- Valid chars: alphanumeric, `-`, `_`, `.`, `/`

**Examples:**
```
✓ /project/notes.md
✓ /docs/api/v2/guide.txt
✗ ../secret/file.txt
✗ /project/
✗ /.hidden/file.md
```

### "Artifact already exists" (409)

Saving does not overwrite. There is no call that updates an artifact in place.

**Solutions:**
1. Delete the existing artifact, then save the new content at the same path
2. Save under a different path

---

## MCP Integration Issues

### Tool Not Working

**Checklist:**
1. MCP server URL correct? It is `https://mcp.penfield.app`
2. Is the connector signed in? The MCP server uses an OAuth sign-in, not an API key. If tools stopped working, [reconnect](../../onboarding/reconnect-penfield.md)
3. Does the tool appear in your client's tool list? See the [tool reference](../../mcp/mcp-integration.md)
4. Do the arguments match the tool's parameters?

### Personality Not Loading (awaken)

**Solution:**
1. Check if personality is configured in Portal
2. Call `awaken` at conversation start
3. Or use `restore_context("awakening")`

---

## Debugging Tips

### Enable Request Logging

```python
import logging
import requests

logging.basicConfig(level=logging.DEBUG)
requests_log = logging.getLogger("requests.packages.urllib3")
requests_log.setLevel(logging.DEBUG)
```

### Check Request ID

Every response includes a `request_id` in the `meta` section:

```json
{
  "meta": {
    "request_id": "req_abc123"
  }
}
```

Include this when contacting support.

### Verify Token Contents

```bash
# Decode JWT (payload section)
echo $TOKEN | cut -d. -f2 | base64 -d 2>/dev/null | jq
```

Check:
- `exp` - Expiration time
- `scopes` - Granted scopes (a list)
- `tenant_id` - Correct tenant

---

## Getting Help

If you're still stuck:

1. **Check the error code** - See [Error Handling Guide](./error-handling.md)
2. **Review the API docs** - Verify request format
3. **Check status page** - For service issues
4. **Contact support** - Include `request_id` and error details
