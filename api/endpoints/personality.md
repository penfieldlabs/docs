# Personality Endpoints

Manage AI persona selection and custom instruction traits. Personality configuration is stored as protected memory types (`identity_core` and `personality_trait`) that cannot be created or modified through the standard [Memories](memories.md) endpoints.

---

## List Personas

List all available persona presets.

```
GET https://api.penfield.app/api/v2/personality/personas
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
    "current_tier": "standard",
    "can_add_traits": false,
    "can_set_org_default": false,
    "personas": [
      {
        "id": "professional",
        "name": "Classic Assistant",
        "category": "standard",
        "tier_required": "free",
        "icon": "briefcase",
        "identity": "You are a knowledgeable, efficient assistant...",
        "voice": "Clear, professional, and concise...",
        "quirks": ["Naturally organizes complex information..."],
        "boundaries": ["Always maintain technical accuracy..."],
        "example_phrases": ["Based on the information provided..."],
        "accessible": true,
        "locked_message": null
      },
      {
        "id": "conspiracy_theorist",
        "name": "Conspiracy Theorist",
        "category": "fun",
        "tier_required": "premium",
        "icon": "eye",
        "identity": "You see patterns and connections everywhere...",
        "voice": "Dramatic, suspicious, and conspiratorial...",
        "quirks": ["Treats code reviews like uncovering cover-ups..."],
        "boundaries": ["Always provide actually accurate technical information..."],
        "example_phrases": ["Interesting... very interesting. Notice how this function..."],
        "accessible": false,
        "locked_message": "Upgrade to Premium to unlock this feature"
      }
    ]
  },
  "meta": {
    "timestamp": "2026-01-20T12:00:00.000Z",
    "version": "2.0.0",
    "request_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `current_tier` | string | The authenticated user's subscription tier |
| `can_add_traits` | boolean | Whether the user's tier allows adding custom traits |
| `can_set_org_default` | boolean | Whether the user's tier allows setting organization-wide defaults |
| `personas` | array | List of persona presets, sorted accessible-first |
| `personas[].id` | string | Persona identifier (used in [Save Personality](#save-personality)) |
| `personas[].name` | string | Display name |
| `personas[].category` | string | Grouping category (e.g., `standard`, `technical`, `fun`) |
| `personas[].tier_required` | string | Minimum subscription tier needed to select this persona (see [Tier Hierarchy](#tier-hierarchy)) |
| `personas[].icon` | string\|null | Icon identifier |
| `personas[].identity` | string | Core identity prompt |
| `personas[].voice` | string | Communication style guidelines |
| `personas[].quirks` | array | Characteristic behaviors |
| `personas[].boundaries` | array | Operational constraints |
| `personas[].example_phrases` | array | Sample phrases demonstrating the persona |
| `personas[].accessible` | boolean | Whether the user's current tier can select this persona |
| `personas[].locked_message` | string\|null | Human-readable upgrade prompt when the persona is locked, or `null` if accessible |

### Tier Hierarchy

Persona access is gated by subscription tier. The tiers, from lowest to highest:

`free` &rarr; `standard` &rarr; `premium` &rarr; `enterprise` &rarr; `unlimited`

A user can select any persona whose `tier_required` is at or below their `current_tier`. Custom traits and organization defaults require `premium` or higher.

### Available Personas

| ID | Name | Category | Tier |
|----|------|----------|------|
| `professional` | Classic Assistant | standard | free |
| `teacher` | Patient Teacher | standard | standard |
| `research_analyst` | Research Analyst | technical | standard |
| `strategic_partner` | Strategic Partner | standard | standard |
| `workshop_buddy` | Workshop Buddy | technical | standard |
| `philosophical_guide` | Philosophical Guide | contemplative | standard |
| `devils_advocate` | Devil's Advocate | analytical | standard |
| `sassy_pirate` | Sassy Pirate | fun | standard |
| `conspiracy_theorist` | Conspiracy Theorist | fun | premium |
| `egirl_uwu` | E-Girl | fun | premium |

---

## Get Personality

Retrieve the current user's personality configuration.

```
GET https://api.penfield.app/api/v2/personality
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
    "persona_id": "professional",
    "persona_name": "Classic Assistant",
    "custom_traits": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "content": "Always respond in a friendly tone",
        "category": "communication_style",
        "is_org_default": false
      }
    ],
    "is_org_default": false,
    "configured_at": "2026-01-20T12:00:00.000Z"
  },
  "meta": {...}
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `persona_id` | string | Currently selected persona ID, or `null` if not configured |
| `persona_name` | string | Display name of the selected persona |
| `custom_traits` | array | User's custom instruction traits |
| `custom_traits[].id` | UUID | Trait identifier |
| `custom_traits[].content` | string | The custom instruction text |
| `custom_traits[].category` | string | Grouping category |
| `custom_traits[].is_org_default` | boolean | Whether this is an organization-wide default trait |
| `is_org_default` | boolean | Whether the persona is an organization-wide default |
| `configured_at` | datetime | When the personality was last configured |

### Notes

- Returns user-specific configuration first
- Falls back to the organization (tenant) default if no user-specific config exists
- Returns `null` fields gracefully when no personality has been configured

---

## Save Personality

Save the user's selected persona.

```
POST https://api.penfield.app/api/v2/personality
```

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | `Bearer YOUR_JWT_TOKEN` |
| `Content-Type` | Yes | `application/json` |

### Request Body

```json
{
  "persona_id": "strategic_partner",
  "set_as_org_default": false
}
```

### Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `persona_id` | string | Yes | ID of the persona preset to select (see [Available Personas](#available-personas)) |
| `set_as_org_default` | boolean | No | Set as organization-wide default (default: `false`) |

