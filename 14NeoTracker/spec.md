# Near Earth Object (NEO) Tracker — Product Specification

## Overview

A single-page web application for tracking near-Earth objects using three NASA APIs. The app is divided into three tabs, each powered by a distinct data source, giving users a full picture of asteroid activity: what's happening now, what has happened and what's coming, and what poses the greatest collision risk.

**API Key:** `ev30OnlMrEFb7SBFVDzxgUu2ON3rIYLX43RNuXkf`  
**Base URL for all NASA APIs:** `https://api.nasa.gov`

---

## Default Landing View — Interactive Globe

### Concept
Before any tab is selected, the app opens on a full-screen interactive globe. This serves as the entry point and persistent spatial anchor for the entire experience. The globe is always visible — either as the landing page before tab selection, or as a sidebar/overlay panel that persists while tabs are open (see Layout section below).

### Visual Design
- **Background:** Pure black (`#000000`). No gradients, no star field on the globe canvas itself — the void is the aesthetic.
- **Globe:** Rendered as a dark sphere (`#0a0a0a` fill) with **gray continent outlines** only (`#555` stroke, no fill). Oceans are invisible — just the black void showing through. Minimalist, almost wireframe.
- **Lunar Distance Rings:** Concentric distance rings at 2, 5, 10, 20, 50 LD with `#555` stroke and `#888` labels — subtle but readable.
- **Graticule:** Subtle longitude/latitude grid lines at 30° intervals in `#111111` — barely visible, adds depth without clutter.
- **Slow auto-rotation:** Globe rotates at ~0.1°/frame on the Y axis when idle. Rotation pauses on user hover/drag interaction and resumes 3 seconds after the user releases.
- **Rendering:** Use an SVG `<canvas>` approach via **D3.js** (`d3-geo` with `geoOrthographic` projection) for the globe. GeoJSON continent data loaded from a public CDN (e.g., `naturalearth-cdn`). Three.js is explicitly **not** used — keep it lightweight and 2D projected.

### NEO Dots — This Week's Objects
- Each NEO from the NeoWs this-week feed is plotted as a **white dot** (`#ffffff`, radius 3px) positioned at its closest approach **azimuth/elevation** converted to a longitude/latitude on the globe surface.
- Dots appear on the globe face visible to the user; dots on the far side of the globe are hidden (clipped by the sphere boundary naturally via D3 projection).
- **Potentially Hazardous Asteroids (PHAs)** render as **amber dots** (`#f59e0b`, radius 4px) with a faint pulsing halo animation.
- On hover, each dot shows a small tooltip: `[Name] — [Miss Distance] km — [Date]`.

### Position Mapping
NEO approach data provides miss distance and velocity vectors but not a geographic position. Use the following approach to derive a meaningful plotted position:

```
Each NEO in the NeoWs feed has a `close_approach_data[].close_approach_date` and
`close_approach_data[].miss_distance.kilometers`.

Map miss distance to a radial distance from Earth center (visual only, not to scale).
Derive a unique longitude from a hash of the NEO's designation so dots are
distributed across the globe rather than clustered at a single point.
Latitude is derived from the object's orbital inclination if available (from the
`orbital_data.inclination` field), otherwise assigned pseudo-randomly but
deterministically from the designation hash.

This is a visual approximation — the globe is a spatial metaphor, not an
orbital simulator.
```

The implementation note must be surfaced to the user as a subtle disclaimer on the globe: *"Positions are illustrative, not orbital"* in `9px` monospace text, bottom-left corner of the globe canvas.

### Selected NEO Highlighting
- When `selectedNEO` is set to **any object from Tab 1 (This Week)**, its dot on the globe:
  - Scales up to radius 7px
  - Color changes to **teal** (`#2dd4bf`)
  - Gets a soft **outer glow effect** (two concentric circles at 20px/15% opacity and 14px/25% opacity)
  - Gets a persistent expanding ring animation (like a radar ping) that repeats every 2s
  - A thin line extends from the dot to a floating label showing the NEO name and miss distance

- When the selected NEO switches to a **different This Week object**, the previous dot returns to its default white/amber state and the new one gets the teal highlight treatment.

