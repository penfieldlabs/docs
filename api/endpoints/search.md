# Search Endpoints

Penfield provides powerful hybrid search combining text matching, semantic similarity, and knowledge graph traversal.

---

## Hybrid Search

Perform comprehensive search combining BM25, vector similarity, and graph expansion.

```
POST https://api.penfield.app/api/v2/search/hybrid
```

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | `Bearer YOUR_JWT_TOKEN` |
| `Content-Type` | Yes | `application/json` |

### Request Body

```json
{
  "query": "Python async programming best practices",
  "limit": 20,
  "bm25_weight": 0.4,
  "vector_weight": 0.4,
  "graph_weight": 0.2,
  "memory_types": ["fact", "insight"],
  "importance_threshold": 0.5,
  "enable_graph_expansion": true,
  "graph_max_depth": 2,
  "include_neighbors": 2
}
```

### Request Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `query` | string | **required** | Search query (1-4,000 characters) |
| `limit` | integer | `20` | Maximum results (1-100) |
| `bm25_weight` | float | `0.4` | BM25 text search weight (0-1) |
| `vector_weight` | float | `0.4` | Vector similarity weight (0-1) |
| `graph_weight` | float | `0.2` | Graph expansion weight (0-1) |
| `memory_types` | array | - | Filter by memory types |
| `source_types` | array | - | Filter by source types: `["direct_input"]`, `["document_upload"]`, etc. (See [SourceType values](#sourcetype-values)) |
| `importance_threshold` | float | - | Minimum importance (0-1) |
| `enable_graph_expansion` | boolean | `true` | Enable knowledge graph expansion |
| `graph_max_depth` | integer | `2` | Max graph traversal depth (1-5) |
| `graph_seed_limit` | integer | `10` | Top results for graph seeds (1-50) |
| `rrf_k` | integer | `60` | RRF (Reciprocal Rank Fusion) constant (1-100). RRF is the algorithm that combines BM25, vector, and graph scores into a single ranking. Higher values produce more balanced fusion across search types; lower values favor top-ranked results more aggressively. |
| `min_score_threshold` | float | - | Minimum final score |
| `include_neighbors` | integer | `2` | Neighboring chunks for document results (0-20) |
| `tags` | string | - | Comma-separated tags to filter by (OR logic - memories with ANY of these tags). Example: `"python,async,programming"` |
| `start_date` | string | - | Filter memories created on or after this date (ISO 8601: `"2025-01-01"` or `"2025-01-01T00:00:00Z"`) |
| `end_date` | string | - | Filter memories created on or before this date (ISO 8601: `"2025-01-09"` or `"2025-01-09T23:59:59Z"`) |

### Weight Constraint

**Weights must sum to 1.0**: `bm25_weight + vector_weight + graph_weight = 1.0`

### SourceType Values

The `source_types` field accepts an array of source type strings:

| Value | Description |
|-------|-------------|
| `direct_input` | Memories created directly via API or MCP |
| `document_upload` | Content from uploaded documents |
| `web_scrape` | Content scraped from web sources |
| `api_import` | Imported via API integration |
| `conversation` | Captured from conversations |
| `reflection` | Generated through reflection/analysis |
| `checkpoint` | Created as session checkpoints |
| `checkpoint_recall` | Restored from checkpoints |

**Example:**
```json
{
  "query": "search text",
  "source_types": ["direct_input", "conversation"]
}
```

### Response

```json
{
  "status": "success",
  "data": {
    "items": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "content": "Python async programming uses asyncio...",
        "snippet": "Python async programming uses asyncio for concurrent...",
        "score": 0.95,
        "score_breakdown": {
          "final_score": 0.95,
          "bm25_score": 0.85,
          "vector_score": 0.92,
          "graph_score": 0.7,
          "rank_bm25": 1.0,
          "rank_vector": 2.0
        },
        "metadata": {
          "memory_type": "fact",
          "importance": 0.8,
          "highlight": null,
          "graph_context": {
            "relationships": [
              {
                "to_id": "...",
                "type": "supports",
                "strength": 0.8
              }
            ],
            "relationship_count": 3
          }
        },
        "source_type": "direct_input",
        "created_at": "2025-01-06T12:00:00.000Z",
        "content_with_context": "...",
        "chunk_index": 0,
        "total_chunks": 1
      }
    ],
    "query": "Python async programming best practices",
    "analyzed_query": {
      "original": "Python async programming best practices",
      "search_type": "hybrid",
      "weights": {
        "bm25": 0.4,
        "vector": 0.4,
        "graph": 0.2
      },
      "graph_expansion_enabled": true
    },
    "search_metadata": {
      "search_method": "hybrid",
      "results_count": 20,
      "stats": {
        "search_time_ms": 45,
        "bm25_time_ms": 12,
        "vector_time_ms": 15,
        "graph_time_ms": 10,
        "fusion_time_ms": 3,
        "bm25_count": 40,
        "vector_count": 40,
        "graph_count": 15,
        "final_count": 20
      },
      "knowledge_cloud": {
        "nodes": [...],
        "edges": [...],
        "stats": {
          "seed_count": 5,
          "cloud_count": 12,
          "edge_count": 15
        }
      }
    }
  },
  "meta": {...}
}
```

### Response Fields

| Field | Description |
|-------|-------------|
| `items[].id` | Memory UUID |
| `items[].content` | Full memory content |
| `items[].snippet` | Truncated content (~200 chars) |
| `items[].score` | Combined relevance score |
| `items[].score_breakdown` | Individual component scores |
| `items[].content_with_context` | Document chunks with neighbors (if applicable) |
| `items[].chunk_index` | Position in parent document (if from document) |
| `items[].total_chunks` | Total chunks in parent document |
| `search_metadata.stats` | Performance timing data |
| `search_metadata.knowledge_cloud` | Graph visualization data |

### Knowledge Cloud

When `enable_graph_expansion` is true, the response includes `knowledge_cloud` data for building graph visualizations. See the response example above for the structure.

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 422 | `VAL_VALIDATION_FAILED` | Weights don't sum to 1.0 |
| 422 | `VAL_FIELD_REQUIRED` | Missing query |

### Examples

**Basic search with filters:**
```bash
curl -X POST https://api.penfield.app/api/v2/search/hybrid \
-H "Authorization: Bearer $JWT_TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "query": "machine learning best practices",
  "limit": 10,
  "memory_types": ["fact", "insight"],
  "importance_threshold": 0.6
}'
```

**Search with tags and date range:**
```bash
curl -X POST https://api.penfield.app/api/v2/search/hybrid \
-H "Authorization: Bearer $JWT_TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "query": "API development",
  "limit": 20,
  "tags": "python,api,backend",
  "start_date": "2025-01-01",
  "end_date": "2025-01-31"
}'
```

**Note:** Tags use OR logic - returns memories with ANY of the specified tags.

---

## Vector Search

Perform pure vector similarity search (for advanced use cases).

```
POST https://api.penfield.app/api/v2/search/vector
```

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | `Bearer YOUR_JWT_TOKEN` |
| `Content-Type` | Yes | `application/json` |

### Request Body

```json
{
  "query": "semantic meaning of async programming",
  "limit": 20,
  "filters": {
    "memory_types": ["fact"],
    "min_importance": 0.5
  },
  "include_neighbors": 2,
  "tags": "python,async"
}
```

### Request Fields

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `query` | string | **required** | Search query (1-4,000 characters) |
| `limit` | integer | `20` | Maximum results (1-100) |
| `filters` | object | - | Search filters (see below) |
| `include_neighbors` | integer | `2` | Neighboring chunks for document results (0-20) |
| `tags` | string | - | Comma-separated tags (OR logic) |

### Filters Object

| Field | Type | Description |
|-------|------|-------------|
| `memory_types` | array | Filter by memory types |
| `min_importance` | float | Minimum importance (0-1) |
| `user_id` | UUID | Filter by user ID |

### Response

```json
{
  "status": "success",
  "data": {
    "items": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "content": "Python async programming uses asyncio...",
        "snippet": "Python async programming uses asyncio for concurrent...",
        "score": 0.95,
        "score_breakdown": {
          "vector_similarity": 0.95
        },
        "metadata": {
          "memory_type": "fact",
          "importance": 0.8,
          "confidence": 0.9,
          "source_type": "direct_input",
          "user_id": "user-uuid",
          "created_at": "2025-01-06T12:00:00.000Z",
          "updated_at": "2025-01-06T12:00:00.000Z"
        },
        "source_type": "direct_input",
        "created_at": "2025-01-06T12:00:00.000Z",
        "tags": ["python", "async"],
        "content_with_context": "...",
        "chunk_index": 0,
        "total_chunks": 1
      }
    ],
    "query": "semantic meaning of async programming",
    "analyzed_query": {
      "original": "semantic meaning of async programming",
      "embedding_generated": true,
      "filters_applied": true,
      "search_type": "vector_similarity",
      "include_neighbors": 2
    },
    "search_metadata": {
      "search_method": "vector_similarity",
      "results_count": 20,
      "max_results": 20,
      "include_neighbors": 2
    }
  },
  "meta": {...}
}
```

### Response Fields

| Field | Description |
|-------|-------------|
| `items[].id` | Memory UUID |
| `items[].content` | Full memory content |
| `items[].snippet` | Truncated content (~200 chars) |
| `items[].score` | Vector similarity score (0-1) |
| `items[].score_breakdown.vector_similarity` | Same as score (for consistency) |
| `items[].metadata` | Memory metadata (type, importance, etc.) |
| `items[].tags` | Array of tag names |
| `items[].content_with_context` | Document chunks with neighbors (if applicable) |
| `items[].chunk_index` | Position in parent document |
| `items[].total_chunks` | Total chunks in parent document |
| `analyzed_query.embedding_generated` | Whether query was embedded |
| `analyzed_query.filters_applied` | Whether filters were used |
| `analyzed_query.search_type` | Always "vector_similarity" |
| `search_metadata.search_method` | Always "vector_similarity" |
| `search_metadata.include_neighbors` | Number of neighbors included |

### Comparison: Hybrid vs Vector Search

Understanding when to use each search type:

| Feature | Hybrid Search | Vector Search |
|---------|---------------|---------------|
| **Search Methods** | BM25 + Vector + Graph | Vector only |
| **Use Case** | General search, best quality | Semantic similarity |
| **Latency** | Higher (3 methods + fusion) | Lower (1 method) |
| **score_breakdown** | `bm25_score`, `vector_score`, `graph_score`, `final_score`, `rank_vector` | `vector_similarity` only |
| **Graph Expansion** | Optional (enabled by default) | Not available |
| **Knowledge Cloud** | Included in response | Not included |
| **Timing Stats** | Detailed breakdown | Simple count |
| **Best For** | Finding relevant context | Finding similar concepts |

### score_breakdown Field Differences

**Hybrid Search:**
```json
{
  "final_score": 0.95,
  "bm25_score": 0.85,
  "vector_score": 0.92,
  "graph_score": 0.7,
  "rank_vector": 2.0
}
```

**Vector Search:**
```json
{
  "vector_similarity": 0.95
}
```

Vector search returns only the similarity score since it uses a single search method (no fusion needed).

### Examples

**Basic vector search:**
```bash
curl -X POST https://api.penfield.app/api/v2/search/vector \
-H "Authorization: Bearer $JWT_TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "query": "machine learning concepts",
  "limit": 10
}'
```

**Vector search with filters:**
```bash
curl -X POST https://api.penfield.app/api/v2/search/vector \
-d '{
  "query": "async patterns",
  "limit": 20,
  "filters": {
    "memory_types": ["fact", "insight"],
    "min_importance": 0.6
  },
  "tags": "python,async"
}'
```

### Notes

- Uses semantic similarity only (no BM25 or graph)
- Best for finding conceptually similar content
- Lower latency than hybrid search
- No RRF fusion needed (single ranking method)
- Returns simpler score_breakdown structure

---

## Search Stats

Get statistics about your searchable content. Returns counts of memories, documents, and relationships, along with embedding coverage.

```
GET https://api.penfield.app/api/v2/search/stats
```

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | `Bearer YOUR_JWT_TOKEN` |

### Response

```json
{
  "status": "success",
  "data": {
    "total_memories": 73,
    "total_documents": 1,
    "total_relationships": 54,
    "embedding_coverage": 0.92,
    "last_indexed": "2025-01-06T12:00:00.000Z"
  },
  "meta": {...}
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `total_memories` | integer | Total number of memories stored |
| `total_documents` | integer | Total number of uploaded documents |
| `total_relationships` | integer | Total number of relationships between memories |
| `embedding_coverage` | float | Fraction of memories with completed embeddings (0-1) |
| `last_indexed` | string | ISO 8601 timestamp of the most recently indexed memory |

### Example

```bash
curl -X GET https://api.penfield.app/api/v2/search/stats \
-H "Authorization: Bearer $JWT_TOKEN"
```

### Notes

- All counts are scoped to your tenant
- `embedding_coverage` indicates what percentage of your memories are searchable via vector similarity
- A coverage below 1.0 means some memories are still pending embedding or had embedding failures

---

## Search Tips

### Optimize for Speed

- Reduce `limit` to minimum needed
- Disable graph expansion for simple queries: `"enable_graph_expansion": false`
- Use filters to narrow search space
- Lower `graph_max_depth` (1-2 is usually sufficient)

### Optimize for Quality

- Use hybrid search (default) for best results
- Increase `graph_seed_limit` for more graph context
- Set appropriate `importance_threshold` to filter low-quality matches
- Use `memory_types` filter when you know what you're looking for

### Query Writing

- Be specific: "Python asyncio error handling" > "Python errors"
- Include context: "machine learning for image classification"
- Use natural language: The vector search understands semantics
