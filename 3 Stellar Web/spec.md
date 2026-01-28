# Stellar Web - Technical Specification

## Overview
Stellar Web is an interactive 3D node network visualization that renders animated geometric shapes composed of interconnected nodes and edges. The application runs entirely in the browser using HTML5 Canvas and vanilla JavaScript.

## Features

### Shape Options
- **Random**: Nodes scattered randomly across the screen
- **Sphere**: 3D sphere with nodes distributed on surface and interior
- **Cube**: 3D cube with nodes along edges and interior
- **Pyramid**: 3D pyramid with nodes along edges and interior

### Interactive Controls

| Control | Range | Default | Description |
|---------|-------|---------|-------------|
| Shape | Dropdown | Random | Selects the overall shape formation |
| Node Size | 1 - 8 | 2 | Size of individual nodes |
| Node Frequency | 20 - 400 | 150 | Total number of nodes |
| Node Lifespan | 1 - 15s | 5s | How long inner nodes live before respawning |
| Connection Thickness | 0.5 - 5 | 1.5 | Thickness of connection lines |
| Connectivity Radius | 50 - 300 | 150 | Maximum distance for node connections |
| Node Color | Color picker | #ffffff | Color of nodes |
| Connection Color | Color picker | #6496ff | Color of connection lines |

### Network Statistics (Real-time)
- **Total Edges**: Number of active connections between nodes
- **Avg Connections/Node**: Average number of connections per visible node
- **Network Density**: Percentage of possible connections that exist (actual edges / max possible edges)

### User Interactions
- **Drag on canvas**: Rotate 3D shapes (Sphere, Cube, Pyramid)
- **Drag on node**: Reposition individual nodes
- **Auto-rotation**: Shapes rotate automatically when not being dragged
- **Touch support**: Full touch interaction for mobile devices

## Technical Architecture

### Node Types
1. **Outer Nodes (40%)**: Permanent nodes positioned on shape surface/edges
   - Never despawn
   - Form the visible wireframe of the shape
   - Evenly distributed using mathematical algorithms (Fibonacci sphere, edge distribution)

2. **Inner Nodes (60%)**: Dynamic nodes that spawn inside the shape
   - Fade in over 500ms
   - Live for configurable lifespan (with ±30% variance)
   - Fade out over 500ms
   - Respawn at new random positions

### 3D Rendering
- Uses rotation matrices for Y-axis and X-axis rotation
- Perspective projection with configurable depth (600px perspective distance)
- Shapes centered at viewport center with 30% viewport scale

### Connection Algorithm
- O(n²) pairwise distance calculation
- Connections drawn for nodes within connectivity radius
- Opacity based on distance (closer = more opaque)
- Line thickness scales with opacity

### Performance Considerations
- RequestAnimationFrame for smooth 60fps animation
- Delta time calculation for consistent animation speed
- Efficient canvas clearing and redrawing

## Browser Compatibility
- Modern browsers with HTML5 Canvas support
- CSS backdrop-filter for control panel blur effect
- Touch events for mobile interaction

## File Structure
```
3 Stellar Web/
├── index.html    # Single-file application (HTML + CSS + JS)
└── spec.md       # This specification document
```

## Live Demo
https://racoonma392.github.io/3-stellar-web/

## Repository
https://github.com/racoonma392/3-stellar-web
