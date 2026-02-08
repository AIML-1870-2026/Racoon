# Turing Pattern Explorer

## Overview
An interactive Gray-Scott reaction-diffusion simulation that generates Turing patterns - the mathematical basis for many natural patterns like animal spots, stripes, and coral formations.

## Features

### Parameter Space Control
- Interactive 2D graph showing feed rate (f) vs kill rate (k)
- Preset pattern markers: Spots, Stripes, Waves, Mitosis, Coral, Maze
- Click and drag points to adjust parameters in real-time
- Click anywhere on empty space to move the first simulation point

### Multiple Simulations
- Add multiple simulations running side-by-side
- Each simulation has its own draggable control point on the parameter space
- Color-coded identification (cyan, red, teal, yellow, etc.)
- Independent pause/reset/seed controls per simulation

### Seed Placement
- Click "Add Seed" to enter placement mode
- Click anywhere on the simulation to place reaction seeds
- Mode stays active for multiple placements
- Auto-respawn: if pattern dies, a new seed appears after 2 seconds

### Visual Customization
- Color slider to adjust the hue of all simulations
- Real-time color changes without resetting

## The Math

### Gray-Scott Model
Two chemicals U and V react and diffuse:

```
∂u/∂t = Du∇²u - uv² + f(1-u)
∂v/∂t = Dv∇²v + uv² - (k+f)v
```

Where:
- **Du, Dv**: Diffusion rates (0.16 and 0.08)
- **f**: Feed rate - how fast U is added
- **k**: Kill rate - how fast V is removed
- **∇²**: Laplacian (spatial diffusion)

### Pattern Types
| Pattern | Feed (f) | Kill (k) | Description |
|---------|----------|----------|-------------|
| Spots | 0.020 | 0.050 | Isolated circular spots |
| Stripes | 0.040 | 0.060 | Parallel stripe patterns |
| Waves | 0.030 | 0.055 | Propagating wave fronts |
| Mitosis | 0.040 | 0.0625 | Spots that divide |
| Coral | 0.060 | 0.0625 | Branching coral-like structures |
| Maze | 0.0375 | 0.050 | Labyrinthine patterns |

## Controls

| Control | Action |
|---------|--------|
| Parameter Space | Drag points or click anywhere to adjust f/k |
| Color Slider | Drag to change pattern hue |
| Add Simulation | Create new simulation panel |
| Reset | Restart simulation with center seed |
| Pause/Resume | Toggle simulation |
| Add Seed | Enter seed placement mode |
| × Button | Remove simulation (when 2+) |

## Technical Details
- Grid size: 200×200 pixels
- Display size: 400×400 pixels
- 10 iterations per animation frame
- Wrapping boundary conditions (toroidal topology)
