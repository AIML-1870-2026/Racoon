# Mandelbrot-Julia Set Explorer

## Overview
An interactive web-based fractal explorer that displays the Mandelbrot set and its corresponding Julia sets side by side.

## Features

### Side-by-Side Display
- Mandelbrot set on the left
- Julia set on the right
- Click on Mandelbrot to select c value for Julia set
- Crosshair marks current c value on Mandelbrot

### Navigation Controls
- **Drag**: Pan the view
- **Hold still for 1 second**: Zoom in at cursor position
- **Right-click hold**: Zoom out
- **Scroll wheel**: Zoom in/out

### Famous Julia Sets Sidebar
Pre-configured buttons for notable Julia sets:
- Lightning (c = -0.7 + 0.27i)
- Dendrite (c = 0 + 1i)
- San Marco (c = -0.75 + 0i)
- Douady Rabbit (c = -0.123 + 0.745i)
- Siegel Disk (c = -0.391 - 0.587i)
- Dragon (c = -0.8 + 0.156i)
- Spiral (c = 0.285 + 0.01i)
- Galaxies (c = -0.4 + 0.6i)
- Starfish (c = 0.355 + 0.355i)
- Basilica (c = -1.25 + 0i)

### Smart View Behavior
- When zoomed out: clicking famous Julia sets moves the dot
- When zoomed in: view pans to center on selected c value
- If dot is outside visible area: view adjusts to show it

### Per-Panel Controls
- Zoom percentage input (editable)
- Reset button to restore default view

### Global Controls
- Max iterations slider (50-500)
- Color scheme selector (Rainbow, Fire, Ocean, Psychedelic, Grayscale)
- Zoom speed slider

## Technical Details

### Canvas Size
- 500x500 pixels per fractal

### Default Views
- Mandelbrot: x ∈ [-2.5, 1.0], y ∈ [-1.75, 1.75]
- Julia: x ∈ [-2.0, 2.0], y ∈ [-2.0, 2.0]

### Escape Radius
- |z| > 2 (escape when z² > 4)

### Color Mapping
Colors determined by iteration count at escape, with multiple palette options.
