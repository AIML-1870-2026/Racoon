# Weather Website — Product Specification

## Overview

A visually striking, single-page weather web app powered by the **OpenWeatherMap free API**. Users search for any city in the world, pick from a live autocomplete dropdown, and view current weather data in their preferred temperature unit — including a surprise "Chaos Mode" unit.

---

## Tech Stack

| Layer | Choice |
|---|---|
| Frontend | HTML / CSS / Vanilla JS (or React) |
| Weather Data | [OpenWeatherMap Current Weather API](https://openweathermap.org/current) (free tier) |
| Hourly Forecast | [OpenWeatherMap 5-Day / 3-Hour Forecast API](https://openweathermap.org/forecast5) (free tier) |
| City Autocomplete | [OpenWeatherMap Geocoding API](https://openweathermap.org/api/geocoding-api) (free tier) |
| API Key | User-supplied free key from openweathermap.org — stored in a config or `.env` file |

---

## Features

### 1. City Search with Autocomplete Dropdown

- A prominent search input is displayed on page load.
- As the user types (after **2+ characters**), the app calls the **OpenWeatherMap Geocoding API** (`/geo/1.0/direct`) to fetch up to **5 matching cities** worldwide.
- Results appear in a styled dropdown below the input, showing:
  - City name
  - State / region (if available)
  - Country code (e.g. `US`, `JP`, `DE`)
- Selecting a city from the dropdown triggers the weather fetch.
- Pressing **Enter** selects the top result.
- Clicking outside the dropdown closes it.
- If no matches are found, a "No cities found" message appears in the dropdown.

### 2. Weather Display

Once a city is selected, the following data is fetched from `/data/2.5/weather` and displayed:

| Field | Source |
|---|---|
| City name + country | `name`, `sys.country` |
| Current temperature | `main.temp` |
| Feels like | `main.feels_like` |
| Min / Max temp | `main.temp_min`, `main.temp_max` |
| Weather condition | `weather[0].main`, `weather[0].description` |
| Weather icon | `weather[0].icon` (via `https://openweathermap.org/img/wn/{icon}@2x.png`) |
| Humidity | `main.humidity` |
| Wind speed | `wind.speed` |
| Visibility | `visibility` |
| Sunrise / Sunset | `sys.sunrise`, `sys.sunset` (converted to local time) |

### 3. Hourly Forecast for the Day

After the current weather card, an **hourly forecast strip** is displayed showing the remaining forecast entries for the current local calendar day, plus enough entries to fill a full 24-hour window if fewer than 8 remain today.

**Data source:** `/data/2.5/forecast` (OWM free tier — returns forecasts in 3-hour intervals, up to 5 days, max 40 entries).

#### Displayed Per Hour Slot

| Field | Source |
|---|---|
| Time | `dt_txt` — formatted as `h AM/PM` in the city's local timezone |
| Temperature | `main.temp` (respects the active unit selection, including Chaos Mode) |
| Weather icon | `weather[0].icon` |
| Condition label | `weather[0].main` (short label, e.g. "Rain", "Clear") |
| Precipitation probability | `pop` — shown as a percentage if > 10% |
| Wind speed | `wind.speed` |

#### Filtering Logic

1. Fetch all forecast entries for the selected city.
2. Filter to entries whose `dt` (Unix timestamp) falls within the **next 24 hours** from the current time — this guarantees a full day's worth of slots (up to 8 entries at 3-hour intervals).
3. If the city's local midnight falls within that window, add a subtle **date divider** between the last entry of today and the first entry of tomorrow.

#### UI / Layout

- **Horizontal scrollable strip** of cards sitting directly below the current weather card.
- Each card is compact: time at the top, icon in the middle, temperature at the bottom.
- Precipitation probability badge (e.g. `💧 60%`) appears on the card when `pop > 0.1`.
- The card representing the **closest upcoming hour** is highlighted (e.g. slightly larger, accented border).
- On desktop, all 8 cards are visible without scrolling if space allows. On mobile, the strip scrolls horizontally with momentum scrolling (`overflow-x: auto; scroll-snap-type: x mandatory`).
- Cards animate in sequentially with a staggered entrance (each card delayed by ~50ms) when the forecast loads.
- When the user changes the temperature unit, all hourly temperatures update instantly without re-fetching.

#### Empty / Error State

- If the forecast API call fails independently of the current weather call, show a muted inline message ("Hourly forecast unavailable") inside the strip area — do not block the main weather card.

---

### 4. Temperature Units

A unit selector (toggle buttons or a dropdown) lets the user choose how temperatures are displayed:

| Unit | Label | Description |
|---|---|---|
| Celsius | `°C` | Standard metric. Query param: `units=metric` |
| Fahrenheit | `°F` | Standard imperial. Query param: `units=imperial` |
| Kelvin | `K` | Scientific absolute scale. Query param: `units=standard` |
| **Chaos Mode** | `°?` | Celsius value multiplied by a **random float** between `1.5` and `9.9`, re-rolled each time the unit is selected. Display the multiplier in small text so the user knows what happened. |

- The selected unit persists across city searches (stored in component state or `localStorage`).
- All temperature fields (current, feels like, min, max, and all hourly forecast temps) update when the unit changes — **without** re-fetching from the API (convert locally where possible, or re-fetch for Kelvin/Celsius/Fahrenheit since OWM handles those server-side).

---

## UI / UX Requirements

### Layout
- **Single page**, no routing needed.
- Full-viewport hero with the search bar centered on load.
- After a city is selected, the weather card appears below (or the hero shrinks and the card slides in), followed immediately by the hourly forecast strip.
- Responsive: works on mobile (320px+) and desktop.

### Search Input
- Placeholder: `"Search for a city..."`
- Debounce autocomplete requests by **300ms** to avoid hammering the API.
- Show a subtle loading spinner inside the input while fetching suggestions.

### Dropdown
- Keyboard navigable: **↑ / ↓** arrows to move, **Enter** to select, **Escape** to close.
- Each row shows: `City, Region, [COUNTRY]`
- Highlight hovered/focused row.

### Weather Card
- Prominent temperature display (large, styled number + unit).
- Weather icon + condition label side by side.
- Secondary stats (humidity, wind, visibility, sunrise/sunset) in a clean grid.
- Subtle background or color shift based on weather condition (e.g. blue-grey for rain, warm amber for clear skies).

### Error States
- **City not found**: Shown if geocoding returns 0 results.
- **API error / network failure**: Friendly error message with a retry button.
- **Invalid API key**: Clear prompt to check the key in config.

---

## Aesthetic Direction

Commit to a **dark, atmospheric, meteorological-data-terminal** aesthetic:

- **Color palette**: Deep navy/slate backgrounds, cool-white text, electric cyan or amber accents.
- **Typography**: A monospaced or semi-monospaced display font for temperature numbers; a clean sans-serif for labels.
- **Texture**: Subtle noise or grain overlay on the background. Optional: faint animated gradient that slowly shifts based on weather condition.
- **Motion**: Search dropdown fades/slides in. Weather card animates in on first load. Temperature number ticks/counts up to its value.
- **Icons**: Use the OWM icon URLs, but style them with a filter or CSS to match the palette.

---

## API Configuration

```js
// config.js (or .env)
const API_KEY = "10f3fca85f99c9bb2e4c17d0858c8a8e";
const BASE_URL = "https://api.openweathermap.org";
```

**Geocoding endpoint:**
```
GET {BASE_URL}/geo/1.0/direct?q={cityName}&limit=5&appid={API_KEY}
```

**Weather endpoint:**
```
GET {BASE_URL}/data/2.5/weather?lat={lat}&lon={lon}&units={units}&appid={API_KEY}
```

**Hourly forecast endpoint:**
```
GET {BASE_URL}/data/2.5/forecast?lat={lat}&lon={lon}&units={units}&cnt=8&appid={API_KEY}
```
> `cnt=8` limits the response to the next 8 forecast slots (24 hours). Both the weather and forecast requests should be fired in **parallel** (`Promise.all`) when a city is selected to minimize load time.

> ⚠️ This key is embedded in the spec for reference. When implementing, move it to a `.env` file and add that file to `.gitignore` to avoid accidentally committing it to a public repo.

---

## Out of Scope (v1)

- Multi-day (5-day) forecast
- User accounts or saved locations
- Map view
- Push notifications
- PWA / offline support

---

## Stretch Goals (v2+)

- Full 5-day forecast view (expandable below the hourly strip)
- Sparkline temperature chart across the hourly entries
- Save favorite cities (localStorage)
- Auto-detect location via browser Geolocation API
- Animated weather backgrounds (rain particles, cloud movement)
- More chaos units (e.g. "in hot dog lengths per fortnight")
