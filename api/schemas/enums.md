# Enum Values

Reference for all enumerated values used in the API.

---

## Memory Types

Classification of memory content.

| Value | Description |
|-------|-------------|
| `fact` | Verified information or knowledge |
| `insight` | Derived understanding, pattern, or realization |
| `conversation` | Conversational context or dialogue |
| `correction` | Update or correction to previous information |
| `reference` | External reference, citation, or link |
| `task` | Action item, todo, or pending work |
| `checkpoint` | Conversation state snapshot |
| `identity_core` | Core AI identity information (immutable) |
| `personality_trait` | AI personality characteristics (evolvable) |
| `relationship` | User-AI relationship information |
| `strategy` | Learned behavior patterns or approaches |

### Usage

```json
{
  "memory_type": "fact"
}
```

---

## Source Types

Origin of memory creation.

| Value | Description |
|-------|-------------|
| `direct_input` | Manually entered by user or AI |
| `document_upload` | Extracted from uploaded document |
| `web_scrape` | Scraped from web page |
| `api_import` | Imported via API from external source |
| `conversation` | Captured from conversation |
| `reflection` | Generated from analysis/reflection |
| `checkpoint` | Created during checkpoint operation |
| `checkpoint_recall` | Restored from checkpoint |

### Usage

```json
{
  "source_type": "direct_input"
}
```

---

## Relationship Types

Types of connections between memories. **24 types total.**

### Knowledge Evolution

| Value | Description |
|-------|-------------|
| `supersedes` | Completely replaces older information |
| `updates` | Partially updates existing information |
| `evolution_of` | Natural evolution or development from |

### Evidence & Support

| Value | Description |
|-------|-------------|
| `supports` | Provides evidence or support for |
| `contradicts` | Conflicts with or contradicts |
| `disputes` | Disagrees with or challenges |

### Hierarchy & Structure

| Value | Description |
|-------|-------------|
| `parent_of` | Contains or encompasses |
| `child_of` | Is subset or part of |
| `sibling_of` | At same level, parallel to |
| `composed_of` | Contains or is made of |
| `part_of` | Belongs to larger structure |

### Causation

| Value | Description |
|-------|-------------|
| `causes` | Directly leads to or causes |
| `influenced_by` | Was shaped or affected by |
| `prerequisite_for` | Required before or necessary for |

### Implementation & Testing

| Value | Description |
|-------|-------------|
| `implements` | Applies or implements concept |
| `documents` | Describes or documents |
| `tests` | Verifies or validates |
| `example_of` | Demonstrates or exemplifies |

### Conversation & Attribution

| Value | Description |
|-------|-------------|
| `responds_to` | Response to previous statement |
| `references` | Refers to or cites |
| `inspired_by` | Inspired or motivated by |

### Sequence & Flow

| Value | Description |
|-------|-------------|
| `follows` | Comes after in sequence |
| `precedes` | Comes before in sequence |

### Dependencies

| Value | Description |
|-------|-------------|
| `depends_on` | Requires or depends upon |

### Usage

```json
{
  "relationship_type": "supports"
}
```

---

## Direction Types

How relationships are traversed.

| Value | Description |
|-------|-------------|
| `DIRECTED` | One-way relationship (A → B) |
| `BIDIRECTIONAL` | Single relationship, queryable both ways |
| `INVERSE_PAIRED` | Two relationships with inverse types |

### Usage

```json
{
  "direction_type": "DIRECTED"
}
```

---

## Embedding Status

Status of embedding generation.

| Value | Description |
|-------|-------------|
| `pending` | Embedding not yet generated |
| `completed` | Embedding successfully generated |
| `failed` | Embedding generation failed |

---

## Document Status

Document processing status.

| Value | Description |
|-------|-------------|
| `uploading` | File is being uploaded |
| `processing` | Document is being chunked |
| `active` | Ready for search |
| `completed` | All processing complete |
| `failed` | Processing failed |

---

## Evolution Types

Types of memory evolution.

| Value | Description |
|-------|-------------|
| `correction` | Fixed an error or inaccuracy |
| `update` | Added new information |
| `expansion` | Expanded with more detail |
| `contradiction` | Conflicting information found |
| `deprecation` | Marked as outdated |

### Usage

```json
{
  "evolution_type": "correction"
}
```

---

## Lifecycle States

Memory lifecycle states.

| Value | Description |
|-------|-------------|
| `active` | Current, active memory |
| `superseded` | Replaced by newer version |
| `corrected` | Corrected by newer version |
| `contradicted` | Contradicted by other information |
| `archived` | Archived, no longer active |

---

## Contradiction Types

Types of contradictions between memories.

| Value | Description |
|-------|-------------|
| `direct` | Completely contradictory statements |
| `partial` | Partially conflicting information |
| `contextual` | Contradictory in specific contexts |

---

## Resolution Status

Contradiction resolution status.

| Value | Description |
|-------|-------------|
| `unresolved` | Not yet addressed |
| `resolved` | Resolved with new information |
| `accepted` | One version accepted as correct |

---

## Traversal Directions

Graph traversal directions.

| Value | Description |
|-------|-------------|
| `OUTBOUND` | Follow from source to target |
| `INBOUND` | Follow from target to source |
| `ANY` | Follow in either direction |

### Usage

```json
{
  "direction": "OUTBOUND"
}
```

---

## Time Windows

Relative time periods for analysis.

| Value | Description |
|-------|-------------|
| `recent` | Last 50 memories |
| `today` | Today's memories |
| `yesterday` | Yesterday's memories |
| `week` | Last 7 days |
| `month` | Last 30 days |
| `quarter` | Last 90 days |
| `year` | Last 365 days |

### Usage

```json
{
  "time_range": "week"
}
```

---

## Sort Orders

Sort direction indicators.

| Prefix | Direction |
|--------|-----------|
| (none) | Ascending |
| `-` | Descending |

### Examples

| Value | Description |
|-------|-------------|
| `created_at` | Oldest first |
| `-created_at` | Newest first (default) |
| `importance` | Lowest first |
| `-importance` | Highest first |

### Usage

```json
{
  "sort": "-created_at"
}
```
