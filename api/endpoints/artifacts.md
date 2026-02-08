# Artifacts Endpoints

Store and retrieve user-created content (diagrams, notes, code) with path-based organization.

---

## Save Artifact

Save an artifact to storage.

```
POST https://api.penfield.app/api/v2/artifacts
```

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | `Bearer YOUR_JWT_TOKEN` |
| `Content-Type` | Yes | `application/json` |

### Request Body

```json
{
  "path": "/project/docs/api-notes.md",
  "content": "# API Notes\n\nImportant information about the API..."
}
```

### Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `path` | string | Yes | Full path including filename |
| `content` | string | Yes | File content to save |

### Path Requirements

- Must start with `/`
- Must specify a filename (not just a directory)
- Maximum 10 levels deep
- Valid characters: alphanumeric, dash, underscore, dot, slash
- No `..` (path traversal)
- No hidden files (starting with `.`)
- No double slashes

### Valid Path Examples

```
/readme.md
/project/docs/api.md
/diagrams/architecture.svg
/notes/2025/january/meeting.md
```

### Invalid Path Examples

```
readme.md          # Missing leading /
/                  # Directory, not file
/project/          # Directory, not file
/../secrets.txt    # Path traversal
/.hidden           # Hidden file
//double.txt       # Double slash
```

### Response

```json
{
  "status": "success",
  "data": {
    "success": true,
    "message": "Artifact saved successfully",
    "path": "/project/docs/api-notes.md",
    "artifact_id": "550e8400-e29b-41d4-a716-446655440000",
    "content_type": "text/markdown",
    "size": 1024,
    "storage_path": "artifacts/project/docs/api-notes.md"
  },
  "meta": {...}
}
```

### Response Fields

| Field | Description |
|-------|-------------|
| `success` | Boolean indicating operation success |
| `message` | Human-readable status message |
| `path` | The artifact path |
| `artifact_id` | Unique identifier for the artifact |
| `content_type` | Detected MIME type |
| `size` | Content size in bytes |
| `storage_path` | Internal storage location |

### Size Limits

- Maximum artifact size: 10MB

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 400 | - | Invalid path format |
| 409 | - | Artifact already exists at path |
| 413 | - | Content exceeds 10MB limit |

### Example

```bash
curl -X POST https://api.penfield.app/api/v2/artifacts \
-H "Authorization: Bearer $JWT_TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "path": "/notes/python-tips.md",
  "content": "# Python Tips\n\n- Use list comprehensions\n- Prefer generators for large data"
}'
```

---

## Retrieve Artifact

Retrieve an artifact's content.

```
GET https://api.penfield.app/api/v2/artifacts?path={path}
```

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Full path of the artifact |

### Response

```json
{
  "status": "success",
  "data": {
    "success": true,
    "content": "# Python Tips\n\n- Use list comprehensions\n- Prefer generators for large data",
    "path": "/notes/python-tips.md",
    "content_type": "text/markdown",
    "size": 78
  },
  "meta": {...}
}
```

> **Note:** Due to a known issue, `.md` files may currently return `text/plain` instead of `text/markdown`. See the known issue note below.

### Response Fields

| Field | Description |
|-------|-------------|
| `success` | Boolean indicating operation success |
| `content` | The artifact content |
| `path` | The artifact path |
| `content_type` | MIME type of the content (see known issue below) |
| `size` | Content size in bytes |

> **Known Issue:** Some file extensions (including `.md`) may return an incorrect `content_type` on retrieve. For example, `.md` files may return `text/plain` instead of `text/markdown`. Use the file extension from the `path` field as a reliable alternative until this is resolved.

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 400 | - | Invalid path format |
| 404 | - | Artifact not found |

### Example

```bash
curl -X GET "https://api.penfield.app/api/v2/artifacts?path=/notes/python-tips.md" \
-H "Authorization: Bearer $JWT_TOKEN"
```

---

## List Artifacts

List artifacts in a directory.

