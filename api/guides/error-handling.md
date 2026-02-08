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
    "details": {}
  },
  "meta": {
    "timestamp": "2025-01-06T12:00:00.000Z",
    "request_id": "uuid-for-debugging"
  }
}
```

---

## Error Code Reference

### Authentication Errors (401)

| Code | Message | Cause | Solution |
|------|---------|-------|----------|
| `AUTH_INVALID_CREDENTIALS` | Invalid credentials | Wrong API key or token | Check API key format |
| `AUTH_TOKEN_EXPIRED` | Token expired | JWT past expiration | Refresh token or re-authenticate |
| `AUTH_TOKEN_INVALID` | Invalid token | Malformed or tampered JWT | Re-authenticate |
| `AUTH_UNAUTHORIZED` | Unauthorized | Missing Authorization header | Add Bearer token |

**Handling:**
```python
if response.status_code == 401:
    error_code = response.json().get("error", {}).get("code")
    if error_code == "AUTH_TOKEN_EXPIRED":
        # Refresh token and retry
        new_token = refresh_token(refresh_token)
        retry_request(new_token)
    else:
        # Re-authenticate
        authenticate()
```

---

### Forbidden Errors (403)

| Code | Message | Cause | Solution |
|------|---------|-------|----------|
| `AUTH_INSUFFICIENT_PERMISSIONS` | Insufficient permissions | Token lacks required scope | Request token with correct scopes |

**Handling:**
```python
if response.status_code == 403:
    # Token is valid but lacks required permissions
    # User needs token with different/additional scopes
    request_elevated_access()
```

---

### Validation Errors (422)

| Code | Message | Cause | Solution |
|------|---------|-------|----------|
| `VAL_VALIDATION_FAILED` | Validation failed | Request body invalid | Check request format |
| `VAL_FIELD_REQUIRED` | Required field missing | Missing required parameter | Add required field |
| `VAL_FIELD_TOO_LONG` | Field too long | Content exceeds max length | Truncate content (max 10,000 chars) |
| `VAL_FIELD_INVALID` | Invalid field value | Wrong type or format | Check field type |
| `VAL_UUID_INVALID` | Invalid UUID format | Malformed UUID | Use valid UUID v4 |
| `VAL_ENUM_INVALID` | Invalid enum value | Value not in allowed list | Use valid enum value |

**Field-Level Errors:**

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

**Handling:**
```python
if response.status_code == 422:
    error = response.json().get("error", {})
    field_errors = error.get("field_errors", {})

    # Check specific field errors
    if "content" in field_errors:
        print(f"Content error: {field_errors['content']}")

    # Or iterate all field errors
    for field, messages in field_errors.items():
        print(f"{field}: {', '.join(messages)}")
```

---

### Not Found Errors (404)

| Code | Message | Cause | Solution |
|------|---------|-------|----------|
| `RES_NOT_FOUND` | Resource not found | Memory/document doesn't exist | Verify resource ID |
| `RES_MEMORY_NOT_FOUND` | Memory not found | Memory ID doesn't exist | Check memory UUID |
| `RES_DOCUMENT_NOT_FOUND` | Document not found | Document ID doesn't exist | Check document UUID |

**Handling:**
```python
if response.status_code == 404:
    # Resource doesn't exist - may need to create or verify ID
    pass
```

---

### Conflict Errors (409)

| Code | Message | Cause | Solution |
|------|---------|-------|----------|
| `RES_ALREADY_EXISTS` | Resource already exists | Duplicate creation attempt | Use existing resource |
| `RES_RELATIONSHIP_EXISTS` | Relationship exists | Duplicate relationship | Skip or update existing |

---

### Rate Limit Errors (429)

| Code | Message | Cause | Solution |
|------|---------|-------|----------|
| `BIZ_RATE_LIMIT_EXCEEDED` | Rate limit exceeded | Too many requests | Wait and retry |

**Headers on 429 responses:**

| Header | Description |
|--------|-------------|
| `x-ratelimit-limit` | Max requests per window |
| `x-ratelimit-remaining` | Requests left in window |
| `x-ratelimit-reset` | Unix timestamp when window resets |
| `retry-after` | Seconds to wait before retrying |

**Handling:**
```python
if response.status_code == 429:
    retry_after = int(response.headers.get("retry-after", 60))
    time.sleep(retry_after)
    retry_request()
```

---

### Server Errors (500, 502, 503)

| Code | Message | Cause | Solution |
|------|---------|-------|----------|
| `SYS_INTERNAL_ERROR` | Internal server error | Server-side issue | Retry with backoff |
| `SYS_SERVICE_UNAVAILABLE` | Service unavailable | Maintenance or overload | Retry later |
| `SYS_BAD_GATEWAY` | Bad gateway | Upstream service issue | Retry later |

**Handling:**
```python
if response.status_code >= 500:
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

Some errors include additional context in `details`:

```json
{
  "error": {
    "code": "VAL_FIELD_TOO_LONG",
    "message": "Content exceeds maximum length",
    "details": {
      "field": "content",
      "max_length": 10000,
      "actual_length": 15234
    }
  }
}
```

Common detail fields:

| Field | Description |
|-------|-------------|
| `field` | Which field caused the error |
| `resource_type` | Type of resource (Memory, Document, etc.) |
| `resource_id` | ID of affected resource |
| `max_length` | Maximum allowed length |
| `allowed_values` | List of valid enum values |

---

## Debugging Tips

1. **Check `request_id`** - Include in support requests
2. **Log full error response** - Don't just log status code
3. **Validate before sending** - Catch errors client-side
4. **Use proper content types** - Always set `Content-Type: application/json`
5. **Check token expiration** - Refresh proactively before expiry
