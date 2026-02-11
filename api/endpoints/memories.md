# Memories Endpoints

Memories are the core data unit in Penfield.

---

## Create Memory

```
POST https://api.penfield.app/api/v2/memories
```

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | `Bearer YOUR_JWT_TOKEN` |
| `Content-Type` | Yes | `application/json` |

### Request Body

```json
{
  "content": "Python async programming uses asyncio",
  "memory_type": "fact",
  "importance": 0.8,
  "tags": ["python", "async"]
}
```

### Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `content` | string | Yes | Memory content (1-10,000 chars) |
| `memory_type` | string | No | Type (fact, insight, etc.) |
| `importance` | float | No | 0.0-1.0 |
| `confidence` | float | No | 0.0-1.0 |
| `tags` | array | No | Up to 10 tags |

### Response

```json
{
  "status": "success",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "content": "Python async programming uses asyncio for concurrent operations",
    "memory_type": "fact",
    "importance": 0.8,
    "confidence": 0.9,
    "surprise_score": 0.5,
    "source_type": "direct_input",
    "user_id": "user_123",
    "access_count": 0,
    "last_accessed": null,
    "created_at": "2025-01-06T12:00:00.000Z",
    "updated_at": "2025-01-06T12:00:00.000Z",
    "metadata": {
      "category": "programming",
      "language": "python"
    },
    "embedding_status": "pending",
    "relationships": []
  },
  "meta": {
    "timestamp": "2025-01-06T12:00:00.000Z",
    "version": "2.0.0",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

### Notes

- Embeddings are generated asynchronously in the background
- `embedding_status` will be `"pending"` initially, then `"completed"` or `"failed"`
- Tags are normalized (lowercase, hyphens replace spaces/underscores)
- Invalid tags are silently skipped

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 401 | `AUTH_UNAUTHORIZED` | Missing or invalid token |
| 403 | `AUTH_FORBIDDEN` | `memory_type` is `identity_core` or `personality_trait` (use `/api/v2/personality` endpoints) |
| 422 | `VAL_VALIDATION_FAILED` | Invalid request body |
| 422 | `VAL_FIELD_TOO_LONG` | Content exceeds 10,000 characters |

---

## Get Memory

Retrieve a specific memory by ID.

```
GET https://api.penfield.app/api/v2/memories/{memory_id}
```

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `memory_id` | UUID | Memory identifier |

### Response

```json
{
  "status": "success",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "content": "Python async programming uses asyncio for concurrent operations",
    "memory_type": "fact",
    "importance": 0.8,
    "confidence": 0.9,
    "surprise_score": 0.5,
    "source_type": "direct_input",
    "user_id": "user_123",
    "access_count": 5,
    "last_accessed": "2025-01-06T14:00:00.000Z",
    "created_at": "2025-01-06T12:00:00.000Z",
    "updated_at": "2025-01-06T12:00:00.000Z",
    "metadata": {},
    "embedding_status": "completed",
    "relationships": []
  },
  "meta": {...}
}
```

### Notes

- The `access_count` and `last_accessed` fields reflect access patterns over time but are not guaranteed to update immediately on each read. Applications should not rely on these fields for real-time read-after-write consistency.

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 404 | `RES_NOT_FOUND` | Memory not found |

---

## List Memories

List memories with pagination and filters.

```
GET https://api.penfield.app/api/v2/memories
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | `1` | Page number (1-based) |
| `per_page` | integer | `20` | Items per page (max 100) |
| `memory_type` | string | - | Filter by memory type |
| `user_id` | string | - | Filter by creator |
| `importance_threshold` | float | - | Minimum importance (0.0-1.0) |
| `date_from` | datetime | - | Filter memories created after this date |
| `date_to` | datetime | - | Filter memories created before this date |
| `time_range` | string | - | Relative time filter: `today`, `yesterday`, `week`, `month`, `quarter`, `year` |
| `tags` | string | - | Comma-separated tags (OR logic) |
| `sort` | string | `-created_at` | Sort field with optional `-` prefix for descending |
| `include_inactive` | boolean | `false` | Include evolved/deprecated memories |

### Response

```json
{
  "status": "success",
  "data": {
    "items": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "content": "Python async programming...",
        "memory_type": "fact",
        "importance": 0.8,
        "confidence": 0.9,
        "surprise_score": 0.5,
        "source_type": "direct_input",
        "user_id": "user_123",
        "access_count": 5,
        "last_accessed": "2025-01-06T14:00:00.000Z",
        "created_at": "2025-01-06T12:00:00.000Z",
        "updated_at": "2025-01-06T12:00:00.000Z",
        "metadata": {},
        "embedding_status": "completed",
        "relationships": []
      }
    ],
    "pagination": {
      "page": 1,
      "per_page": 20,
      "total": 150,
      "pages": 8,
      "has_next": true,
      "has_prev": false
    }
  },
  "meta": {...}
}
```

### Sort Options

