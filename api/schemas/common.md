# Common Response Schemas

All API responses follow consistent structures.

---

## Success Response

All successful responses follow this structure:

```json
{
  "status": "success",
  "data": { ... },
  "meta": {
    "timestamp": "2025-01-06T12:00:00.000Z",
    "version": "2.0.0",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | Always `"success"` for successful responses |
| `data` | object/array | Response payload (varies by endpoint) |
| `meta` | object | Request metadata |
| `meta.timestamp` | datetime | Server timestamp (ISO 8601) |
| `meta.version` | string | API version |
| `meta.request_id` | UUID | Unique request identifier for debugging |

---

## Error Response

All error responses follow this structure:

```json
{
  "status": "error",
  "error": {
    "code": "RES_NOT_FOUND",
    "message": "Memory not found",
    "details": {
      "resource_type": "Memory",
      "resource_id": "550e8400-e29b-41d4-a716-446655440000"
    }
  },
  "meta": {
    "timestamp": "2025-01-06T12:00:00.000Z",
    "version": "2.0.0",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

### Error Fields

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | Always `"error"` for error responses |
| `error.code` | string | Machine-readable error code |
| `error.message` | string | Human-readable error message |
| `error.details` | object | Additional error context (optional) |

---

## Pagination

Paginated responses include pagination metadata:

```json
{
  "status": "success",
  "data": {
    "items": [ ... ],
    "pagination": {
      "page": 1,
      "per_page": 20,
      "total": 150,
      "pages": 8,
      "has_next": true,
      "has_prev": false
    }
  },
  "meta": { ... }
}
```

### Pagination Fields

| Field | Type | Description |
|-------|------|-------------|
| `page` | integer | Current page number (1-based) |
| `per_page` | integer | Items per page |
| `total` | integer | Total items across all pages |
| `pages` | integer | Total number of pages |
| `has_next` | boolean | More pages available after current |
| `has_prev` | boolean | Pages available before current |

### Query Parameters

| Parameter | Default | Max | Description |
|-----------|---------|-----|-------------|
| `page` | 1 | - | Page number |
| `per_page` | 20 | 100 | Items per page |

---

## Bulk Operations

Bulk operation responses include results and errors:

```json
{
  "status": "success",
  "data": {
    "created": [ ... ],
    "updated": [ ... ],
    "deleted": [ ... ],
    "errors": [
      {
        "index": 5,
        "error": {
          "code": "RES_NOT_FOUND",
          "message": "Memory not found"
        }
      }
    ]
  },
  "meta": { ... }
}
```

---

## Memory Object

The core memory data structure:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "content": "Memory content text...",
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
```

### Memory Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Unique identifier |
| `content` | string | Memory content (1-10,000 chars) |
| `memory_type` | string | Type classification |
| `importance` | float | Importance score (0-1) |
| `confidence` | float | Confidence score (0-1) |
| `surprise_score` | float | Novelty score (0-1) |
| `source_type` | string | Origin of memory |
| `user_id` | string | Creator identifier |
| `access_count` | integer | Times accessed (not real-time) |
| `last_accessed` | datetime | Last access timestamp (not real-time) |
| `created_at` | datetime | Creation timestamp |
| `updated_at` | datetime | Last update timestamp |
| `metadata` | object | Custom metadata |
| `embedding_status` | string | Embedding generation status |
| `relationships` | array | Related memories (when populated) |

> **Note:** The `access_count` and `last_accessed` fields reflect access patterns over time but are not updated synchronously on each read. Do not rely on these fields for real-time consistency.

---

## Relationship Object

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "from_id": "memory-uuid-1",
  "to_id": "memory-uuid-2",
  "relationship_type": "supports",
  "direction_type": "DIRECTED",
  "strength": 0.8,
  "confidence": 0.9,
  "evidence": {},
  "metadata": {},
  "created_at": "2025-01-06T12:00:00.000Z",
  "updated_at": "2025-01-06T12:00:00.000Z"
}
```

### Relationship Fields

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Unique identifier |
| `from_id` | UUID | Source memory ID |
| `to_id` | UUID | Target memory ID |
| `relationship_type` | string | Type of relationship |
| `direction_type` | string | Direction handling |
| `strength` | float | Relationship strength (0-1) |
| `confidence` | float | Confidence in relationship (0-1) |
| `evidence` | object | Supporting evidence |
| `metadata` | object | Reserved (system-managed) |

---

## Document Object

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "filename": "document.pdf",
  "content_type": "application/pdf",
  "size": 1048576,
  "chunk_count": 45,
  "processing_status": "completed",
  "embedding_progress": {
    "total": 45,
    "completed": 45,
    "pending": 0,
    "failed": 0,
    "percent_complete": 100.0
  },
  "metadata": {},
  "created_at": "2025-01-06T12:00:00.000Z",
  "updated_at": "2025-01-06T12:00:00.000Z"
}
```

---

## Search Result Object

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "content": "Full memory content...",
  "snippet": "Truncated content...",
  "score": 0.95,
  "score_breakdown": {
    "final_score": 0.95,
    "bm25_score": 0.85,
    "vector_score": 0.92,
    "graph_score": 0.7
  },
  "metadata": {
    "memory_type": "fact",
    "importance": 0.8,
    "highlight": "Matching <b>terms</b>..."
  },
  "source_type": "direct_input",
  "created_at": "2025-01-06T12:00:00.000Z",
  "content_with_context": "...",
  "chunk_index": 0,
  "total_chunks": 1
}
```

---

## Timestamp Format

All timestamps use ISO 8601 format with timezone:

```
2025-01-06T12:00:00.000Z
```

- Always UTC (indicated by `Z` suffix)
- Millisecond precision
- Format: `YYYY-MM-DDTHH:mm:ss.sssZ`

---

## UUID Format

All identifiers use UUID v4:

```
550e8400-e29b-41d4-a716-446655440000
```

- 36 characters including hyphens
- Format: `xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx`
- Case-insensitive in requests
