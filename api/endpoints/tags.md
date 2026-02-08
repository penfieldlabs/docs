# Tags Endpoints

Organize memories with tags for easier discovery and filtering.

---

## Add Tags to Memory

Add one or more tags to a memory.

```
POST https://api.penfield.app/api/v2/memories/{memory_id}/tags
```

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | `Bearer YOUR_JWT_TOKEN` |
| `Content-Type` | Yes | `application/json` |

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `memory_id` | UUID | Memory identifier |

### Request Body

```json
{
  "tags": ["python", "async", "best-practices"]
}
```

### Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `tags` | array | Yes | Tags to add (max 10 total per memory) |

### Tag Normalization

Tags are automatically normalized:
- Converted to lowercase
- Spaces and underscores replaced with hyphens
- Invalid characters removed

| Input | Normalized |
|-------|------------|
| `"Python"` | `"python"` |
| `"Best Practices"` | `"best-practices"` |
| `"async_await"` | `"async-await"` |

### Response

```json
{
  "status": "success",
  "data": {
    "memory_id": "550e8400-e29b-41d4-a716-446655440000",
    "tags": ["python", "async", "best-practices"],
    "warnings": null
  },
  "meta": {...}
}
```

### Response Fields

| Field | Description |
|-------|-------------|
| `memory_id` | The memory identifier |
| `tags` | Complete list of tags now on the memory |
| `warnings` | Array of warning messages, or null if none |

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 404 | `RES_NOT_FOUND` | Memory not found |
| 422 | `VAL_VALIDATION_FAILED` | Would exceed 10 tags per memory |

---

## Remove Tags from Memory

Remove specific tags from a memory.

```
DELETE https://api.penfield.app/api/v2/memories/{memory_id}/tags
```

### Request Body

```json
{
  "tags": ["best-practices"]
}
```

### Response

```json
{
  "status": "success",
  "data": {
    "memory_id": "550e8400-e29b-41d4-a716-446655440000",
    "tags": ["python", "async"],
    "warnings": null
  },
  "meta": {...}
}
```

### Response Fields

| Field | Description |
|-------|-------------|
| `memory_id` | The memory identifier |
| `tags` | Remaining tags after removal |
| `warnings` | Array of warning messages, or null |

---

## Get Memory Tags

Get all tags for a specific memory.

```
GET https://api.penfield.app/api/v2/memories/{memory_id}/tags
```

### Response

```json
{
  "status": "success",
  "data": ["python", "async", "best-practices"],
  "meta": {...}
}
```

Returns an array of tag names. Empty array if the memory has no tags.

---

## List All Tags

List all tags with usage counts.

```
GET https://api.penfield.app/api/v2/tags
```

### Response

```json
{
  "status": "success",
  "data": {
    "items": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "name": "python",
        "usage_count": 45,
        "created_at": "2025-01-01T00:00:00.000Z"
      },
      {
        "id": "660f9500-f30c-52e5-b827-557766551111",
        "name": "javascript",
        "usage_count": 32,
        "created_at": "2025-01-02T00:00:00.000Z"
      },
      {
        "id": "770e8400-e29b-41d4-a716-446655440002",
        "name": "machine-learning",
        "usage_count": 28,
        "created_at": "2025-01-03T00:00:00.000Z"
      }
    ],
    "total": 156
  },
  "meta": {...}
}
```

### Notes

- Tags are sorted by usage count (most used first)
- `usage_count` shows how many memories use each tag
- Tags with zero usage may be cleaned up periodically

---

## Get Memories by Tag

Get all memories with a specific tag (paginated).

```
GET https://api.penfield.app/api/v2/tags/{tag}/memories
```

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `tag` | string | Tag name (normalized) |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | `1` | Page number (minimum 1) |
| `per_page` | integer | `20` | Results per page (1-100) |

### Response

```json
{
  "status": "success",
  "data": {
    "items": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "content": "Python async programming uses asyncio...",
        "memory_type": "fact",
        "importance": 0.8,
        "confidence": 0.9,
        "created_at": "2025-01-01T00:00:00.000Z",
        "tags": ["python", "async"],
        ...
      }
    ],
    "pagination": {
      "page": 1,
      "per_page": 20,
      "total": 45,
      "pages": 3
    }
  },
  "meta": {...}
}
```

### Response Fields

| Field | Description |
|-------|-------------|
| `items` | Array of Memory objects with the specified tag |
| `pagination` | Pagination metadata (page, per_page, total, pages) |

### Notes

- Tag name is normalized before matching (lowercase, hyphens)
- Returns empty results (`items: []`, `total: 0`) for non-existent tags
- Memories are returned with full Memory schema
- Use pagination for large result sets

### Example

```bash
# Get first 10 memories tagged with "python"
curl -X GET "https://api.penfield.app/api/v2/tags/python/memories?page=1&per_page=10" \
-H "Authorization: Bearer $JWT_TOKEN"

# Get second page
curl -X GET "https://api.penfield.app/api/v2/tags/python/memories?page=2&per_page=10" \
-H "Authorization: Bearer $JWT_TOKEN"
```

---

## Filtering Memories by Tags

Alternative: Use the memories list endpoint to filter by multiple tags:

```
GET https://api.penfield.app/api/v2/memories?tags=python,async
```

### Tag Filter Behavior

- Multiple tags use **OR logic** (memories matching ANY tag)
- Tags are normalized before matching
- Case-insensitive matching

### Example

```bash
# Get memories tagged with either "python" or "javascript"
curl -X GET "https://api.penfield.app/api/v2/memories?tags=python,javascript" \
-H "Authorization: Bearer $JWT_TOKEN"
```

---

## Best Practices

### Tag Naming

- Use lowercase and hyphens: `machine-learning` not `Machine Learning`
- Be consistent: decide between `ml` and `machine-learning`
- Use hierarchical tags: `python-async`, `python-testing`
- Avoid overly specific tags: `my-project-v2-feature-x`

### Tag Organization

| Category | Example Tags |
|----------|--------------|
| Language | `python`, `javascript`, `rust` |
| Topic | `machine-learning`, `web-dev`, `devops` |
| Type | `tutorial`, `reference`, `snippet` |
| Status | `verified`, `needs-review`, `deprecated` |
| Project | `project-alpha`, `project-beta` |

### Tag Limits

| Limit | Value |
|-------|-------|
| Max tags per memory | 10 |
| Max tag length | 100 characters |
| Allowed characters | `a-z`, `0-9`, `-` |
