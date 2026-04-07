# WesternWashingtonTransitExplorer
Interactive transit map of Eastern Washington
# Seattle CSA Transit Network Map

An interactive map of the Seattle Combined Statistical Area transit network, built with Leaflet.js and live data from the ArcGIS FeatureServer.

## Features

* **Live data** — pulls directly from the ArcGIS FeatureServer, filtered to currently active routes
* **Service level visualization** — routes colored by service level (frequency/type)
* **Click-to-inspect panel** — shows segment attributes and all transit routes serving that segment
* **Route badges** — color-coded by agency with custom per-route colors from the RouteAttribute table
* **Agency tooltip** — hover over a route badge to see the full agency name
* **AM / Midday / PM service bar** — visual breakdown of service level by time of day

## Data Sources

* **Seattle\_CSA\_Transit\_Network / FeatureServer** — ArcGIS REST API hosted by King County

  * Route layers (spatial)
  * Transit\_Table (route–segment join)
  * RouteAttribute table (agency, short label, badge colors)

## Usage

Open `index.html` in a browser. The map will automatically fetch and render live data. No build step or server required — it runs entirely client-side.

> \*\*Note:\*\* Due to browser security restrictions, open via a local server or GitHub Pages rather than directly from the file system (`file://`).

## Live Map

View the live version at: `https://zhenghan1994.github.io/WesternWashingtonTransitExplorer/`

## Tech Stack

* [Leaflet.js](https://leafletjs.com/) — map rendering
* [Leaflet.PolylineOffset](https://github.com/bbecquet/Leaflet.PolylineOffset) — parallel line offset
* [ArcGIS REST API](https://developers.arcgis.com/rest/) — data source
* IBM Plex Sans / IBM Plex Mono — typography

