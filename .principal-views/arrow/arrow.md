# Arrow creation

The arrow tool allows users to create arrows that connect shapes or points on the canvas. Arrows are special shapes that can bind to other shapes, automatically updating their endpoints when connected shapes move.

## What it does

Arrow creation supports:

- **Freeform arrows** - Draw arrows between any two points on the canvas
- **Bound arrows** - Connect arrows to shapes, maintaining the connection as shapes move
- **Multiple arrow styles** - Straight, curved, and elbow (right-angle) arrows
- **Smart snapping** - Arrows snap to shape edges, centers, and connection points

## How it works

### Creation flow

1. **Idle hover** - As the user hovers with the arrow tool, the system detects potential binding targets and shows snap indicators
2. **Pointing** - On pointer down, a precision timer starts to enable more accurate snapping
3. **Shape creation** - When the user drags, an arrow shape is instantiated
4. **Handle dragging** - The arrow's end handle follows the pointer, with bindings created/updated as it passes over shapes
5. **Completion** - On pointer up, the arrow is finalized with its bindings

### Binding system

When an arrow endpoint is over a shape:
- The system calculates potential snap points (center, edges, edge-points)
- Based on precision mode and arrow kind, it chooses the best snap
- A binding record is created linking the arrow to the target shape
- The binding stores the normalized anchor position (0-1 within shape bounds)

When a bound shape moves:
- The arrow's endpoint is automatically updated
- The arrow may be reparented to maintain proper z-ordering

### Snap modes

- **Center** - Snaps to shape center (for closed shapes when precise)
- **Edge** - Snaps to nearest point on shape edge
- **Edge-point** - Snaps to specific points on edges (for elbow arrows)
- **Center-axis** - Snaps to center on one axis (for elbow arrows)

## Design decisions

### Precision timeout

When hovering over a potential target, a 500ms timer starts. After this timeout, "precise" mode activates, enabling more accurate edge snapping. This prevents accidental bindings during quick movements while allowing deliberate precision when the user pauses.

### Arrow reparenting

Arrows are automatically reparented to be siblings of (or above) their bound shapes. This ensures:
- Arrows render on top of connected shapes
- Group operations work correctly
- Z-order remains logical

### Elbow arrows

Elbow arrows have special snapping that prefers axis-aligned connections, snapping to the center of edges rather than arbitrary edge points.
