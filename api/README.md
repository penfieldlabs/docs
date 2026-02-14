# Penfield API Documentation

**Version:** 2.0.0
**Base URL:** `https://api.penfield.app/api/v2`

Penfield is a persistent memory and knowledge graph system for AI assistants.

**New to Penfield?** Start with the [Quick Start Guide](guides/quick-start.md).

---

## How Penfield Works

**Memories** are individual pieces of knowledge—facts, insights, decisions, or context from conversations. Think of them as thoughts that persist across sessions.

**Relationships** connect memories into a knowledge graph. When you link "Python uses asyncio" to "asyncio enables concurrent I/O", you're building a web of understanding that search can traverse.

**Documents** you upload get chunked and embedded for search. Each chunk becomes searchable alongside your memories.

**Artifacts** are files you create and store—notes, diagrams, code snippets. Unlike memories, they're organized by path (`/project/notes.md`) rather than searched semantically.

**Hybrid Search** combines three methods: BM25 (keyword matching), vector similarity (semantic meaning), and graph expansion (following relationships). This finds relevant memories even when your query doesn't match exact words.

---

## Documentation Index

### Getting Started
- [Quick Start](guides/quick-start.md) - First memory in 5 minutes
- [Authentication](endpoints/authentication.md) - OAuth flows, API keys, tokens

### Guides
- [Best Practices](guides/best-practices.md) - Effective usage patterns
- [Common Patterns](guides/common-patterns.md) - Session handoffs, investigations, multi-agent
- [Memory Types](guides/memory-types.md) - The 11 memory types explained
- [Relationship Types](guides/relationship-types.md) - The 24 relationship types
- [Error Handling](guides/error-handling.md) - Error codes and recovery
- [Troubleshooting](guides/troubleshooting.md) - Solutions to common issues

### API Reference
- [Memories](endpoints/memories.md) - Store and retrieve memories
- [Search](endpoints/search.md) - Hybrid search (BM25 + vector + graph)
- [Relationships](endpoints/relationships.md) - Connect memories
- [Documents](endpoints/documents.md) - Upload and process documents
- [Artifacts](endpoints/artifacts.md) - File storage
- [Analysis](endpoints/analysis.md) - Memory analysis tools
- [Tags](endpoints/tags.md) - Tag management

### Integration
- [MCP Integration](../mcp/mcp-integration.md) - Model Context Protocol tools
- [Python Examples](examples/python.md) - Complete Python client
- [cURL Examples](examples/curl.md) - Command-line examples

---

## Authentication

Penfield uses OAuth 2.1 with dynamic endpoint discovery.

### Step 1: Discover Endpoints

**Always discover endpoints from `.well-known`:**

```bash
DISCOVERY=$(curl -s https://api.penfield.app/.well-known/oauth-authorization-server)
```

### Step 2: Choose Your Auth Method

| Use Case | Method |
|----------|--------|
| Quick start / testing | API Key Exchange |
| Server-to-server | API Key Exchange |
| CLI tools / headless | Device Code Flow |
| Desktop/web apps | Authorization Code + PKCE |

See [Authentication](endpoints/authentication.md) for complete details.

### API Key Exchange (Simplest)

```bash
curl -X POST https://api.penfield.app/api/v2/auth/token \
-H "Authorization: Bearer YOUR_API_KEY" \
-H "Content-Type: application/json"
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "token_type": "Bearer",
    "expires_in": 259200,
    "refresh_token": "eyJhbGciOiJIUzI1NiIs..."
  }
}
```

### Use Token

