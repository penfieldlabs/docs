# Common Patterns Guide

Practical patterns for using Penfield effectively.

---

## Session Handoffs

### Pattern: Save and Restore Investigation Context

When ending a session:

```json
POST /api/v2/memories
{
  "content": "{\"checkpoint_name\":\"Auth Bug Investigation\",\"description\":\"[Session] Found root cause: JWT scope missing...\",\"memory_ids\":[\"uuid-1\",\"uuid-2\"]}",
  "memory_type": "checkpoint",
  "importance": 0.9,
  "tags": ["context", "checkpoint", "Auth Bug Investigation"]
}
```

When resuming:

```json
GET /api/v2/memories?memory_type=checkpoint&per_page=100
```

Then find the checkpoint by name and fetch referenced memories.

### Pattern: Structured Handoff Description

Include these sections in checkpoint descriptions:

```
[Session Name - Date]

SITUATION: What you were investigating
PROGRESS: What you discovered (with memory references)
HYPOTHESIS: Your current theory
EVIDENCE: What supports/contradicts it
BLOCKED ON: What's preventing progress
NEXT STEPS: What the next agent should do
KEY MEMORIES: UUIDs or descriptions of critical memories
```

---

## Investigation Threads

### Pattern: Track an Investigation

1. **Create a root memory:**
```json
{
  "content": "[Investigation: API Latency] Started investigating slow response times on /search endpoint. Initial hypothesis: database queries taking too long.",
  "memory_type": "task",
  "importance": 0.9,
  "tags": ["investigation", "api-latency"]
}
```

2. **Add findings as children:**
```json
{
  "content": "[API Latency] FACT: Database query averages 45ms, within acceptable range. This contradicts the database hypothesis.",
  "memory_type": "fact",
  "importance": 0.8,
  "tags": ["investigation", "api-latency"]
}
```

3. **Connect with relationships:**
```json
POST /api/v2/relationships
{
  "from_id": "finding-uuid",
  "to_id": "root-uuid",
  "relationship_type": "child_of",
  "strength": 0.9
}
```

4. **Track corrections:**
```json
{
  "content": "[API Latency] CORRECTION: Issue was not database but serialization. JSON encoding taking 200ms for large responses.",
  "memory_type": "correction",
  "importance": 0.9
}
```

Then connect with `supersedes`:
```json
{
  "from_id": "correction-uuid",
  "to_id": "hypothesis-uuid",
  "relationship_type": "supersedes"
}
```

---

## Evidence Chains

### Pattern: Build Argument Structures

For claims that need evidence:

```
Claim (hypothesis)
├── Evidence 1 (supports)
├── Evidence 2 (supports)
├── Counter-evidence (contradicts)
└── Resolution (supersedes original claim)
```

**Example:**

```json
// Claim
{
  "content": "[Hypothesis] The memory leak is caused by unclosed database connections",
  "memory_type": "insight",
  "tags": ["memory-leak", "hypothesis"]
}

// Evidence
{
  "content": "[Evidence] Connection pool shows 47/50 connections in use after 10 minutes",
  "memory_type": "fact",
  "tags": ["memory-leak", "evidence"]
}

// Relationship
{
  "from_id": "evidence-uuid",
  "to_id": "claim-uuid",
  "relationship_type": "supports",
  "strength": 0.9
}
```

---

## Learning and Research

### Pattern: Study Session Notes

Structure learning notes with connections:

```json
// Main concept
{
  "content": "[ML Study] Transformers use self-attention mechanisms to weigh relationships between all tokens in a sequence",
  "memory_type": "fact",
  "importance": 0.8,
  "tags": ["ml", "transformers", "study"]
}

// Related insight
{
  "content": "[ML Study] INSIGHT: Self-attention is like a soft lookup table where queries find relevant keys",
  "memory_type": "insight",
  "tags": ["ml", "transformers", "study"]
}

// Connect
{
  "from_id": "insight-uuid",
  "to_id": "concept-uuid",
  "relationship_type": "example_of"
}
```

### Pattern: Question and Answer Tracking

```json
// Question
{
  "content": "[Question] How does the embedding service handle rate limiting?",
  "memory_type": "task",
  "tags": ["question", "embeddings"]
}

// Answer (later)
{
  "content": "[Answer] Embedding service uses token bucket algorithm with 100 req/min burst and 20 req/min sustained",
  "memory_type": "fact",
  "tags": ["answer", "embeddings"]
}

// Connect
{
  "from_id": "answer-uuid",
  "to_id": "question-uuid",
  "relationship_type": "responds_to"
}
```

