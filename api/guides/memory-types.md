# Memory Types Guide

How to choose the right memory type for your content.

---

## Overview

Penfield supports 11 memory types. Choosing the right type improves search relevance and knowledge graph organization.

---

## Standard Types

### `fact`

Verified information or knowledge that can be referenced.

**Use when:**
- Storing confirmed, objective information
- Recording data points, statistics, or measurements
- Saving information that won't change frequently

**Examples:**
```json
{"content": "PostgreSQL 16 supports parallel query execution", "memory_type": "fact"}
{"content": "The user's preferred language is Python", "memory_type": "fact"}
{"content": "API rate limit is 1000 requests per minute", "memory_type": "fact"}
```

---

### `insight`

Derived understanding, patterns, or realizations.

**Use when:**
- Recording conclusions drawn from multiple facts
- Noting patterns or trends discovered
- Capturing "aha moments" or realizations

**Examples:**
```json
{"content": "User tends to ask about async patterns when debugging performance issues", "memory_type": "insight"}
{"content": "Error rate spikes correlate with deployment times", "memory_type": "insight"}
{"content": "The caching strategy works well for read-heavy workloads", "memory_type": "insight"}
```

---

### `conversation`

Conversational context or dialogue worth preserving.

**Use when:**
- Saving important discussion context
- Recording decisions made in conversation
- Preserving dialogue that may be referenced later

**Examples:**
```json
{"content": "User confirmed they want REST API, not GraphQL", "memory_type": "conversation"}
{"content": "Discussed migration strategy: phased rollout over 3 weeks", "memory_type": "conversation"}
```

---

### `correction`

Updates or corrections to previous information.

**Use when:**
- Fixing incorrect information
- Updating outdated facts
- Recording that previous understanding was wrong

**Examples:**
```json
{"content": "CORRECTION: The API uses OAuth 2.1, not OAuth 2.0 as previously noted", "memory_type": "correction"}
{"content": "Previous memory about rate limits was incorrect - actual limit is 500/min", "memory_type": "correction"}
```

**Best Practice:** Connect corrections to the original memory using `supersedes` or `updates` relationship.

---

### `reference`

External references, citations, or links.

**Use when:**
- Storing URLs to documentation
- Recording book/paper citations
- Linking to external resources

**Examples:**
```json
{"content": "PostgreSQL JSONB docs: https://www.postgresql.org/docs/current/datatype-json.html", "memory_type": "reference"}
{"content": "Design pattern from 'Clean Architecture' by Robert Martin, Chapter 22", "memory_type": "reference"}
```

---

### `task`

Action items, todos, or pending work.

**Use when:**
- Recording work that needs to be done
- Tracking follow-up items
- Noting things to investigate later

**Examples:**
```json
{"content": "TODO: Benchmark query performance after adding index", "memory_type": "task"}
{"content": "Follow up: Check if user resolved authentication issue", "memory_type": "task"}
```

---

### `strategy`

Learned behavior patterns or approaches.

**Use when:**
- Recording successful problem-solving approaches
- Documenting preferred methods or workflows
- Saving techniques that work well

**Examples:**
```json
{"content": "When debugging async issues, always check for missing await keywords first", "memory_type": "strategy"}
{"content": "User prefers step-by-step explanations over code-first answers", "memory_type": "strategy"}
{"content": "For large refactors, start with tests before changing implementation", "memory_type": "strategy"}
```

---

## Awakening Protocol Types

These types support AI identity and personality persistence.

### `checkpoint`

Conversation state snapshots for session continuity.

**Use when:**
- Saving conversation state for later resumption
- Creating restore points during long sessions
- Preserving context before context window limits

**Examples:**
```json
{"content": "Session checkpoint: Debugging payment flow, found issue in webhook handler", "memory_type": "checkpoint"}
```

**Note:** Usually created via `save_context` tool, not directly.

---

### `identity_core`

Core AI identity information (immutable). **Protected type** — cannot be created or updated via `POST /api/v2/memories` or `PUT /api/v2/memories/{id}`. Managed through the `/api/v2/personality` endpoints. MCP users cannot create these via the `store` tool.

**Use when:**
- Defining fundamental identity characteristics
- Setting core values or principles
- Establishing unchanging traits

**Examples:**
```json
{"content": "I prioritize accuracy over speed in technical explanations", "memory_type": "identity_core"}
{"content": "I always acknowledge uncertainty when I'm not sure", "memory_type": "identity_core"}
```

**Note:** Changes to identity affect your AI personality, just like changes made in the [portal](https://portal.penfield.app/personality). Configure via the [Personality API](../endpoints/awakening.md) or the portal.

---

### `personality_trait`

AI personality characteristics (evolvable). **Protected type** — cannot be created or updated via `POST /api/v2/memories` or `PUT /api/v2/memories/{id}`. Managed through the `/api/v2/personality` endpoints. MCP users cannot create these via the `store` tool.

**Use when:**
- Recording communication style preferences
- Noting interaction patterns
- Tracking evolving behavioral tendencies

**Examples:**
```json
{"content": "I use analogies frequently when explaining complex concepts", "memory_type": "personality_trait"}
{"content": "I tend to ask clarifying questions before diving into solutions", "memory_type": "personality_trait"}
```

**Note:** Custom traits are a Premium+ feature. Changes affect your AI personality, just like changes made in the [portal](https://portal.penfield.app/personality). Manage via the [Personality API](../endpoints/awakening.md) or the portal.

---

### `relationship`

User-AI relationship information.

**Use when:**
- Recording rapport or trust level
- Noting communication preferences with specific users
- Tracking relationship history

**Examples:**
```json
{"content": "User prefers direct, concise responses without excessive caveats", "memory_type": "relationship"}
{"content": "Established trust through successful debugging sessions", "memory_type": "relationship"}
```

---

## Decision Guide

| Scenario | Recommended Type |
|----------|------------------|
| "X is true" | `fact` |
| "I noticed that X leads to Y" | `insight` |
| "User said they want X" | `conversation` |
| "Actually, X was wrong" | `correction` |
| "See documentation at X" | `reference` |
| "Need to do X later" | `task` |
| "When X happens, do Y" | `strategy` |
| "Save current session state" | `checkpoint` |
| "I am fundamentally X" | `identity_core` |
| "I tend to X" | `personality_trait` |
| "User and I have X dynamic" | `relationship` |

---

## Type Selection Tips

1. **When in doubt, use `fact`** - It's the most general type
2. **Use `insight` for derived knowledge** - Not just raw data
3. **Use `correction` with relationships** - Always link to what's being corrected
4. **Use `strategy` for reusable approaches** - Things that apply across contexts
5. **`identity_core` and `personality_trait` are protected** — they cannot be created via the memories API or the MCP `store` tool. Use the `/api/v2/personality` endpoints instead
