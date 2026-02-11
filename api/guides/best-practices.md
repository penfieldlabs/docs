# Best Practices Guide

How to get the most out of Penfield's memory system.

---

## Memory Storage

### Write Rich, Contextual Memories

**Good:**
```json
{
  "content": "[API Investigation] FACT: The /users endpoint returns 401 when the JWT token lacks the 'read:users' scope. Verified by comparing headers between working and failing requests. Error message: 'Insufficient scope'. Solution: Request 'read:users' scope during OAuth flow.",
  "memory_type": "fact",
  "importance": 0.8,
  "tags": ["api", "auth", "debugging"]
}
```

**Poor:**
```json
{
  "content": "API returns 401",
  "memory_type": "fact"
}
```

The rich memory includes:
- Context (what you were doing)
- Evidence (how you verified it)
- Solution (what to do about it)

### Use Appropriate Memory Types

| If you're storing... | Use type... |
|---------------------|-------------|
| Verified information | `fact` |
| Pattern or realization | `insight` |
| Bug fix or update | `correction` |
| Link to documentation | `reference` |
| Something to do later | `task` |
| Session handoff data | `checkpoint` |

See [Memory Types Guide](./memory-types.md) for all 11 types.

### Set Meaningful Importance

```
0.9-1.0: Critical facts, key decisions, major insights
0.7-0.8: Important findings, useful patterns
0.5-0.6: General notes, context
0.3-0.4: Minor details, background info
0.1-0.2: Ephemeral data, temporary notes
```

Higher importance = more likely to appear in search results.

### Use Tags Consistently

Create a tag taxonomy and stick to it:

```
# Project tags
project:penfield, project:webapp

# Type tags
bug, feature, research, meeting

# Status tags
resolved, pending, blocked

# Technology tags
python, api, database
```

---

## Knowledge Graph

### Always Connect Related Memories

After storing a memory, ask yourself:
1. Does this update or correct something? → `supersedes`, `updates`, `correction`
2. Does this provide evidence? → `supports`, `contradicts`
3. Is this an example of something? → `example_of`, `implements`
4. What came before/after? → `follows`, `precedes`

### Build Evidence Chains

```
Hypothesis → Evidence 1 (supports) → Evidence 2 (supports) → Conclusion (supersedes)
```

This creates traversable paths in your knowledge graph.

### Use Appropriate Relationship Strength

```
0.9-1.0: Direct, strong connection
0.7-0.8: Clear relationship
0.5-0.6: Related but indirect
0.3-0.4: Weak association
```

### Avoid Orphan Memories

Every memory should connect to at least one other memory. Orphans are harder to find through graph traversal.

---

## Search Optimization

### Write Searchable Content

Include keywords naturally:
```json
{
  "content": "The authentication flow uses OAuth 2.0 with PKCE for security. Tokens can be refreshed using the refresh_token grant type."
}
```

This memory will match searches for: "authentication", "OAuth", "PKCE", "tokens", "refresh".

### Use Source Type Filters

```json
{
  "query": "API documentation",
  "source_type": "document"  // Only search uploaded documents
}
```

```json
{
  "query": "debugging notes",
  "source_type": "memory"  // Only search memories, not documents
}
```

### Use Date Filters for Time-Sensitive Queries

```json
{
  "query": "project updates",
  "start_date": "2025-01-01",
  "end_date": "2025-01-31"
}
```

---

## Context Management

### Save Checkpoints at Milestones

Checkpoint names are **unique per tenant** — saving with a duplicate name will fail with an error. Use descriptive names that include context like dates or session identifiers (e.g., `"Auth Investigation - 2026-02-11"`, `"Sprint 4 Handoff"`).

Good times to save context:
- End of a debugging session
- Before switching tasks
- After making a key discovery
- When handing off to another agent

### Write Detailed Checkpoint Descriptions

```json
{
  "name": "Auth Bug Investigation - Jan 6",
  "description": "[Session Summary] Investigated 401 errors on /users endpoint. Root cause: missing scope in OAuth request. Key memory: 'scope discovery' (uuid-123). Verified fix works in staging. TODO: Deploy to production. Blocked on: need prod access."
}
```

Include:
- What you investigated
- What you found
- Key memories to reference
- What's next
- Any blockers

### Use Awakening for Personality

Always call `awaken` at the start of a conversation to load user preferences.

---

## API Usage

### Handle Tokens Properly

```python
# Store tokens securely
access_token = response["data"]["access_token"]
refresh_token = response["data"].get("refresh_token")  # May be None
expires_at = time.time() + response["data"]["expires_in"]

# Refresh proactively (before expiry)
if refresh_token and time.time() > expires_at - 300:  # 5 min buffer
    new_tokens = refresh_tokens(refresh_token)
    access_token = new_tokens["access_token"]
    refresh_token = new_tokens.get("refresh_token", refresh_token)
```

### Implement Retry Logic

```python
import time
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=1, max=60)
)
def api_call(endpoint, data):
    response = requests.post(endpoint, json=data, headers=headers)
    if response.status_code == 429:
        retry_after = int(response.headers.get("Retry-After", 60))
        time.sleep(retry_after)
        raise Exception("Rate limited")
    response.raise_for_status()
    return response.json()
```

### Batch Operations When Possible

Instead of:
```python
for item in items:
    store_memory(item)  # 100 API calls
```

Consider:
```python
# Group related items
batch_content = "\n\n".join(items)
store_memory(batch_content)  # 1 API call
```

---

## Security

### Never Log Tokens

```python
# BAD
logger.info(f"Using token: {access_token}")

# GOOD
logger.info(f"Using token: {access_token[:10]}...")
```

### Validate Inputs

```python
# Validate content length
if len(content) > 10000:
    raise ValueError("Content too long")

# Validate importance
if not 0 <= importance <= 1:
    raise ValueError("Importance must be 0-1")

# Validate memory type
VALID_TYPES = ["fact", "insight", "conversation", ...]
if memory_type not in VALID_TYPES:
    raise ValueError(f"Invalid memory type: {memory_type}")
```

### Use Appropriate Scopes

Request only the scopes you need:
- `read` - Read memories, search
- `write` - Create, update, delete memories
- `profile` - Access user profile information
- `offline_access` - Get refresh tokens (OAuth flows)

**Discover available scopes** from `.well-known/oauth-authorization-server`.

---

## Performance

### Limit Search Results

```json
{
  "query": "debugging",
  "limit": 20  // Don't fetch more than needed
}
```

### Use Graph Depth Wisely

```json
{
  "start_memory": "uuid",
  "max_depth": 2  // Deep traversals are expensive
}
```

### Cache Frequently Accessed Data

```python
from functools import lru_cache

@lru_cache(maxsize=100)
def get_memory(memory_id):
    return api.get_memory(memory_id)
```

---

## Anti-Patterns

### Don't Store Ephemeral Data

Avoid storing:
- Temporary calculations
- Intermediate processing results
- Data that changes every second

### Don't Create Circular Relationships

```
A → supports → B → supports → A  // Circular!
```

### Don't Ignore Errors

```python
# BAD
try:
    store_memory(data)
except:
    pass

# GOOD
try:
    store_memory(data)
except ValidationError as e:
    logger.error(f"Invalid memory data: {e}")
    # Handle appropriately
except RateLimitError:
    time.sleep(60)
    store_memory(data)  # Retry
```

### Don't Hardcode IDs

```python
# BAD
IMPORTANT_MEMORY = "550e8400-e29b-41d4-a716-446655440000"

# GOOD
result = search("important finding about X")
important_memory = result["items"][0]["id"]
```
