# TouringTogether

A private, real-time coordination app for **motorcycle touring groups of 2–6
riders**. Share live GPS positions on one map, agree on a single destination,
get an individual route each, and talk over Discord voice.

Static frontend — HTML5, CSS3, vanilla ES modules, Firebase (Auth + Realtime
Database), Leaflet and OpenStreetMap. No build step, no server to keep running,
deployable on GitHub Pages and wrappable in an Android WebView (AppMint).

---

## Quick start

```bash
cd TouringTogether
python3 -m http.server 8080
# open http://localhost:8080
```

Out of the box it runs in **demo mode**: Firebase is not configured, so accounts
and rooms are stored in your browser (`js/mock-firebase.js`). Everything is
clickable — register, create a room, join with the code, share a location. An
amber banner says so.

### Going live

1. Create a Firebase project, enable Email/Password auth and Realtime Database.
2. Copy your SDK config into `js/config.js` (see `firebase/README.md`).
3. Deploy `firebase/database.rules.json` (and `storage.rules` for avatars).

```bash
firebase deploy --only database
```

---

## Features

| Area | What works |
| --- | --- |
| Auth | Email + password, registration, password reset, persisted sessions, sign-out |
| Profile | Display name, avatar URL or upload (with Storage), motorcycle, private e-mail |
| Rooms | Create (random id + 50-bit invitation code), join by code, leave, close, delete, rotate code, kick riders |
| Map | Leaflet + OSM tiles, avatar rider markers, live/stale/offline states, legend, recenter, fit-all, route toggle, collapsible rider list |
| GPS | `watchPosition()` with distance + time throttling, accuracy display, permission/denied/timeout/stale handling, stops on leave or sign-out |
| Presence | Realtime Database `.info/connected` + `onDisconnect()`, heartbeats, "last seen" |
| Destination | Nominatim place search (debounced, must be confirmed) **or** manual `lat, lng` with validation and optional reverse geocoding |
| Routing | OSRM-compatible provider with fallback, per-rider route from own position, distance + ETA, external navigation fallback |
| Discord | Validated https invite/channel link, Android intent with browser fallback, owner-configurable |
| PWA | Manifest, service worker (offline shell), install icons |

---

## Project structure

```
index.html                 splash → redirects by auth state
manifest.webmanifest       PWA manifest (relative paths)
sw.js                      service worker (offline app shell)
.nojekyll                  required so GitHub Pages serves /js and /pages

css/
  tokens.css               colour, type, spacing, shadow tokens
  base.css                 reset, buttons, inputs, typography
  components.css           cards, modals, toasts, riders, map markers
  layout.css               page shells, touring screen, bottom sheet
  responsive.css           phone-first breakpoints

js/
  config.js                Firebase config + all tunable constants  ← edit this
  firebase.js              SDK bootstrap, demo-mode switch, path helpers
  mock-firebase.js         localStorage stand-in used only in demo mode
  auth.js                  sign-in / register / reset / session
  profile.js               profile read, write, avatar helpers
  rooms.js                 create, join, leave, destination, settings
  presence.js              connection state and onDisconnect()
  location.js              GPS publishing with throttling
  map.js                   Leaflet markers, route line, view helpers
  destinations.js          Nominatim search + reverse geocoding
  routing.js               OSRM requests, recalculation policy
  discord.js               URL validation + Android intent handling
  permissions.js           room roles, geolocation permission
  router.js                route guards, page navigation
  ui.js                    top bar, toasts, modals, busy states
  geo.js                   haversine, parsing, formatting, debounce
  app.js                   splash bootstrap + service worker
  pages/                   one controller per screen

pages/                     login, register, dashboard, touring, profile
firebase/                  database.rules.json, storage.rules, README
assets/icons               SVG + PNG icons, default avatar
```

All links and asset paths are **relative**, so the app works unchanged under
`https://USERNAME.github.io/REPOSITORY/` — no `<base>` hacks, no absolute URLs.

---

## Deploying to GitHub Pages

1. Push the repository.
2. **Settings → Pages → Build and deployment → Source: Deploy from a branch**,
   branch `main`, folder `/root` (or `/docs` if you move the folder there).
3. Wait for the build, then open the URL.
4. In Firebase: **Authentication → Settings → Authorized domains** → add
   `USERNAME.github.io`.

`.nojekyll` is included; without it GitHub Pages hides directories that start
with an underscore and can break asset paths.

### Android WebView wrapper (AppMint / similar)

* Point the wrapper at your Pages URL.
* Grant **location permission** at the native level too; the WebView delegates
  `navigator.geolocation` to the app, and a denied app permission never reaches
  the browser prompt.
* Allow `https://nominatim.openstreetmap.org`, `https://router.project-osrm.org`
  and the tile/CDN hosts, or the map and search will silently fail.
* Keep the screen on while riding: browsers throttle or freeze timers and GPS in
  the background (see limitations).

---

## Tuning

Everything adjustable lives in `js/config.js`:

* `APP_CONFIG.gps` — `enableHighAccuracy`, `maximumAge`, `timeout`,
  `minDistanceMeters` (publish after this much movement), `minIntervalMs`,
  `stationaryIntervalMs`, `staleAfterMs` (when a rider stops counting as live).
* `APP_CONFIG.room` — `maxMembers` (6), `minMembersForActive` (2), code length.
* `APP_CONFIG.map` — tile provider, zoom, fallback centre.
* `APP_CONFIG.geocoding` — endpoint, debounce, result limit.
* `APP_CONFIG.routing` — provider list (first one that answers wins),
  `recalcDistanceMeters`, `recalcIntervalMs`.
* `APP_CONFIG.discord` — allowed hosts, Android package.

---

## Honest limitations

* **Background GPS is unreliable.** A page loaded in a browser or WebView is
  suspended when the screen locks; updates stop. The app detects this on resume
  and takes a fresh fix, but for guaranteed background tracking you need a
  native Android foreground service.
* **Routing is a car profile over OpenStreetMap data.** It is not traffic aware,
  not guaranteed, and not verified as motorcycle-safe (no road-class, surface or
  toll filtering). Treat ETA as a free-flow estimate.
* **No push notifications.** Destination changes are announced inside the app
  only.
* **Nominatim usage policy** applies to destination search: debounced requests,
  no heavy automated traffic. Swap in a commercial geocoder if you outgrow it.
* **Capacity (6) and `memberCount` are enforced client-side** — see
  `firebase/README.md` §5 for why and what to do about it.
* **Discord** is only a link hand-off. Joining a channel, mute, push-to-talk and
  permissions are entirely Discord's business; TouringTogether never asks for
  Discord credentials or tokens.

---

## Browser support

Chromium-based browsers (Chrome/Edge/Android WebView), Firefox and Safari 16+.
ES modules, `crypto.getRandomValues`, `crypto.subtle` and the Permissions API
are used; `crypto.subtle` and the service worker require a secure context
(https or localhost).
