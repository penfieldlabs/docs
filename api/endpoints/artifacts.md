# Artifacts Endpoints

Artifacts are file storage for you and your AI: a private folder tree where files are saved, read back, listed and deleted by path.

---

## What Artifacts Are For

A memory is one piece of knowledge and holds up to 10,000 characters. An artifact is a whole file and holds up to 1 MB. Use an artifact when the content is a file rather than a fact:

- **Code** - scripts, modules, configuration files, snippets worth keeping whole
- **Long documents** - specs, reports, research notes, transcripts, anything past the memory size limit
- **Structured data** - JSON, YAML and CSV
- **Diagrams** - SVG and other text-based diagram formats
- **Templates and working drafts** - PR templates, prompts, checklists, text that gets rewritten over time

| | Memories | Documents | Artifacts |
|---|----------|-----------|-----------|
| What it is | A piece of knowledge | A file you upload so it can be searched | A file stored and read back as a file |
| Size limit | 10,000 characters | 20 MB | 1 MB |
| Found by search | Yes | Yes, split into chunks | No |
| Read back whole | Yes | Yes, as a download | Yes, by path |
| Organized by | Tags and relationships | Document list | Folders and file names |
| Created through | API and MCP tools | API upload and the portal | API and MCP tools |

**Artifacts are not searched.** Search and recall never return an artifact, its content or its file name. An artifact is reached in two ways: by its path, or by browsing the folder tree. To make an artifact turn up when it is relevant, store a memory that describes it and gives its path. See [Making Artifacts Findable](#making-artifacts-findable).

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
- Maximum 10 folders deep (the file name does not count)
- Valid characters: letters `A-Z` and `a-z`, digits, dash, underscore, dot, slash. Spaces and other characters are refused
- No `..` (path traversal)
- No hidden files (starting with `.`)
- No double slashes

### Valid Path Examples

```
/readme.md
/project/docs/api.md
/diagrams/architecture.svg
/notes/2026/january/meeting.md
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

- Maximum artifact size: 1MB (1,048,576 bytes, counted on the UTF-8 encoded content)

### Saving Does Not Overwrite

A path can hold one artifact. Saving to a path that already exists is refused with `409`. To change a file, [delete](#delete-artifact) it and save the new content, or save the new version under a new path.

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 400 | `VAL_VALIDATION_FAILED` | Invalid path format (the message says which rule) |
| 403 | `AUTH_INSUFFICIENT_PERMISSIONS` | The token does not have the `write` scope |
| 409 | - | Artifact already exists at path |
| 413 | - | Content exceeds 1MB limit |

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

### Response Fields

| Field | Description |
|-------|-------------|
| `success` | Boolean indicating operation success |
| `content` | The artifact content |
| `path` | The artifact path |
| `content_type` | MIME type of the content |
| `size` | Content size in bytes |

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 400 | `VAL_VALIDATION_FAILED` | Invalid path format |
| 404 | `RES_NOT_FOUND` | Artifact not found |

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
| `name_pattern` | string | - | Case-insensitive substring filter applied to file and folder names |

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
        "last_modified": "2026-01-06T12:00:00.531000+00:00"
      },
      {
        "name": "notes.txt",
        "size": 512,
        "content_type": "text/plain",
        "last_modified": "2026-01-05T10:00:00.411000+00:00"
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
- `name_pattern` is applied server-side against both file names and folder names within the listed directory

### Example

```bash
# List root directory
curl -X GET "https://api.penfield.app/api/v2/artifacts/list" \
-H "Authorization: Bearer $JWT_TOKEN"

# List specific directory
curl -X GET "https://api.penfield.app/api/v2/artifacts/list?prefix=/project/docs" \
-H "Authorization: Bearer $JWT_TOKEN"

# Filter by name (case-insensitive substring)
curl -X GET "https://api.penfield.app/api/v2/artifacts/list?prefix=/project/docs&name_pattern=meeting" \
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

- This operation is permanent and cannot be undone
- **Requires the `delete` scope**
- Any memories you created that reference this artifact path will not be affected - update or delete those separately
- A folder is no longer listed once it has no files left in it

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 400 | `VAL_VALIDATION_FAILED` | Invalid path format |
| 403 | `AUTH_INSUFFICIENT_PERMISSIONS` | The token does not have the `delete` scope |
| 404 | `RES_NOT_FOUND` | Artifact not found |

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
| `.xml` | `text/xml` |
| `.yaml`, `.yml` | `text/yaml` |
| `.csv` | `text/csv` |
| Unknown extension | `application/octet-stream` |

The same content type is returned when the artifact is saved, retrieved and listed.

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
| Searchable | No | Yes (hybrid search) |
| Relationships | No | Yes (knowledge graph) |
| Use Case | Store files and full-length content | Store searchable knowledge |

Artifacts are files. They are not included in search or recall results. To retrieve an artifact, you need its path.

## Making Artifacts Findable

There are three ways to reach an artifact:

1. **By path** - If you know the path, call [Retrieve Artifact](#retrieve-artifact) directly.
2. **By browsing** - Use [List Artifacts](#list-artifacts) to walk the folder tree. `name_pattern` filters the names in one folder.
3. **Through a memory** - Store a memory that says what the file is and gives its path. When that memory comes up in search, follow the path to retrieve the full artifact.

The third way is the one that makes an artifact part of your AI's working knowledge. The memory is searchable, can carry tags, and can be connected to other memories with relationships. The artifact holds the full content. A good pointer memory:

- says what the file contains and when it is useful, in the words someone would search for
- gives the exact path
- is updated when the file is replaced or moved, and deleted when the file is deleted

For example, a long research document can be stored as an artifact, with a shorter summary memory that references the artifact path:

```bash
# 1. Store the full document as an artifact
curl -X POST https://api.penfield.app/api/v2/artifacts \
-H "Authorization: Bearer $JWT_TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "path": "/research/sec-analysis.md",
  "content": "# Full 50-page SEC analysis document content..."
}'

# 2. Create a searchable summary memory that points to it
curl -X POST https://api.penfield.app/api/v2/memories \
-H "Authorization: Bearer $JWT_TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "content": "Regulatory analysis of silver bullion P2P trading under SEC guidelines. Covers exemptions, reporting requirements, and state-level variations. Full document: /research/sec-analysis.md",
  "memory_type": "reference",
  "tags": ["sec", "silver", "regulations"]
}'
```

When an agent searches for "SEC silver trading regulations", the summary memory appears in results. The agent reads the path from the content and calls `retrieve_artifact` to get the full document.
