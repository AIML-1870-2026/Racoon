# Readable Explorer - Specification

## Overview
Readable Explorer is a web-based color accessibility tool that helps users check contrast ratios, simulate color blindness, and ensure their color choices meet WCAG accessibility guidelines.

## Core Features

### 1. Text Color Controls
- RGB sliders (0-255) for Red, Green, and Blue channels
- Editable integer input fields synced with sliders (bidirectional)
- Color preview box showing current text color
- Real-time luminosity display

### 2. Background Color Controls
- RGB sliders (0-255) for Red, Green, and Blue channels
- Editable integer input fields synced with sliders (bidirectional)
- Color preview box showing current background color
- Real-time luminosity display

### 3. Text Size Control
- Slider for adjusting text size (12px - 48px)
- Editable integer input field synced with slider
- Affects sample text in contrast preview

### 4. Color Presets Dropdown
- **Custom** - Default, allows manual RGB adjustment
- **Contrast Examples:**
  - High Contrast (black on white)
  - Low Contrast (similar grays)
  - WCAG Fail Pairing (colors that fail accessibility)
- **Brand Colors:**
  - Google (link blue on white)
  - Bing (blue on white)
  - Yahoo (purple on white)
- **Accessibility:**
  - AA Colors (~4.5:1 ratio)
  - AAA Colors (~7:1 ratio)

### 5. Color Blindness Swatch
- Full spectrum rainbow gradient
- Demonstrates how colors appear under different color blindness simulations

### 6. Contrast Preview
- Sample Lorem ipsum text displayed with selected text/background colors
- Text size reflects the text size slider value
- Displays calculated contrast ratio (X.XX:1 format)

### 7. WCAG Results Display
- **AA Normal** - Pass/Fail indicator (4.5:1 required)
- **AA Large** - Pass/Fail indicator (3:1 required)
- **AAA Normal** - Pass/Fail indicator (7:1 required)
- **AAA Large** - Pass/Fail indicator (4.5:1 required)
- Green highlight for PASS, red highlight for FAIL

### 8. Color Blindness Simulator (Side Panel)
- Accessible via "Views" toggle button
- Simulation options:
  - Normal Vision
  - Protanopia (red-blind)
  - Deuteranopia (green-blind)
  - Tritanopia (blue-blind)
  - Protanomaly (red-weak)
  - Deuteranomaly (green-weak)
  - Tritanomaly (blue-weak)
  - Achromatopsia (complete color blindness)
- Applies SVG filter to entire page to simulate color vision deficiency

## Technical Requirements

### Contrast Ratio Calculation
- Uses WCAG relative luminance formula
- Formula: (L1 + 0.05) / (L2 + 0.05) where L1 is lighter, L2 is darker
- Luminance calculated using sRGB color space conversion

### Real-time Updates
- All changes update immediately without page refresh
- Sliders and inputs stay synchronized
- Contrast ratio recalculates on any color change

### Accessibility
- Keyboard navigable controls
- Clear visual indicators for pass/fail states
- Link to WCAG guidelines for reference

## File Structure
```
10Readable/
├── index.html    # Single-file application (HTML, CSS, JS)
└── spec.md       # This specification document
```

## Browser Support
- Modern browsers with CSS filter support
- SVG filter support for color blindness simulation
