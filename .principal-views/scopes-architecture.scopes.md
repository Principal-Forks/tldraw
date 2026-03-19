# tldraw instrumentation scopes

## Overview

This document describes the OpenTelemetry instrumentation scopes for tldraw. Scopes are divided into two categories:

1. **Library scopes** - Core SDK packages embedded in applications
2. **Application scopes** - Deployed services with their own telemetry

## Scope hierarchy

```
Application (tldraw-web)
  ├── App scope (@tldraw/dotcom-client)     ← app-specific spans
  └── Library scopes (embedded):
        ├── @tldraw/editor                  ← editor.* spans
        ├── @tldraw/tldraw                  ← tldraw.* spans
        ├── @tldraw/store                   ← store.* spans
        ├── @tldraw/state                   ← state.* spans
        └── @tldraw/sync                    ← sync.client.* spans
```

A single service can emit spans from multiple scopes - its own app scope plus any embedded library scopes.

## Core SDK scopes (library scopes)

These scopes are embedded in applications and emit telemetry from within them.

| Scope | Package | Span Patterns | Description |
|-------|---------|---------------|-------------|
| `@tldraw/editor` | packages/editor | `editor.canvas.*`, `editor.shape.*`, `editor.tool.*` | Core editor engine - canvas, shapes, tools, geometry |
| `@tldraw/tldraw` | packages/tldraw | `tldraw.ui.*`, `tldraw.action.*`, `tldraw.export.*` | Full SDK - default UI, actions, shortcuts, export |
| `@tldraw/store` | packages/store | `store.transaction.*`, `store.query.*`, `store.persist.*` | Reactive database - transactions, queries, persistence |
| `@tldraw/state` | packages/state | `state.compute.*`, `state.effect.*` | Reactive signals - computations, effects |
| `@tldraw/sync` | packages/sync | `sync.client.*`, `sync.socket.*`, `sync.presence.*` | Multiplayer client - WebSocket, presence, sync protocol |

### Embedding relationships

| Library Scope | Embedded In |
|---------------|-------------|
| `@tldraw/editor` | dotcom-client, docs, vscode-extension |
| `@tldraw/tldraw` | dotcom-client, docs, vscode-extension |
| `@tldraw/store` | dotcom-client |
| `@tldraw/state` | (used internally by editor/store) |
| `@tldraw/sync` | dotcom-client |

## Application scopes

### Multiplayer (server scopes)

| Scope | Service | Span Patterns | Description |
|-------|---------|---------------|-------------|
| `@tldraw/sync-worker` | tldraw-multiplayer | `sync.server.*`, `durable-object.*`, `websocket.*` | WebSocket server, Durable Objects, doc sync |
| `@tldraw/bemo-worker` | tldraw-bemo | `merge.*`, `transform.*` | OT/conflict resolution |

### Storage & assets (server scopes)

| Scope | Service | Span Patterns | Description |
|-------|---------|---------------|-------------|
| `@tldraw/asset-upload-worker` | tldraw-assets | `upload.*`, `r2.*` | File uploads, R2 storage |
| `@tldraw/image-resize-worker` | tldraw-images | `image.*`, `resize.*` | Image optimization |

### Frontend (app scopes)

| Scope | Service | Span Patterns | Description |
|-------|---------|---------------|-------------|
| `@tldraw/dotcom-client` | tldraw-web | `app.*`, `route.*`, `network.*` | App-specific logic, routing |
| `@tldraw/docs` | tldraw-docs | `page.*`, `search.*` | Page navigation, search |

### Other (app scopes)

| Scope | Service | Span Patterns | Description |
|-------|---------|---------------|-------------|
| `@tldraw/analytics-worker` | tldraw-consent | `consent.*` | Cookie consent |
| `@tldraw/vscode-extension` | tldraw-vscode | `extension.*`, `file.*` | Extension lifecycle, file ops |
| `@tldraw/zero-cache` | tldraw-zero-cache | `cache.*`, `db.*`, `replication.*` | Cache, PostgreSQL, replication |

## Span naming conventions

### Library scopes (core SDK)

| Scope | Prefix | Examples |
|-------|--------|----------|
| `@tldraw/editor` | `editor.` | `editor.canvas.render`, `editor.shape.create`, `editor.tool.activate` |
| `@tldraw/tldraw` | `tldraw.` | `tldraw.ui.toolbar.click`, `tldraw.action.undo`, `tldraw.export.svg` |
| `@tldraw/store` | `store.` | `store.transaction.commit`, `store.query.execute`, `store.persist.save` |
| `@tldraw/state` | `state.` | `state.compute.derived`, `state.effect.run` |
| `@tldraw/sync` | `sync.client.` | `sync.client.connect`, `sync.socket.message`, `sync.presence.update` |

### Application scopes

| Scope | Prefix | Examples |
|-------|--------|----------|
| `@tldraw/sync-worker` | `sync.server.`, `durable-object.` | `sync.server.handle`, `durable-object.fetch` |
| `@tldraw/bemo-worker` | `merge.`, `transform.` | `merge.changes`, `transform.operation` |
| `@tldraw/asset-upload-worker` | `upload.`, `r2.` | `upload.validate`, `r2.put` |
| `@tldraw/image-resize-worker` | `image.`, `resize.` | `image.fetch`, `resize.execute` |
| `@tldraw/dotcom-client` | `app.`, `route.` | `app.init`, `route.navigate` |
| `@tldraw/docs` | `page.`, `search.` | `page.load`, `search.query` |
| `@tldraw/analytics-worker` | `consent.` | `consent.check`, `consent.update` |
| `@tldraw/vscode-extension` | `extension.`, `file.` | `extension.activate`, `file.open` |
| `@tldraw/zero-cache` | `cache.`, `db.`, `replication.` | `cache.get`, `db.query`, `replication.sync` |

## Distinguishing client vs server sync

Both `@tldraw/sync` (client) and `@tldraw/sync-worker` (server) handle sync operations. They use different prefixes:

| Operation | Client Scope | Server Scope |
|-----------|--------------|--------------|
| Connection | `sync.client.connect` | `sync.server.accept` |
| Message | `sync.socket.send` | `websocket.receive` |
| Presence | `sync.presence.update` | `sync.server.presence.broadcast` |

## Validation

All scopes are validated against `library.yaml`:

```yaml
owned-scopes:
  # Core SDK libraries
  - "@tldraw/editor"
  - "@tldraw/tldraw"
  - "@tldraw/store"
  - "@tldraw/state"
  - "@tldraw/sync"
  # Application scopes
  - "@tldraw/sync-worker"
  - "@tldraw/bemo-worker"
  # ... etc
```

## Design decisions

**Library scopes vs app scopes**

Library scopes instrument reusable SDK code. They're embedded in applications but maintain their own identity. This allows:
- Filtering by library vs app code in traces
- Understanding which SDK layer caused an issue
- Consistent span naming across all apps using tldraw

**One scope per package/service**

Each npm package and deployed service has exactly one scope. This keeps scope boundaries clear and predictable.

**Span prefix ownership**

Each scope "owns" certain span prefixes. Prefixes like `editor.*` always come from `@tldraw/editor`, making traces self-documenting.
