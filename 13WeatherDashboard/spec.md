# Weather Terminal — Implementation Documentation

## Overview

A visually striking, single-page weather web app powered by the **OpenWeatherMap free API**. Users search for any city in the world, pick from a live autocomplete dropdown, and view current weather data with animated weather-reactive backgrounds. Supports multiple temperature units including a "Chaos Mode" for fun.

---

## Tech Stack

| Layer | Implementation |
|---|---|
| Frontend | Single HTML file with embedded CSS & Vanilla JS |
| Weather Data | [OpenWeatherMap Current Weather API](https://openweathermap.org/current) (free tier) |
| Hourly Forecast | [OpenWeatherMap 5-Day / 3-Hour Forecast API](https://openweathermap.org/forecast5) (free tier) |
| City Autocomplete | [OpenWeatherMap Geocoding API](https://openweathermap.org/api/geocoding-api) (free tier) |
| Typography | Google Fonts (JetBrains Mono + Inter) |

---

## Features Implemented

### 1. City Search with Autocomplete Dropdown

- Prominent search input centered on page load
- Autocomplete triggers after **2+ characters** with **300ms debounce**
- Fetches up to **5 matching cities** via Geocoding API
- Dropdown displays:
  - City name
  - State/region (if available)
  - Country code badge (styled cyan chip)
- **Keyboard navigation**: Arrow keys to move, Enter to select, Escape to close
- Loading spinner appears during fetch
- "No cities found" message for empty results
- Clicking outside closes dropdown

### 2. Weather Display Card

Displays data from `/data/2.5/weather`:

| Field | Display |
|---|---|
| City + Country | Header with country badge |
| Temperature | Large monospace gradient text |
| Feels Like | Secondary text below temp |
| Min/Max | Color-coded (amber high, blue low) |
| Condition | Icon + main + description |
| Humidity | Percentage with droplet icon |
| Wind Speed | m/s or mph based on unit |
| Visibility | Kilometers |
| Sunrise/Sunset | Local time formatted |

### 3. 24-Hour Forecast Strip

- Horizontal scrollable strip below weather card
- Shows next 24 hours of 3-hour forecast intervals
- Each card displays:
  - Time (h AM/PM format)
  - Weather icon
  - Temperature (respects unit selection)
  - Condition label
  - Precipitation % (if > 10%)
  - Wind speed
- **Date divider** appears when crossing midnight
- First card highlighted with cyan accent border
- **Staggered animation** on load (50ms delay per card)
- Scroll snap on mobile for smooth swiping
- Graceful fallback if forecast unavailable

### 4. Temperature Units

Toggle buttons for unit selection:

| Unit | Symbol | Implementation |
|---|---|---|
| Celsius | °C | `units=metric` API param |
| Fahrenheit | °F | `units=imperial` API param |
| Kelvin | K | `units=standard` API param |
| **Chaos Mode** | °? | Celsius × random multiplier (1.5–9.9) |

- Unit preference persists in `localStorage`
- Chaos multiplier displayed when active
- New multiplier generated each time Chaos Mode is selected
- All temperatures update instantly on unit change

### 5. Animated Weather Backgrounds

Dynamic full-screen weather effects that match current conditions:

| Weather | Background | Effects |
|---|---|---|
| Clear (Day) | Warm gradient | Pulsing sun with rotating light rays |
| Clear (Night) | Deep navy | Twinkling stars (100 animated) |
| Cloudy | Slate gray | Drifting blurred clouds (8 layers) |
| Rain | Dark blue-gray | Falling rain drops (100 particles) |
| Drizzle | Dark blue-gray | Light rain (50 particles) |
| Thunderstorm | Near-black | Heavy rain + dark clouds + lightning flashes |
| Snow | Cool gray-blue | Falling snowflakes with rotation |
| Mist/Fog/Haze | Soft gray | 3-layer drifting fog effect |

- Day/night detection via weather icon code
- Smooth transitions between weather states
- Subtle noise texture overlay on all backgrounds

---

## UI/UX Details

### Layout
- Single page, no routing
- Full-viewport hero on load with centered search
- Hero collapses when weather loads
- Weather card and forecast slide in with animation
- Responsive design (320px+ mobile to desktop)

### Design System
- **Colors**: Navy/slate backgrounds, white text, cyan/amber accents
- **Typography**: JetBrains Mono (numbers), Inter (labels)
- **Cards**: Semi-transparent with subtle borders
- **Accents**: Cyan for primary, amber for Chaos Mode and highs

### Error Handling
- "No cities found" in dropdown
- API error message with retry button
- Invalid API key detection
- Forecast failure doesn't block weather display

---

## API Configuration

```js
const API_KEY = "10f3fca85f99c9bb2e4c17d0858c8a8e";
const BASE_URL = "https://api.openweathermap.org";
```

**Endpoints used:**
```
GET /geo/1.0/direct?q={query}&limit=5&appid={key}
GET /data/2.5/weather?lat={lat}&lon={lon}&units={units}&appid={key}
GET /data/2.5/forecast?lat={lat}&lon={lon}&units={units}&appid={key}
```

Weather and forecast requests fire in **parallel** via `Promise.all` for faster load times.

---

## File Structure

```
13WeatherDashboard/
├── index.html    # Complete app (HTML + CSS + JS)
└── spec.md       # This documentation
```

---

## Future Enhancements

- Full 5-day forecast view
- Sparkline temperature charts
- Save favorite cities
- Browser geolocation auto-detect
- More chaos units ("hot dog lengths per fortnight")
