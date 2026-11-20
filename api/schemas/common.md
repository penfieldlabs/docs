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
    "timestamp": "2026-01-06T12:00:00.390742+00:00",
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
| `meta.request_id` | string | Unique request identifier for debugging. Treat it as an opaque string |

---

## Error Response

All error responses follow this structure:

```json
{
  "status": "error",
  "error": {
    "code": "RES_NOT_FOUND",
    "message": "Memory with id '550e8400-e29b-41d4-a716-446655440000' not found",
    "details": {
      "resource_type": "Memory",
      "resource_id": "550e8400-e29b-41d4-a716-446655440000"
    },
    "field_errors": null
  },
  "meta": {
    "timestamp": "2026-01-06T12:00:00.390742+00:00",
    "version": "2.0.0",
    "request_id": "dd0e8400e29b41d4a716446655440008"
  }
}
```

### Error Fields

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | Always `"error"` for error responses |
| `error.code` | string | Machine-readable error code |
| `error.message` | string | Human-readable error message |
| `error.details` | object or null | Additional error context |
| `error.field_errors` | object or null | On a 422, one entry per invalid field: the field name mapped to an array of messages. `null` on other errors |

A validation error (422) looks like this:

```json
{
  "status": "error",
  "error": {
    "code": "VAL_VALIDATION_FAILED",
    "message": "Request validation failed",
    "details": null,
    "field_errors": {
      "content": ["Field required"]
    }
  },
  "meta": {...}
}
```

See the [Error Handling Guide](../guides/error-handling.md) for the list of codes.

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

Bulk operation responses list what was changed:

```json
{
  "status": "success",
  "data": {
    "created": [ ... ],
    "updated": [ ... ],
    "deleted": [ ... ],
    "errors": []
  },
  "meta": { ... }
}
```

Bulk operations are all-or-nothing. If one entry is refused, nothing is changed and the call returns an [error response](#error-response) whose message and `details.index` identify the entry.

---

## Memory Object

The core memory data structure:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "content": "Memory content text...",
  "memory_type": "fact",
  "importance": 0.8,
  "confidence": 0.8,
  "surprise_score": 0.5,
  "source_type": "direct_input",
  "user_id": "880e8400-e29b-41d4-a716-446655440003",
  "access_count": 5,
  "last_accessed": "2026-01-06T14:00:00.000000Z",
  "created_at": "2026-01-06T12:00:00.288284Z",
  "updated_at": "2026-01-06T12:00:00.288284Z",
  "metadata": {},
  "embedding_status": "completed",
  "relationships": [],
  "tags": ["async", "python"]
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
| `user_id` | UUID | Creator identifier |
| `access_count` | integer | Times accessed (not real-time) |
| `last_accessed` | datetime | Last access timestamp (not real-time) |
| `created_at` | datetime | Creation timestamp |
| `updated_at` | datetime | Last update timestamp |
| `metadata` | object | Custom metadata |
| `embedding_status` | string | Embedding generation status |
| `relationships` | array | Related memories (when populated) |
| `tags` | array | Tag names, normalized, in alphabetical order |

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
  "created_at": "2026-01-06T12:00:00.953480Z",
  "updated_at": "2026-01-06T12:00:00.953480Z"
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
  "created_at": "2026-01-06T12:00:00.502632Z",
  "updated_at": "2026-01-06T12:00:02.502112Z"
}
```

---

## Search Result Object

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "content": "Full memory content...",
  "snippet": "Truncated content...",
  "score": 0.71,
  "score_breakdown": {
    "final_score": 0.71,
    "bm25_score": 24.51,
    "vector_score": 0.78,
    "graph_score": 0.0,
    "rank_bm25": 1.0,
    "rank_vector": 1.0
  },
  "metadata": {
    "memory_type": "fact",
    "importance": 0.8,
    "highlight": null
  },
  "source_type": "direct_input",
  "created_at": "2026-01-06T12:00:00.162108Z",
  "tags": ["async", "python"],
  "content_with_context": null,
  "chunk_index": null,
  "total_chunks": null
}
```

This is a [hybrid search](../endpoints/search.md#hybrid-search) result. `content_with_context`, `chunk_index` and `total_chunks` are filled in for document chunks and `null` for other memories. See the search page for the field-by-field description and for the vector search variant.

---

## Timestamp Format

All timestamps use ISO 8601 format with timezone:

```
2026-01-06T12:00:00.288284Z
2026-01-06T12:00:00.390742+00:00
```

- Always UTC
- Microsecond precision
- The UTC marker is written either as `Z` or as `+00:00`, depending on the field (`meta.timestamp` uses `+00:00`, most resource fields use `Z`). Both mean the same thing. Use an ISO 8601 parser and do not compare timestamps as strings

---

## UUID Format

All identifiers use UUID v4:

```
550e8400-e29b-41d4-a716-446655440000
```

- 36 characters including hyphens
- Format: `xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx`
- Case-insensitive in requests
