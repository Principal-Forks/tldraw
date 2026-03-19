# tldraw package architecture

The tldraw SDK is organized as a monorepo containing multiple packages that work together to provide an infinite canvas experience for React applications.

## Package hierarchy

The architecture follows a layered design where higher-level packages depend on lower-level ones:

### Core packages

- **@tldraw/tldraw** - The complete "batteries-included" SDK that most developers use. Includes full UI, default shapes, and tools.
- **@tldraw/editor** - The foundational canvas engine. Provides the core editing experience without any shapes, tools, or UI.
- **@tldraw/store** - Reactive client-side database for document persistence and state management.
- **@tldraw/tlschema** - Type definitions, validators, and migrations for all tldraw data structures.

### Support packages

- **@tldraw/state** - Reactive signals library (Atom, Computed) for state management.
- **@tldraw/utils** - Shared utility functions used across packages.
- **@tldraw/validate** - Lightweight validation library for runtime type checking.
- **@tldraw/assets** - Bundled icons, fonts, and translations.
- **@tldraw/sync** - Multiplayer synchronization SDK for real-time collaboration.

### Applications

- **examples** - SDK examples and demos showcasing different use cases.
- **docs** - Documentation site at tldraw.dev.
- **dotcom** - The tldraw.com web application.
- **vscode** - VSCode extension for editing .tldr files.
- **sync-worker** - Cloudflare Worker for multiplayer backend.

## Design principles

1. **Composable** - Each package can be used independently or combined with others.
2. **Reactive** - All state changes propagate automatically through the signal system.
3. **Extensible** - Custom shapes, tools, and UI components can be added easily.
4. **Type-safe** - Full TypeScript support with strict type checking.

## Common patterns

- Use `@tldraw/tldraw` for most applications
- Use `@tldraw/editor` when you need full control over shapes and UI
- Add `@tldraw/sync` for multiplayer support
