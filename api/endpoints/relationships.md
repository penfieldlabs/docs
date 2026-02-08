# Relationships Endpoints

Create and traverse relationships between memories to build a knowledge graph.

---

## Create Relationship

Create a relationship between two memories.

```
POST https://api.penfield.app/api/v2/relationships
```

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | `Bearer YOUR_JWT_TOKEN` |
| `Content-Type` | Yes | `application/json` |

### Request Body

```json
{
  "from_id": "550e8400-e29b-41d4-a716-446655440000",
  "to_id": "660f9500-f30c-52e5-b827-557766551111",
  "relationship_type": "supports",
  "direction_type": "DIRECTED",
  "strength": 0.8,
  "confidence": 0.9,
  "evidence": {
    "reason": "Both discuss async patterns",
    "source": "manual_connection"
  }
}
```

### Request Fields

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `from_id` | UUID | Yes | - | Source memory ID |
| `to_id` | UUID | Yes | - | Target memory ID |
| `relationship_type` | string | Yes | - | Type of relationship (see [Relationship Types](#relationship-types)) |
| `direction_type` | string | No | `"DIRECTED"` | Direction handling (see [Direction Types](#direction-types)) |
| `inverse_type` | string | No | - | Inverse relationship type (for `INVERSE_PAIRED`) |
| `strength` | float | No | `0.5` | Relationship strength (0.0-1.0) |
| `confidence` | float | No | `0.8` | Confidence in relationship (0.0-1.0) |
| `evidence` | object | No | `{}` | Supporting evidence/reasoning |
| `traversal_properties` | object | No | `{}` | Properties for graph algorithms |

### Direction Types

| Type | Description |
|------|-------------|
| `DIRECTED` | One-way relationship (A → B) |
| `BIDIRECTIONAL` | Single relationship queryable both ways |
| `INVERSE_PAIRED` | Creates two relationships with inverse types (requires `inverse_type`) |

### Response

```json
{
  "status": "success",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "from_id": "550e8400-e29b-41d4-a716-446655440000",
    "to_id": "660f9500-f30c-52e5-b827-557766551111",
    "relationship_type": "supports",
    "direction_type": "DIRECTED",
    "strength": 0.8,
    "confidence": 0.9,
    "evidence": {"reason": "Both discuss async patterns"},
    "metadata": {},
    "created_at": "2025-01-06T12:00:00.000Z",
    "updated_at": "2025-01-06T12:00:00.000Z"
  },
  "meta": {...}
}
```

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 422 | `VAL_VALIDATION_FAILED` | Invalid relationship data |
| 422 | `VAL_FIELD_REQUIRED` | `inverse_type` required for `INVERSE_PAIRED` |
| 404 | `RES_NOT_FOUND` | Source or target memory not found |
| 409 | `RES_ALREADY_EXISTS` | Relationship already exists |

### Example

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

---

## Get Relationship

Retrieve a specific relationship by ID.

```
GET https://api.penfield.app/api/v2/relationships/{relationship_id}
```

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `relationship_id` | UUID | Relationship identifier |

### Response

```json
{
  "status": "success",
  "data": {
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
  },
  "meta": {...}
}
```

---

## List Relationships

List relationships with optional filters.

```
GET https://api.penfield.app/api/v2/relationships
```

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `from_id` | UUID | Filter by source memory |
| `to_id` | UUID | Filter by target memory |
| `memory_id` | UUID | Filter by either source or target |
| `relationship_type` | string | Filter by type |
| `page` | integer | Page number (default 1) |
| `per_page` | integer | Items per page (default 20, max 100) |

### Response

```json
{
  "status": "success",
  "data": {
    "items": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "from_id": "memory-uuid-1",
        "to_id": "memory-uuid-2",
        "relationship_type": "supports",
        "strength": 0.8,
        "confidence": 0.9,
        "evidence": {},
        "metadata": {},
        "created_at": "2025-01-06T12:00:00.000Z",
        "updated_at": "2025-01-06T12:00:00.000Z"
      }
    ],
    "pagination": {
      "page": 1,
      "per_page": 20,
      "total": 45,
      "pages": 3,
      "has_next": true,
      "has_prev": false
    }
  },
  "meta": {...}
}
```

### Example

```bash
# Get all relationships for a specific memory
curl -X GET "https://api.penfield.app/api/v2/relationships?memory_id=550e8400-e29b-41d4-a716-446655440000" \
-H "Authorization: Bearer $JWT_TOKEN"
```

---

## Update Relationship

Update relationship properties.

```
PATCH /api/v2/relationships/{relationship_id}
```

### Request Body

```json
{
  "strength": 0.9,
  "confidence": 0.95,
  "evidence": {
    "reason": "Updated based on new findings"
  }
}
```

### Updatable Fields

| Field | Type | Description |
|-------|------|-------------|
| `strength` | float | New strength (0.0-1.0) |
| `confidence` | float | New confidence (0.0-1.0) |
| `evidence` | object | New evidence |

### Response

Returns the updated relationship.

---

## Delete Relationship

Delete a relationship by ID.

```
DELETE https://api.penfield.app/api/v2/relationships/{relationship_id}
```

### Response

```
HTTP 204 No Content
```

### Notes

- For `INVERSE_PAIRED` relationships, you need to delete both relationships separately
- Deleting a relationship does not affect the connected memories

---

## Delete Relationship Between Memories

Delete the relationship between two specific memories.

```
DELETE https://api.penfield.app/api/v2/relationships/between?from_id={from_id}&to_id={to_id}
```

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `from_id` | UUID | Yes | Source memory ID |
| `to_id` | UUID | Yes | Target memory ID |

### Response

```
HTTP 204 No Content
```

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 404 | `RES_NOT_FOUND` | No relationship exists between these memories |

---

## Bulk Create Relationships

Create multiple relationships in a single transaction.

```
POST https://api.penfield.app/api/v2/relationships/bulk
```

### Request Body

```json
[
  {
    "from_id": "memory-uuid-1",
    "to_id": "memory-uuid-2",
    "relationship_type": "supports",
    "strength": 0.8
  },
  {
    "from_id": "memory-uuid-2",
    "to_id": "memory-uuid-3",
    "relationship_type": "follows",
    "strength": 0.9
  }
]
```

### Response

```json
{
  "status": "success",
  "data": {
    "created": [
      {"id": "550e8400-e29b-41d4-a716-446655440000", ...},
      {"id": "660f9500-f30c-52e5-b827-557766551111", ...}
    ],
    "updated": [],
    "deleted": [],
    "errors": []
  },
  "meta": {...}
}
```

### Notes

- All relationships must succeed or all are rolled back
- Use for creating complex relationship patterns efficiently

---

## Graph Traversal

Traverse the knowledge graph starting from a memory.

```
POST https://api.penfield.app/api/v2/relationships/traverse
```

### Request Body

```json
{
  "start_memory_id": "550e8400-e29b-41d4-a716-446655440000",
  "max_depth": 3,
  "direction": "OUTBOUND",
  "relationship_types": ["supports", "references"],
  "include_scores": true
}
```

### Request Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `start_memory_id` | UUID | **required** | Starting memory for traversal |
| `max_depth` | integer | `3` | Maximum traversal depth (1-10) |
| `direction` | string | `"ANY"` | Traversal direction: `OUTBOUND`, `INBOUND`, `ANY` |
| `relationship_types` | array | - | Filter by relationship types |
| `include_scores` | boolean | `true` | Include path scores |

### Response

```json
{
  "status": "success",
  "data": {
    "paths": [
      {
        "nodes": [
          "550e8400-e29b-41d4-a716-446655440000",
          "660f9500-f30c-52e5-b827-557766551111"
        ],
        "relationships": [
          "770e8400-e29b-41d4-a716-446655440002"
        ],
        "total_score": 0.4,
        "path_metadata": {
          "depth": 1,
          "start_node": "550e8400-e29b-41d4-a716-446655440000",
          "direction": "ANY"
        }
      }
    ],
    "visited_chunks": [
      "550e8400-e29b-41d4-a716-446655440000",
      "660f9500-f30c-52e5-b827-557766551111"
    ],
    "max_depth_reached": 2,
    "total_paths_found": 8
  },
  "meta": {...}
}
```

### Response Fields

| Field | Description |
|-------|-------------|
| `paths` | Array of paths found during traversal |
| `paths[].nodes` | Array of memory UUIDs in the path |
| `paths[].relationships` | Array of relationship UUIDs connecting the nodes |
| `paths[].total_score` | Combined score for the path |
| `paths[].path_metadata` | Metadata about the path |
| `visited_chunks` | Array of all memory UUIDs visited during traversal |
| `max_depth_reached` | Actual maximum depth traversed |
| `total_paths_found` | Total number of paths discovered |

### Traversal Directions

| Direction | Description |
|-----------|-------------|
| `OUTBOUND` | Follow relationships from source to target |
| `INBOUND` | Follow relationships from target to source |
| `ANY` | Follow relationships in either direction |

---

## Relationship Types

All 24 relationship types are working as of 2026-01-22.

### Knowledge Evolution

| Type | Description | Example |
|------|-------------|---------|
| `supersedes` | Completely replaces old understanding | "New policy supersedes old" |
| `updates` | Partially modifies existing knowledge | "Correction updates fact" |
| `evolution_of` | Developed from earlier concept | "V2 is evolution of V1" |

### Evidence & Support

| Type | Description | Example |
|------|-------------|---------|
| `supports` | Provides evidence for claim/hypothesis | "Study supports hypothesis" |
| `contradicts` | Challenges existing belief | "New finding contradicts theory" |
| `disputes` | Disagrees with perspective | "Alternative view disputes claim" |

### Hierarchy & Structure

| Type | Description | Example |
|------|-------------|---------|
| `parent_of` | Encompasses more specific concept | "Category parent of item" |
| `child_of` | Is subset of broader concept | "Specific child of general" |
| `sibling_of` | Parallels related concept at same level | "Option A sibling of Option B" |
| `composed_of` | Contains component parts | "System composed of modules" |
| `part_of` | Belongs to larger structure | "Module part of system" |

### Cause & Prerequisites

| Type | Description | Example |
|------|-------------|---------|
| `causes` | Leads to effect/outcome | "Action causes result" |
| `influenced_by` | Shaped by contributing factor | "Decision influenced by factor" |
| `prerequisite_for` | Required for next concept | "Step A prerequisite for Step B" |

### Implementation & Testing

| Type | Description | Example |
|------|-------------|---------|
| `implements` | Applies theoretical concept | "Code implements algorithm" |
| `documents` | Describes system/process | "Doc documents API" |
| `tests` | Verifies or validates | "Test suite tests implementation" |
| `example_of` | Demonstrates general principle | "Case example of pattern" |

### Conversation & Attribution

| Type | Description | Example |
|------|-------------|---------|
| `responds_to` | Response to previous statement | "Reply responds to question" |
| `references` | Refers to or cites | "Doc references specification" |
| `inspired_by` | Inspired or motivated by | "Design inspired by nature" |

### Sequence & Flow

| Type | Description | Example |
|------|-------------|---------|
| `follows` | Comes after previous step | "Step 2 follows Step 1" |
| `precedes` | Comes before next step | "Setup precedes execution" |

### Dependencies

| Type | Description | Example |
|------|-------------|---------|
| `depends_on` | Requires prerequisite | "Feature depends on library" |

### Complete List (24)

1. supersedes | 7. parent_of | 13. implements | 19. inspired_by
2. updates | 8. child_of | 14. documents | 20. follows
3. evolution_of | 9. sibling_of | 15. tests | 21. precedes
4. supports | 10. causes | 16. example_of | 22. depends_on
5. contradicts | 11. influenced_by | 17. responds_to | 23. composed_of
6. disputes | 12. prerequisite_for | 18. references | 24. part_of
