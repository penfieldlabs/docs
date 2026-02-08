# Relationship Types Guide

How to connect memories to build knowledge graphs.

---

## Quick Reference

| Category | Types | Use For |
|----------|-------|---------|
| Knowledge Evolution | `supersedes`, `updates`, `evolution_of` | Tracking changes over time |
| Evidence & Support | `supports`, `contradicts`, `disputes` | Arguments and evidence |
| Hierarchy | `parent_of`, `child_of`, `sibling_of`, `composed_of`, `part_of` | Organization |
| Causation | `causes`, `influenced_by`, `prerequisite_for` | Cause and effect |
| Implementation | `implements`, `documents`, `example_of`, `tests` | Code and docs |
| Conversation | `responds_to`, `references`, `inspired_by` | Attribution |
| Sequence | `follows`, `precedes` | Temporal order |
| Dependencies | `depends_on` | Requirements |

---

## Overview

Penfield supports 24 relationship types organized into 7 categories. Relationships are directional: `from_id` → `to_id`.

---

## Knowledge Evolution

Track how information changes over time.

### `supersedes`

Completely replaces older information.

```json
{
  "from_id": "new-memory-uuid",
  "to_id": "old-memory-uuid",
  "relationship_type": "supersedes"
}
```

**Example:** "API uses OAuth 2.1" supersedes "API uses OAuth 2.0"

---

### `updates`

Partially updates existing information.

```json
{
  "from_id": "update-memory-uuid",
  "to_id": "original-memory-uuid",
  "relationship_type": "updates"
}
```

**Example:** "Rate limit increased to 1000/min" updates "Rate limit is 500/min"

---

### `evolution_of`

Natural development or evolution from earlier concept.

```json
{
  "from_id": "evolved-memory-uuid",
  "to_id": "original-memory-uuid",
  "relationship_type": "evolution_of"
}
```

**Example:** "Microservices architecture" evolution_of "Monolithic design"

---

## Evidence & Support

Express agreement or disagreement between memories.

### `supports`

Provides evidence or support for a claim.

```json
{
  "from_id": "evidence-uuid",
  "to_id": "claim-uuid",
  "relationship_type": "supports"
}
```

**Example:** "Benchmark shows 50% speedup" supports "Caching improves performance"

---

### `contradicts`

Conflicts with or contradicts existing information.

```json
{
  "from_id": "new-finding-uuid",
  "to_id": "contradicted-uuid",
  "relationship_type": "contradicts"
}
```

**Example:** "Tests show memory leak" contradicts "Memory management is stable"

---

### `disputes`

Disagrees with or challenges (weaker than contradicts).

```json
{
  "from_id": "disputing-uuid",
  "to_id": "disputed-uuid",
  "relationship_type": "disputes"
}
```

**Example:** "Some users report issues" disputes "Feature works perfectly"

---

## Hierarchy & Structure

Organize memories into hierarchies.

### `parent_of`

Contains or encompasses (broader concept).

```json
{
  "from_id": "broader-uuid",
  "to_id": "narrower-uuid",
  "relationship_type": "parent_of"
}
```

**Example:** "Authentication system" parent_of "OAuth implementation"

---

### `child_of`

Is subset or part of (narrower concept).

```json
{
  "from_id": "narrower-uuid",
  "to_id": "broader-uuid",
  "relationship_type": "child_of"
}
```

**Example:** "JWT validation" child_of "Authentication system"

---

### `sibling_of`

At same level, parallel to (peer concepts).

```json
{
  "from_id": "concept-a-uuid",
  "to_id": "concept-b-uuid",
  "relationship_type": "sibling_of"
}
```

**Example:** "REST API" sibling_of "GraphQL API"

---

### `composed_of`

Contains or is made of (composition).

```json
{
  "from_id": "whole-uuid",
  "to_id": "part-uuid",
  "relationship_type": "composed_of"
}
```

**Example:** "Search system" composed_of "BM25 module"

---

### `part_of`

