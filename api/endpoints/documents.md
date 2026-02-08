# Documents Endpoints

Upload and manage documents that are automatically chunked, indexed, and made searchable.

---

## Upload Document

Upload and process a document.

```
POST https://api.penfield.app/api/v2/documents/upload
```

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Authorization` | Yes | `Bearer YOUR_JWT_TOKEN` |
| `Content-Type` | Yes | `multipart/form-data` |

### Request Body (multipart/form-data)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file` | file | Yes | Document file to upload |
| `metadata` | string | No | JSON string with additional metadata |

### Supported File Types

| Type | Extensions | Max Size |
|------|------------|----------|
| Text | `.txt` | 20MB |
| Markdown | `.md` | 20MB |
| PDF | `.pdf` | 20MB |
| EPUB | `.epub` | 20MB |
| Word | `.doc`, `.docx` | 20MB |
| HTML | `.html`, `.htm` | 20MB |
| JSON | `.json` | 20MB |
| YAML | `.yaml`, `.yml` | 20MB |
| Code | `.py`, `.js`, `.ts` | 20MB |

### Chunking Strategy

Documents are automatically split into searchable chunks. Approximate behavior:

| Parameter | Value |
|-----------|-------|
| Target chunk size | ~400 characters |
| Maximum chunk size | ~450 characters |
| Chunk overlap | ~50 characters |

Different file types use optimized chunking strategies:

| File Type | Strategy | Notes |
|-----------|----------|-------|
| Text | Sentence-based | Splits at sentence boundaries |
| Markdown | Structure-aware | Respects headers, code blocks, tables |
| PDF | Page/section-aware | Uses TOC when available |
| EPUB | Chapter-aware | Preserves chapter boundaries |
| Code | Markdown-style | Keeps code blocks together |

### Response

```json
{
  "status": "success",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "filename": "python-guide.pdf",
    "content_type": "application/pdf",
    "size": 1048576,
    "chunk_count": 45,
    "processing_status": "completed",
    "embedding_progress": {
      "total": 45,
      "completed": 45,
      "pending": 0,
      "failed": 0,
      "percent_complete": 100.0
    },
    "metadata": {
      "chunk_ids": ["uuid1", "uuid2", "..."],
      "message": "Document processed into 45 chunks. Embeddings are being generated.",
      "processing_time": 2.34
    },
    "created_at": "2025-01-06T12:00:00.000Z",
    "updated_at": "2025-01-06T12:00:00.000Z"
  },
  "meta": {...}
}
```

### Processing Status

| Status | Description |
|--------|-------------|
| `uploading` | File is being uploaded |
| `processing` | Document is being chunked |
| `active` | Ready for search |
| `completed` | All embeddings generated |
| `failed` | Processing failed |

### Embedding Progress

Documents are chunked synchronously, but embeddings are generated asynchronously. The `embedding_progress` field shows:

- `total`: Total chunks created
- `completed`: Chunks with embeddings
- `pending`: Chunks awaiting embedding
- `failed`: Chunks that failed embedding
- `percent_complete`: Completion percentage

### Duplicate Detection

If you upload a file with identical content (matching hash), the existing document is returned:

```json
{
  "metadata": {
    "duplicate_upload": true,
    "message": "Document already exists. Returning existing document with 45 chunks."
  }
}
```

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 413 | - | File too large (max 20MB) |
| 422 | `VAL_VALIDATION_FAILED` | Unsupported file type |
| 501 | `NOT_IMPLEMENTED` | File type not yet supported |

### Example

```bash
curl -X POST https://api.penfield.app/api/v2/documents/upload \
-H "Authorization: Bearer $JWT_TOKEN" \
-F "file=@/path/to/document.pdf" \
-F 'metadata={"project": "documentation", "version": "2.0"}'
```

---

## Get Document

Retrieve document metadata and optionally its chunks.

```
GET https://api.penfield.app/api/v2/documents/{document_id}
```

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `document_id` | UUID | Document identifier |

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `include_chunks` | boolean | `false` | Include document chunks in response |

### Response (without chunks)

```json
{
  "status": "success",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "filename": "python-guide.pdf",
    "content_type": "application/pdf",
    "size": 1048576,
    "chunk_count": 45,
    "processing_status": "completed",
    "embedding_progress": {
      "total": 45,
      "completed": 45,
      "pending": 0,
      "failed": 0,
      "percent_complete": 100.0
    },
    "metadata": {},
    "created_at": "2025-01-06T12:00:00.000Z",
    "updated_at": "2025-01-06T12:00:00.000Z"
  },
  "meta": {...}
}
```