```
GET https://api.penfield.app/api/v2/artifacts/list?prefix={prefix}
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `prefix` | string | `/` | Directory path to list |

### Response

```json
{
  "status": "success",
  "data": {
    "success": true,
    "path": "/project",
    "folders": ["docs", "diagrams"],
    "files": [
      {
        "name": "readme.md",
        "size": 2048,
        "content_type": "text/markdown",
        "last_modified": "2025-01-06T12:00:00.000Z"
      },
      {
        "name": "notes.txt",
        "size": 512,
        "content_type": "text/plain",
        "last_modified": "2025-01-05T10:00:00.000Z"
      }
    ]
  },
  "meta": {...}
}
```

### Response Fields

| Field | Description |
|-------|-------------|
| `path` | The directory path being listed |
| `folders` | Array of folder names (strings) |
| `files[].name` | File name |
| `files[].size` | File size in bytes |
| `files[].content_type` | MIME type |
| `files[].last_modified` | Last modification timestamp |

### Notes

- Returns immediate contents only (not recursive)
- Folders are returned as name strings; combine with `path` to get full paths
- Empty directories are not shown

### Example

```bash
# List root directory
curl -X GET "https://api.penfield.app/api/v2/artifacts/list" \
-H "Authorization: Bearer $JWT_TOKEN"

# List specific directory
curl -X GET "https://api.penfield.app/api/v2/artifacts/list?prefix=/project/docs" \
-H "Authorization: Bearer $JWT_TOKEN"
```

---

## Delete Artifact

Permanently delete an artifact.

```
DELETE https://api.penfield.app/api/v2/artifacts?path={path}
```

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Full path of the artifact |

### Response

```json
{
  "status": "success",
  "data": {
    "message": "Artifact deleted: /notes/python-tips.md"
  },
  "meta": {...}
}
```

### Notes

- This operation is permanent
- Associated memory record is also deleted
- Parent directories are not deleted (even if empty)

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 400 | - | Invalid path format |
| 404 | - | Artifact not found |

---

## Content Type Detection

Content types are automatically detected based on file extension:

| Extension | Content Type |
|-----------|--------------|
| `.md` | `text/markdown` |
| `.txt` | `text/plain` |
| `.json` | `application/json` |
| `.html` | `text/html` |
| `.css` | `text/css` |
| `.js` | `application/javascript` |
| `.py` | `text/x-python` |
| `.svg` | `image/svg+xml` |
| `.xml` | `application/xml` |
| `.yaml`, `.yml` | `application/x-yaml` |
| `.csv` | `text/csv` |
| Other | `application/octet-stream` |

> **Known Issue:** Some file extensions (including `.md`) may return an incorrect `content_type` when retrieving artifacts. The table above shows the intended behavior. As a workaround, use the file extension from the `path` field to determine the file type.

---

## Use Cases

### Documentation Storage

```json
{
  "path": "/docs/api/authentication.md",
  "content": "# Authentication\n\nAPI authentication uses JWT tokens..."
}
```

### Code Snippets

```json
{
  "path": "/snippets/python/async-example.py",
  "content": "import asyncio\n\nasync def main():\n    await asyncio.sleep(1)"
}
```

### Diagrams (SVG)

```json
{
  "path": "/diagrams/architecture.svg",
  "content": "<svg xmlns=\"http://www.w3.org/2000/svg\">...</svg>"
}
```

### Configuration Files

```json
{
  "path": "/config/settings.json",
  "content": "{\"debug\": false, \"log_level\": \"info\"}"
}
```

---

## Artifacts vs Memories

| Feature | Artifacts | Memories |
|---------|-----------|----------|
| Organization | Path-based hierarchy | Flat with tags |
| Content | Files (text, code, diagrams) | Knowledge snippets |
| Searchable | No (planned) | Yes (hybrid search) |
| Relationships | No | Yes (knowledge graph) |
| Use Case | Store outputs | Store knowledge |