| Value | Description |
|-------|-------------|
| `created_at` | Sort by creation date (ascending) |
| `-created_at` | Sort by creation date (descending, default) |
| `importance` | Sort by importance (ascending) |
| `-importance` | Sort by importance (descending) |
| `updated_at` | Sort by update date (ascending) |
| `-updated_at` | Sort by update date (descending) |

### Example

```bash
# Get recent high-importance memories tagged with "python"
curl -X GET "https://api.penfield.app/api/v2/memories?importance_threshold=0.7&tags=python&sort=-importance" \
-H "Authorization: Bearer $JWT_TOKEN"
```

---

## Update Memory

Update an existing memory.

```
PUT https://api.penfield.app/api/v2/memories/{memory_id}
```

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `memory_id` | UUID | Memory identifier |

### Request Body

Only include fields you want to update:

```json
{
  "content": "Updated content...",
  "importance": 0.9,
  "metadata": {
    "updated": true
  }
}
```

### Request Fields

| Field | Type | Description |
|-------|------|-------------|
| `content` | string | New content (1-10,000 characters) |
| `memory_type` | string | New memory type |
| `importance` | float | New importance (0.0-1.0) |
| `confidence` | float | New confidence (0.0-1.0) |
| `metadata` | object | New metadata (replaces existing) |
| `tags` | array | New tags (replaces existing) |

### Update Behavior

| Field | `null` | Empty (`[]` or `{}`) |
|-------|--------|----------------------|
| `tags` | No change | Clears all tags |
| `metadata` | No change | Clears metadata |

### Response

Returns the updated memory object.

### Notes

- Only provided fields are updated
- Updating `content` will regenerate embeddings
- The `updated_at` timestamp is automatically set

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 403 | `AUTH_FORBIDDEN` | `memory_type` is `identity_core` or `personality_trait` (use `/api/v2/personality` endpoints) |
| 404 | `RES_NOT_FOUND` | Memory not found |
| 422 | `VAL_VALIDATION_FAILED` | Invalid update data |

---

## Delete Memory

Permanently delete a memory.

```
DELETE https://api.penfield.app/api/v2/memories/{memory_id}
```

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `memory_id` | UUID | Memory identifier |

### Response

```
HTTP 204 No Content
```

### Notes

- This operation is permanent and cannot be undone
- Associated relationships are also deleted
- Tags are not deleted (they may be used by other memories)
- **Reserved for Portal/API Use:** This endpoint requires `write` scope and is not available via standard MCP access. It is intended for administrative use through the Penfield Portal or API keys with elevated permissions.

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 404 | `RES_NOT_FOUND` | Memory not found |

---

## Response Fields

All memory responses include these fields:

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Unique memory identifier |
| `content` | string | The memory content |
| `memory_type` | string | Type classification |
| `importance` | float | Significance score (0.0-1.0) |
| `confidence` | float | Certainty score (0.0-1.0) |
| `surprise_score` | float | Novelty indicator (0.0-1.0). Higher values indicate more unexpected or novel content. Default: 0.5. Currently set at creation; future versions may update dynamically. |
| `source_type` | string | Origin of the memory |
| `user_id` | string | User who created the memory |
| `access_count` | integer | Number of times accessed (not real-time; see note below) |
| `last_accessed` | datetime | When last accessed, or null (not real-time; see note below) |
| `embedding_status` | string | `"pending"`, `"completed"`, or `"failed"` |
| `created_at` | datetime | Creation timestamp |
| `updated_at` | datetime | Last modification timestamp |

### Access Tracking Note

The `access_count` and `last_accessed` fields track access patterns over time. These fields are not updated synchronously on every read request. Applications should not depend on these values for immediate read-after-write consistency. Use `created_at` or `updated_at` if you need reliable timestamps for ordering or change detection.

---

## Memory Types

| Type | Value | Description |
|------|-------|-------------|
| Fact | `fact` | Verified information |
| Insight | `insight` | Derived understanding or pattern |
| Conversation | `conversation` | Conversational context |
| Correction | `correction` | Update to previous information |
| Reference | `reference` | External reference or citation |
| Task | `task` | Action item or todo |
| Checkpoint | `checkpoint` | Conversation state snapshot |
| Identity Core | `identity_core` | Immutable AI identity |
| Personality Trait | `personality_trait` | Evolvable AI traits |
| Relationship | `relationship` | User-AI relationship info |
| Strategy | `strategy` | Learned behavior pattern |

---

## Source Types

| Type | Value | Description |
|------|-------|-------------|
| Direct Input | `direct_input` | Manually entered |
| Document Upload | `document_upload` | Extracted from uploaded document |
| Web Scrape | `web_scrape` | Scraped from web page |
| API Import | `api_import` | Imported via API |
| Conversation | `conversation` | From conversation context |
| Reflection | `reflection` | Generated from analysis |
| Checkpoint | `checkpoint` | From checkpoint operation |
| Checkpoint Recall | `checkpoint_recall` | Restored from checkpoint |
