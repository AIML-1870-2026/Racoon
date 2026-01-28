# Starfield Particle System - Specification

## Overview
An interactive starfield particle system that creates animated stars radiating from the center of the screen with customizable trails and patterns. Built as a single HTML file with embedded CSS and JavaScript.

## Features

### Particle System
- Stars radiate outward from the center of the screen
- Each star leaves a trail that draws its path
- Stars accelerate as they move further from the center
- Glow effect on star heads for visual depth
- Stars automatically reset when they leave the screen

### Patterns
Four distinct radiation patterns are available:

1. **Default** - Stars radiate in random directions from a circular origin at the center
2. **Star** - Stars spawn from a 5-pointed star-shaped outline and radiate outward
3. **Flower** - Stars follow curving spiral paths with rotational movement
4. **Spiral** - Stars follow a continuous spiral pattern emanating from the origin

### Controls
All controls are accessible via a dropdown panel in the top-left corner.

#### Pattern Selector
- Dropdown menu to select between Default, Star, Flower, and Spiral patterns

#### Manual Mode Toggle
- Switch to enable/disable manual mode
- When ON: Stars only emit when user clicks or holds down on the canvas
- When OFF: Stars animate automatically
- Displays instructional popup message when enabled

#### Sliders
| Control | Range | Default | Description |
|---------|-------|---------|-------------|
| Star Count | 50-500 | 200 | Number of stars in the particle system |
| Speed | 1-20 | 8 | Velocity of star movement |
| Trail Length | 1-50 | 20 | Length of the trail behind each star |
| Star Size | 1-5 | 2 | Size of star particles |
| Trail Fade | 1-30 | 10 | Rate at which trails fade out |

#### Color Gradient
- Visual gradient bar showing full hue spectrum (0-360)
- Click or drag to select star color
- White selector indicator shows current position

### User Interactions
- **Click dropdown header** - Toggle control panel open/closed
- **Press H key** - Toggle control panel open/closed
- **Click/hold on canvas** (Manual mode) - Emit stars from origin

## Technical Details

### Technologies
- HTML5 Canvas for rendering
- Vanilla JavaScript (ES6+)
- CSS3 with transitions and backdrop-filter

### Browser Compatibility
- Modern browsers supporting HTML5 Canvas
- CSS backdrop-filter for blur effects

### Performance
- RequestAnimationFrame for smooth 60fps animation
- Trail rendering using canvas fade technique
- Efficient star pooling (reuses inactive stars in manual mode)

## File Structure
```
2 Starfield/
├── index.html    # Main application file (single-file implementation)
└── spec.md       # This specification document
```

## Usage
1. Open `index.html` in a web browser
2. Use the control panel to adjust settings
3. Select different patterns to change the animation style
4. Enable Manual mode to control star emission with mouse clicks
5. Experiment with color, speed, and trail settings
