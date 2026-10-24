# CA Watch

A live, ZIP-code-specific hazard dashboard for California. Enter a California ZIP code and get
active NWS alerts, a 10-day outlook, 48-hour hazard readouts, nearby river gauges, active
wildfires, neighborhood weather stations, plus road-camera and power-outage links, all in one
page that installs to a phone's home screen. No login, no backend, no build step.

Live at: https://trumedicines.github.io/CA-warning-system/ once you publish it (see below).
The repo name `CA-warning-system` is an assumption that follows your WA and OR repos; if you
use a different name, only the URL changes.

Adapted from [WA Watch](https://trumedicines.github.io/wa-warning-system/) and
[OR Watch](https://trumedicines.github.io/OR-warning-system/), the Washington and Oregon
versions of the same dashboard.

## What it does

- **Alerts** — active National Weather Service watches, warnings and advisories for the ZIP,
  including Red Flag Warnings, flood and winter-storm alerts. The section header is green when
  clear and turns red when an alert is active; open it for the full text and "what to do"
  instructions.
- **10-day outlook** — 12-hour day and night panels. Each day panel shows an icon for that
  day's weather (sun, partly sunny, cloud, rain, snow, thunderstorm, fog) and each night panel
  shows the moon phase. Panels include wind, precipitation chance, sea-level pressure with a
  rising/falling arrow, and sunrise or sunset. The first seven days are the NWS forecast; days
  8–10 (dashed panels) are Open-Meteo model forecasts and are less certain.
- **Hazard readouts, next 48 hours** — gauges for wind gusts, snow, rain/flood, lightning
  (NWS Lightning Activity Level), cold exposure and heat.
- **River & stream gauges** — nearest USGS gauges.
- **Hyper-local stations** — Weather Underground personal weather stations (see the API key
  note below).
- **Wildfire activity** — active fire perimeters near the ZIP. Burn scars also carry
  flash-flood and debris-flow risk during heavy rain, which matters across California.
- **Traffic cameras** — links to Caltrans QuickMap and highway conditions.
- **Power outages** — links to major utility outage maps, plus a note about Public Safety
  Power Shutoffs.
- **Winter outlook** — a short, hand-written summary of the Old Farmer's Almanac forecast.
- **More California hazard tools** — CAL FIRE incidents, the AirNow smoke map, USGS
  earthquakes, MyShake earthquake alerts, Cal OES MyHazards, and the Sierra Avalanche Center.
- **Get alerts** — browser notifications, plus optional free text messages (below).

Every section is collapsible: tap its header to open or close it. Only the 10-day outlook
starts open. Change the `open` class on any `<section>` in `index.html` to alter the defaults.

## Data sources

| Section | Source | Live? |
|---|---|---|
| Alerts, forecast days 1–7 | [National Weather Service API](https://www.weather.gov/documentation/services-web-api) | Yes, no key |
| Forecast days 8–10, pressure, sunrise/sunset | [Open-Meteo](https://open-meteo.com/) | Yes, no key |
| Lightning risk | NWS gridpoint data (Lightning Activity Level) | Yes, no key |
| River & stream gauges | [USGS Water Services](https://waterservices.usgs.gov/) | Yes, no key; nearest gauges within ~30 mi |
| Wildfire perimeters | [NIFC / WFIGS](https://data-nifc.opendata.arcgis.com/) | Yes, no key; active fires within ~75 mi |
| Neighborhood stations | [Weather Underground](https://www.wunderground.com/) | Links by default; live readings need an API key |
| ZIP → location | [Zippopotam.us](https://www.zippopotam.us/) | Yes, no key |
| Traffic cameras | [Caltrans QuickMap](https://quickmap.dot.ca.gov/) | Links only |
| Power outages | PG&E, Southern California Edison, SDG&E, LADWP, SMUD | Links only |
| Winter outlook | [The Old Farmer's Almanac](https://www.almanac.com/winter-extended-forecast-farmers-almanac) | Static text, updated once a year |
| California hazard tools | CAL FIRE, AirNow, USGS, MyShake, Cal OES, Sierra Avalanche Center | Links only |

### Why cameras and outages are links, not live feeds

**Traffic cameras:** Caltrans QuickMap is California's official source for road conditions,
closures, chain controls and camera snapshots. Caltrans recommends linking to QuickMap rather
than embedding it in other sites, so this section links straight to QuickMap and to Caltrans'
highway-conditions page.

**Power outages:** there is no single free, official statewide outage feed. Each utility runs
its own map, so the section links to five major ones. California also has many other
municipal utilities and co-ops. To add yours, add one line to the `UTILITY_LINKS` list in
`index.html`:

```js
{ name: 'Your utility', url: 'https://example.com/outage-map' },
```

### The Weather Underground API key (optional)

Weather Underground's data API requires a key. Without one, that section shows links to
Wundermap and the city's Weather Underground page. To show live nearby-station readings,
paste a key into `WU_API_KEY` near the top of the script in `index.html`. Anything in that
file is visible to every visitor, so only use a key you are comfortable making public.

### The winter outlook section

The Almanac has no API and its forecast changes once a year, so this is a hand-written
summary, labeled with the date it was written. California spans two of the Almanac's regions
(Pacific Southwest and, in the north, Pacific Northwest), and the Almanac's region boundaries
don't follow county lines, so the text points readers to its regional map. To refresh it each
year, read the new regional forecasts, rewrite the blurb in your own words in the
`data-key="almanac"` section of `index.html`, and update the "Last updated" date.

## 1. Deploy it (free, about 5 minutes)

**GitHub Pages**

1. Create the repo `trumedicines/CA-warning-system` on GitHub and push everything in this
   folder to it (see the commands below).
2. In the repo go to **Settings → Pages**, set Source to "Deploy from a branch", Branch
   `main` / `(root)`, and save.
3. After a minute the site is live at `https://trumedicines.github.io/CA-warning-system/`.

```bash
git init -b main
git add .
git commit -m "CA Watch: California weather hazard dashboard"
git remote add origin https://github.com/trumedicines/CA-warning-system.git
git push -u origin main
```

Any other free static host (Cloudflare Pages, Netlify, Vercel) works the same way.

## 2. Getting real text messages (the honest tradeoffs)

A static site can't send a text message while your phone is asleep and the site is closed;
that always needs something running on a server. Three options, cheapest first:

### Option A: Browser notifications (built in, $0)

Click "Enable browser alerts" in the "Get alerts" section. While the tab is open, or the
site is installed to your home screen and running, you get a notification within about a
minute of a new NWS alert. It does not work if the app is fully closed.

### Option B: Free SMS via GitHub Actions + carrier email gateway

This repo includes a ready-to-use version:

- `scripts/check-alerts.js` checks alerts for everyone in `registrations.json` every 15
  minutes (via `.github/workflows/alert-check.yml`, free on public repos).
- New alerts are emailed to `<phone>@<carrier-gateway>` (for example
  `4155551234@tmomail.net`), which most US carriers deliver as a free text. Only California ZIP
  codes are processed.

**Setup**

1. Add each subscriber to `registrations.json`:
   `{ "zip": "94102", "phone": "4155551234", "carrier": "tmobile" }`. Supported carriers are
   listed at the top of `scripts/check-alerts.js`. The file starts empty on purpose.
2. Create a Gmail account (or use an existing one) and generate an
   [App Password](https://myaccount.google.com/apppasswords).
3. In the GitHub repo, go to **Settings → Secrets and variables → Actions** and add
   `EMAIL_USER` (the Gmail address) and `EMAIL_PASS` (the App Password).
4. The workflow then runs every 15 minutes. You can also trigger it from the **Actions** tab.

**Caveats:** carrier email-to-text gateways are unofficial and can be delayed, rate-limited
or blocked. Phone numbers committed to a public repo are public, so use a private repo or a
small database if you open this up beyond a few people.

### Option C: Paid SMS provider (most reliable)

Use [Twilio](https://www.twilio.com/) or similar: roughly $0.80 per 100 texts, plus a small
serverless function to replace the email step in `check-alerts.js`. The registration list,
alert polling and schedule stay the same.

## 3. Customizing

- **Colors, fonts, layout** are in the `<style>` block at the top of `index.html`. Each
  section header has its own pastel tint in a `section[data-key="..."] .section-toggle` rule.
- **Search radii** (30 mi for river gauges, 75 mi for wildfires) are the `milesBBox(...)`
  calls inside `loadRivers` and `loadFires`.
- **App icon** is in `icons/`. `icon.svg` is the editable source; the PNGs are exports of it
  (`icon-192.png`, `icon-512.png`, a padded `icon-maskable-512.png` for Android, and
  `apple-touch-icon.png` for iPhone). The 32 px browser-tab icon and the iPhone icon are also
  embedded directly in the `<head>` of `index.html`.
- **Sponsor bar** at the bottom rotates slots using the vendored
  [AdShuffle](https://github.com/codions/ad-shuffle) library (MIT license) in
  `vendor/ad-shuffle.js`. Each `.rb-random-ads` block in `index.html` is one slot; add, edit
  or delete them freely.

## Limits and disclaimer

This is a community tool built on public data, not an official emergency system. Always
follow guidance from the National Weather Service, CAL FIRE and local emergency management
for life-safety decisions, including evacuation orders. Third-party services (NWS,
Open-Meteo, USGS, NIFC, Weather Underground) can occasionally be slow or briefly unavailable;
each section is built to fail on its own with a plain message rather than break the whole
dashboard.
