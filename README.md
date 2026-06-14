[README.md](https://github.com/user-attachments/files/28931669/README.md)
# 🚲 Burlington Bike Route Planner

An interactive map for planning low-traffic bike routes in Burlington, Vermont and the surrounding region. Color-coded by road type and traffic level, with route drawing, elevation profiles, and GPX export.

**Live site:** [bikebtv.github.io](https://bikebtv.github.io) *(update with your URL)*

---

## What it does

Burlington has great cycling infrastructure, but it can be hard to find routes that feel safe and low-traffic. This map surfaces cycleways, quiet residential streets, and gravel paths so you can plan rides and commutes without ending up on busy roads.

- **Browse** the road network color-coded by tier — from dedicated cycleways down to heavy arterials
- **Draw routes** by clicking waypoints; they snap automatically to roads and bridge small gaps
- **View elevation profiles** for any saved route
- **Export to GPX** for use in Strava, Ride with GPS, Garmin, or Komoot
- **Toggle and reverse** saved routes
- **Undo waypoints** while drawing

---

## Road tiers

| Color | Tier | Description | Default |
|-------|------|-------------|---------|
| 🟣 `#542788` | **Cycleway** | Dedicated paved bike paths & separated lanes | On |
| 🟤 `#a67c52` | **Gravel** | Unpaved tracks, dirt roads, gravel paths | Off |
| 💜 `#998ec3` | **Low traffic** | Quiet residential & service roads | On |
| 🟠 `#e08214` | **Medium traffic** | Secondary & tertiary roads | Off |
| 🟫 `#b35806` | **Heavy traffic** | Primary roads & arterials | Off |

Zoom-dependent rendering keeps the map clean at regional view — cycleways only below zoom 13, all active tiers at zoom 13+.

---

## How to use

### Browsing the map
Use the tier filter buttons in the toolbar to show or hide road classes. Hover over any button for a description of that tier.

### Drawing a route
1. Click **Draw route** in the toolbar
2. Click anywhere near a road — waypoints snap to the nearest road within ~100m
3. Each leg routes along actual roads using a local OSM graph, with OSRM cycling as fallback
4. Small gaps between disconnected road features bridge automatically
5. Use **↩ Undo** to remove the last waypoint
6. Double-click (or click **Save route**) to finish

### Managing routes
Saved routes appear in the right panel. From there you can:
- **👁️ Toggle** visibility on/off
- **⇄ Reverse** the direction
- **↗ Elevation** — view the elevation profile (fetches from USGS 3DEP)
- **GPX** — export for Strava, Ride with GPS, etc.

Routes are saved in your browser's **localStorage** — they persist between sessions on your device but are not shared with other users.

---

## Technical details

### Stack
- **No build step** — single `index.html`, pure vanilla JS + HTML + CSS
- **No API keys required** — all data sources are free and open

### Data sources

| Source | Used for |
|--------|----------|
| [OpenStreetMap](https://www.openstreetmap.org) via [Overpass API](https://overpass-api.de) | Road network data |
| [OSRM](https://project-osrm.org) (cycling profile) | Routing fallback |
| [USGS 3DEP Elevation](https://www.usgs.gov/3d-elevation-program) | Elevation profiles |
| [CartoDB Positron](https://carto.com/basemaps) | Base map tiles |

### Region
The map queries OSM data for a bounding box covering:
- **North:** St. Albans
- **South:** Hinesburg
- **East:** Waterbury / Richmond

```
BBOX: 44.27,-73.35,44.80,-72.85
```

### Routing logic
1. A local graph is built from all OSM way geometries at load time — every node becomes a routable vertex, including all intermediate shape points on cycleways
2. Dijkstra's algorithm finds the shortest path between snapped waypoints
3. If the local graph can't find a connected route, OSRM cycling API is tried as fallback
4. If OSRM returns a detour >3× the straight-line distance, a straight-line bridge segment is used instead

### Dead-end filtering
Short dead-end spurs on the low-traffic tier (under 150m with no through-connection) are filtered from the display to reduce clutter, while remaining in the routing graph for connectivity.

---

## Local development

No dependencies to install. Just open the file:

```bash
git clone https://github.com/yourusername/burlington-bike-routes.git
cd burlington-bike-routes
open index.html   # macOS
# or: start index.html  (Windows)
# or: xdg-open index.html  (Linux)
```

Or serve it locally to avoid any browser file:// restrictions:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

### Deploying to GitHub Pages
1. Push `index.html` (and this `README.md`) to the `main` branch of your repo
2. Go to **Settings → Pages**
3. Set source to **Deploy from a branch → main → / (root)**
4. Your site will be live at `https://yourusername.github.io/repo-name`

---

## Customizing

### Change the region
Edit `BBOX` in the JS constants at the top of `index.html`:
```js
const BBOX = '44.27,-73.35,44.80,-72.85'; // S,W,N,E
```

### Change default visible tiers
```js
let layerVisibility = { cy:true, gr:false, t1:true, t2:false, t3:false };
```

### Change colors
```js
const TIER_COLORS = { cy:'#542788', gr:'#a67c52', t1:'#998ec3', t2:'#e08214', t3:'#b35806' };
```

### Change the basemap
Replace the CartoDB Positron tile URL in `init()`:
```js
tileLayer = L.tileLayer('https://{s}.basemaps.cartocdn.com/light_all/{z}/{x}/{y}{r}.png', ...);
```
Other good options: `dark_all` (dark matter), `rastertiles/voyager` (Voyager).

---

## Contributing

This is a community resource for Burlington cyclists. Contributions welcome:

- **Bug reports** — open an issue
- **Route suggestions** — if you know of a great low-traffic route that isn't showing up correctly, check whether it's tagged in OSM and consider adding it at [openstreetmap.org](https://www.openstreetmap.org)
- **Feature ideas** — open an issue or PR

---

## License

Code: [MIT](LICENSE)

Map data © [OpenStreetMap contributors](https://www.openstreetmap.org/copyright), available under the [ODbL](https://opendatacommons.org/licenses/odbl/).