```bash
curl -X GET https://api.penfield.app/api/v2/memories \
-H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

---

## Quick Start

### 1. Store a Memory

```bash
curl -X POST https://api.penfield.app/api/v2/memories \
-H "Authorization: Bearer $TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "content": "Python async programming uses asyncio for concurrent operations",
  "memory_type": "fact",
  "importance": 0.8,
  "tags": ["python", "async", "programming"]
}'
```

### 2. Search Memories

```bash
curl -X POST https://api.penfield.app/api/v2/search/hybrid \
-H "Authorization: Bearer $TOKEN" \
-H "Content-Type: application/json" \
-d '{"query": "Python async programming", "limit": 10}'
```

### 3. Connect Related Memories

```bash
curl -X POST https://api.penfield.app/api/v2/relationships \
-H "Authorization: Bearer $TOKEN" \
-H "Content-Type: application/json" \
-d '{"from_id": "uuid-1", "to_id": "uuid-2", "relationship_type": "supports", "strength": 0.8}'
```

---

## Core Concepts

### Memories

The fundamental unit of storage:

- **content**: Text (max 10,000 characters)
- **memory_type**: Classification (fact, insight, conversation, etc.)
- **importance**: Score 0-1
- **confidence**: Score 0-1
- **tags**: Up to 10 tags

### Memory Types

| Type | Description |
|------|-------------|
| `fact` | Verified information |
| `insight` | Derived understanding |
| `conversation` | Conversational context |
| `correction` | Update to previous information |
| `reference` | External reference |
| `task` | Action item |
| `checkpoint` | State snapshot |
| `identity_core` | Core AI identity |
| `personality_trait` | AI personality |
| `relationship` | User-AI relationship |
| `strategy` | Learned behavior |

### Relationships

24 types across categories:

| Category | Types |
|----------|-------|
| Knowledge Evolution | `supersedes`, `updates`, `evolution_of` |
| Evidence & Support | `supports`, `contradicts`, `disputes` |
| Hierarchy | `parent_of`, `child_of`, `sibling_of`, `composed_of`, `part_of` |
| Causation | `causes`, `influenced_by`, `prerequisite_for` |
| Implementation | `implements`, `documents`, `tests`, `example_of` |
| Conversation | `responds_to`, `references`, `inspired_by` |
| Sequence | `follows`, `precedes` |
| Dependencies | `depends_on` |

### Hybrid Search

Combines via Reciprocal Rank Fusion (RRF):
- BM25 text search (keywords)
- Vector similarity (semantic)
- Graph expansion (relationships)

Default weights: BM25=0.4, Vector=0.4, Graph=0.2

---

## Rate Limits

Rate limits may be enforced.

**Headers:**

| Header | Description |
|--------|-------------|
| `x-ratelimit-limit` | Max requests per window |
| `x-ratelimit-remaining` | Requests remaining |
| `x-ratelimit-reset` | Unix timestamp when window resets |
| `retry-after` | Seconds to wait (on 429) |

---

## Error Handling

```json
{
  "status": "error",
  "error": {
    "code": "RES_NOT_FOUND",
    "message": "Memory not found"
  }
}
```

| Code | Status | Description |
|------|--------|-------------|
| `AUTH_INVALID_CREDENTIALS` | 401 | Invalid credentials |
| `AUTH_TOKEN_EXPIRED` | 401 | Token expired |
| `AUTH_INSUFFICIENT_PERMISSIONS` | 403 | Missing required scope |
| `VAL_VALIDATION_FAILED` | 422 | Validation failed |
| `RES_NOT_FOUND` | 404 | Resource not found |
| `BIZ_RATE_LIMIT_EXCEEDED` | 429 | Rate limited |

---

## Environments

| Environment | API | Auth | Portal |
|-------------|-----|------|--------|
| Production | `api.penfield.app` | `auth.penfield.app` | `portal.penfield.app` |
| Development | `api-dev.penfield.app` | `auth-dev.penfield.app` | `portal-dev.penfield.app` |

**Always use discovery.** The `.well-known/oauth-authorization-server` endpoint returns the correct auth server URLs.

---

## Support

- **API Issues**: support@penfield.app
- **Documentation**: https://github.com/penfieldlabs/docs

---

© 2026 Penfield™. All Rights Reserved.
