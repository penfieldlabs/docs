# Error Handling Guide

How to handle API errors gracefully.

---

## Error Response Structure

All errors follow this format:

```json
{
  "status": "error",
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message",
    "details": {},
    "field_errors": null
  },
  "meta": {
    "timestamp": "2026-01-06T12:00:00.390742+00:00",
    "version": "2.0.0",
    "request_id": "id-for-debugging"
  }
}
```

| Field | Description |
|-------|-------------|
| `error.code` | Machine-readable code. Branch on this together with the HTTP status |
| `error.message` | Human-readable reason |
| `error.details` | Extra context, or `null` |
| `error.field_errors` | On a 422: each invalid field mapped to an array of messages. Otherwise `null` |
| `meta.request_id` | Include this in support requests |

---

## Error Code Reference

### Authentication Errors (401)

| Code | Message | Cause | Solution |
|------|---------|-------|----------|
| `AUTH_UNAUTHORIZED` | `Invalid API key` | The API key sent to `/auth/token` was not accepted | Check the key (format `tm_..._ak_...`) and that it has not been revoked |
| `AUTH_UNAUTHORIZED` | `Invalid token` | The bearer token is not a valid access token. This is also what you get when an API key is sent where an access token is expected | Exchange the API key for a token first, or refresh |
| `AUTH_UNAUTHORIZED` | `No access token provided (expected Authorization header or cookie)` | Missing `Authorization` header | Add `Authorization: Bearer <token>` |
| `AUTH_TOKEN_INVALID` | `Invalid refresh token` | The refresh token sent to `/auth/refresh` was not accepted | Re-authenticate |

**Handling:**
```python
if response.status_code == 401:
    # The access token is missing, invalid or expired.
    # Refresh once and retry; if that fails, authenticate again.
    try:
        new_token = refresh_access_token(refresh_token)
        retry_request(new_token)
    except AuthError:
        authenticate()
```

---

### Forbidden Errors (403)

| Code | Message | Cause | Solution |
|------|---------|-------|----------|
| `AUTH_INSUFFICIENT_PERMISSIONS` | `Insufficient permissions. Required scope(s): memories:write, write, admin` | The token lacks the scope the call needs. The message lists the scopes that would allow it | Use a token or API key that has the scope (`write` to create and update, `delete` to delete) |
| `AUTH_FORBIDDEN` | Cannot create/set {type} via this endpoint | Attempted to create or update a memory with a protected type (`identity_core`, `personality_trait`) | Use `/api/v2/personality` endpoints instead |

**Protected Memory Types:**

`identity_core` and `personality_trait` cannot be created or updated via `POST /api/v2/memories` or `PUT /api/v2/memories/{id}`. These types are managed exclusively through the `/api/v2/personality` endpoints.

```json
{
  "status": "error",
  "error": {
    "code": "AUTH_FORBIDDEN",
    "message": "Cannot create identity_core memories via this endpoint. Use /personality endpoints instead.",
    "details": null,
    "field_errors": null
  },
  "meta": {
    "timestamp": "2026-02-11T10:13:32.467536+00:00",
    "version": "2.0.0",
    "request_id": "id-for-debugging"
  }
}
```

**Handling:**
```python
if response.status_code == 403:
    error_code = response.json().get("error", {}).get("code")
    if error_code == "AUTH_FORBIDDEN":
        # Protected memory type
        # Use /personality endpoints for identity_core and personality_trait
        pass
    else:
        # AUTH_INSUFFICIENT_PERMISSIONS:
        # the token is valid but lacks the required scope
        request_elevated_access()
```

---

### Validation Errors (422 and 400)

Every validation failure uses the code `VAL_VALIDATION_FAILED`. The HTTP status tells you which kind it is.

| Status | When | Where the reason is |
|--------|------|---------------------|
| 422 | The request does not match the schema: a required field is missing, a value has the wrong type, a string is too long, an enum value or UUID is not valid | `error.field_errors` |
| 400 | The request is well formed but breaks a rule: search weights that do not sum to 1.0, a relationship that already exists, a memory that would have more than 10 tags, an unsupported upload type, an invalid artifact path | `error.message` (and `error.details` on some endpoints) |

**Field-Level Errors (422):**

Validation errors include a `field_errors` object that identifies exactly which fields failed:

```json
{
  "status": "error",
  "error": {
    "code": "VAL_VALIDATION_FAILED",
    "message": "Request validation failed",
    "details": null,
    "field_errors": {
      "content": ["Field required"],
      "importance": ["Input should be a valid number, unable to parse string as a number"]
    }
  },
  "meta": {...}
}
```

Each key in `field_errors` is the field name, and the value is an array of error messages for that field.

**Rule Violations (400):**

```json
{
  "status": "error",
  "error": {
    "code": "VAL_VALIDATION_FAILED",
    "message": "Search weights must sum to 1.0 (current sum: 1.5)",
    "details": {
      "bm25_weight": 0.9,
      "vector_weight": 0.4,
      "graph_weight": 0.2
    },
    "field_errors": null
  },
  "meta": {...}
}
```

**Handling:**
```python
if response.status_code == 422:
    error = response.json().get("error", {})
    field_errors = error.get("field_errors") or {}

    # Check specific field errors
    if "content" in field_errors:
        print(f"Content error: {field_errors['content']}")

    # Or iterate all field errors
    for field, messages in field_errors.items():
        print(f"{field}: {', '.join(messages)}")

elif response.status_code == 400:
    error = response.json().get("error", {})
    print(f"Rejected: {error.get('message')}")
```

---

### Not Found Errors (404)

