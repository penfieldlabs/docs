# Analysis Endpoints

Analyze patterns and insights from your memories.

---

## Reflect

Analyze recent memories to identify patterns, topics, and insights.

```
POST https://api.penfield.app/api/v2/analysis/reflect
```

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | `Bearer YOUR_JWT_TOKEN` |
| `Content-Type` | Yes | `application/json` |

### Request Body

```json
{
  "time_window": "week",
  "include_documents": false,
  "include_patterns": true,
  "include_relationships": true,
  "include_clusters": true,
  "max_memories": 50,
  "start_date": "2025-01-01",
  "end_date": "2025-01-31",
  "recency_weight": 0.4,
  "access_weight": 0.3,
  "importance_weight": 0.2,
  "relationship_weight": 0.1
}
```

### Request Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `time_window` | string | `"recent"` | Time period to analyze (see Time Windows below) |
| `include_documents` | boolean | `false` | Include document chunks in analysis |
| `include_patterns` | boolean | `true` | Include emerging patterns analysis |
| `include_relationships` | boolean | `true` | Include relationship insights |
| `include_clusters` | boolean | `true` | Include memory clustering |
| `max_memories` | integer | `50` | Maximum memories to analyze (1-1000) |
| `start_date` | string | - | Filter memories created on/after this date (ISO 8601) - overrides time_window |
| `end_date` | string | - | Filter memories created on/before this date (ISO 8601) - overrides time_window |
| `recency_weight` | float | `0.4` | Weight for recency in ranking (0-1) |
| `access_weight` | float | `0.3` | Weight for access frequency in ranking (0-1) |
| `importance_weight` | float | `0.2` | Weight for importance score in ranking (0-1) |
| `relationship_weight` | float | `0.1` | Weight for relationship count in ranking (0-1) |

**Important:** The four weight parameters must sum to 1.0.

### Time Windows

| Value | Description |
|-------|-------------|
| `recent` | Last 50 memories |
| `today` | Today's memories |
| `week` | Last 7 days |
| `month` | Last 30 days |

### Response

```json
{
  "status": "success",
  "data": {
    "time_window": "week",
    "memories": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "content": "Python async programming uses asyncio...",
        "memory_type": "fact",
        "importance": 0.9,
        "access_count": 0,
        "created_at": "2025-01-06T12:00:00.000Z",
        "age_hours": 2.16,
        "score": 0.597
      }
    ],
    "active_topics": ["python", "async", "programming", "api"],
    "memory_clusters": [
      {
        "name": "Recent Focus",
        "memory_count": 50,
        "description": "Memories from the last 24 hours"
      }
    ],
    "emerging_patterns": [
      {
        "pattern": "High Access Frequency",
        "memory_count": 5,
        "description": "Memories accessed more than 5 times"
      }
    ],
    "relationship_insights": [
      {
        "type": "supports",
        "count": 12,
        "description": "12 supports relationships"
      }
    ],
    "statistics": {
      "total_memories_analyzed": 150,
      "memories_returned": 50,
      "average_age_hours": 2.54,
      "average_importance": 0.61,
      "total_access_count": 0,
      "unique_topics": 20
    }
  },
  "meta": {...}
}
```

### Response Fields

| Field | Description |
|-------|-------------|
| `time_window` | The requested time window |
| `memories` | Array of memory objects with relevance scores |
| `memories[].age_hours` | Hours since memory creation |
| `memories[].score` | Relevance score for the time window |
| `active_topics` | Array of topic keywords extracted from memories |
| `memory_clusters` | Grouped memories by pattern |
| `emerging_patterns` | Detected behavioral patterns |
| `relationship_insights` | Summary of relationship types |
| `statistics` | Aggregate statistics for the analysis |

### Use Cases

#### Weekly Review

```bash
curl -X POST https://api.penfield.app/api/v2/analysis/reflect \
-H "Authorization: Bearer $JWT_TOKEN" \
-H "Content-Type: application/json" \
-d '{"time_window": "week"}'
```

#### Daily Reflection

```bash
curl -X POST https://api.penfield.app/api/v2/analysis/reflect \
-d '{"time_window": "today"}'
```

#### Include Documents

```bash
curl -X POST https://api.penfield.app/api/v2/analysis/reflect \
-d '{"time_window": "month", "include_documents": true}'
```

---

## Summarize Memories

Create a statistical summary of specific memories or recent activity.

```
POST https://api.penfield.app/api/v2/analysis/summarize
```

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | `Bearer YOUR_JWT_TOKEN` |
| `Content-Type` | Yes | `application/json` |

### Request Body

```json
{
  "memory_ids": ["550e8400-e29b-41d4-a716-446655440000", "..."],
  "max_memories": 50,
  "include_relationships": true
}
```

