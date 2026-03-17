# Drawing telemetry

## Overview

This canvas documents the telemetry events for the drawing gesture - freehand drawing and highlighting in tldraw. The Drawing state machine handles both the draw tool and highlight tool.

## User flow

1. User presses down with draw/highlight tool selected
2. Shape is created at pointer position
3. As user moves pointer, points are added to the shape
4. User can hold Shift to switch to straight-line mode
5. On pointer up, shape is marked complete
6. User can cancel or be interrupted, rolling back the shape

## Events

### drawing.started

Emitted when the drawing gesture begins. Captures initial state including input device type (pen with pressure vs mouse) and starting segment mode.

### drawing.updated

Emitted as points are added to the shape. This is a summary event emitted periodically (not per-point) to avoid telemetry overhead. Captures current point count, segment count, and line length.

### drawing.mode.changed

Emitted when the segment mode changes between free-form and straight-line drawing. Triggered by Shift key press/release.

### drawing.overflow

Emitted when the shape exceeds the maximum points limit and auto-splits into a new shape. The original shape is completed and a new shape continues from the current position.

### drawing.complete

Emitted when the drawing gesture completes successfully. Captures final statistics including total points, segments, line length, and whether the shape is closed.

### drawing.cancelled

Emitted when the drawing is cancelled or interrupted. The shape is rolled back via history mark. Captures the reason for cancellation.

## Workflows

Two workflows use these events:

- **draw-shape**: Freehand drawing with the draw tool
- **highlight-shape**: Highlighting with the highlight tool

Both use the same events but are semantically different operations.

## Implementation notes

The Drawing state machine (`packages/tldraw/src/lib/shapes/draw/toolStates/Drawing.ts`) handles:

- Pen vs mouse detection with pressure sensitivity
- Free-form and straight-line segment modes
- Automatic shape splitting when max points exceeded
- History mark for rollback on cancel/interrupt
- Closed shape detection when endpoints meet
