# Python Examples

Complete Python examples for interacting with the Penfield API.

---

## Setup

### Installation

```bash
pip install httpx  # or requests
```

### Client Class

```python
import httpx
from typing import Optional, List, Dict, Any
from datetime import datetime


class PenfieldClient:
    """Penfield API client with automatic token management."""

    def __init__(self, api_key: str, base_url: str = "https://api.penfield.app"):
        self.api_key = api_key
        self.base_url = base_url
        self._token: Optional[str] = None
        self._token_expires: Optional[datetime] = None
        self._client = httpx.Client(timeout=30.0)

    @property
    def token(self) -> str:
        """Get valid JWT token, refreshing if needed."""
        if self._token is None or self._is_token_expired():
            self._authenticate()
        return self._token

    def _is_token_expired(self) -> bool:
        if self._token_expires is None:
            return True
        # Refresh 5 minutes before expiry
        return datetime.utcnow() >= self._token_expires

    def _authenticate(self) -> None:
        """Exchange API key for JWT token."""
        response = self._client.post(
            f"{self.base_url}/api/v2/auth/token",
            headers={"Authorization": f"Bearer {self.api_key}"}
        )
        response.raise_for_status()
        data = response.json()["data"]
        self._token = data["access_token"]
        # Parse expiry (expires_in is seconds)
        from datetime import timedelta
        self._token_expires = datetime.utcnow() + timedelta(
            seconds=data["expires_in"] - 300  # 5 min buffer
        )

    def _headers(self) -> Dict[str, str]:
        return {
            "Authorization": f"Bearer {self.token}",
            "Content-Type": "application/json"
        }

    def _request(self, method: str, endpoint: str, **kwargs) -> Dict[str, Any]:
        """Make authenticated request."""
        url = f"{self.base_url}/api/v2{endpoint}"
        response = self._client.request(
            method, url, headers=self._headers(), **kwargs
        )
        response.raise_for_status()
        return response.json()

    # Memory operations
    def create_memory(
        self,
        content: str,
        memory_type: str = "fact",
        importance: float = 0.5,
        tags: Optional[List[str]] = None,
        metadata: Optional[Dict] = None
    ) -> Dict[str, Any]:
        """Create a new memory."""
        payload = {
            "content": content,
            "memory_type": memory_type,
            "importance": importance,
        }
        if tags:
            payload["tags"] = tags
        if metadata:
            payload["metadata"] = metadata
        return self._request("POST", "/memories", json=payload)

    def get_memory(self, memory_id: str) -> Dict[str, Any]:
        """Get a memory by ID."""
        return self._request("GET", f"/memories/{memory_id}")

    def list_memories(
        self,
        page: int = 1,
        per_page: int = 20,
        memory_type: Optional[str] = None,
        tags: Optional[List[str]] = None,
        importance_threshold: Optional[float] = None
    ) -> Dict[str, Any]:
        """List memories with filters."""
        params = {"page": page, "per_page": per_page}
        if memory_type:
            params["memory_type"] = memory_type
        if tags:
            params["tags"] = ",".join(tags)
        if importance_threshold:
            params["importance_threshold"] = importance_threshold
        return self._request("GET", "/memories", params=params)

    def update_memory(
        self,
        memory_id: str,
        content: Optional[str] = None,
        importance: Optional[float] = None,
        metadata: Optional[Dict] = None
    ) -> Dict[str, Any]:
        """Update a memory."""
        payload = {}
        if content is not None:
            payload["content"] = content
        if importance is not None:
            payload["importance"] = importance
        if metadata is not None:
            payload["metadata"] = metadata
        return self._request("PUT", f"/memories/{memory_id}", json=payload)

    def delete_memory(self, memory_id: str) -> None:
        """Delete a memory."""
        self._client.delete(
            f"{self.base_url}/api/v2/memories/{memory_id}",
            headers=self._headers()
        )

    # Search operations
    def search(
        self,
        query: str,
        limit: int = 20,
        memory_types: Optional[List[str]] = None,
        importance_threshold: Optional[float] = None
    ) -> Dict[str, Any]:
        """Perform hybrid search."""
        payload = {
            "query": query,
            "limit": limit,
            "bm25_weight": 0.4,
            "vector_weight": 0.4,
            "graph_weight": 0.2,
        }
        if memory_types:
            payload["memory_types"] = memory_types
        if importance_threshold:
            payload["importance_threshold"] = importance_threshold
        return self._request("POST", "/search/hybrid", json=payload)

    # Relationship operations
    def create_relationship(
        self,
        from_id: str,
        to_id: str,
        relationship_type: str,
        strength: float = 0.5
    ) -> Dict[str, Any]:
        """Create a relationship between memories."""
        payload = {
            "from_id": from_id,
            "to_id": to_id,
            "relationship_type": relationship_type,
            "strength": strength
        }
        return self._request("POST", "/relationships", json=payload)

    def traverse_graph(
        self,
        start_memory_id: str,
        max_depth: int = 3,
        direction: str = "ANY"
    ) -> Dict[str, Any]:
        """Traverse the knowledge graph."""
        payload = {
            "start_memory_id": start_memory_id,
            "max_depth": max_depth,
            "direction": direction
        }
        return self._request("POST", "/relationships/traverse", json=payload)

    # Document operations
    def upload_document(
        self,
        file_path: str,
        metadata: Optional[Dict] = None
    ) -> Dict[str, Any]:
        """Upload a document."""
        import json
        with open(file_path, "rb") as f:
            files = {"file": f}
            data = {}
            if metadata:
                data["metadata"] = json.dumps(metadata)
            response = self._client.post(
                f"{self.base_url}/api/v2/documents/upload",
                headers={"Authorization": f"Bearer {self.token}"},
                files=files,
                data=data
            )
        response.raise_for_status()
        return response.json()

    def close(self):
        """Close the client connection."""
        self._client.close()

    def __enter__(self):
        return self

    def __exit__(self, *args):
        self.close()
```