### Request Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `memory_ids` | array | - | Specific memory UUIDs to summarize (optional) |
| `max_memories` | integer | `50` | Maximum memories to include |
| `include_relationships` | boolean | `true` | Include relationship statistics |

### Behavior

- If `memory_ids` provided: Summarizes those specific memories
- If `memory_ids` omitted: Summarizes recent memories (last 24 hours)
- Limited to `max_memories` results

### Response

```json
{
  "status": "success",
  "data": {
    "summary": "Analyzed 50 memories: 45 fact, 3 insight, 2 task. Key topics: python, async, api, testing, deployment. Relationships: 12 supports, 5 references. Average importance: 0.72.",
    "memory_count": 50,
    "memory_types": {
      "fact": 45,
      "insight": 3,
      "task": 2
    },
    "relationships": [
      {
        "type": "supports",
        "count": 12,
        "description": "12 supports relationships"
      },
      {
        "type": "references",
        "count": 5,
        "description": "5 references relationships"
      }
    ],
    "time_range": {
      "oldest": "2025-01-06T10:00:00.000Z",
      "newest": "2025-01-06T14:30:00.000Z",
      "span_hours": 4.5
    }
  },
  "meta": {...}
}
```

### Response Fields

| Field | Description |
|-------|-------------|
| `summary` | Human-readable text summary |
| `memory_count` | Number of memories analyzed |
| `memory_types` | Distribution of memory types (type → count) |
| `relationships` | Relationship statistics (type, count, description) |
| `time_range` | Time span of analyzed memories (oldest, newest, span_hours) |

### Use Cases

**Summarize specific memories:**
```bash
curl -X POST https://api.penfield.app/api/v2/analysis/summarize \
-H "Authorization: Bearer $JWT_TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "memory_ids": ["uuid-1", "uuid-2", "uuid-3"],
  "include_relationships": true
}'
```

**Summarize recent activity:**
```bash
curl -X POST https://api.penfield.app/api/v2/analysis/summarize \
-d '{"max_memories": 100}'
```

### Notes

- Summary text includes: memory type distribution, key topics (word frequency), relationship counts, average importance
- Topic extraction uses simple word frequency (words > 5 characters from first 10 memories)
- Time range shows oldest/newest memory and span in hours
- Empty result returns "No memories found to summarize."

---

## Memory Insights

Get high-level insights and statistics about the memory system.

```
GET https://api.penfield.app/api/v2/analysis/insights
```

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | `Bearer YOUR_JWT_TOKEN` |

### Query Parameters

None - returns insights for all active memories.

### Response

```json
{
  "status": "success",
  "data": {
    "memory_type_distribution": {
      "fact": 120,
      "insight": 45,
      "task": 12,
      "checkpoint": 3
    },
    "statistics": {
      "total_memories": 180,
      "memories_last_24h": 15,
      "memories_last_week": 67,
      "memories_last_month": 156,
      "average_importance": 0.68,
      "total_access_count": 342
    },
    "insights": [
      {
        "insight": "Memory Creation Rate",
        "value": "15 memories/day",
        "trend": "stable"
      },
      {
        "insight": "Most Common Type",
        "value": "fact",
        "count": 120
      }
    ]
  },
  "meta": {...}
}
```

### Response Fields

| Field | Description |
|-------|-------------|
| `memory_type_distribution` | Count of memories by type |
| `statistics.total_memories` | Total active memories |
| `statistics.memories_last_24h` | Memories created in last 24 hours |
| `statistics.memories_last_week` | Memories created in last 7 days |
| `statistics.memories_last_month` | Memories created in last 30 days |
| `statistics.average_importance` | Mean importance score across all memories |
| `statistics.total_access_count` | Sum of all memory access counts |
| `insights` | Array of calculated insights |

### Insights Provided

| Insight | Description |
|---------|-------------|
| Memory Creation Rate | Memories created per day (last 24h) |
| Most Common Type | Memory type with highest count |

### Example

```bash
curl -X GET https://api.penfield.app/api/v2/analysis/insights \
-H "Authorization: Bearer $JWT_TOKEN"
```

### Use Cases

- Dashboard statistics
- Memory system health monitoring
- Usage pattern analysis
- Planning memory organization

---

## Memory Evolution History

Track how a memory has evolved over time.

```
GET https://api.penfield.app/api/v2/memories/{memory_id}/history
```

### Response

```json
{
  "status": "success",
  "data": {
    "current": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "content": "Current memory content...",
      "memory_type": "fact",
      "importance": 0.8,
      "confidence": 0.9,
      "created_at": "2025-01-06T12:00:00.000Z",
      "updated_at": "2025-01-06T12:00:00.000Z"
    },
    "history": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "version": 1,
        "content": "Original content...",
        "created_at": "2025-01-01T00:00:00.000Z",
        "lifecycle_state": "active",
        "is_active": true
      }
    ],
    "total_versions": 1
  },
  "metadata": {
    "memory_id": "550e8400-e29b-41d4-a716-446655440000",
    "versions_count": 1
  }
}
```