| Code | Message | Cause | Solution |
|------|---------|-------|----------|
| `RES_NOT_FOUND` | `Memory with id '...' not found` | The memory, document, relationship, artifact, trait or contradiction does not exist | Verify the ID or path |

`error.details` names what was looked for:

```json
{
  "code": "RES_NOT_FOUND",
  "message": "Document with id '550e8400-e29b-41d4-a716-446655440000' not found",
  "details": {
    "resource_type": "Document",
    "resource_id": "550e8400-e29b-41d4-a716-446655440000"
  },
  "field_errors": null
}
```

A memory that has been replaced through [evolve](../endpoints/analysis.md#evolve-memory) also returns 404 on `GET /memories/{id}`. Use the id of the new version.

**Handling:**
```python
if response.status_code == 404:
    # Resource doesn't exist - may need to create or verify ID
    pass
```

---

### Conflict Errors (409)

| Status | Message | Cause | Solution |
|--------|---------|-------|----------|
| 409 | `Artifact already exists at path: /notes/python-tips.md` | An artifact is already stored at that path | Delete it first, or save under another path |

**Checkpoint Name Uniqueness:**

Context checkpoint names are unique. The MCP `save_context` tool refuses a name that is already in use and returns a `DUPLICATE_NAME` error code. See the [MCP Integration Guide](../../mcp/mcp-integration.md#save_context) for details.

---

### Payload Too Large (413)

| Status | Cause | Solution |
|--------|-------|----------|
| 413 | A document upload over 20MB, or an artifact over 1MB | Split the file |

---

### Rate Limit Errors (429)

| Code | Message | Cause | Solution |
|------|---------|-------|----------|
| `BIZ_RATE_LIMIT_EXCEEDED` | Rate limit exceeded | Too many requests | Wait and retry |

```json
{
  "status": "error",
  "error": {
    "code": "BIZ_RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded",
    "details": {
      "retry_after": 3,
      "limit": 35
    }
  },
  "meta": {...}
}
```

**Headers:**

| Header | Description |
|--------|-------------|
| `x-ratelimit-limit` | Max requests per window |
| `x-ratelimit-remaining` | Requests left in window |
| `x-ratelimit-reset` | Unix timestamp when window resets |
| `retry-after` | Seconds to wait before retrying (on 429) |

The limit depends on the endpoint. Read it from the headers rather than assuming a number.

**Handling:**
```python
if response.status_code == 429:
    retry_after = int(response.headers.get("retry-after", 60))
    time.sleep(retry_after)
    retry_request()
```

---

### Server Errors (500, 501, 503)

| Status | Code | Message | Cause | Solution |
|--------|------|---------|-------|----------|
| 500 | `SYS_INTERNAL_ERROR` | Internal server error | Server-side issue | Retry with backoff. If it persists, contact support with the `request_id` |
| 501 | `SYS_NOT_IMPLEMENTED` | Not yet implemented | The feature is not available yet (Word document upload, document reprocess) | Use a supported alternative |
| 503 | `EXT_SEARCH_SERVICE_ERROR` | An error occurred | The search service could not complete the request | Retry later |

**Handling:**
```python
if response.status_code in (500, 503):
    # Exponential backoff retry
    for attempt in range(max_retries):
        time.sleep(2 ** attempt)
        response = retry_request()
        if response.status_code < 500:
            break
```

---

## Retry Strategy

Recommended retry logic:

```python
import time
from typing import Callable

def with_retry(
    fn: Callable,
    max_retries: int = 3,
    base_delay: float = 1.0,
    max_delay: float = 60.0
):
    """Execute with exponential backoff retry."""
    last_error = None

    for attempt in range(max_retries):
        try:
            response = fn()

            # Don't retry client errors (except rate limits)
            if response.status_code < 500 and response.status_code != 429:
                return response

            # Handle rate limits
            if response.status_code == 429:
                retry_after = int(response.headers.get("retry-after", base_delay))
                time.sleep(min(retry_after, max_delay))
                continue

            # Server error - exponential backoff
            delay = min(base_delay * (2 ** attempt), max_delay)
            time.sleep(delay)

        except Exception as e:
            last_error = e
            delay = min(base_delay * (2 ** attempt), max_delay)
            time.sleep(delay)

    raise last_error or Exception("Max retries exceeded")
```

---

## Error Details

Some errors include additional context in `details`. A request that fails schema validation carries its reasons in `field_errors` instead. For example, content over the 10,000 character limit:

```json
{
  "error": {
    "code": "VAL_VALIDATION_FAILED",
    "message": "Request validation failed",
    "details": null,
    "field_errors": {
      "content": ["Value error, Memory content is 10,001 characters, exceeds the 10,000 character limit. For longer content, upload as a document (POST /api/v2/documents/upload) ..."]
    }
  }
}
```

Content that is too long for one memory can be split into several memories, uploaded as a [document](../endpoints/documents.md), or stored as an [artifact](../endpoints/artifacts.md) with a summary memory that points to it.

Common detail fields:

| Field | Description |
|-------|-------------|
| `resource_type` | Type of resource (Memory, Document, Relationship, etc.) on a 404 |
| `resource_id` | ID of the resource that was not found |
| `errors` | Array of reasons on a relationship that was refused (400) |
| `retry_after`, `limit` | Seconds to wait and the limit that was hit, on a 429 |

---

## Debugging Tips

1. **Check `request_id`** - Include in support requests
2. **Log full error response** - Don't just log status code
3. **Validate before sending** - Catch errors client-side
4. **Use proper content types** - Always set `Content-Type: application/json`
5. **Check token expiration** - Refresh proactively before expiry
