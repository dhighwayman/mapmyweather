# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development

Serve locally with any static HTTP server — Google Maps requires HTTP, not `file://`:

```bash
python3 -m http.server 8787
# then open http://localhost:8787
```

No build step, no dependencies to install. The project is a single self-contained HTML file.

## Architecture

The entire application lives in **`index.html`** (~900 lines). There is no framework, bundler, or module system.

**`cities.json`** — compact city dataset (~500 cities, population > ~50K) used for fast client-side bounding-box lookup at low zoom levels. Format: `[{n, lat, lng, p}, ...]`.

### External dependencies (CDN, no API keys)
- **Leaflet 1.9.4** — map rendering and marker management
- **CartoDB Voyager tiles** — map tiles (free, no key)
- **Open-Meteo** — weather forecast API (free, no key). Endpoint: `https://api.open-meteo.com/v1/forecast`
- **Nominatim** — reverse geocoding + place search (free, no key). Endpoints: `/reverse` and `/search`
- **Overpass API** — OpenStreetMap place lookup at high zoom (free, no key). Used only at `zoom >= 10` to avoid rate-limit issues at lower zooms.

### JS structure (inside `<script>`)

All code is in a single `<script>` block, organised into these sections:

| Section | What it does |
|---|---|
| Constants & helpers | `FORECAST_DAYS`, `MARK_STEP`, `OVERPASS_ZOOM`, `wxInfo()`, `tempCol()` |
| `generateTimes()` + `initHourIdx()` | Builds the UTC hour array for the timeline without waiting for any API |
| `buildTimeline()` | Constructs the scrollable timeline DOM; drag + touch support |
| `moveCursor()` / `selectHour()` | Updates the needle position and triggers `updateMarkers()` |
| City resolution | `citiesFromDataset()` (local JSON, zoom < 10) + `citiesFromOverpass()` (zoom ≥ 10) |
| `fetchCityWeather()` / `loadWeatherForCities()` | Fetches Open-Meteo hourly data with batching (4 at a time) and retry on 429 |
| `WxMarker` class | Leaflet `DivIcon`-based marker. Stores pending data in `_data`/`_hi` until `addTo()` is called (which triggers Leaflet's `onAdd`) |
| `syncMarkers()` / `updateMarkers()` | Adds/removes `WxMarker` instances to match the current city list; `updateMarkers()` is called on timeline scrub |
| `onMapMove()` | Debounced handler for `moveend`/`zoomend`; debounce is longer (`1800ms`) when using Overpass |
| Search | Nominatim autocomplete with 320ms debounce, keyboard navigation |
| Drawer | Click-on-map → fetch 5-day daily + hourly forecast → slide-in panel; expand/collapse per day |
| `initLeaflet()` | Creates the map, tile layer, registers all event listeners, runs geolocation |
| Bootstrap | `buildTimeline()` → `loadCitiesDataset()` → `initLeaflet()` |

### Key state variables

```js
let lmap          // Leaflet map instance
let hourIdx       // current selected hour index into allTimes[]
let allTimes      // array of UTC ISO strings generated at startup
const wxCache     // { "lat,lng" → { name, temps, codes, prob, precip } }
const activeMarkers // Map<cacheKey, WxMarker> — currently visible markers
```

### Adding forecast variables

To add a new Open-Meteo field (e.g. wind speed):
1. Append it to the `hourly=` param in `fetchCityWeather()`
2. Store it in `wxCache[city.key]`
3. Read it in `WxMarker._render()` and/or `renderDrawerDays()`

### Changing zoom thresholds for city density

Edit `maxCitiesForZoom()` (for local dataset) and the `placeFilter`/`maxRows` logic in `citiesFromOverpass()`.
