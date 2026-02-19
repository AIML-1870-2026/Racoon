# RGB Color Explorer - Specification

## Overview
An interactive web application for exploring RGB color mixing, palette generation, accessibility checking, and color blindness simulation.

## Features

### 1. RGB Color Mixer with Flashlight Visualization
- **Visual metaphor**: Three flashlights (red, green, blue) shine colored light beams that overlap and mix in the center
- **Primary color selection**: Users select two "primary" colors using explicit Primary 1 / Primary 2 button pickers
- **Lightness channel**: The third (non-selected) color acts as a "lightness" control that shifts the mixed color toward white
- **Constraint system**: The lightness slider is capped at the minimum value of the two primary colors, visually shown with a MAX marker and greyed-out disabled zone
- **Live explanation**: Dynamic text explains the current mix (e.g., "Red + Green = Yellow. Blue adds lightness toward white.")

### 2. Color Wheel Discovery
- Interactive color wheel that reveals colors as you discover them through mixing
- Shows count of discovered colors
- Option to reveal the full color wheel for reference
- Colors positioned by hue (angle) and saturation (distance from center)

### 3. Palette Generator
- Generates harmonious color palettes based on the current mixed color
- Palette types:
  - Complementary (2 colors)
  - Triadic (3 colors)
  - Analogous (3 colors)
  - Split Complementary (3 colors)
  - Tetradic (4 colors)
  - Monochromatic (5 colors)
- Accessible mode (AA/AAA) adjusts palette colors to meet WCAG contrast requirements

### 4. Contrast Checker
- Check WCAG contrast ratios between any two colors
- Input via color picker or hex code
- Click palette colors to select them for contrast checking
- Shows pass/fail status for:
  - AA Normal (4.5:1)
  - AA Large (3:1)
  - AAA Normal (7:1)
  - AAA Large (4.5:1)
- Live preview of text on background

### 5. Color Blindness Simulator
- Collapsible side panel with hamburger menu toggle
- Simulates different types of color vision deficiency:
  - Normal Vision
  - Protanopia (red-blind)
  - Deuteranopia (green-blind)
  - Tritanopia (blue-blind)
  - Protanomaly (red-weak)
  - Deuteranomaly (green-weak)
  - Tritanomaly (blue-weak)
  - Achromatopsia (complete color blindness)
- Applies filter to entire page for accurate simulation

## Technical Implementation

### Color Conversions
- RGB to HEX
- RGB to HSL
- HSL to RGB
- HEX to RGB

### WCAG Contrast Calculation
- Uses relative luminance formula
- Calculates contrast ratio per WCAG 2.1 guidelines

### Color Blindness Simulation
- SVG feColorMatrix filters applied to body element
- Matrices based on color vision deficiency research

## Educational Purpose
This tool teaches the concept that in RGB additive color mixing:
- Two primary colors combine to create secondary colors (Yellow, Cyan, Magenta)
- The third color channel adds "lightness" toward white
- The lightness channel is constrained to not exceed the minimum of the two primaries

## File Structure
- `index.html` - Single-file application containing HTML, CSS, and JavaScript