---

## Usage Examples

### Basic Memory Operations

```python
# Initialize client
client = PenfieldClient(api_key="tm_your_tenant_ak_your_key")

# Create a memory
result = client.create_memory(
    content="Python's asyncio provides async/await syntax for concurrent programming",
    memory_type="fact",
    importance=0.8,
    tags=["python", "async", "concurrency"]
)
memory_id = result["data"]["id"]
print(f"Created memory: {memory_id}")

# Get the memory
memory = client.get_memory(memory_id)
print(f"Content: {memory['data']['content']}")

# Update the memory
client.update_memory(
    memory_id,
    importance=0.9,
    metadata={"verified": True}
)

# List memories
memories = client.list_memories(
    memory_type="fact",
    importance_threshold=0.7,
    per_page=10
)
for item in memories["data"]["items"]:
    print(f"- {item['content'][:50]}...")
```

### Search

```python
# Hybrid search
results = client.search(
    query="Python async programming patterns",
    limit=5,
    memory_types=["fact", "insight"]
)

for item in results["data"]["items"]:
    print(f"Score: {item['score']:.2f}")
    print(f"Content: {item['snippet']}")
    print("---")
```

### Building Knowledge Graph

```python
# Create memories
py_async = client.create_memory(
    content="Python asyncio uses an event loop to manage async operations",
    memory_type="fact",
    tags=["python", "asyncio"]
)

py_await = client.create_memory(
    content="await keyword suspends execution until the awaited coroutine completes",
    memory_type="fact",
    tags=["python", "asyncio"]
)

# Connect them
client.create_relationship(
    from_id=py_async["data"]["id"],
    to_id=py_await["data"]["id"],
    relationship_type="supports",
    strength=0.9
)

# Traverse the graph
graph = client.traverse_graph(
    start_memory_id=py_async["data"]["id"],
    max_depth=2,
    direction="OUTBOUND"
)

for path in graph["data"]["paths"]:
    print(f"Path score: {path['total_score']}")
    for node in path["nodes"]:
        print(f"  - {node['content'][:40]}...")
```

### Document Upload

```python
# Upload a document
result = client.upload_document(
    file_path="/path/to/document.pdf",
    metadata={"project": "research", "topic": "AI"}
)
doc_id = result["data"]["id"]
print(f"Uploaded document: {doc_id}")
print(f"Chunks created: {result['data']['chunk_count']}")

# Search document content
results = client.search(
    query="specific topic from document",
    source_types=["document_upload"]
)
```

---

## Async Client

For async applications:

```python
import httpx
import asyncio
from typing import Optional, List, Dict, Any


class AsyncPenfieldClient:
    """Async Penfield API client."""

    def __init__(self, api_key: str, base_url: str = "https://api.penfield.app"):
        self.api_key = api_key
        self.base_url = base_url
        self._token: Optional[str] = None
        self._client = httpx.AsyncClient(timeout=30.0)

    async def _authenticate(self) -> None:
        response = await self._client.post(
            f"{self.base_url}/api/v2/auth/token",
            headers={"Authorization": f"Bearer {self.api_key}"}
        )
        response.raise_for_status()
        self._token = response.json()["data"]["access_token"]

    async def _request(self, method: str, endpoint: str, **kwargs) -> Dict:
        if self._token is None:
            await self._authenticate()

        url = f"{self.base_url}/api/v2{endpoint}"
        response = await self._client.request(
            method, url,
            headers={
                "Authorization": f"Bearer {self._token}",
                "Content-Type": "application/json"
            },
            **kwargs
        )
        response.raise_for_status()
        return response.json()

    async def create_memory(self, content: str, **kwargs) -> Dict:
        return await self._request("POST", "/memories", json={"content": content, **kwargs})

    async def search(self, query: str, limit: int = 20) -> Dict:
        return await self._request("POST", "/search/hybrid", json={"query": query, "limit": limit})

    async def close(self):
        await self._client.aclose()

    async def __aenter__(self):
        return self

    async def __aexit__(self, *args):
        await self.close()


# Usage
async def main():
    async with AsyncPenfieldClient("your_api_key") as client:
        # Create memories concurrently
        tasks = [
            client.create_memory(f"Memory {i}", memory_type="fact")
            for i in range(5)
        ]
        results = await asyncio.gather(*tasks)
        print(f"Created {len(results)} memories")

        # Search
        search_results = await client.search("Memory")
        print(f"Found {len(search_results['data']['items'])} results")


asyncio.run(main())
```

---

## Error Handling

```python
import httpx


def safe_api_call(client: PenfieldClient, memory_id: str):
    """Example with error handling."""
    try:
        memory = client.get_memory(memory_id)
        return memory["data"]

    except httpx.HTTPStatusError as e:
        if e.response.status_code == 404:
            print(f"Memory {memory_id} not found")
            return None
        elif e.response.status_code == 401:
            print("Authentication failed - check your API key")
            raise
        elif e.response.status_code == 429:
            # Rate limited - get retry time
            retry_after = e.response.headers.get("Retry-After", "60")
            print(f"Rate limited. Retry after {retry_after} seconds")
            raise
        else:
            error_data = e.response.json()
            print(f"API Error: {error_data['error']['code']}")
            print(f"Message: {error_data['error']['message']}")
            raise

    except httpx.RequestError as e:
        print(f"Network error: {e}")
        raise
```
