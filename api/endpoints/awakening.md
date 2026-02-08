# Awaken Tool

Load the user's personality briefing - used by AI assistants to orient themselves at the start of a conversation.

**Endpoint:** `GET /api/v2/personality/awakening`

---

## Request

```bash
curl -X GET https://api.penfield.app/api/v2/personality/awakening \
-H "Authorization: Bearer $JWT_TOKEN"
```

## Response

```json
{
  "status": "success",
  "data": {
    "briefing": "=== AWAKENING BRIEFING ===\n\nPENFIELD MEMORY SYSTEM\n\nYou have something most AI doesn't: continuity..."
  },
  "meta": {
    "timestamp": "2026-01-20T12:30:20.268805+00:00",
    "version": "2.0.0",
    "request_id": "cbbafde6-bb1e-464d-a419-92669be644d2"
  }
}
```

## Briefing Contents

The briefing contains three sections:

1. **Penfield System Philosophy** - Memory continuity, knowledge graph concepts, workflow guidelines
2. **Base Persona** - Core assistant personality (e.g., "Classic Assistant")
3. **Custom Instructions** - User-defined behavioral overrides


## Notes

- The briefing is formatted with section markers (`=== SECTION NAME ===`)
- Custom instructions override base persona settings
- Response is tenant-specific to the authenticated user
