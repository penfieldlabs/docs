# MCP Integration Guide

Penfield provides a Model Context Protocol (MCP) server for direct integration with AI assistants like Claude Desktop, Cursor, and other MCP-compatible clients.

---

## Overview

The MCP server exposes Penfield's memory and knowledge graph capabilities as tools that AI assistants can use during conversations. This enables persistent memory across sessions without requiring direct API integration.

---

## Available Tools

### Initialization Tools

| Tool | Description | API Equivalent |
|------|-------------|----------------|
| `awaken` | Load personality configuration | `GET /api/v2/personality/awakening` |

### Memory Tools

| Tool | Description | API Equivalent |
|------|-------------|----------------|
| `store` | Store a new memory | `POST /api/v2/memories` |
| `recall` | Hybrid search (BM25 + vector + graph) for context retrieval | `POST /api/v2/search/hybrid` |
| `fetch` | Get a specific memory | `GET /api/v2/memories/{id}` |
| `search` | Semantic search for fuzzy concept matching | `POST /api/v2/search/hybrid` |
| `update_memory` | Update an existing memory | `PUT /api/v2/memories/{id}` |

### Knowledge Graph Tools

| Tool | Description | API Equivalent |
|------|-------------|----------------|
| `connect` | Create relationship between memories | `POST /api/v2/relationships` |
| `explore` | Traverse the knowledge graph | `POST /api/v2/relationships/traverse` |

> **Note:** `disconnect` (relationship removal) is registered as an MCP tool but returns `403 Forbidden` for standard MCP users. It requires elevated permissions (`RELATIONSHIPS_DELETE`, `DELETE`, or `ADMIN` scope). Use the REST API with `write` scope or Penfield Portal instead.

### Context Management Tools

| Tool | Description | API Equivalent |
|------|-------------|----------------|
| `save_context` | Create a checkpoint | `POST /api/v2/memories` (type: checkpoint) |
| `restore_context` | Recall a checkpoint | `GET /api/v2/memories` + fetch |
| `list_contexts` | List saved checkpoints | `GET /api/v2/memories?memory_type=checkpoint` |
| `reflect` | Analyze recent memories | `POST /api/v2/analysis/reflect` |

### Storage Tools

| Tool | Description | API Equivalent |
|------|-------------|----------------|
| `save_artifact` | Store a file | `POST /api/v2/artifacts` |
| `retrieve_artifact` | Get a stored file | `GET /api/v2/artifacts` |
| `list_artifacts` | List stored files | `GET /api/v2/artifacts/list` |
| `delete_artifact` | Remove a stored file | `DELETE /api/v2/artifacts` |

---

## Tool Parameters

> **Note:** MCP tool parameters use different names than the REST API. The MCP server maps them internally:
> - MCP `from_memory` / `to_memory` → REST API `from_id` / `to_id`
> - MCP `start_memory` → REST API `start_memory_id`
> - MCP `memory_id` → REST API `id` (in path)
>
> This documentation shows MCP parameter names. For direct REST API usage, see the [endpoint documentation](../api/endpoints/).

### awaken

Load the user's personality briefing at the start of a conversation.

**Parameters:** None

```json
{}
```

**Response:**
```json
{
  "success": true,
  "briefing": "=== AWAKENING BRIEFING ===\n..."
}
```

---

### store

Store a new memory with automatic type detection.

```json
{
  "content": "Python async uses asyncio for concurrent operations",
  "tags": ["python", "async"],
  "importance": 0.8
}
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `content` | string | Yes | Memory content (1-10,000 chars) |
| `tags` | array | No | Tags for categorization |
| `importance` | number | No | 0.0-1.0 (auto-calculated if omitted) |

**Note:** Memory type is auto-detected from content. Valid types: fact, insight, conversation, correction, reference, task, checkpoint, relationship, strategy.

**Protected types:** `identity_core` and `personality_trait` are managed exclusively through the `/api/v2/personality` endpoints and cannot be created via `store`. The auto-detection will not produce these types.

---

### recall

Hybrid search (BM25 + vector + graph). Use when you need context before responding, resuming a topic, or looking up prior decisions.

```json
{
  "query": "async programming patterns",
  "limit": 20,
  "source_type": "memory",
  "tags": ["python"],
  "start_date": "2025-01-01",
  "end_date": "2025-01-31"
}
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | Yes | Search query |
| `limit` | number | No | Max results (default: 10, max: 100) |
| `source_type` | string | No | Filter: "memory", "document", or null for all |
| `tags` | array | No | Filter by tags (OR logic) |
| `start_date` | string | No | Filter by date (ISO 8601) |
| `end_date` | string | No | Filter by date (ISO 8601) |