---

## Project Documentation

### Pattern: Living Architecture Docs

Store architecture decisions and keep them connected:

```json
// Decision
{
  "content": "[ADR-001] Decision: Use PostgreSQL for memory storage. Rationale: pgvector support for embeddings, mature ecosystem, ACID compliance.",
  "memory_type": "reference",
  "importance": 0.9,
  "tags": ["adr", "architecture", "database"]
}

// Implementation
{
  "content": "[Implementation] PostgreSQL deployed with pgvector 0.6.0. Using HNSW indexes for similarity search.",
  "memory_type": "fact",
  "tags": ["implementation", "database"]
}

// Connect
{
  "from_id": "implementation-uuid",
  "to_id": "decision-uuid",
  "relationship_type": "implements"
}
```

---

## Multi-Agent Workflows

### Pattern: Agent Handoff Protocol

**Agent A ending session:**
```json
{
  "content": "{\"checkpoint_name\":\"task-handoff\",\"description\":\"Completed steps 1-3. Agent B should continue with step 4. Key finding: [uuid-123] shows the config issue.\",\"memory_ids\":[\"uuid-123\",\"uuid-456\"]}",
  "memory_type": "checkpoint",
  "importance": 0.9,
  "tags": ["handoff", "task-handoff"]
}
```

**Agent B resuming:**
1. Search for handoff: `GET /api/v2/memories?memory_type=checkpoint`
2. Parse checkpoint content
3. Fetch referenced memories
4. Continue work

### Pattern: Shared Knowledge Base

Multiple agents contributing to same project:

```json
// Tag with project and agent
{
  "content": "[Project: Dashboard] [Agent: Code Review] Found potential XSS vulnerability in user input handling",
  "memory_type": "fact",
  "tags": ["dashboard", "security", "code-review-agent"]
}
```

Other agents can search:
```json
{
  "query": "dashboard security",
  "tags": ["dashboard"]
}
```

---

## Document Processing

### Pattern: Extract and Connect Insights from Documents

After uploading a document:

1. **Search for key content:**
```json
POST /api/v2/search/hybrid
{
  "query": "main conclusions",
  "source_type": "document",
  "limit": 5
}
```

2. **Create summary memory:**
```json
{
  "content": "[Document Summary: Research Paper] Key findings: 1) X improves by 20%, 2) Y has no effect. See document chunks [uuid-1, uuid-2] for details.",
  "memory_type": "insight",
  "tags": ["document-summary", "research"]
}
```

3. **Connect to document chunks:**
```json
{
  "from_id": "summary-uuid",
  "to_id": "chunk-uuid",
  "relationship_type": "references"
}
```

---

## Artifact Management

### Pattern: Organize Artifacts by Project

```
/project-a/
  /docs/
    architecture.md
    api-spec.md
  /diagrams/
    flow.svg
  /notes/
    meeting-2025-01-06.md
/project-b/
  ...
```

**Save:**
```json
POST /api/v2/artifacts
{
  "path": "/project-a/docs/architecture.md",
  "content": "# Architecture\n\n..."
}
```

**List:**
```json
GET /api/v2/artifacts/list?prefix=/project-a/docs/
```

### Pattern: Version Artifacts with Timestamps

```
/project/docs/architecture-2025-01-05.md
/project/docs/architecture-2025-01-06.md
```

Or use memories to track artifact history:
```json
{
  "content": "[Artifact Update] Updated /project/docs/architecture.md - Added section on caching layer",
  "memory_type": "fact",
  "tags": ["artifact-history", "project"]
}
```

---

## Error Recovery

### Pattern: Graceful Degradation

```python
def store_with_fallback(content, tags):
    try:
        # Try full operation
        return api.store(content=content, tags=tags, importance=0.8)
    except ValidationError as e:
        # Fall back to minimal
        return api.store(content=content[:5000])
    except RateLimitError:
        # Queue for later
        queue.add({"content": content, "tags": tags})
        return {"status": "queued"}
```

### Pattern: Idempotent Operations

Before creating, check if it exists:

```python
def store_or_update(content, identifier):
    # Search for existing
    results = api.search(query=identifier, limit=1)

    if results["items"]:
        # Update existing
        return api.update_memory(
            memory_id=results["items"][0]["id"],
            content=content
        )
    else:
        # Create new
        return api.store(content=content, tags=[identifier])
```
