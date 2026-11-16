# Dawarich

Self-hosted location history: your own Google Maps Timeline, on your own server.

Dawarich stores the places you have been and draws them on a map. Import a history you already
have, or point a phone app at it and let it collect new points continuously.

## Core features

- Interactive map of your whole history, with heatmaps, routes, point clustering and a
  fog-of-war layer showing how much of the world you have actually covered
- Imports from Google Maps Timeline (phone and account takeout), Owntracks, Immich,
  Photoprism, Strava, GeoJSON and GPX
- Live tracking from Owntracks, Overland, GPSLogger, Dawarich's own iOS and Android apps, or
  anything that can POST to its API
- Visits and places: clusters of points become suggested visits you confirm, rename or merge
- Stats per month and year — points, distance, countries and cities visited, with a trip list
- Reverse geocoding against a geocoder you run yourself, so the history never has to leave
  your network

## Configuration notes

> [!IMPORTANT]
> **A default admin account is created on first start: `demo@dawarich.app` with the password
> `safepassword`.** Dawarich runs its seed task on every boot, and that task creates this admin
> whenever the instance has no users. Sign in and change the address and password immediately,
> before exposing the app — otherwise anyone who can reach it has admin access with a password
> published in Dawarich's source.

- **Secret key base** — Rails signs sessions and cookies with it. Runtipi generates one on
  install (equivalent to `openssl rand -hex 64`). Keep it stable: changing it invalidates every
  session and signs everyone out.
- **The exposed domain is wired in for you.** Rails refuses any request whose `Host` header is
  not on its allow-list, answering `403`, so `APPLICATION_HOSTS` is set from the domain you
  expose plus localhost. If you reach the app by some *other* name or by IP, add it to
  `APPLICATION_HOSTS` through a user-config override, or it will refuse to answer.
- **Reverse geocoding is off until you configure it.** Place and city names come from a geocoder
  you choose, set either in the app's own settings or with `PHOTON_API_HOST` /
  `NOMINATIM_API_HOST` (plus the `GEOAPIFY_API_KEY` and `LOCATIONIQ_API_KEY` alternatives) via a
  user-config override. Point it at a Photon or Nominatim you host yourself to keep your
  coordinates on your own network. Without it, points are still recorded and mapped; they just
  have no place names.
- **Two-factor authentication needs three extra keys** that this app does not set:
  `OTP_ENCRYPTION_PRIMARY_KEY`, `OTP_ENCRYPTION_DETERMINISTIC_KEY` and
  `OTP_ENCRYPTION_KEY_DERIVATION_SALT`. Upstream ships them commented out and documents no
  format, so add them through a user-config override if you want 2FA.
- **Web and background jobs share one container.** `SIDEKIQ_EMBEDDED` is on and
  `WEB_CONCURRENCY` is `0`, matching upstream's current compose — imports and geocoding run
  inside the same process as the web server. Raise
  `BACKGROUND_PROCESSING_CONCURRENCY` (default `2`) if large imports are slow.
- **ARM** — Dawarich needs PostGIS, and the official `postgis/postgis` images are amd64 only.
  This app uses `imresamu/postgis`, the multi-arch rebuild Dawarich points ARM users at, so the
  same configuration works on both architectures.

## Data layout

- `${APP_DATA_DIR}/data/postgres` — the PostGIS database: every point, visit, place and trip.
  **This is the one to back up.**
- `${APP_DATA_DIR}/data/storage` — Active Storage: uploaded import files and generated exports.
- `${APP_DATA_DIR}/data/public` — generated assets served by the app.
- `${APP_DATA_DIR}/data/watched` — drop files here and Dawarich imports them automatically,
  one subfolder per user email (see the *Watched folder* section of the docs).
- `${APP_DATA_DIR}/data/redis` — the job queue. Rebuilt on its own, so it needs no backup.

Source: <https://github.com/Freika/dawarich> · Docs: <https://dawarich.app/docs> · Docker image:
`freikin/dawarich`