---

### search

Semantic search (higher vector weight). Use for fuzzy concept search when you don't have exact terms. Returns ChatGPT-compatible citation format.

```json
{
  "query": "async programming",
  "limit": 10
}
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | Yes | Search query |
| `limit` | number | No | Max results (default: 10, max: 100) |

**Response includes citation fields:** `id`, `title`, `url`, `text`

---

### fetch

Get a specific memory by ID with citation support.

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000"
}
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | UUID | Yes | Memory ID to fetch |

---

### update_memory

Update an existing memory.

```json
{
  "memory_id": "550e8400-e29b-41d4-a716-446655440000",
  "content": "Updated content...",
  "importance": 0.9,
  "tags": ["updated", "important"]
}
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `memory_id` | UUID | Yes | Memory ID to update |
| `content` | string | No | New content |
| `importance` | number | No | New importance (0-1) |
| `tags` | array | No | New tags (replaces existing) |

---

### connect

Create a relationship between two memories.

```json
{
  "from_memory": "uuid-1",
  "to_memory": "uuid-2",
  "relationship_type": "supports",
  "strength": 0.8
}
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `from_memory` | UUID | Yes | Source memory ID |
| `to_memory` | UUID | Yes | Target memory ID |
| `relationship_type` | enum | Yes | One of 24 relationship types |
| `strength` | number | No | 0.0-1.0 (default: 0.5) |

> **MCP ↔ REST Parameter Mapping:**
> - MCP `from_memory` → REST API `from_id`
> - MCP `to_memory` → REST API `to_id`

---

### explore

Traverse the knowledge graph from a starting memory.

```json
{
  "start_memory": "550e8400-e29b-41d4-a716-446655440000",
  "max_depth": 3,
  "relationship_types": ["supports", "references"]
}
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `start_memory` | UUID | Yes | Starting memory ID |
| `max_depth` | number | No | Max traversal depth (default: 3, max: 10) |
| `relationship_types` | array | No | Filter by relationship types |

---

### reflect

Analyze memory patterns and generate insights.

```json
{
  "time_window": "week",
  "include_documents": false,
  "start_date": "2025-01-01",
  "end_date": "2025-01-07"
}
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `time_window` | enum | No | "recent", "today", "week" (default: "recent") |
| `include_documents` | boolean | No | Include document chunks (default: false) |
| `start_date` | string | No | Filter by date (ISO 8601) |
| `end_date` | string | No | Filter by date (ISO 8601) |

---

### save_context

Create a checkpoint of cognitive state for session handoffs.

```json
{
  "name": "API Investigation",
  "description": "[Session] Found the bug in auth flow. Key discovery in memory_id: 550e8400-e29b-41d4-a716-446655440000. Also see memory: 'JWT validation issue'. Next: test fix."
}
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Checkpoint name (must be unique per tenant) |
| `description` | string | No | Detailed cognitive handoff description |

**Unique Names:**

Checkpoint names are unique per tenant. Saving with a name that already exists returns an error instead of silently overwriting:

```json
{
  "success": false,
  "error": "Context name 'My Checkpoint' already exists. Use a unique name.",
  "error_code": "DUPLICATE_NAME",
  "operation": "save_context"
}
```

To avoid collisions, include a timestamp or session identifier in the name (e.g., `"Auth Investigation - 2026-02-11"`).

**Memory References in Descriptions**

You can reference memories in the description to link them to the checkpoint. Two formats are supported:

| Format | Reliability | Example |
|--------|-------------|---------|
| `memory_id: UUID` | **Reliable** — direct lookup, always resolves | `memory_id: 550e8400-e29b-41d4-a716-446655440000` |
| `memory: 'phrase'` | **Best-effort** — search + substring matching, may not resolve | `see memory: 'JWT validation issue'` |

Prefer `memory_id: UUID` references whenever you have the UUID. Semantic phrase references use hybrid search with substring matching and may silently fail to resolve if the memory isn't in the top results.

**Response:**

```json
{
  "success": true,
  "message": "Context 'API Investigation' saved",
  "context_id": "uuid",
  "memories_included": 8,
  "name": "API Investigation",
  "references_extracted": 2,
  "references_resolved": 2
}
```

