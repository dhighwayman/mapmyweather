# MapMyWeather

An interactive weather map that shows real-time forecasts for cities worldwide. Click anywhere on the map to get a detailed 5-day forecast with hourly breakdown.

![MapMyWeather](https://img.shields.io/badge/weather-open--meteo-blue) ![Map](https://img.shields.io/badge/map-leaflet%20%2B%20OpenStreetMap-green) ![License](https://img.shields.io/badge/license-MIT-lightgrey)

## Features

- **Live weather markers** — cities in the visible map area show current conditions: icon, temperature, and precipitation probability
- **Dynamic city loading** — cities update as you pan and zoom; at high zoom levels small towns and villages appear via OpenStreetMap
- **5-day forecast drawer** — click any point on the map to see a detailed forecast with daily max/min, precipitation, and expandable hourly breakdown
- **Location search** — Nominatim-powered search with keyboard navigation
- **Timeline scrubber** — scroll up to 5 days ahead; markers update in real time as you move the cursor
- **Play mode** — auto-advances the timeline to animate the forecast
- **Geolocation** — centers the map on your location on first load

## Running locally

```bash
# Any static HTTP server works — file:// does not (Leaflet needs HTTP)
python3 -m http.server 8787
```

Then open [http://localhost:8787](http://localhost:8787).

## Stack

| Concern | Solution |
|---|---|
| Map | [Leaflet](https://leafletjs.com/) + CartoDB Voyager tiles |
| Weather data | [Open-Meteo](https://open-meteo.com/) hourly & daily forecast |
| City search | [Nominatim](https://nominatim.openstreetmap.org/) (OpenStreetMap) |
| Small-town lookup | [Overpass API](https://overpass-api.de/) at zoom ≥ 10 |
| City dataset | Bundled `cities.json` (~500 major cities) for fast low-zoom lookup |

No API keys. No build step. No framework. Single `index.html`.

## Architecture

Everything lives in `index.html`. See [CLAUDE.md](CLAUDE.md) for a detailed breakdown of the code structure.

## License

MIT