### Out-of-Week NEO — Relative Plot
When `selectedNEO` is set to an object **not in this week's NeoWs feed** (i.e., selected from Tab 2 or Tab 3):

1. **Fetch current position** using the JPL Horizons API:
   ```
   GET https://ssd.jpl.nasa.gov/api/horizons.api
     ?format=json
     &COMMAND='{designation}'
     &OBJ_DATA=NO
     &MAKE_EPHEM=YES
     &EPHEM_TYPE=OBSERVER
     &CENTER=500@399          ← geocenter
     &START_TIME='{today}'
     &STOP_TIME='{today+1}'
     &STEP_SIZE=1d
     &QUANTITIES=1,20         ← RA/Dec and distance
   ```
   Parse `RA` and `Dec` from the response ephemeris table. Convert RA/Dec to globe longitude/latitude using standard spherical coordinate mapping: `lon = RA - GMST`, `lat = Dec`.

2. **Plot the dot** at the derived position as a **dim blue-white dot** (`#93c5fd`, radius 5px) with a dashed-circle halo to distinguish it from the regular this-week dots.

3. A label reads: *"[Name] — current sky position (not a close approach)"* to avoid confusion.

4. **Ephemeral lifetime:** This dot exists **only** while that specific NEO is selected. The moment `selectedNEO` changes to any other object (even another out-of-week object), the dot is immediately removed with a brief fade-out animation (300ms opacity → 0). A new Horizons API call is made for the newly selected out-of-week object, and its dot appears with a fade-in (300ms).

5. **Error state:** If the Horizons API call fails or returns no ephemeris data, show a small notice on the globe: *"Position unavailable for [Name]"* — no dot is plotted.