| Field | Description |
|-------|-------------|
| `context_id` | UUID of the created checkpoint memory |
| `memories_included` | Total memories attached to the checkpoint |
| `references_extracted` | Number of memory references parsed from description |
| `references_resolved` | Number of references that matched a memory |
| `unresolved_references` | List of phrases that didn't resolve (only present when > 0) |
| `hint` | Suggestion to use UUID format (only present when there are failures) |

---

### restore_context

Restore a saved checkpoint.

```json
{
  "name": "API Investigation",
  "limit": 20
}
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `name` | string | Yes | Checkpoint name to restore |
| `limit` | number | No | Max memories to return (default: 20) |

**Special case:** `name: "awakening"` loads the user's personality configuration.

---

### list_contexts

List saved checkpoints with optional filtering and pagination.

**Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | integer | `20` | Maximum contexts to return (1-100) |
| `offset` | integer | `0` | Number of contexts to skip (pagination) |
| `name_pattern` | string | - | Filter by name (case-insensitive substring match) |
| `include_descriptions` | boolean | `false` | Include full descriptions in results |

**Example - Basic:**
```json
{
  "limit": 10
}
```

**Example - With filtering:**
```json
{
  "limit": 20,
  "name_pattern": "investigation",
  "include_descriptions": true
}
```

**Response:**
```json
{
  "success": true,
  "total": 3,
  "limit": 20,
  "offset": 0,
  "has_more": false,
  "contexts": [
    {
      "id": "uuid",
      "name": "API Investigation",
      "description": "Detailed context from API debugging session...",
      "memory_count": 15,
      "created": "2025-01-06T12:00:00Z"
    }
  ]
}
```

| Field | Description |
|-------|-------------|
| `total` | Total number of matching contexts |
| `limit` | Page size used |
| `offset` | Number of results skipped |
| `has_more` | Whether more results exist beyond this page |

**Notes:**
- `name_pattern` performs case-insensitive substring matching across all pages
- `include_descriptions` adds full description text (can be large)
- Contexts are ordered by creation date (newest first)
- `total` reflects the full count of matching contexts, regardless of `limit`

---

### save_artifact

Save a file artifact.

```json
{
  "content": "# Notes\n\nContent here...",
  "path": "/project/notes.md"
}
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `content` | string | Yes | File content |
| `path` | string | Yes | Path starting with / (e.g., /project/file.txt) |

**Path rules:**
- Must start with `/`
- Must specify filename (not directory)
- No path traversal (`..`)
- No hidden files (`.`)
- Max depth: 10 levels
- Valid characters: alphanumeric, `-`, `_`, `.`, `/`

---

### retrieve_artifact

Get a stored artifact.

```json
{
  "path": "/project/notes.md"
}
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Artifact path to retrieve |

---

### list_artifacts

List artifacts in a directory.

```json
{
  "path_prefix": "/project/"
}
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path_prefix` | string | No | Directory prefix (default: "/") |

---

### delete_artifact

Delete a stored artifact.

```json
{
  "path": "/project/old-notes.md"
}
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Artifact path to delete |

---

## Restricted Tools (Not Available via Standard MCP)

These tools require elevated permissions and are only available via the REST API with `write` scope or through the Penfield Portal.

### disconnect

Remove a relationship between memories.

**Status:** Registered as MCP tool but **requires elevated permissions**.

**Why it fails:** The tool is visible in MCP tool lists, but calling it returns `403 Forbidden` for standard MCP users. The underlying API endpoint requires `RELATIONSHIPS_DELETE`, `DELETE`, or `ADMIN` scope, which standard MCP access does not include.

**Recommendation:** Use the REST API with `write` scope or access via Penfield Portal for relationship removal.

**Parameters:**
- `from_memory` (UUID): Source memory ID
- `to_memory` (UUID): Target memory ID

**API Endpoint:** `DELETE /api/v2/relationships/between?from_id={id}&to_id={id}`

**Error when called via MCP:**
```json
{
  "error": "Forbidden",
  "status_code": 403
}
```

---

### delete_memory

Permanently delete a memory.

**Why restricted:** Memory deletion is a destructive operation. To prevent accidental data loss, it's excluded from MCP tools and requires explicit API access with `write` scope.

**API Endpoint:** `DELETE /api/v2/memories/{id}`

---

## Deprecated Tools

These tools are deprecated and will be removed in a future release.

### summarize

**Status:** Deprecated - use `save_context` instead.

