# tldraw span conventions

## Overview

This document defines the vocabulary of operations (spans) that tldraw services emit. Edges in the canvas represent valid parent-child relationships.

## Span layers

Spans are organized into three layers based on their role:

| Layer | Span Kind | Color | Purpose |
|-------|-----------|-------|---------|
| **Entry** | SERVER/CONSUMER | Red | Request/connection entry points |
| **Domain** | INTERNAL | Blue/Purple | Business logic & SDK operations |
| **Infrastructure** | CLIENT | Green | External calls & storage |

## Entry layer spans

Entry spans are the root of most traces. They represent incoming requests or connections.

| Pattern | Kind | Description | Scope |
|---------|------|-------------|-------|
| `http.request` | SERVER | HTTP request handling | Workers |
| `websocket.connect` | SERVER | WebSocket connection established | sync-worker |
| `websocket.message` | SERVER | WebSocket message received | sync-worker |
| `durable-object.fetch` | SERVER | Durable Object request | sync-worker |
| `app.route.*` | INTERNAL | Client-side route navigation | dotcom-client |
| `extension.activate` | INTERNAL | VS Code extension activation | vscode-extension |

## Domain layer spans

Domain spans represent business logic operations. They're children of entry spans.

### Editor operations (@tldraw/editor)

| Pattern | Description | Example Children |
|---------|-------------|------------------|
| `editor.tool.*` | Tool state changes | `editor.shape.*` |
| `editor.shape.*` | Shape CRUD operations | `store.transaction.*`, `editor.canvas.*` |
| `editor.canvas.*` | Canvas rendering/viewport | (leaf) |

### UI operations (@tldraw/tldraw)

| Pattern | Description | Example Children |
|---------|-------------|------------------|
| `tldraw.ui.*` | UI component interactions | `tldraw.action.*` |
| `tldraw.action.*` | User actions (undo, copy, etc) | `editor.shape.*` |
| `tldraw.export.*` | Export to SVG/PNG/etc | `editor.canvas.*` |

### Store operations (@tldraw/store)

| Pattern | Description | Example Children |
|---------|-------------|------------------|
| `store.transaction.*` | Store transactions | `store.persist.*`, `sync.client.*` |

### Sync operations (@tldraw/sync, @tldraw/sync-worker)

| Pattern | Location | Description | Example Children |
|---------|----------|-------------|------------------|
| `sync.client.*` | Client | Sync client operations | `sync.socket.*` |
| `sync.server.*` | Server | Sync server operations | `db.*`, `cache.*` |
| `sync.presence.*` | Both | Presence updates | `db.*` |

### Other domain operations

| Pattern | Scope | Description | Example Children |
|---------|-------|-------------|------------------|
| `merge.*` | bemo-worker | Merge/OT operations | (leaf) |
| `upload.*` | asset-upload-worker | Upload processing | `r2.*` |

## Infrastructure layer spans

Infrastructure spans represent calls to external systems.

| Pattern | Kind | Description | Scope |
|---------|------|-------------|-------|
| `store.persist.*` | CLIENT | IndexedDB persistence | dotcom-client |
| `sync.socket.*` | CLIENT | WebSocket send/receive | sync client |
| `r2.*` | CLIENT | Cloudflare R2 storage | Workers |
| `db.*` | CLIENT | PostgreSQL queries | sync-worker, zero-cache |
| `cache.*` | CLIENT | Cache operations | zero-cache |
| `image.*` | CLIENT | Image fetch/resize | image-resize-worker |

## Valid parent-child relationships

Edges in the canvas define which spans can be children of which. Key relationships:

### Server-side traces

```
http.request
  └── upload.*
        └── r2.*

websocket.connect
  └── sync.server.*
        ├── db.*
        └── cache.*

websocket.message
  ├── sync.server.*
  │     ├── db.*
  │     └── cache.*
  └── merge.*

durable-object.fetch
  ├── sync.server.*
  └── sync.presence.*
        └── db.*
```

### Client-side traces

```
app.route.*
  ├── editor.tool.*
  │     └── editor.shape.*
  │           ├── editor.canvas.*
  │           └── store.transaction.*
  │                 ├── store.persist.*
  │                 └── sync.client.*
  │                       └── sync.socket.*
  └── sync.client.*
        └── sync.socket.*

tldraw.ui.*
  └── tldraw.action.*
        └── editor.shape.*
              └── ...

tldraw.export.*
  └── editor.canvas.*
```

### VS Code traces

```
extension.activate
  └── editor.tool.*
        └── ...
```

## Span attributes

### Common attributes

| Attribute | Type | Description |
|-----------|------|-------------|
| `tldraw.document.id` | string | Document identifier |
| `tldraw.user.id` | string | User identifier |
| `tldraw.shape.type` | string | Shape type (for shape operations) |
| `tldraw.tool.name` | string | Tool name (for tool operations) |

### HTTP spans

| Attribute | Type | Description |
|-----------|------|-------------|
| `http.method` | string | HTTP method |
| `http.route` | string | Route pattern |
| `http.status_code` | int | Response status |

### WebSocket spans

| Attribute | Type | Description |
|-----------|------|-------------|
| `websocket.message.type` | string | Message type |
| `websocket.message.size` | int | Message size in bytes |

### Storage spans

| Attribute | Type | Description |
|-----------|------|-------------|
| `r2.bucket` | string | R2 bucket name |
| `r2.key` | string | Object key |
| `db.statement` | string | Database query |

## Invalid relationships

These span hierarchies are NOT valid (no edge exists):

| Parent | Invalid Child | Reason |
|--------|---------------|--------|
| `db.*` | `sync.server.*` | Infrastructure doesn't call domain |
| `r2.*` | `upload.*` | Infrastructure doesn't call domain |
| `editor.canvas.*` | `editor.shape.*` | Canvas is a leaf (render only) |
| `sync.client.*` | `sync.server.*` | Client/server are separate traces |

## Design decisions

**Three-layer architecture**

Spans follow a strict layered architecture: Entry → Domain → Infrastructure. This makes traces predictable and easy to understand.

**Wildcard patterns**

Most patterns use wildcards (`editor.shape.*`) to allow specific operations like `editor.shape.create`, `editor.shape.delete`. The canvas documents the pattern; implementations use specific names.

**Edge = valid relationship**

If there's no edge between two spans in the canvas, that parent-child relationship is invalid. This catches instrumentation errors where spans are incorrectly nested.

**CLIENT kind for external calls**

All infrastructure spans use `CLIENT` kind because they represent outgoing calls to external systems (storage, database, etc.), even when running on the server.
