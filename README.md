# Artemis II Mission Tracker

A real-time 3D visualization of NASA's Artemis II crewed lunar flyby mission, built as a single-page web app using Three.js.

**[View Live](https://unlox775-code-dot-org.github.io/artemis-watch/)**

![Artemis II Tracker](https://img.shields.io/badge/mission-Artemis%20II-blue)
![Three.js](https://img.shields.io/badge/three.js-r163-green)
![License](https://img.shields.io/badge/license-MIT-brightgreen)

## What It Shows

- **3D Earth-Moon system** with textured Earth (TopoJSON country boundaries), Moon, Sun, and visible planets
- **Spacecraft trajectory** color-coded by mission day (10 days), with smooth orbital curves
- **Real-time telemetry** — velocity, distance from Earth, distance from Moon
- **Live ticker** showing the last confirmed data point from NASA with timestamp
- **Mission Elapsed Time** clock counting from launch (April 1, 2026 22:35 UTC)
- **Interactive camera** — click and drag to rotate the view (trackball-style), auto-wobble resumes after 60s
- **Minimap** showing the sub-spacecraft ground track with country identification
- **Exaggerated vs. To-Scale** toggle for the Earth-Moon distance
- **Mission phase display** tracking the current flight phase
- **Settings persistence** — scale, units, and label preferences saved to localStorage

## Data Sources

### Primary: NASA AROW / JSC Flight Dynamics

The tracker's primary data source is NASA's **AROW (Artemis Real-time Orbit Website)** ephemeris — the same operational trajectory data used by NASA's own real-time tracker. This is published by the **Flight Dynamics Operations (FDO) group at Johnson Space Center** and contains state vectors (position + velocity) at high resolution:

- **~105-second intervals** during early flight
- **4-minute intervals** during cruise
- **2-second intervals** near maneuvers and burns

The app fetches this data entirely in-browser: it scrapes the NASA tracking page for the latest OEM (Orbital Ephemeris Message) ZIP file URL, downloads it directly from nasa.gov, unzips it in memory using [JSZip](https://stuk.github.io/jszip/), and parses the CCSDS OEM format. The coordinate frame is rotated from EME2000 (equatorial J2000) to ecliptic J2000 to match the rendering frame.

New AROW versions are checked every 30 minutes. The filename includes a version number (e.g., `v3`) so the app skips re-downloading if nothing has changed.

### Fallback: JPL Horizons API

If the AROW fetch fails (CORS issues, NASA page changes, etc.), the tracker falls back to the **[JPL Horizons API](https://ssd.jpl.nasa.gov/horizons/)** querying object `-1024` (Orion spacecraft). Horizons is an archival system maintained by JPL's Solar System Dynamics group — it provides accurate trajectory data but may lag the operational data by hours to days, since it's updated on an irregular schedule by the navigation team.

The Horizons fallback fetches at 30-minute resolution with additional 1-minute fine data near both perigee passes for smooth trajectory curves.

### Full Data Source Table

| Data | Source | Resolution | Update Frequency |
|------|--------|-----------|-----------------|
| Spacecraft trajectory | NASA AROW OEM (JSC/FDO) | ~105s to 4 min | Checked every 30 min |
| Spacecraft trajectory (fallback) | JPL Horizons API (-1024) | 30 min + 1 min near perigees | On page load |
| Live confirmed position | AROW data point nearest to "now" | Closest available point | Every 15 min |
| Live confirmed position (fallback) | JPL Horizons real-time query | 5 min window around now | Every 15 min |
| Moon ephemeris | JPL Horizons API (object 301) | 1 hour | On page load |
| Country boundaries | [Natural Earth](https://github.com/topojson/world-atlas) TopoJSON | 110m resolution | Static |
| Sun position | Simplified solar longitude formula | Continuous | Every frame |
| Planet positions | Keplerian orbital elements | Computed once at startup | Static |

### What's Projected vs. Confirmed

- **Live ticker** (bottom of screen): Shows the most recent **confirmed data point** — an actual state vector from NASA's trajectory data at a specific moment in time. This is labeled "Live from AROW" or "Live from Horizons" with how long ago that data point was recorded.
- **Telemetry panels** (left side): Show **interpolated values** — smoothly computed between confirmed data points using Catmull-Rom spline interpolation. These update every frame and will be very close to (but not exactly) the ticker values.
- **Trajectory line**: The full mission trajectory rendered from all available data points. When using AROW data, this is 3,200+ points from JSC Flight Dynamics.
- **Pre-launch phase** (first ~4.5 hours): The AROW/Horizons data starts after ICPS separation (~T+3.5h). The initial orbit loops (LEO parking orbit and TLI burn) are **back-propagated** using 2-body Keplerian physics from the first confirmed data point. These are physics-based estimates, not live data.
- **Reentry phase** (after last data point): Forward-propagated using 2-body dynamics, blending to a known splashdown target ~50 miles off San Diego (~32.5N, 117.5W). NASA has indicated Artemis II will use a steep direct entry (no skip reentry) based on lessons from Artemis I heat shield analysis.

## How It's Built

This entire project is a **single `index.html` file** — no build step, no dependencies to install, no server required. Just open it in a browser.

- **3D rendering**: [Three.js](https://threejs.org/) v0.163.0 via CDN ES module importmap
- **Earth texture**: Generated at runtime from TopoJSON country boundaries rendered onto a canvas
- **ZIP decompression**: [JSZip](https://stuk.github.io/jszip/) 3.10.1 for in-browser OEM file extraction
- **Coordinate transforms**: GMST-based Earth rotation, ecliptic/equatorial frame conversion, equirectangular map projection
- **Orbital mechanics**: Catmull-Rom spline interpolation with linear fallback near Earth, Velocity Verlet integration for propagated segments
- **CORS handling**: Direct fetch where possible (nasa.gov allows cross-origin for uploads), corsproxy.io fallback for page scraping

### Generated with AI

This project was built collaboratively using **[Claude Code](https://claude.ai/code)** by Anthropic. The trajectory math, Three.js rendering, orbital mechanics, data source integration, and UI were developed through iterative conversation with Claude (Opus). The backward orbital propagation was computed once using a Python helper script and the results embedded as static data.

## Contributing

Contributions welcome! Some ideas:

- Higher-resolution Earth texture (Blue Marble, etc.)
- Spacecraft 3D model instead of a glowing dot
- Audio/comms integration from NASA feeds
- Additional telemetry graphs (acceleration, thermal)
- Better mobile touch gestures
- OCR pipeline for YouTube live feed telemetry overlay (for truly real-time data)
- Integration with NASA's OEM ephemeris updates if a predictable URL pattern emerges

### Running Locally

```bash
# Just serve the directory with any static file server
python3 -m http.server 9123

# Then open http://localhost:9123
```

## License

MIT
