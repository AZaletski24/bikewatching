# 🚴🏼‍♀️ Bikewatching

Explore how Bluebikes traffic across Boston and Cambridge ebbs and flows throughout the day, with a map you can filter by time.

An interactive map visualization of Boston and Cambridge bike infrastructure and Bluebikes station traffic, built for DSC 106 Lab 7. Combines Mapbox GL JS for map rendering with D3.js for data-driven SVG overlays.

---

## Features by Lab Step

### Step 0 — Project Setup
- `index.html` scaffolded with semantic HTML, viewport meta, and linked stylesheets
- `global.css` — body layout: flex column, max-width 80em, auto margins, 1em padding
- `assets/favicon.svg` — custom bike emoji favicon displayed in the browser tab

### Base Map
- Mapbox GL JS loaded via CDN as an ES module
- Map centered on Boston/Cambridge (`[-71.09415, 42.36027]`) at zoom 12
- Zoom constrained between 5 and 18
- `map.css` sizes the map to fill available page height with a subtle shadow and rounded corners

### Bike Lanes
- **Boston bike lanes** loaded from the City of Boston open data GeoJSON feed and rendered as a `line` layer
- **Cambridge bike lanes** loaded from the Cambridge GIS GitHub repository and rendered as a second `line` layer
- Shared `bikeLineStyle` object (`#32D400`, width 5, opacity 0.6) applied to both layers to keep styling consistent

### Station Markers
- D3.js loaded via CDN as an ES module
- Bluebikes station JSON fetched from `dsc106.com` and parsed with `d3.json()`
- `<svg>` overlay injected inside `#map`, absolutely positioned at full size with `pointer-events: none`
- `getCoords(station)` — converts station `lon`/`lat` to pixel coordinates using `map.project()`
- Circles appended for each station and repositioned on `move`, `zoom`, `resize`, and `moveend` map events via `updatePositions()`

### Traffic Volume
- Bluebikes trip CSV (~260k rows, March 2024) fetched with `d3.csv()`
- `computeStationTraffic(stations, timeFilter)` — computes `arrivals`, `departures`, and `totalTraffic` per station using `d3.rollup()`
- Circle radius scaled by `totalTraffic` using `d3.scaleSqrt()` (range `[0, 25]`) so area encodes traffic, not radius
- `<title>` tooltip on each circle shows exact trip count, departures, and arrivals on hover

### Time Filter Slider
- Range slider (`min=-1`, `max=1440`) lets users filter trips to a ±60-minute window around any time of day
- `formatTime(minutes)` — converts slider value (minutes since midnight) to a 12-hour AM/PM string
- `minutesSinceMidnight(date)` — extracts elapsed minutes from a `Date` object
- **Performance optimization:** trips are pre-bucketed into two 1440-element arrays (`departuresByMinute`, `arrivalsByMinute`) at load time so filtering never iterates the full dataset
- `filterByMinute(tripsByMinute, minute)` — slices the pre-bucketed arrays for the selected window, handling midnight wraparound correctly
- `updateScatterPlot(timeFilter)` — recomputes station traffic and updates circle radii; expands radius range to `[3, 50]` when a filter is active so stations remain visible with sparser data
- `updateTimeDisplay()` — syncs the `<time>` element and "(any time)" label with the slider, then triggers `updateScatterPlot()`

### Traffic Flow Color
- `stationFlow` — `d3.scaleQuantize()` maps departure ratio `[0, 1]` to three discrete values `[0, 0.5, 1]`
- Each circle gets a `--departure-ratio` CSS custom property set from `stationFlow`
- CSS `color-mix(in oklch, steelblue, darkorange)` blends the departure color (steelblue) and arrival color (darkorange) according to the ratio — no JS color math needed
- Legend below the map shows the three color states: **More departures**, **Balanced**, **More arrivals**

---

## Data Sources

| Dataset | Source |
|---|---|
| Boston existing bike network | City of Boston Open Data (ArcGIS) |
| Cambridge bike facilities | City of Cambridge GIS (GitHub) |
| Bluebikes station info | `dsc106.com` mirror of Bluebikes GBFS feed |
| Bluebikes trip history (March 2024) | `dsc106.com` mirror of Bluebikes trip data |

## Tech Stack

- [Mapbox GL JS v2.15](https://docs.mapbox.com/mapbox-gl-js/) — base map, tile rendering, coordinate projection
- [D3.js v7.9](https://d3js.org/) — data fetching, rollups, scales, SVG rendering
- Vanilla HTML/CSS/JS — no build step required
