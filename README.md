# CropCast

A single-page web app that shows recent rainfall on an interactive map and suggests which crop fits best in each area. The map sits on the left and the suggestions sit on the right.

It is built almost entirely from existing tools: Leaflet for the map, Open-Meteo for weather, ethers.js for Web3, and HTMX for the interactions. The only custom code is the glue and the crop-scoring rules, all inside one `index.html`.

## Features

- **Rain tracking:** the last 30 days of rain, the next 7 days of forecast, and mean temperature for any point you click.
- **Crop suggestions:** the top 3 crops for that point, each with a fit score and its ideal temperature and rainfall range.
- **Map-wide scan:** samples a 5×4 grid across the visible map and places the best-fit crop on each cell, plus a summary of how many areas suit each crop.
- **Place search:** jump to any town or region.
- **Signed field reports:** connect a browser wallet and sign a report of the selected location. The page verifies the signature and shows the signer's address. No gas, no transaction.

## Tech stack

| Purpose | Tool | Key needed |
|---|---|---|
| Map | [Leaflet](https://leafletjs.com) 1.9.4 with OpenStreetMap tiles | No |
| Weather and rain | [Open-Meteo](https://open-meteo.com) forecast API | No |
| Place search | Open-Meteo geocoding API | No |
| Web3 | [ethers.js](https://docs.ethers.org/v6/) 6.13.4 with any EIP-1193 wallet (MetaMask, etc.) | No |
| UI interactions | [HTMX](https://htmx.org) 2.0.4 | No |

All libraries load from CDNs (cdnjs and jsDelivr), so there is nothing to install or build.

## Running it

You need an internet connection and a modern browser. Serve the folder over HTTP rather than opening the file directly. Some browsers and wallet extensions behave differently on `file://` pages, and OpenStreetMap may refuse tile requests that carry no referrer.

```bash
cd cropcast
python3 -m http.server 8000
```

Then open <http://localhost:8000>. Any static server works (`npx serve`, VS Code Live Server, and so on).

## Using it

1. The map opens over Kenya. Use the search box to go somewhere else.
2. Click anywhere on the map. The right panel shows rain, temperature and the top 3 crops.
3. Click **Scan this view** to place the best-fit crop on each part of the current map view. It also runs once on load. Zoom or pan, then scan again.
4. Click a crop marker to open its full suggestions in the panel.
5. Optional: click **Connect wallet**, then **Sign field report** to sign the current result.

## How the prediction works

For each point, the app takes the total rain over the last 30 days and the mean temperature over the same period. Each crop gets a score from 0 to 1:

```
fit = temperatureScore × rainScore
```

Each part scores 1 when the value is inside the crop's ideal range and falls off linearly to 0 as it moves outside. Temperature falls to 0 at 6 °C outside the range. Rain falls to 0 at 60% of the range's upper bound outside it.

The built-in ranges:

| Crop | Temperature (°C) | Rain (mm / 30 days) |
|---|---|---|
| Maize | 18–27 | 60–150 |
| Beans | 15–24 | 50–130 |
| Rice | 22–30 | 150–400 |
| Sorghum | 22–33 | 30–90 |
| Wheat | 10–22 | 40–100 |
| Potatoes | 10–20 | 50–120 |
| Tea | 13–24 | 100–250 |
| Coffee | 15–25 | 80–200 |
| Cassava | 22–32 | 50–150 |
| Bananas | 23–30 | 100–250 |

These are rough approximations set by hand, not official agronomy data. To change or add crops, edit the `CROPS` array near the top of the script in `index.html`.

## How HTMX is used

HTMX normally fetches HTML fragments from a server. This project has no server, so each `hx-get` path (`/advice`, `/grid`, `/search`, `/connect`, `/sign`) is intercepted in the `htmx:beforeRequest` event. A matching JavaScript function builds the HTML and HTMX-enabled elements inside it keep working through `htmx.process`. If you later add a backend, you can delete the interceptor and return the same HTML from real endpoints.

## Limitations

- **Short window:** predictions use the last 30 days only. They show current conditions, not the long-term climate or growing season.
- **No soil, altitude or irrigation data.** These matter a lot for real planting decisions.
- **Coarse grid:** the scan samples 20 points per view, so zooming in gives finer detail.
- **Open-Meteo limits:** the free API has fair-use rate limits. If you scan very often you may see an error message until it resets.
- **Wallet required for signing only:** everything else works without a wallet.
- **Not tested on real devices yet.** Please report anything that misbehaves.

Check long-term climate and soil conditions with your local agricultural extension office before planting.

## Troubleshooting

- **Blank map or grey tiles:** serve the folder over HTTP (see above) and check your internet connection.
- **"No wallet found":** install a browser wallet extension and reload the page.
- **Weather error message:** wait a minute and try again, or zoom in to reduce the area scanned.
- **Search finds nothing:** try a larger nearby town or a different spelling.

## Possible next steps

- Add long-term climate averages (for example Open-Meteo's archive API) for more reliable seasonal predictions.
- Add a rain-radar overlay to the map.
- Add soil data from a service such as ISRIC SoilGrids.
- Let users edit crop ranges in the UI.

## Credits and licenses

Map data © OpenStreetMap contributors. Weather data from Open-Meteo (free for non-commercial use; check their terms for commercial use). Leaflet, HTMX and ethers.js are open-source projects under their own licenses.