Belongs to larger structure.

```json
{
  "from_id": "part-uuid",
  "to_id": "whole-uuid",
  "relationship_type": "part_of"
}
```

**Example:** "Vector index" part_of "Search system"

---

## Causation

Express cause and effect.

### `causes`

Directly leads to or causes.

```json
{
  "from_id": "cause-uuid",
  "to_id": "effect-uuid",
  "relationship_type": "causes"
}
```

**Example:** "Missing index" causes "Slow query performance"

---

### `influenced_by`

Was shaped or affected by.

```json
{
  "from_id": "influenced-uuid",
  "to_id": "influencer-uuid",
  "relationship_type": "influenced_by"
}
```

**Example:** "API design" influenced_by "REST best practices"

---

### `prerequisite_for`

Required before or necessary for.

```json
{
  "from_id": "prerequisite-uuid",
  "to_id": "dependent-uuid",
  "relationship_type": "prerequisite_for"
}
```

**Example:** "Database migration" prerequisite_for "New feature deployment"

---

## Implementation & Testing

Connect concepts to their implementations.

### `implements`

Applies or implements a concept.

```json
{
  "from_id": "implementation-uuid",
  "to_id": "concept-uuid",
  "relationship_type": "implements"
}
```

**Example:** "PaymentService class" implements "Payment processing strategy"

---

### `documents`

Describes or documents.

```json
{
  "from_id": "documentation-uuid",
  "to_id": "documented-uuid",
  "relationship_type": "documents"
}
```

**Example:** "API reference" documents "REST endpoints"

---

### `example_of`

Demonstrates or exemplifies.

```json
{
  "from_id": "example-uuid",
  "to_id": "concept-uuid",
  "relationship_type": "example_of"
}
```

**Example:** "User login flow" example_of "Authentication pattern"

---

### `tests`

Verifies or validates.

```json
{
  "from_id": "test-uuid",
  "to_id": "tested-uuid",
  "relationship_type": "tests"
}
```

**Example:** "Integration test suite" tests "API endpoints"

---

## Conversation & Attribution

Track dialogue and sources.

### `responds_to`

Response to previous statement.

```json
{
  "from_id": "response-uuid",
  "to_id": "question-uuid",
  "relationship_type": "responds_to"
}
```

**Example:** "Solution explanation" responds_to "User question about caching"

---

### `references`

Refers to or cites.

```json
{
  "from_id": "citing-uuid",
  "to_id": "cited-uuid",
  "relationship_type": "references"
}
```

**Example:** "Implementation note" references "Architecture decision record"

---

### `inspired_by`

Inspired or motivated by.

```json
{
  "from_id": "inspired-uuid",
  "to_id": "inspiration-uuid",
  "relationship_type": "inspired_by"
}
```

**Example:** "New caching approach" inspired_by "Redis documentation"

---

## Sequence & Flow

Express temporal ordering.

### `follows`

Comes after in sequence.

```json
{
  "from_id": "later-uuid",
  "to_id": "earlier-uuid",
  "relationship_type": "follows"
}
```

**Example:** "Deploy to production" follows "Run integration tests"

---

### `precedes`

Comes before in sequence.

```json
{
  "from_id": "earlier-uuid",
  "to_id": "later-uuid",
  "relationship_type": "precedes"
}
```

**Example:** "Code review" precedes "Merge to main"

---

## Dependencies

Express dependencies between concepts.

### `depends_on`

Requires or depends upon.

```json
{
  "from_id": "dependent-uuid",
  "to_id": "dependency-uuid",
  "relationship_type": "depends_on"
}
```

**Example:** "Search feature" depends_on "Embedding service"

---

## Best Practices

1. **Use specific types** - `supports` is better than a generic link
2. **Direction matters** - Think about which memory is the subject
3. **Combine with strength** - Use `strength` parameter (0-1) to indicate confidence
4. **Build chains** - A → B → C creates traversable paths
5. **Connect corrections** - Always link `correction` memories to originals
