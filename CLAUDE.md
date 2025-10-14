# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a web-based GIS application that visualizes school facilities in Aichi Prefecture, Japan using Leaflet.js and GeoJSON data from the Ministry of Land, Infrastructure, Transport and Tourism (MLIT).

**Key characteristics:**
- Static single-page application (no build process)
- Client-side only (no backend server)
- Uses vanilla JavaScript (no framework)
- Data source: National Land Numerical Information (School Data P29-2023)

## Development Commands

### Running the Application

The application requires a local web server due to CORS restrictions when loading GeoJSON files:

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
2. Verify map loads with school markers
3. Check popup displays when clicking markers
4. Verify legend and info panel display correctly

## Architecture

### File Structure

```
├── index.html                    # Single-file application with embedded CSS/JS
├── P29-23_23.geojson            # School facility data (944KB, ~7000+ features)
├── P29-23_23_structure.md       # GeoJSON schema documentation
├── README.md                     # User-facing documentation
└── LICENSE                       # MIT (code) + CC BY 4.0 (data)
```

### Application Flow

1. **Map Initialization**: Leaflet map centered on Aichi Prefecture (35.1°N, 137.0°E)
2. **Data Loading**: Async fetch of `P29-23_23.geojson` from same directory
3. **Marker Rendering**: Each GeoJSON feature converted to color-coded marker
4. **Interaction**: Click handlers bind popups with facility details

### Data Schema

GeoJSON features use property codes from MLIT standard:
- `P29_001`: Administrative area code
- `P29_002`: School code
- `P29_003`: School type code (16011 = kindergarten, etc.)
- `P29_004`: School name
- `P29_005`: Address
- `P29_006`: **Operator code** (3 = public/municipal, 4 = private)
- `P29_007`: Closure flag (0 = active, 1 = closed)
- `P29_008`: Campus code
- `P29_009`: Notes (optional)

### Marker Color Logic

Markers are color-coded by operator type (P29_006):
- Blue (#3498db): Public schools (code "3")
- Red (#e74c3c): Private schools (code "4")
- Gray (#95a5a6): Other/unknown

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
- CSS `.custom-marker` class (currently unused but available)

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
