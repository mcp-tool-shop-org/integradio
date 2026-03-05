---
title: Reference
description: SemanticBlocks API, architecture internals, embedder, registry, HNSW, circuit breaker, events, and security scope.
sidebar:
  order: 5
---

Complete API reference and architecture overview for Integradio.

## SemanticBlocks API

`SemanticBlocks` is the main entry point. It extends `gr.Blocks` with a component registry and embedding pipeline.

### Constructor

```python
from integradio import SemanticBlocks

with SemanticBlocks(
    db_path=None,                        # SQLite path (None = in-memory)
    cache_dir=None,                      # Embedding cache directory
    ollama_url="http://localhost:11434",  # Ollama server URL
    embed_model="nomic-embed-text",      # Embedding model name
) as demo:
    ...
```

### Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `search(query, k=10)` | `list[SearchResult]` | Semantic search across all registered components |
| `find(query)` | `SearchResult` | Get the single most relevant component |
| `trace(component)` | `TraceResult` | Get upstream/downstream dependency chain |
| `map()` | `dict` | Export graph as D3.js-compatible JSON |
| `describe(component)` | `dict` | Full metadata dump for a component |
| `summary()` | `str` | Text report of all registered components |
| `add_api_routes(app)` | `None` | Mount FastAPI routes on a FastAPI/Starlette app |

### SearchResult

```python
@dataclass
class SearchResult:
    id: str            # Component ID
    label: str         # Gradio label
    type: str          # Component type name (e.g., "Textbox")
    intent: str        # Semantic intent string
    score: float       # Cosine similarity (0.0 to 1.0)
    tags: list[str]    # Metadata tags
    metadata: dict     # Additional metadata from specialized wrappers
```

### TraceResult

```python
@dataclass
class TraceResult:
    component: SearchResult        # The traced component
    upstream: list[SearchResult]   # Components that feed into this one
    downstream: list[SearchResult] # Components that this one feeds into
    events: list[EventInfo]        # Event connections (click, change, etc.)
```

## Architecture

Integradio consists of four core subsystems:

```
┌─────────────────────────────────────────┐
│              SemanticBlocks              │
│  (extends gr.Blocks with registry)      │
├──────────┬──────────┬───────────────────┤
│ Embedder │ Registry │  Event Introspect │
│ (Ollama) │ (HNSW)   │  (dataflow graph) │
└──────────┴──────────┴───────────────────┘
```

### Embedder

The embedder communicates with Ollama to generate vector representations of intent strings.

- **Model:** `nomic-embed-text` (768 dimensions, default)
- **Protocol:** HTTP POST to `http://localhost:11434/api/embeddings`
- **Caching:** Optional on-disk cache to avoid re-embedding identical strings
- **Resilience:** Circuit breaker protects against Ollama downtime

```python
# The embedder is created automatically by SemanticBlocks
# You rarely need to interact with it directly
from integradio.embedder import OllamaEmbedder

embedder = OllamaEmbedder(
    url="http://localhost:11434",
    model="nomic-embed-text",
    cache_dir="./cache",
)

vector = embedder.embed("user enters search terms")
# Returns: numpy array of shape (768,)
```

### Registry (HNSW + SQLite)

The registry stores component metadata and their vector embeddings. It uses HNSW (Hierarchical Navigable Small World) for approximate nearest-neighbor search.

- **Vector index:** HNSW via `hnswlib` (in-process, no external service)
- **Metadata store:** SQLite (in-memory by default, optionally persisted to disk)
- **Search:** Cosine similarity with configurable `k` parameter

```python
from integradio.registry import ComponentRegistry

registry = ComponentRegistry(db_path="components.db")

# Register a component
registry.add(
    id="comp_001",
    label="Search Query",
    component_type="Textbox",
    intent="user enters search terms",
    vector=embedder.embed("user enters search terms"),
    tags=["input", "text"],
)

# Search
results = registry.search(query_vector, k=10)
```

### Circuit breaker

The circuit breaker protects against Ollama failures (server down, model not loaded, timeout).

```
States:
  CLOSED  → Normal operation. Requests pass through.
  OPEN    → Ollama is down. Requests fail immediately (no waiting).
  HALF    → Testing recovery. One request allowed through.
```

**Thresholds:**

| Parameter | Default | Description |
|-----------|---------|-------------|
| `failure_threshold` | 5 | Failures before opening circuit |
| `recovery_timeout` | 30s | Time before trying again (OPEN → HALF) |
| `success_threshold` | 2 | Successes in HALF before closing |

When the circuit is open, `semantic()` calls still work — the component is registered without an embedding and will be embedded when the circuit closes.

### Event introspection

Integradio inspects Gradio event listeners to build a dataflow graph:

```python
# When you write:
search_btn.click(fn=search, inputs=query, outputs=results)

# Integradio records:
# Edge: query → search_btn (input to click handler)
# Edge: search_btn → results (click handler output)
```

This graph powers `demo.trace()`, `demo.map()`, and all visualization exports.

## Events and WebSocket mesh

For real-time applications, Integradio provides an event system with WebSocket support:

```python
from integradio.events import EventMesh

mesh = EventMesh(secret="your-hmac-secret")

# Subscribe to component changes
@mesh.on("component.updated")
async def handle_update(event):
    print(f"Component {event.component_id} updated")

# Events are HMAC-signed for integrity
mesh.emit("component.updated", {
    "component_id": "comp_001",
    "new_intent": "user types a search query",
})
```

**Event types:**

| Event | Trigger |
|-------|---------|
| `component.registered` | New component added to registry |
| `component.updated` | Component metadata changed |
| `component.searched` | Search query executed |
| `graph.changed` | Event listener added/removed |
| `circuit.opened` | Circuit breaker tripped |
| `circuit.closed` | Circuit breaker recovered |

## Security scope

Integradio is a **local-first** library. It does not phone home, collect telemetry, or send data to external services.

| Aspect | Scope |
|--------|-------|
| **Network** | Local Ollama only (`localhost:11434`) |
| **Storage** | In-memory HNSW + optional SQLite file |
| **File system** | Optional embedding cache directory |
| **Telemetry** | None |
| **Cloud APIs** | None |
| **Secrets** | HMAC secret for event signing (optional, user-provided) |

For the full security policy, see [SECURITY.md](https://github.com/mcp-tool-shop-org/integradio/blob/main/SECURITY.md) in the repository.
