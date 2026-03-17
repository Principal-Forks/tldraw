# Selection gesture

The selection gesture allows users to select shapes on the canvas for manipulation. Selection is the foundation for all editing operations - shapes must be selected before they can be transformed, styled, deleted, or grouped.

## What it does

Selection supports multiple methods:
- **Single click** - Click on a shape to select it, replacing any existing selection
- **Shift-click** - Add or remove a shape from the current selection (toggle)
- **Brush selection** - Drag on the canvas to create a marquee rectangle that selects all shapes it touches
- **Click on canvas** - Click on empty canvas to deselect all shapes

## How it works

The selection gesture is implemented as a state machine within the SelectTool:

1. **Idle state** - Waiting for user interaction
2. **PointingShape** - User has clicked on a shape, waiting to see if they drag (transform) or release (select)
3. **PointingCanvas** - User has clicked on canvas, waiting to see if they drag (brush) or release (deselect)
4. **Brushing** - User is dragging to create a marquee selection rectangle

The system uses hit testing to determine what's under the pointer and geometry intersection tests during brush selection to find shapes that overlap the marquee rectangle.

## Design decisions

### Immediate vs deferred selection

When clicking a shape, selection can happen immediately (on pointer down) or be deferred (on pointer up). Immediate selection occurs when:
- The shape has an onClick handler
- The shape is already selected (enables quick transforms)
- The shape is a group being entered

Deferred selection allows the system to distinguish between "select" and "start dragging".

### Brush selection filtering

During brush selection, certain shapes are excluded:
- Locked shapes cannot be selected
- Group shapes are transparent to selection (children are selected instead)
- Shapes on locked pages are excluded

### Selection persistence

When a brush selection is cancelled (e.g., by pressing Escape), the original selection from before the brush started is restored. This allows users to abort an accidental brush without losing their previous selection.
