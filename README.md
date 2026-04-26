# Puget Sound Transit Explorer

An interactive web map visualizing the Western Washington transit network with multiple viewing modes, built with ArcGIS JS SDK.

![Puget Sound Transit Explorer](https://via.placeholder.com/800x400?text=Puget+Sound+Transit+Explorer)

## Implementation Note

This project has two branches:
- **`ArcGIS_JS_API`**: The production version using ArcGIS JS SDK 4.30 (connected to GitHub Pages)
- **`leaflet`**: Alternative implementation using Leaflet.js

**The ArcGIS JS SDK version was adopted** due to:
- **Faster loading time**: Native FeatureLayer handles pagination and streaming automatically
- **Better compatibility**: Direct integration with ArcGIS Online Feature Server without manual GeoJSON conversion
- **CIM symbol support**: Advanced cartographic symbols with geometric offset effects
- **Automatic projection handling**: SDK manages coordinate transformations internally

The Leaflet branch remains available for reference and for scenarios requiring more granular control or offline-first capabilities.

## Features

### Three Viewing Modes

- **Service Level Mode**: Visualize transit frequency by time period (Daytime, AM Peak, Mid-day, PM Peak)
  - Color-coded by service frequency: Light Rail, Very Frequent, Frequent, etc.
  - Dynamic period switching to see how service changes throughout the day
  
- **Trolleybus Mode**: View electric trolleybus infrastructure
  - Active trolley wire networks
  - Planned and under-construction routes
  - Inactive infrastructure

- **Transit Right-of-Way Mode**: Explore dedicated transit infrastructure
  - Grade-separated light rail
  - Dedicated transit-only lanes
  - Bus-only streets and managed lanes
  - HOV/Toll facilities

### Interactive Features

- **Smart Draw Order**: Transit routes are rendered with priority stacking — higher-frequency routes (Light Rail, Very Frequent) appear on top of lower-frequency routes
- **Hover Highlights**: Mouse over any route segment to highlight it
- **Click for Details**: Click any segment to view:
  - Service level breakdown (AM/MD/PM periods)
  - Transit right-of-way type
  - All transit routes operating on that segment
  - Agency information with color-coded badges
- **Resizable Sidebar**: Drag the separator to adjust map/panel layout
- **Mobile Responsive**: Optimized layout for phone and tablet screens
- **Auto-refresh**: Automatically updates at midnight to show current service

### Technical Implementation

- **Two Implementations**:
  - `seattle-transit-arcgis.html`: ArcGIS JS SDK 4.30 with CIM symbols and geometric offset
  - `seattle-transit-leaflet.html`: Leaflet.js with manual REST API queries
  
- **Multi-layer Architecture** (ArcGIS version): Service Level mode uses separate FeatureLayers per service level code to ensure correct visual stacking order, as `orderBy` property conflicts with CIM symbol rendering

- **CIM Symbol Offset**: All three modes use -1.1pt geometric offset to visually separate overlapping routes on shared corridors

- **Live Data**: Pulls from ArcGIS FeatureServer with temporal filtering based on segment start/end dates

## Data Source

**ArcGIS Feature Service** (created and maintained by the project author):
- Base URL: `https://services7.arcgis.com/m6uLpqj7MgjPU371/arcgis/rest/services/Western_Washington_Transit_Network/FeatureServer`
- Main Layer: Layer 2 (transit network segments)
- Transit Routes: Layer 4 (route attributes and agency information)

The feature service aggregates and synthesizes transit data from multiple regional agencies including King County Metro, Sound Transit, Pierce Transit, Community Transit, and others. The data includes service level classifications, transit right-of-way types, trolleybus infrastructure, and temporal attributes for tracking network changes over time.

### Service Level Codes

| Code | Description | Typical Headway |
|------|-------------|----------------|
| 100 | Light Rail | Grade-separated rail |
| 105 | Very Frequent | ≤10 minutes |
| 104 | Frequent | 10-15 minutes |
| 106 | Frequent (peak) | 10-15 minutes |
| 107 | Less Frequent | 15-30 minutes |
| 108 | Regular | 30 minutes |
| 109 | Limited Trips | >30 minutes |
| 116 | Peak Only | Peak hours only |
| 199 | DART | Dial-a-Ride Transit |
| 295-999 | No Regular Service | Inactive or planned |

### Transit Right-of-Way Types

| Code | Type |
|------|------|
| 7 | Grade-Separated Light Rail |
| 6 | Dedicated Transit Only |
| 5 | Freight and Bus |
| 4 | All-day Transit-Only |
| 3 | Managed Lane (HOV/Toll) |
| 2 | Time Restricted Bus-only |
| 1 | None |

## Installation

### Static Deployment (GitHub Pages, Netlify, etc.)

1. Clone the repository:
```bash
git clone https://github.com/zhenghan1994/PugetSoundTransitExplorer.git
cd PugetSoundTransitExplorer
```

2. **Use the ArcGIS_JS_API branch** (recommended - connected to GitHub Pages):
```bash
git checkout ArcGIS_JS_API
```

   Or switch to the Leaflet version:
```bash
git checkout leaflet
```

3. Deploy the HTML file:
   - **GitHub Pages**: Already configured to deploy from `ArcGIS_JS_API` branch
   - **Netlify**: Drag and drop the HTML file or connect your GitHub repo
   - **Local**: Open `seattle-transit-arcgis.html` directly in a browser

No build process or dependencies required — it's a single self-contained HTML file.

### Branch Comparison

| Feature | `ArcGIS_JS_API` | `leaflet` |
|---------|----------------|-----------|
| Initial load time | ~2-3 seconds | ~4-6 seconds |
| Feature Server integration | Native | Manual REST queries |
| Projection handling | Automatic | Manual GeoJSON conversion |
| CIM symbols | ✓ | ✗ (uses simple Leaflet paths) |
| Offline after load | Partial | Full (pre-loads all data) |
| File size | ~50KB | ~52KB |
| GitHub Pages | ✓ Connected | Available |

**Recommendation**: Use the `ArcGIS_JS_API` branch for production deployments.

## Configuration

### Customizing the Map

The map can be customized by editing JavaScript constants at the top of the file:

```javascript
// Data source
const BASE = 'https://services7.arcgis.com/.../FeatureServer';
const LAYER_ID = 2;

// Initial view
const view = new MapView({
  center: [-122.3321, 47.6062],  // Seattle coordinates
  zoom: 11
});

// Line styling
const offset = -1.1;  // CIM symbol offset in points
const width = 2;      // Line width for categorized routes
```

### Service Level Colors

Edit the `SL_META` object to customize service level colors:

```javascript
const SL_META = {
  100: { bg: '#E60000', label: 'Light Rail' },
  105: { bg: '#00A650', label: 'Very Frequent' },
  // ... etc
};
```

### Draw Order

Control visual stacking by editing the `DRAW_ORDER` object:

```javascript
const DRAW_ORDER = {
  SL1: [100, 105, 104, 106, ...],  // First = on top
  TROLLEY: [1, 2, 3, 4, 0],
  BUSLANE: [7, 6, 5, 4, 2, 3, 9, 10, 1]
};
```

## Browser Support

- **Modern browsers**: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- **ArcGIS JS SDK**: Requires ES6+ support
- **Mobile**: iOS Safari 14+, Chrome Mobile 90+

## Performance Notes

- **Initial Load**: ~2-3 seconds to fetch and render ~50,000 line segments
- **Service Level Mode**: Creates 17 separate FeatureLayers (one per service level code) to ensure correct visual stacking
- **Hover/Click**: Uses `hitTest` API for fast spatial queries
- **Memory**: ~80-120MB for full dataset in browser

## Known Issues & Limitations

1. **orderBy with CIM Symbols**: ArcGIS JS SDK's `orderBy` property does not reliably control draw order when using CIM symbols with geometric offset effects. Solution: Multi-layer approach with separate FeatureLayers.

2. **Mobile Legend**: Legend may overflow on very small screens (<320px width)

3. **Midnight Refresh**: Refreshes at midnight local time; does not account for timezone changes when traveling

## Development

### Repository Structure

```
PugetSoundTransitExplorer/
├── ArcGIS_JS_API branch (production)
│   └── seattle-transit-arcgis.html    # ArcGIS JS SDK implementation
├── leaflet branch
│   └── seattle-transit-leaflet.html   # Leaflet.js implementation
└── README.md                           # This file (both branches)
```

### Code Organization

Both implementations follow the same structure:

```javascript
// 1. Constants & Configuration
// 2. Domain lookups (service levels, agencies, etc.)
// 3. Utility functions
// 4. Renderer builders (CIM symbols)
// 5. Legend builder
// 6. Event handlers (hover, click, mode switch)
// 7. Main init function
// 8. Map/View/Layer setup
```

### Adding a New Mode

1. Add mode button HTML:
```html
<button class="mode-btn" data-mode="newmode">New Mode</button>
```

2. Create renderer builder:
```javascript
function buildNewModeRenderer(offset) {
  return new UniqueValueRenderer({
    field: 'YourField',
    uniqueValueInfos: YOUR_DRAW_ORDER.slice().reverse().map(code => ({
      value: String(code),
      symbol: makeCIMSym(colors[code], 2, offset)
    }))
  });
}
```

3. Add case to mode switcher:
```javascript
case 'newmode':
  // Create layers or set renderer
  break;
```

4. Update legend builder to handle the new mode

## Credits

- **Data Collection & Processing**: Transit network data compiled, synthesized, and maintained by the project author
- **Data Source**: Western Washington Transit Network Feature Service (ArcGIS Online)
- **Base Map**: Esri World Imagery
- **Mapping Libraries**: 
  - ArcGIS JS SDK 4.30
  - Leaflet 1.9.4
- **Fonts**: IBM Plex Sans, IBM Plex Mono

## License

MIT License - feel free to use and modify for your own transit mapping projects.

## Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-mode`)
3. Commit your changes (`git commit -am 'Add transit speed mode'`)
4. Push to the branch (`git push origin feature/new-mode`)
5. Open a Pull Request

## Contact

For questions or issues, please open a GitHub issue at [https://github.com/zhenghan1994/PugetSoundTransitExplorer/issues](https://github.com/zhenghan1994/PugetSoundTransitExplorer/issues) or contact zhenghan1994@gmail.com

---

**Live Demo**: [https://zhenghan1994.github.io/PugetSoundTransitExplorer](https://zhenghan1994.github.io/PugetSoundTransitExplorer)
