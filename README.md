# Artemis II Mission Tracker

A real-time 3D visualization of NASA's Artemis II crewed lunar flyby mission, built as a single-page web app using Three.js.

**[View Live](https://unlox775-code-dot-org.github.io/artemis-watch/)**

![Artemis II Tracker](https://img.shields.io/badge/mission-Artemis%20II-blue)
![Three.js](https://img.shields.io/badge/three.js-r163-green)
![License](https://img.shields.io/badge/license-MIT-brightgreen)

## What It Shows

- **3D Earth-Moon system** with textured Earth (TopoJSON country boundaries), Moon, Sun, and visible planets
- **Spacecraft trajectory** color-coded by mission day (10 days), with smooth orbital curves
- **Real-time telemetry** — velocity, distance from Earth, distance from Moon — updated every 15 minutes from JPL Horizons
- **Mission Elapsed Time** clock counting from launch (April 1, 2026 22:30 UTC)
- **Interactive camera** — click and drag to rotate the view (trackball-style)
- **Minimap** showing the sub-spacecraft ground track with country identification
- **Exaggerated vs. To-Scale** toggle for the Earth-Moon distance
- **Mission phase display** tracking the current flight phase

## Data Sources

| Data | Source | Update Frequency |
|------|--------|-----------------|
| Spacecraft trajectory | [JPL Horizons API](https://ssd.jpl.nasa.gov/horizons/) (object -1024, Orion) | Loaded at page start |
| Current position/velocity | JPL Horizons API (real-time query) | Every 15 minutes |
| Moon ephemeris | JPL Horizons API (object 301) | Loaded at page start |
| Country boundaries | [Natural Earth](https://github.com/topojson/world-atlas) TopoJSON | Static |

### What's Projected vs. Live

- **Live data**: The "Live telemetry" ticker at the bottom shows the most recent position and velocity fetched from JPL Horizons. This is the actual predicted ephemeris from NASA's navigation team.
- **Trajectory line**: The full mission trajectory is loaded from Horizons at 30-minute resolution (with 1-minute resolution near Earth perigee passes for smooth curves). This is NASA's official predicted trajectory, not a simulation.
- **Pre-Horizons launch phase** (first ~3.5 hours): JPL Horizons doesn't publish data for the LEO/TLI phase. The initial orbit loops shown near Earth are **reconstructed** using 2-body Keplerian backward propagation from the first Horizons data point. This is a physics-based estimate, not live data.
- **Earth rotation**: Computed using GMST (Greenwich Mean Sidereal Time) formula.
- **Sun/planet positions**: Computed from simplified Keplerian orbital elements.

## How It's Built

This entire project is a **single `index.html` file** — no build step, no dependencies to install, no server required. Just open it in a browser.

The 3D rendering uses [Three.js](https://threejs.org/) v0.163.0 loaded from CDN via ES module importmap. The Earth texture is generated at runtime from TopoJSON country boundary data rendered onto a canvas.

### Generated with AI

This project was built collaboratively using **[Claude Code](https://claude.ai/code)** by Anthropic. The trajectory math, Three.js rendering, orbital mechanics computations, and UI were developed through iterative conversation with Claude (Opus). The backward orbital propagation was computed once using a Python helper script and the results embedded as static data.

## Contributing

Contributions are welcome! Feel free to fork and submit a pull request.

Some ideas for improvements:
- Higher-resolution Earth texture
- Spacecraft 3D model instead of a dot
- Audio/comms integration
- Additional telemetry displays
- Improved mobile touch gestures
- Dark/light theme toggle

### Running Locally

```bash
# Just serve the directory with any static file server
python3 -m http.server 9123

# Then open http://localhost:9123
```

## License

MIT
