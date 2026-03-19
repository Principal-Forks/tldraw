# tlschema

The @tldraw/tlschema package defines the data model for tldraw documents. It provides type definitions, validators, and migrations for all record types used in the editor.

## Core concepts

### TLStore

The main interface for interacting with tldraw documents. It manages all records and provides reactive queries and transactions.

### TLRecord

A union type representing all possible record types in a tldraw document. Records are the fundamental unit of data storage.

## Record types

### Document records

- **TLDocument** - Document-level metadata (name, version)
- **TLPage** - Individual pages within a document

### Shape records

- **TLShape** - Base type for all shapes on the canvas
- Concrete shapes: Geo, Text, Draw, Arrow, Image, Video, Frame, Note, Embed, Bookmark, Line, Highlight, Group

### Instance records

- **TLInstance** - Editor instance state (current tool, zoom, etc.)
- **TLCamera** - Viewport position and zoom level
- **TLPointer** - Pointer/cursor state
- **TLPresence** - User presence for multiplayer

### Asset records

- **TLAsset** - Base type for external assets
- **ImageAsset** - Image file references
- **VideoAsset** - Video file references
- **BookmarkAsset** - URL bookmark metadata

### Binding records

- **TLBinding** - Relationships between shapes (e.g., arrow endpoints)
- **ArrowBinding** - Arrow-to-shape connections

## Support systems

### Style system

Manages shape styles (color, size, dash, etc.) with type-safe style definitions.

### Migrations

Handles schema evolution across versions. Each record type defines migrations to transform data from older versions.

### Validation

Runtime type checking using @tldraw/validate to ensure data integrity.

## Usage

```typescript
import { createTLSchema, TLStore } from '@tldraw/tlschema'

const schema = createTLSchema()
const store = new TLStore({ schema })
```
