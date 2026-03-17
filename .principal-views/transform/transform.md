# Transform gesture

The transform gesture allows users to manipulate selected shapes through translation (moving), resizing, and rotation. These are the core editing operations that let users arrange and adjust shapes on the canvas.

## What it does

Transform supports three operations:

- **Translate** - Drag selected shapes to move them to a new position
- **Resize** - Drag selection handles to scale shapes larger or smaller
- **Rotate** - Drag the rotation handle to spin shapes around their center

All transforms operate on the current selection and can affect multiple shapes simultaneously.

## How it works

### Translate (drag-to-move)

Translation begins when the user drags a selected shape (not on a handle). The system:

1. Captures a snapshot of all selected shapes' positions
2. Calculates the delta from the drag origin to current pointer position
3. Applies the delta to all shapes, respecting parent transforms
4. Optionally snaps to grid, other shapes, or alignment guides
5. On release, finalizes the new positions

**Alt-drag cloning**: Holding Alt while dragging creates copies of the shapes and moves those instead.

**Axis constraint**: Holding Shift constrains movement to the X or Y axis.

### Resize (scale)

Resizing begins when the user drags a selection handle (corner or edge):

1. Determines the scale origin (opposite handle position)
2. Calculates scale factors based on pointer distance from origin
3. Applies scale to all shapes, respecting their resize capabilities
4. Optionally locks aspect ratio (Shift key or shape property)
5. Handles negative scales (flipping) when dragging past the origin

**Corner handles** scale both axes, **edge handles** scale one axis.

### Rotate

Rotation begins when the user drags the rotation handle:

1. Captures the rotation center (selection bounds center)
2. Calculates angle from center to pointer
3. Computes delta from initial angle
4. Applies rotation snapping (Shift = 15° increments, otherwise 1°)
5. Rotates all shapes around the common center

## Design decisions

### Snapshot-based transforms

All transforms work by capturing a snapshot of the initial state and computing final positions/scales/rotations relative to that snapshot. This ensures:
- Smooth updates without cumulative floating-point errors
- Clean cancellation by simply discarding the changes
- Consistent behavior regardless of intermediate positions

### Unified cancellation

Any transform can be cancelled by pressing Escape or through interruption. Cancellation restores shapes to their exact pre-transform state using history marks.

### Snapping hierarchy

During transforms, snapping is evaluated in priority order:
1. Grid snapping (if grid is enabled)
2. Shape-to-shape snapping (alignment guides)
3. Rotation angle snapping (for rotate)

Snapping is suppressed during fast movements and only activates when the pointer velocity drops below a threshold.