The summarize tool produced statistical summaries rather than useful cognitive handoffs. It has been replaced by `save_context` which preserves narrative context and memory references.

---

### get_storage_access

**Status:** Deprecated - use Penfield Portal instead.

Direct MinIO storage access has been removed for security reasons. Use the [Penfield Portal](https://portal.penfield.app/documents) to manage documents.

---

## Relationship Types

When using `connect`, choose from these 24 relationship types:

### Knowledge Evolution
- `supersedes` - Replaces outdated information
- `updates` - Modifies existing knowledge
- `evolution_of` - Develops from earlier concept

### Evidence & Support
- `supports` - Provides evidence for
- `contradicts` - Challenges existing belief
- `disputes` - Disagrees with

### Hierarchy & Structure
- `parent_of` - Encompasses more specific concept
- `child_of` - Subset of broader concept
- `sibling_of` - Parallel concept at same level
- `composed_of` - Contains component parts
- `part_of` - Belongs to larger structure

### Causation
- `causes` - Leads to effect/outcome
- `influenced_by` - Was shaped by
- `prerequisite_for` - Required for next concept

### Implementation & Testing
- `implements` - Applies theoretical concept
- `documents` - Describes system/process
- `tests` - Verifies or validates
- `example_of` - Demonstrates general principle

### Conversation & Attribution
- `responds_to` - Response to previous statement
- `references` - Refers to or cites
- `inspired_by` - Inspired or motivated by

### Sequence & Flow
- `follows` - Comes after in sequence
- `precedes` - Comes before in sequence

### Dependencies
- `depends_on` - Requires prerequisite

---

## Configuration

MCP server configuration varies by client. Contact Penfield support for:

- MCP server URL
- Authentication credentials
- Client-specific setup instructions

### Claude Desktop

Add to your Claude Desktop MCP configuration:

```json
{
  "mcpServers": {
    "penfield": {
      "url": "YOUR_MCP_SERVER_URL",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

---

## Best Practices

### Memory Storage
1. Store detailed, contextual information
2. Let the system auto-detect memory types
3. Use descriptive tags for organization
4. Include reasoning and evidence in content

### Knowledge Graph
1. Always connect related memories after storing
2. Use specific relationship types (see the 24 available types above)
3. Build chains of evidence with `supports` relationships
4. Track corrections with `supersedes` relationships

### Context Management
1. Save checkpoints at investigation milestones
2. Include detailed descriptions in checkpoints
3. Reference memories by UUID (`memory_id: UUID`) for reliable linking
4. Use meaningful checkpoint names

### Performance
1. Use `recall` with appropriate limits
2. Filter by source_type when searching documents vs memories
3. Limit graph exploration depth for large graphs
4. Use `reflect` sparingly (computationally expensive)

---

## Error Handling

MCP tools return errors in this format:

```json
{
  "success": false,
  "error": "Memory not found",
  "error_code": "NOT_FOUND",
  "operation": "fetch"
}
```

Common error codes:

| Code | Description |
|------|-------------|
| `NOT_FOUND` | Memory or resource doesn't exist |
| `VALIDATION_ERROR` | Invalid parameters |
| `INVALID_PATH` | Invalid artifact path |
| `ALREADY_EXISTS` | Resource already exists |
| `RATE_LIMITED` | Too many requests |

---

## API Mapping Reference

| MCP Tool | HTTP Method | Endpoint |
|----------|-------------|----------|
| `awaken` | GET | `/api/v2/personality/awakening` |
| `store` | POST | `/api/v2/memories` |
| `recall` | POST | `/api/v2/search/hybrid` |
| `search` | POST | `/api/v2/search/hybrid` |
| `fetch` | GET | `/api/v2/memories/{id}` |
| `update_memory` | PUT | `/api/v2/memories/{id}` |
| `connect` | POST | `/api/v2/relationships` |
| `explore` | POST | `/api/v2/relationships/traverse` |
| `save_context` | POST | `/api/v2/memories` |
| `restore_context` | GET | `/api/v2/memories` |
| `list_contexts` | GET | `/api/v2/memories?memory_type=checkpoint` |
| `reflect` | POST | `/api/v2/analysis/reflect` |
| `save_artifact` | POST | `/api/v2/artifacts` |
| `retrieve_artifact` | GET | `/api/v2/artifacts` |
| `list_artifacts` | GET | `/api/v2/artifacts/list` |
| `delete_artifact` | DELETE | `/api/v2/artifacts` |
