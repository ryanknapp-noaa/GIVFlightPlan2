# G-IV Synoptic Surveillance Pattern Generator

A single-page web tool for building the NOAA/AOML **G-IV Over-Storm** flight pattern from a storm center fix. It plots a double circumnavigation (outer and inner rings) plus a figure-four on an interactive map, checks the route against a range limit, and exports the result in the formats crews and planners actually use: HRD-style drops and turns sheets, CSV, KML, the onboard 800 UI point list, and route links for Windy, SkyVector, and ForeFlight.

> **Planning aid only.** Always verify against the official NOAA/AOML mission tasking before flight.

Reference pattern: [AOML 2026 HFP Early Stage — G-IV Over-Storm](https://www.aoml.noaa.gov/wp-content/uploads/2026/05/2026HFP_EarlyStage_Flight_Patterns_GIV_overstorm.pdf)

---

## Quick start

1. Open `index.html` in any modern browser (Chrome, Edge, Firefox, Safari). No build step, no server, no API keys.
2. Enter the storm **center fix** (decimal degrees, with N/S and E/W).
3. Optionally enter **departure** and **arrival** airports.
4. Click **Generate Pattern**.
5. Adjust on the map if needed, then export.

To share it, drop `index.html` on any static host (Netlify, GitHub Pages, an internal web server). Everything runs in the browser.

---

## What the pattern looks like

| Element | Points | Radius | Notes |
|---|---|---|---|
| Outer circumnavigation | 1–12 | 120, 150, or 180 nmi | 12 points every 30°. Point 1 is due south; point 7 is due north (before any rotation). |
| Inner circumnavigation | 13–20 | 60 or 90 nmi | 8 points every 45°. Point 13 is due south; point 17 is due north. |
| Figure-four vertices | continue after the last ring point (e.g. 21–23) | same as inner radius | Default legs: NW → NE → SW. The SE leg is off by default. |
| Midpoint drops (M) | labeled `M<vertex#>` | half the inner radius | Optional; sit halfway between center and each vertex. |
| Center fix | `C` | — | Shown on the map; can't be dragged or removed. |

**Figure-four behavior**

- **Inner ring on (standard):** entry is from point 20, crossing the center to the NW vertex, then NE, then SW. Each vertex sits on the inner ring's diagonal point, so it's a second drop at the same location. This matches the real HRD mission convention (e.g. 21/22/23 in the GABRIELLE sample plan).
- **Inner ring off:** the figure-four is flown as its own chain: NW → center → SE, transit to NE → center → SW.
- Any leg can be unchecked (AOML allows dropping one crossing leg for safety). Numbering always picks up right after the last ring point in use.

---

## Sidebar controls

### Center Fix
Latitude/longitude in decimal degrees with hemisphere selectors. The sample value (28.60N, 61.60W) matches the reference pattern.

### Airports
- Type an ICAO code (e.g. `KMCF`, `KLAL`, `MRLB`) and the coordinates fill in automatically from the built-in list of ~290 US airports (plus MRLB, Liberia, Costa Rica). Anything not on the list can be entered by hand.
- Leave the identifier blank to omit that airport.
- **First point flown / Last point flown:** pick where the route actually enters and leaves the pattern. Mileage, flight time, and every export are recalculated for just that slice, which is useful for an early departure from the storm or a late entry.

### Pattern Geometry
- Outer radius (120/150/180 nmi) and inner radius (60/90 nmi), each with an on/off toggle.
- Figure-four leg checkboxes (NW, NE, SW, SE) and a midpoint-drops toggle.
- **Figure-four rotation offset:** rotates only the figure-four. With the inner ring on, a non-zero value moves the vertices off the ring's diagonal points (the hint turns amber to warn you).
- **Pattern orientation offset:** rotates the whole pattern clockwise from true north, e.g. to align with storm motion.

### Mission Info & Export
- Mission/storm name, aircraft tail (defaults to N49RF), and proposed takeoff time (e.g. `21/1730Z`). These go into the export headers.
- **Ground speed** (default 450 kt) is used for leg times and total flight time.
- **Max mission range** (default 3,200 nmi). The banner above the map turns green or red depending on whether the total route fits.

---

## Working on the map

- **Drag** any point (except the center) to reposition it. Bearing and range from the center are recalculated automatically.
- **Click** a point to remove it, or tick points in the Fix List and press **Remove Selected Points**.
- Departure and arrival markers can be dragged too; their coordinate fields update to match.
- After any edit, the remaining points are renumbered in sequence and everything downstream (table, mileage, flight time, previews, exports) updates. The map then draws a simple connect-the-dots path in flight order.
- **Generate Pattern** again resets all drags and removals.
- Hover any marker for its label, original point number, bearing/range, and coordinates.

### Basemaps
The layer button in the top-right corner of the map switches between four basemaps. None of them need an API key:

| Basemap | Source | Good for |
|---|---|---|
| Dark Gray (default) | Esri World Dark Gray Canvas + labels | General planning; matches the page theme |
| Ocean | Esri World Ocean Base | Bathymetry and coastline detail over water |
| Satellite | Esri World Imagery | Visual reference |
| OpenStreetMap (dark) | OSM standard tiles, color-inverted | Fallback |

Your choice is remembered in the browser. If the active basemap fails to load, the map switches to the next one automatically and shows a short notice.

*Why the change:* the tool previously used CARTO's Dark Matter tiles. In late August 2026 CARTO began stamping "API KEY REQUIRED" across every tile requested without a key, which is the watermark that showed up on the map background. Switching providers avoids embedding a key in a public HTML file.

---

## Outputs

All exports respect the First/Last point selectors and any edits you've made on the map.

| Output | Button | What you get |
|---|---|---|
| Fix List | (on page) | Sequence, point label, type, bearing, range, and lat/lon in decimal and DMS |
| CSV | **CSV** | `giv_fix_list.csv` with the full fix list |
| Drops sheet | **Drops** | `<mission>_drops.txt` in HRD mission-plan format: point, lat/lon (deg min), radius/azimuth from center, elapsed time. Airports excluded. Also previewed on the page. |
| Turns sheet | **Turns** | `<mission>_turns.txt`: track-distance table with leg and cumulative nmi, elapsed time, time to IP, and mission duration. Also previewed on the page. |
| Windy | **Open Route / Copy URL** | Opens Windy's Distance & Planning tool with the pattern plotted as a track (airports and midpoints left out, duplicates kept so the track draws correctly) |
| SkyVector | **Open Route / Copy URL** | SkyVector flight plan using ICAO-style coordinate fixes (`DDMMSSNDDDMMSSW`) plus airport idents |
| ForeFlight | **Open Route / Copy String** | Opens ForeFlight Mobile on a device that has it installed, or copies a route string (`DDMMSSN/DDDMMSSW` fixes) to paste into the FPL editor |
| Onboard 800 UI | **Download 800 UI Points** | `<mission>_800ui.txt` fixed-width point list (index, lat deg/min, signed lon deg/min) |
| KML | **Download KML** | `<mission>_pattern.kml` with the flight path and labeled placemarks, for Google Earth |

> The 800 UI format was matched to a sample file; there's no published spec for it. Spot-check the first few lines against the system before loading.

---

## How the math works

- Ring points, vertices, and midpoints are placed with the **great-circle destination formula** on a spherical Earth (R = 3,440.065 nmi).
- Leg distances use the **haversine formula**; dragged points get their bearing and range back from the inverse calculation.
- Flight time = total distance ÷ ground speed. It's a straight still-air estimate with no winds, climb, or descent.
- Degrees-minutes output rounds to the nearest whole minute; DMS output rounds to the nearest second.

---

## Point numbering rules

The same rules are used everywhere (table, map labels, drops/turns sheets, KML):

- Only ring points and figure-four vertices take a sequence number (1, 2, 3…).
- Midpoints are labeled after their vertex (`M21`), the center is `C`, and airports show their identifier.
- After removing points, the numbers close up. The tooltip and Fix List still show each point's original number.

---

## Files and dependencies

```
index.html   — the entire app (HTML, CSS, JS, airport list)
README.md    — this file
```

Loaded from public CDNs at runtime, so an internet connection is required:

- [Leaflet 1.9.4](https://leafletjs.com/) (cdnjs)
- Google Fonts: Space Grotesk, IBM Plex Mono
- Map tiles from Esri ArcGIS Online and OpenStreetMap

Nothing is sent anywhere except tile and font requests. The only thing stored locally is your basemap choice.

---

## Known limitations

- Spherical-Earth math; differences from WGS-84 ellipsoid calculations are small at these distances but not zero.
- Flight time ignores wind, climb, and descent.
- The airport list is built in and US-focused; add rows to `US_AIRPORTS` in the script for others (format: `["ICAO","Name","City","State",lat,lon]`).
- Windy, SkyVector, and ForeFlight URL formats are set by those services and could change.
- Esri tiles stop at native zoom 16 (Ocean at 13), so very close zooms are upscaled. That's far beyond what pattern planning needs.

---

## Changelog

**2026-10-28**
- Replaced the CARTO basemap (now watermarked "API KEY REQUIRED" without a key) with keyless Esri and OpenStreetMap basemaps.
- Added a basemap switcher, remembered selection, and automatic fallback if a tile provider is down.
- Added KMCF (MacDill AFB) to the airport list; the input hints used it as the example but it wasn't on the list.
- Added this README.