### Globe Interaction
- **Drag to rotate:** Click and drag rotates the globe freely (D3 drag handler updates the projection's `rotate` property).
- **Scroll to zoom:** Mouse wheel adjusts the projection scale between 50% and 200% of the default.
- **Click a dot:** Sets `selectedNEO` — same as clicking the card/row in a tab.
- **Double-click globe surface:** Clears `selectedNEO` and deselects everything.
- **Reset button:** A small "↺ Reset View" button (bottom-right of globe) snaps back to default rotation and zoom with a 600ms animated transition.

### Layout — Globe + Tabs Coexistence
```
┌─────────────────────────────────────────────────┐
│  NEO TRACKER                          [UTC clock]│
├──────────────┬──────────────────────────────────┤
│              │  [Tab Bar: Week | Archive | Risk] │
│    GLOBE     │                                   │
│   (40% W)    │   Active Tab Content              │
│              │   (scrollable)                    │
│              │                                   │
│              │                                   │
└──────────────┴──────────────────────────────────┘
```
- On landing (no tab selected), globe expands to fill ~70% of viewport width, centered.
- Once a tab is selected, globe collapses to a **fixed left sidebar** at 38% viewport width. The tab content panel occupies the remaining 62%.
- On screens < 900px wide, the globe moves to a **collapsible top panel** (toggle button: `🌍 Globe`), collapsed by default.
- The collapse/expand transition is 400ms ease-in-out.

### Performance Notes
- GeoJSON continent data is loaded once and cached in module scope — not re-fetched on re-render.
- The globe canvas re-draws only when: rotation changes (animation frame), `selectedNEO` changes, or the NeoWs data loads.
- The auto-rotation uses `requestAnimationFrame` and is cancelled on component unmount.
- Horizons API calls are debounced 300ms and results cached per designation for the session — switching back to a previously fetched NEO does not re-fetch.

---

## Cross-Tab NEO Sync

### Concept
A single **globally selected NEO** (`selectedNEO`) is held in top-level React state and shared across all three tabs. When a user selects any object in any tab, the other two tabs automatically navigate to and highlight the same object — giving a unified view of that asteroid's close approach history, future risk, and this-week activity simultaneously.

### Shared State Shape
```js
selectedNEO: {
  designation: string,   // canonical SPK-ID or designation, e.g. "2021 PJ1"
  name: string,          // display name
  sourceTab: "neows" | "sbdb" | "sentry"
}
```
`designation` is the canonical cross-tab key. All three APIs use JPL designations, so it is reliable for matching across datasets.

### Selection Triggers
| Tab | How to Select |
|-----|--------------|
| This Week | Click an asteroid card → sets `selectedNEO` |
| Archive & Predictions | Click a table row → sets `selectedNEO` |
| Collision Risk | Click a risk table row → sets `selectedNEO` |

A selected object gets a persistent **teal outline glow** on its card/row across all tabs. A small "synced across tabs" indicator (chain-link icon + object name) appears in the tab bar beneath the active tab label whenever a selection is active. Clicking it clears the selection.

### Sync Behavior Per Tab

**Tab 1 — This Week**
- If `selectedNEO` is set and the object appears in the current week's feed, its card is automatically scrolled into view and highlighted.
- If the object does not appear this week, a dismissable banner reads: *"[Name] has no close approach this week — showing nearest date instead"* with a link to the SBDB tab.

**Tab 2 — Archive & Predictions**
- When `selectedNEO` is set, Tab 2 automatically filters the table to show **only** that object's approaches (equivalent to firing `?des={designation}`).
- A "Show All" button restores the full dataset without clearing the global selection.
- The table scrolls to the closest upcoming approach (or the most recent historical one if no future passes exist).

**Tab 3 — Collision Risk**
- If `selectedNEO` matches a Sentry entry, the Sentry table scrolls to and highlights that row.
- If the object is **not** on the Sentry watch list, a banner reads: *"[Name] is not currently tracked by Sentry — no significant impact probability detected."* styled as a green "all clear" notice.

### Cross-Tab Navigation Shortcut
Each tab, when an object is selected, shows a **jump bar** beneath its header:
```
[ ☄️ Selected: 2029 XB1 ]   [→ View in Archive]   [→ View in Risk Table]
```
Clicking a jump button switches to that tab and triggers the scroll/highlight described above.

### Identity Resolution
Because each API uses slightly different name formats, a normalization function strips parentheses, leading zeros, and whitespace before comparing designations:
```js
function normalizeDesignation(raw) {
  return raw.replace(/[()]/g, "").trim().toLowerCase().replace(/\s+/g, " ");
}
```
Matching is attempted first by exact designation, then by normalized name substring. If no match is found across APIs, the tab shows the "not found" banner described above.

### URL Persistence
The selected NEO designation is written to the URL hash so links can be shared:
```
https://app.url/#neo=2029+XB1
```
On load, if a `#neo=` hash is present, the app pre-selects that object and opens the tab it was most recently found in.

---

## Aesthetic Direction

**Theme:** Deep-space observatory terminal — dark background evoking the void of space, with a monospaced/technical feel balanced by clean data visualization. Color palette centers on near-black backgrounds (`#08090d`), cool deep-navy accents, and sharp amber/orange highlights to evoke radar screens and threat indicators. Typography pairs a geometric display font (e.g., `Chakra Petch` or `Orbitron`) for headers with a readable monospace (e.g., `JetBrains Mono`) for data values. Subtle star-field CSS background, scanline overlays on headers, and glowing border effects on cards. Animated radar "ping" on page load.

---

## Tab 1 — This Week (NeoWs API)

### Purpose
Show all near-Earth objects making close approaches in the current 7-day window. Give users an at-a-glance summary of the week's asteroid traffic.

### API
**NeoWs Feed Endpoint**
```
GET https://api.nasa.gov/neo/rest/v1/feed
  ?start_date={YYYY-MM-DD}       ← today
  &end_date={YYYY-MM-DD}         ← today + 6 days
  &api_key=ev30OnlMrEFb7SBFVDzxgUu2ON3rIYLX43RNuXkf
```

### Data Displayed

**Summary Bar (top of tab)**
- Total NEO count this week
- Count of potentially hazardous asteroids (PHAs)
- Closest approach distance (km) and the asteroid's name
- Largest asteroid diameter this week

**Day-by-Day Timeline**
- Vertical or horizontal scrollable timeline, one section per day
- Each asteroid card shows:
  - Name / designation
  - Estimated diameter range (min–max meters)
  - Miss distance (lunar distances and km)
  - Relative velocity (km/s)
  - Potentially Hazardous badge (amber glow) if `is_potentially_hazardous_asteroid: true`
  - Link to NASA JPL detail page (`nasa_jpl_url`)

**Sorting & Filtering**
- Sort by: miss distance, velocity, diameter, date
- Filter: show only PHAs toggle

### Visual Design Notes
- Cards sorted closest-first by default
- Miss distance rendered as a small visual "proximity bar" — a thin horizontal bar where 0 = Earth surface, 1 LD = lunar distance marker, full bar = safe distance
- PHA cards have a pulsing red-amber border

---

## Tab 2 — Archive & Predictions (SBDB Close Approach API)

### Purpose
Explore historical close approaches and future predicted passes from the full JPL Small-Body Database. Unlike the NeoWs feed (limited to 7 days), this dataset spans decades in both directions.

### API
**SBDB Close-Approach Data API**
```
GET https://ssd-api.jpl.nasa.gov/cad.api
  ?dist-max=0.05           ← max approach distance in AU (≈ 7.5M km)
  &date-min=1900-01-01     ← archive start
  &date-max=2200-01-01     ← far future predictions
  &sort=date               ← chronological
  &fullname=true           ← include full object name
```
*Note: This API is hosted at `ssd-api.jpl.nasa.gov`, not `api.nasa.gov`. It is free/public and does not require the NASA API key, but the key may be appended without issue.*

**For user-initiated searches (by object name):**
```
GET https://ssd-api.jpl.nasa.gov/cad.api
  ?des={object-name-or-designation}
  &fullname=true
```

### Data Displayed

**Mode Toggle**
- **Historical** — approaches with `cd` (close-approach date) before today
- **Upcoming** — approaches with `cd` after today

**Data Table (paginated, 50 rows/page)**

| Column | Source Field | Notes |
|--------|-------------|-------|
| Object Name | `fullname` | Linked to JPL SBDB lookup |
| Close Approach Date | `cd` | Format: `YYYY-Mon-DD HH:MM` |
| Miss Distance | `dist` (AU) | Show in AU + km |
| Miss Distance Uncertainty | `dist_min` / `dist_max` | ± range |
| Relative Velocity | `v_rel` (km/s) | |
| Absolute Magnitude | `h` | Proxy for size |

**Search / Filter Bar**
- Search by object name/designation
- Date range picker (start / end)
- Max miss distance slider (0.01 AU → 0.05 AU)
- Min magnitude filter

**Object Detail Drawer**
- Clicking a row opens a side drawer with full approach details and a mini timeline of all recorded/predicted approaches for that object

### Visual Design Notes
- Historical rows styled in muted cool tones; future rows in slightly brighter tones with a subtle forward-arrow motif
- "Today" line visually separates past from future in the table
- Uncertainty range rendered as a small error-bar glyph in the miss distance cell

---

## Tab 3 — Collision Risk (Sentry API)

### Purpose
Display NASA's Sentry impact risk monitoring table — objects that have a non-zero probability of impacting Earth in the coming century. This is the most alarming (and most fascinating) dataset.

### API
**Sentry Summary List**
```
GET https://ssd-api.jpl.nasa.gov/sentry.api
  ?removed=false       ← only currently monitored objects
```

**Sentry Object Detail**
```
GET https://ssd-api.jpl.nasa.gov/sentry.api
  ?des={designation}
```
*Also does not require the NASA API key but key can be included.*

### Data Displayed

**Risk Overview Panel (top)**
- Total objects currently on the Sentry watch list
- Highest Palermo Scale value in the list (most significant risk)
- Highest cumulative impact probability object

**Risk Table**

| Column | Source Field | Notes |
|--------|-------------|-------|
| Object Name | `fullname` / `des` | |
| Impact Year Range | `range` | e.g., "2025–2118" |
| Potential Impacts | `n_imp` | Number of possible impact scenarios |
| Cumulative Impact Probability | `ip` | Format as percentage (e.g., 0.0012%) |
| Palermo Scale | `ps_cum` | Logarithmic hazard scale |
| Torino Scale | `ts_max` | 0–10 integer scale with color |
| Estimated Diameter | `diameter` (km) | |
| Absolute Magnitude | `h` | |

**Torino Scale Color Coding**

| Value | Color | Label |
|-------|-------|-------|
| 0 | Gray | No Hazard |
| 1–3 | Yellow | Meriting Attention |
| 4–6 | Orange | Threatening |
| 7 | Orange-Red | Certain Collision – Limited |
| 8–9 | Red | Certain Collision – Regional/Global |
| 10 | Deep Red | Certain Collision – Global Catastrophe |

**Palermo Scale Visual Indicator**
- A small horizontal bar gauge per row
- Negative values (< 0) = below background risk (gray)
- 0 to 2 = elevated concern (yellow → orange)
- > 2 = extreme (red, rare)

**Object Detail Modal**
- Triggered by clicking a row
- Shows: full impact scenario table (year, probability, velocity, energy in megatons, Torino/Palermo per event), object orbit class, discovery info, and a link to the official Sentry page at `https://cneos.jpl.nasa.gov/sentry/`

**Educational Sidebar / Tooltip**
- Hovering the Palermo Scale column header shows a tooltip explaining the scale
- Same for Torino Scale

### Visual Design Notes
- Default sort: Palermo Scale descending (highest risk first)
- Rows with Torino > 0 get a persistent amber glow
- Empty state (if Sentry list is ever cleared): celebratory "No known threats" screen with star animation
- Animate probability values counting up on load for drama

---

## Global UI Elements

### Tab Bar
- Three tabs with icons:
  - 📡 **This Week** (NeoWs)
  - 🗃️ **Archive & Predictions** (SBDB)
  - ☄️ **Collision Risk** (Sentry)
- Active tab underline with glowing accent color
- Tab switching is instant (no page reload); data is fetched once per session and cached
- When a NEO is selected, a sync pill appears below the tab bar: `🔗 Synced: [Object Name]  ✕` — clicking ✕ clears the global selection across all tabs

### Header
- App title: **NEO TRACKER**
- Subtitle: *Near-Earth Object Surveillance Dashboard*
- Live clock (UTC) in top-right corner
- NASA attribution logo/link in footer

### Loading States
- Skeleton loaders styled as terminal scanlines
- Animated radar sweep SVG while API data loads

### Error States
- If an API call fails: show the HTTP status, a human-readable explanation, and a retry button
- If the Sentry list is empty: special "all clear" state

### Responsiveness
- Desktop-first but fully usable on tablet
- On mobile: table columns collapse to the 3 most important fields; full data accessible via row tap → drawer

### Accessibility
- All color-coded risk indicators include text labels (not color alone)
- Table rows are keyboard-navigable
- ARIA labels on all icon buttons

---

## Technical Stack (Recommended)

| Concern | Choice |
|---------|--------|
| Framework | React (single `.jsx` artifact) |
| Styling | Tailwind CSS utility classes + inline CSS variables |
| Data fetching | Native `fetch` with `useEffect` + local state |
| Charts/gauges | Recharts (already available) |
| Icons | lucide-react |
| Fonts | Google Fonts CDN (`Orbitron` + `JetBrains Mono`) |

---

## API Reference Summary

| Tab | API Name | Base URL | Auth |
|-----|----------|----------|------|
| This Week | NeoWs Feed | `https://api.nasa.gov/neo/rest/v1/feed` | `api_key` param |
| Archive & Predictions | SBDB Close-Approach | `https://ssd-api.jpl.nasa.gov/cad.api` | None required |
| Collision Risk | Sentry System | `https://ssd-api.jpl.nasa.gov/sentry.api` | None required |

**NASA API Key:** `ev30OnlMrEFb7SBFVDzxgUu2ON3rIYLX43RNuXkf`

---

## Out of Scope (v1)

- User accounts or saved searches
- Push notifications for new Sentry entries
- 3D orbital visualization
- Comparison between multiple objects
- Export to CSV/PDF