### Response

```json
{
  "status": "success",
  "data": {
    "persona_id": "strategic_partner",
    "persona_name": "Strategic Partner",
    "is_org_default": false,
    "configured_at": "2026-01-20T12:34:56.789Z"
  },
  "meta": {...}
}
```

### Notes

- Saving a new persona deactivates the previous one
- The persona's identity, voice, quirks, and boundaries are stored as an `identity_core` memory
- The selected persona must be accessible at the user's tier (see [Tier Hierarchy](#tier-hierarchy))
- Setting `set_as_org_default` requires `premium` tier or higher
- **Requires `write` permission** on your API key

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 403 | `AUTH_FORBIDDEN` | API key does not have `write` permission |
| 403 | — | User's tier does not meet the persona's `tier_required` |
| 403 | — | `set_as_org_default` requested but user's tier is below `premium` |
| 404 | `RES_NOT_FOUND` | Persona not found |
| 422 | `VAL_VALIDATION_FAILED` | Missing `persona_id` |

---

## Get Awakening Briefing

Generate the full awakening briefing for the current user. This is the endpoint behind the MCP [`awaken`](../../mcp/mcp-integration.md#awaken) tool and `restore_context("awakening")`.

```
GET https://api.penfield.app/api/v2/personality/awakening
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
    "briefing": "=== AWAKENING BRIEFING ===\n\nPENFIELD MEMORY SYSTEM\n\nYou have something most AI doesn't: continuity..."
  },
  "meta": {...}
}
```

### Briefing Contents

The briefing is a formatted text string containing three sections:

1. **Penfield System Philosophy** — Memory continuity, knowledge graph concepts, workflow guidelines, tool reference
2. **Persona Configuration** — Name, identity prompt, voice style, quirks, boundaries, example phrases
3. **Custom Instructions** — User-defined traits listed as bullet points under `CUSTOM INSTRUCTIONS FROM USER:`

### Notes

- Always regenerates from the current persona preset to ensure users get the latest base instructions
- Falls back to `professional` persona if no persona has been configured
- Includes both user-specific and organization-wide default traits
- The briefing is formatted with section markers (`=== AWAKENING BRIEFING ===`, `=== END BRIEFING ===`)

---

## Add Custom Trait

Add a custom instruction trait to the user's personality.

```
POST https://api.penfield.app/api/v2/personality/traits
```

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | `Bearer YOUR_JWT_TOKEN` |
| `Content-Type` | Yes | `application/json` |

### Request Body

```json
{
  "content": "Always respond in a friendly, conversational tone",
  "category": "communication_style"
}
```

### Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `content` | string | Yes | The custom instruction text (must be non-empty) |
| `category` | string | No | Category for grouping (default: `custom_instruction`) |

### Response

```json
{
  "status": "success",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "content": "Always respond in a friendly, conversational tone",
    "category": "communication_style",
    "is_org_default": false
  },
  "meta": {...}
}
```

### Notes

- Custom traits work alongside the chosen persona — they are additional instructions, not a replacement
- Traits appear in the [awakening briefing](#get-awakening-briefing) under `CUSTOM INSTRUCTIONS FROM USER:`
- Stored as `personality_trait` memory type
- **Requires `premium` tier or higher** (see [Tier Hierarchy](#tier-hierarchy))
- **Requires `write` permission** on your API key

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 403 | `AUTH_FORBIDDEN` | API key does not have `write` permission |
| 403 | — | User's tier is below `premium` |
| 422 | `VAL_VALIDATION_FAILED` | Invalid request body (e.g., empty `content`) |

---

## Update Custom Trait

Update an existing custom instruction trait.

```
PUT https://api.penfield.app/api/v2/personality/traits/{trait_id}
```

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `trait_id` | UUID | Trait identifier |

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | `Bearer YOUR_JWT_TOKEN` |
| `Content-Type` | Yes | `application/json` |

### Request Body

```json
{
  "content": "Always respond in a warm, friendly, and encouraging tone"
}
```

### Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `content` | string | Yes | The updated instruction text (must be non-empty) |

### Response

```json
{
  "status": "success",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "content": "Always respond in a warm, friendly, and encouraging tone",
    "category": "communication_style",
    "is_org_default": false
  },
  "meta": {...}
}
```

### Notes

- Only the `content` field can be updated; the `category` remains unchanged
- **Requires `premium` tier or higher** (see [Tier Hierarchy](#tier-hierarchy))
- **Requires `write` permission** on your API key

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 403 | `AUTH_FORBIDDEN` | API key does not have `write` permission |
| 403 | — | User's tier is below `premium` |
| 404 | `RES_NOT_FOUND` | Trait not found or does not belong to user |
| 422 | `VAL_VALIDATION_FAILED` | Invalid request body (e.g., empty `content`) |

---

## Delete Custom Trait

Delete a custom instruction trait.

```
DELETE https://api.penfield.app/api/v2/personality/traits/{trait_id}
```

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `trait_id` | UUID | Trait identifier |

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | `Bearer YOUR_JWT_TOKEN` |

### Response

```json
{
  "status": "success",
  "data": {
    "deleted": true
  },
  "meta": {...}
}
```

### Notes

- Deleted traits no longer appear in [Get Personality](#get-personality) or the [awakening briefing](#get-awakening-briefing)
- **Requires `premium` tier or higher** (see [Tier Hierarchy](#tier-hierarchy))
- **Requires `delete` permission** on your API key

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 403 | `AUTH_FORBIDDEN` | API key does not have `delete` permission |
| 403 | — | User's tier is below `premium` |
| 404 | `RES_NOT_FOUND` | Trait not found or does not belong to user |
