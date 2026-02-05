# Fish Simulation (Koi Pond)

A boid-based simulation featuring koi fish swimming in a pond with lilypads and lotus flowers.

## Features

### Koi Fish
- Teardrop-shaped bodies with wiggling tails
- Color variations: orange, red, white, and gold with contrasting spots
- Smooth, natural swimming motion using boid flocking algorithm

### Flocking Behaviors
- **Separation**: Fish avoid crowding nearby fish
- **Alignment**: Fish steer toward the average heading of neighbors
- **Cohesion**: Fish move toward the center of nearby fish

### Environment
- Lilypads with notch detail
- Lotus flowers with 3 layers of rounded petals (8 outer, 5 middle, 3 inner)
- Fish swim around obstacles (lilypads)

## Controls

### Sliders
| Control | Range | Description |
|---------|-------|-------------|
| Fish Count | 10-400 | Number of fish in the pond |
| Separation Weight | 0-5 | How strongly fish avoid crowding |
| Alignment Weight | 0-5 | How much fish match neighbor heading |
| Cohesion Weight | 0-5 | How strongly fish move toward group center |
| Neighbor Radius | 20-150 | Detection range for nearby fish |
| Max Speed | 1-10 | Maximum swimming speed |

### Buttons
- **Pause/Play**: Toggle simulation
- **Reset**: Reset all parameters and regenerate fish/lilypads

### Presets
- **Schooling**: High alignment, medium cohesion, low separation - creates coordinated movement
- **Chaotic Swarm**: Low alignment, low cohesion, small radius - erratic scattered behavior
- **Tight Cluster**: High cohesion, moderate separation - fish clump together

## Statistics
- **Fish Count**: Current number of fish
- **Avg Speed**: Average velocity of all fish
- **Avg Neighbors**: Average neighbors per fish
- **FPS**: Frames per second

## Technical Details
- Built with HTML5 Canvas
- Pure JavaScript, no dependencies
- Responsive canvas sizing
- Real-time parameter adjustments
