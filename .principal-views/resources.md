# tldraw service resources & scopes

## Overview

This document describes the OpenTelemetry resources and instrumentation scopes for the tldraw platform. Each deployable service is represented as a resource with its own instrumentation scope.

## Resource hierarchy

```
Resource (service.name + service.namespace)
  └── Scope (instrumentation library)
        └── Spans (operations within that service)
```

## Namespaces

| Namespace | Purpose |
|-----------|---------|
| `tldraw.backend` | Server-side services (Cloudflare Workers, caches) |
| `tldraw.frontend` | Client-side applications (web, docs) |
| `tldraw.ide` | IDE integrations (VS Code) |

## Backend services (Cloudflare Workers)

### Multiplayer services

| Resource | Service Name | Scope | Description |
|----------|--------------|-------|-------------|
| tldraw-multiplayer | `tldraw-multiplayer` | `@tldraw/sync-worker` | Real-time sync & collaboration using Durable Objects |
| tldraw-bemo | `tldraw-bemo` | `@tldraw/bemo-worker` | Merge server for operational transformation |

**Durable objects in sync-worker:**
- `TLFileDurableObject` - Document state management
- `TLUserDurableObject` - User presence tracking
- `TLPostgresReplicator` - Database replication
- `TLLoggerDurableObject` - Log aggregation
- `TLStatsDurableObject` - Statistics collection

### Storage services

| Resource | Service Name | Scope | Description |
|----------|--------------|-------|-------------|
| tldraw-assets | `tldraw-assets` | `@tldraw/asset-upload-worker` | Asset upload handling (R2 storage) |
| tldraw-images | `tldraw-images` | `@tldraw/image-resize-worker` | On-the-fly image optimization |

### Analytics services

| Resource | Service Name | Scope | Description |
|----------|--------------|-------|-------------|
| tldraw-consent | `tldraw-consent` | `@tldraw/analytics-worker` | Cookie consent & tracking preferences |

### Database services

| Resource | Service Name | Scope | Description |
|----------|--------------|-------|-------------|
| tldraw-zero-cache | `tldraw-zero-cache` | `@tldraw/zero-cache` | Rocicorp Zero database cache layer |

## Frontend services

| Resource | Service Name | Scope | Description |
|----------|--------------|-------|-------------|
| tldraw-web | `tldraw-web` | `@tldraw/dotcom-client` | tldraw.com web application |
| tldraw-docs | `tldraw-docs` | `@tldraw/docs` | tldraw.dev documentation site |

## IDE services

| Resource | Service Name | Scope | Description |
|----------|--------------|-------|-------------|
| tldraw-vscode | `tldraw-vscode` | `@tldraw/vscode-extension` | VS Code extension |

## Common resource attributes

All services share these resource attributes:

| Attribute | Example | Description |
|-----------|---------|-------------|
| `service.name` | `tldraw-multiplayer` | Logical service name |
| `service.namespace` | `tldraw.backend` | Service grouping |
| `service.version` | `1.2.3` | Service version (from CF_VERSION_METADATA or git) |
| `deployment.environment` | `production` | Environment: `development`, `staging`, `production` |

### Cloudflare Worker-specific attributes

| Attribute | Example | Description |
|-----------|---------|-------------|
| `cloudflare.worker.name` | `tldraw-multiplayer` | Worker script name |
| `cloudflare.durable_object.class` | `TLFileDurableObject` | Durable Object class (if applicable) |
| `cloudflare.storage.type` | `r2` | Storage type (if applicable) |
| `cloudflare.storage.bucket` | `uploads` | Storage bucket (if applicable) |

### Browser-specific attributes

| Attribute | Example | Description |
|-----------|---------|-------------|
| `client.runtime.name` | `browser` | Runtime environment |
| `browser.brands` | `Chrome 120` | User agent brands |

## Environment configuration

Services are deployed to multiple environments:

| Environment | Domain Pattern | Worker Suffix |
|-------------|----------------|---------------|
| `development` | `*.localhost` | `dev-*` |
| `staging` | `*.tldraw.xyz` | `main-*` or `canary-*` |
| `production` | `*.tldraw.com` | (no prefix) |

## Existing observability

### Error tracking (Sentry)
- Browser: `@sentry/react` in dotcom client
- Workers: `toucan-js` via `createSentry()` factory
- Configuration: `SENTRY_DSN` environment variable

### Metrics (Cloudflare Analytics Engine)
- Datasets: `MEASURE`, `BEMO_ANALYTICS`
- Used via `writeDataPoint()` function

### User analytics
- PostHog for event tracking
- React-GA4 for Google Analytics

## Design decisions

**One scope per resource**

Following OTEL best practices, each service has exactly one instrumentation scope. Scopes identify the instrumentation library, not features or subsystems. Different operations are differentiated via span names.

**Namespace grouping**

Services are grouped by their deployment context:
- Backend services run on Cloudflare's edge network
- Frontend services run in user browsers
- IDE services run in VS Code's extension host

**Environment as resource attribute**

Environment (`development`, `staging`, `production`) is a resource attribute, not part of the service name. This allows consistent service identity across environments while enabling environment-specific filtering.