### Response Fields

| Field | Description |
|-------|-------------|
| `current` | Full memory object for the current version |
| `history` | Array of version records |
| `history[].version` | Version number |
| `history[].lifecycle_state` | State: `active`, `evolved`, `deprecated` |
| `history[].is_active` | Whether this version is current |
| `total_versions` | Total number of versions |

### Evolution Types

| Type | Description |
|------|-------------|
| `correction` | Fixed an error |
| `update` | Added new information |
| `expansion` | Expanded with more detail |
| `contradiction` | Conflicting information |
| `deprecation` | Marked as outdated |

---

## Report Contradiction

Report a contradiction between two memories.

```
POST https://api.penfield.app/api/v2/memories/contradictions
```

### Status

~~**Not Yet Available**~~ - This endpoint is now **AVAILABLE**. Use it to report contradictions between memories:

```json
POST /api/v2/relationships
{
  "from_id": "memory-1",
  "to_id": "memory-2",
  "relationship_type": "contradicts"
}
```

### Request Body

```json
{
  "source_memory_id": "550e8400-e29b-41d4-a716-446655440000",
  "target_memory_id": "conflicting-memory-uuid",
  "contradiction_type": "direct",
  "confidence": 0.8,
  "metadata": {
    "description": "These memories state opposite facts"
  }
}
```

### Contradiction Types

| Type | Description |
|------|-------------|
| `direct` | Completely contradictory statements |
| `partial` | Partially conflicting information |
| `contextual` | Contradictory in specific contexts |

### Response

```json
{
  "status": "success",
  "data": {
    "id": "contradiction-uuid",
    "source_memory_id": "550e8400-e29b-41d4-a716-446655440000",
    "target_memory_id": "conflicting-memory-uuid",
    "contradiction_type": "direct",
    "resolution_status": "unresolved",
    "detected_at": "2025-01-06T12:00:00.000Z",
    "detected_by": "user"
  },
  "meta": {...}
}
```

---

## List Contradictions

Get contradictions for a memory.

```
GET https://api.penfield.app/api/v2/memories/{memory_id}/contradictions
```

### Response

```json
{
  "status": "success",
  "data": {
    "contradictions": [
      {
        "id": "contradiction-uuid",
        "target_memory_id": "conflicting-memory-uuid",
        "contradiction_type": "direct",
        "resolution_status": "unresolved",
        "confidence": 0.85,
        "detected_at": "2025-01-06T12:00:00.000Z"
      }
    ]
  },
  "meta": {...}
}
```

---

## Resolve Contradiction

Mark a contradiction as resolved.

```
POST https://api.penfield.app/api/v2/memories/contradictions/{contradiction_id}/resolve
```

### Status

This endpoint is now **AVAILABLE**. Use it to resolve reported contradictions.

### Request Body

```json
{
  "resolution": "choose_memory_1",
  "new_content": "The corrected information",
  "reason": "Target memory was more accurate"
}
```

### Resolution Options

| Resolution | Description |
|------------|-------------|
| `create_new` | Create a new memory with corrected information (requires `new_content`) |
| `choose_memory_1` | Accept source memory as correct |
| `choose_memory_2` | Accept target memory as correct |
| `keep_both` | Keep both memories (mark as contextual) |

### Resolution Status

| Status | Description |
|--------|-------------|
| `unresolved` | Contradiction not yet addressed |
| `resolved` | Contradiction resolved with new information |
| `accepted` | One version accepted as correct |

---

## Evolve Memory

Create a new version of a memory.

```
POST https://api.penfield.app/api/v2/memories/{memory_id}/evolve
```

### Request Body

```json
{
  "new_content": "Updated and corrected content...",
  "evolution_type": "correction",
  "reason": "Fixed factual error based on new information"
}
```

### Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `new_content` | string | Yes | New memory content |
| `evolution_type` | string | Yes | Type of evolution |
| `reason` | string | No | Explanation for change |

### Response

```json
{
  "status": "success",
  "data": {
    "original_memory_id": "550e8400-e29b-41d4-a716-446655440000",
    "new_memory_id": "new-version-uuid",
    "evolution_id": "evolution-uuid",
    "evolution_type": "correction",
    "version": 2,
    "created_at": "2025-01-06T12:00:00.000Z"
  },
  "meta": {...}
}
```

### Notes

- Creates a new memory linked to the original
- Original memory's `lifecycle_state` is updated
- Relationship created between versions
- Use `include_inactive=true` in list to see old versions