### Response (with chunks)

```json
{
  "status": "success",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "filename": "python-guide.pdf",
    "content_type": "application/pdf",
    "size": 1048576,
    "chunk_count": 45,
    "processing_status": "completed",
    "metadata": {},
    "created_at": "2025-01-06T12:00:00.000Z",
    "updated_at": "2025-01-06T12:00:00.000Z",
    "chunks": [
      {
        "id": "chunk-uuid-1",
        "document_id": "550e8400-e29b-41d4-a716-446655440000",
        "chunk_index": 0,
        "content": "Chapter 1: Introduction to Python...",
        "start_char": 0,
        "end_char": 1500,
        "metadata": {
          "page": 1,
          "section": "Introduction"
        },
        "embedding_status": "completed",
        "created_at": "2025-01-06T12:00:00.000Z"
      },
      {
        "id": "chunk-uuid-2",
        "document_id": "550e8400-e29b-41d4-a716-446655440000",
        "chunk_index": 1,
        "content": "Python is a versatile programming language...",
        "start_char": 1500,
        "end_char": 3000,
        "metadata": {},
        "embedding_status": "completed",
        "created_at": "2025-01-06T12:00:00.000Z"
      }
    ]
  },
  "meta": {...}
}
```

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 404 | `RES_NOT_FOUND` | Document not found |

---

## List Documents

List all documents for the tenant.

```
GET https://api.penfield.app/api/v2/documents
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | `1` | Page number |
| `per_page` | integer | `20` | Items per page (max 100) |

### Response

```json
{
  "status": "success",
  "data": {
    "items": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "filename": "python-guide.pdf",
        "content_type": "application/pdf",
        "size": 1048576,
        "chunk_count": 45,
        "processing_status": "completed",
        "embedding_progress": {
          "total": 45,
          "completed": 45,
          "pending": 0,
          "failed": 0,
          "percent_complete": 100.0
        },
        "metadata": {},
        "created_at": "2025-01-06T12:00:00.000Z",
        "updated_at": "2025-01-06T12:00:00.000Z"
      }
    ],
    "pagination": {
      "page": 1,
      "per_page": 20,
      "total": 5,
      "pages": 1,
      "has_next": false,
      "has_prev": false
    }
  },
  "meta": {...}
}
```

---

## Download Document

Download the original uploaded document file.

```
GET https://api.penfield.app/api/v2/documents/{document_id}/download
```

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `document_id` | UUID | Document identifier |

### Response

Returns the raw file content with appropriate headers:

```
Content-Type: application/pdf
Content-Disposition: attachment; filename="python-guide.pdf"
```

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 404 | `RES_NOT_FOUND` | Document not found or file missing from storage |

### Example

```bash
curl -X GET https://api.penfield.app/api/v2/documents/550e8400-e29b-41d4-a716-446655440000/download \
-H "Authorization: Bearer $JWT_TOKEN" \
-o downloaded-file.pdf
```

---

## Delete Document

Permanently delete a document and all its chunks.

```
DELETE https://api.penfield.app/api/v2/documents/{document_id}
```

### Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `document_id` | UUID | Document identifier |

### Response

```
HTTP 204 No Content
```

### Notes

- This operation is permanent and cannot be undone
- All associated chunks are deleted
- The original file is removed from storage
- Embeddings are also removed

### Errors

| Status | Code | Description |
|--------|------|-------------|
| 404 | `RES_NOT_FOUND` | Document not found |

---

## Reprocess Document

Re-chunk and re-embed a document. Useful when chunking algorithms are updated.

```
POST https://api.penfield.app/api/v2/documents/{document_id}/reprocess
```

### Status

**Not Yet Implemented** - Returns 501

### Future Behavior

When implemented, this will:
1. Retrieve the original document from storage
2. Re-run chunking with current algorithms
3. Replace existing chunks
4. Generate new embeddings

---

## Document Search

Document chunks are automatically searchable via the hybrid search endpoint:

```bash
curl -X POST https://api.penfield.app/api/v2/search/hybrid \
-H "Authorization: Bearer $JWT_TOKEN" \
-H "Content-Type: application/json" \
-d '{
  "query": "python async programming",
  "source_types": ["document_upload"],
  "include_neighbors": 2
}'
```

### Context Expansion

When searching documents, use `include_neighbors` to get surrounding context:

```json
{
  "query": "specific topic",
  "include_neighbors": 2
}
```

This includes 2 chunks before and after each matching chunk, providing better context.
