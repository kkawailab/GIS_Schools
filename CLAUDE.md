# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a web-based GIS application that visualizes school facilities in Aichi Prefecture, Japan using Leaflet.js and GeoJSON data from the Ministry of Land, Infrastructure, Transport and Tourism (MLIT). It doubles as a teaching repository: `EXERCISES.md` defines a ladder of feature-extension exercises (filtering, search, clustering, PWA, etc.) that users will commonly ask to implement here.

**Key characteristics:**
- Static single-page application (no build process, no framework, no backend)
- All application code lives in `index.html` (embedded CSS/JS)
- Data source: National Land Numerical Information (School Data P29-2023)
- Documentation, UI text, and commit messages are written in Japanese — follow that convention

## Development Commands

### Running the Application

The application requires a local web server because `fetch()` of the GeoJSON file is blocked under the `file://` protocol:

```bash
# Python 3 (recommended)
python -m http.server 8000

# Node.js alternative
npx http-server -p 8000
```

Access at `http://localhost:8000`

**Important:** Do NOT open `index.html` directly in a browser (file:// protocol) as it will fail to load the GeoJSON data due to CORS policy.

### Testing

No automated tests. Manual testing workflow:
1. Start local server
2. Verify map loads with school markers and the total count appears in the info panel
3. Check popup displays when clicking markers
4. Verify legend and info panel display correctly

## Architecture

### File Structure

```
├── index.html                    # Single-file application with embedded CSS/JS
├── P29-23_23.geojson            # School facility data (~966KB, 2,641 point features)
├── P29-23_23_structure.md       # GeoJSON schema documentation
├── EXERCISES.md                  # Feature-extension exercises (基礎〜エキスパート)
├── README.md                     # User-facing documentation (Japanese)
└── LICENSE                       # MIT (code) + CC BY 4.0 (data)
```

### Application Flow

1. **Map Initialization**: Leaflet map centered on Aichi Prefecture (35.1°N, 137.0°E), zoom 10
2. **Data Loading**: Async fetch of `P29-23_23.geojson` from same directory
3. **Marker Rendering**: Each GeoJSON feature converted to a color-coded `divIcon` marker
4. **Interaction**: `onEachFeature` binds popups with facility details

### Data Schema

GeoJSON Point features in `[longitude, latitude]` order, CRS JGD2011 (EPSG:6668) — coordinates work directly with Leaflet's default handling. **All property values are strings** — compare against string literals (`'3'`), never numbers.

Property codes from the MLIT P29 standard:
- `P29_001`: Administrative area code
- `P29_002`: School code
- `P29_003`: School type code (e.g. 16001 = elementary, 16011 = kindergarten). The data contains 13 distinct codes; `P29-23_23_structure.md` does not enumerate them all — see the MLIT code list: https://nlftp.mlit.go.jp/ksj/gml/codelist/SchoolClassCd.html
- `P29_004`: School name
- `P29_005`: Address
- `P29_006`: **Operator code** — the actual data contains `"0"`–`"4"`, not just the two values documented in `P29-23_23_structure.md`. Distribution: `"3"` municipal public (1,494), `"4"` private (944), `"2"` prefectural (185), `"1"` national (17), `"0"` other (1)
- `P29_007`: Closure flag (`"0"` = active, `"1"` = closed; closed schools get a 【廃校】 prefix in popups)
- `P29_008`: Campus code
- `P29_009`: Notes (optional, often null)

### Marker Color Logic

`getMarkerColor()` colors by operator type (P29_006):
- Blue (#3498db): `"3"` public (municipal)
- Red (#e74c3c): `"4"` private
- Gray (#95a5a6): everything else — **this includes all 202 national (`"1"`) and prefectural (`"2"`) schools** (e.g. prefectural high schools), which the legend does not mention. Extending the color scheme to those codes is a natural improvement users may request.

## Code Modification Guidelines

### Adding New Data Layers

To add additional GeoJSON datasets:
1. Place new `.geojson` file in root directory
2. Add second `L.geoJSON()` call with similar structure
3. Update legend to include new layer colors/categories

### Modifying Marker Appearance

Marker styling is controlled by:
- `getMarkerColor()`: Returns hex color based on property value
- `createCustomIcon()`: Generates Leaflet divIcon with inline styles
- CSS `.custom-marker` class (assigned to the divIcon but currently has no CSS rule)

### Error Handling

The application includes enhanced error messages for common issues:
- HTTP status errors (404, 500, etc.)
- CORS/file protocol warnings
- JSON parse failures

Errors are displayed in the loading overlay with troubleshooting guidance.

## Data Licensing

**Critical:** When modifying or redistributing:
- Application code (index.html): MIT License
- GeoJSON data: CC BY 4.0 - **MUST** include attribution:
  ```
  「国土数値情報（学校データ P29-2023）」（国土交通省）
  (https://nlftp.mlit.go.jp/ksj/gml/datalist/KsjTmplt-P29-2023.html) を加工して作成
  ```

## Deployment

### GitHub Pages

The repository is configured for GitHub Pages deployment:
- Homepage URL: https://kkawailab.github.io/GIS_Schools/
- Deploy from `main` branch root
- No build step required (static files only)

### CDN Dependencies

External resources loaded from CDN:
- Leaflet.js 1.9.4 (unpkg.com)
- OpenStreetMap tiles

Ensure these are accessible in production environment.
