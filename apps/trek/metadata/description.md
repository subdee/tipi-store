# TREK

A self-hosted, real-time collaborative travel planner — with maps, budgets, packing lists, a journal, and AI built in.

TREK lets you plan your trips end to end and keep everything on your own server. Build day-by-day itineraries, plot routes on interactive maps, split expenses, track budgets, keep packing lists, and journal your journeys — collaborating with fellow travellers in real time.

## Core features

- **Trip planner** — day plans, routes and interactive maps
- **Real-time collaboration** — plan trips together, live
- **Budgets & costs** — expense tracking and splitting
- **Journey journal** — capture memories as you go
- **Atlas** — visualise the countries you've visited
- **Collections** — saved lists of places
- **Packing lists** — never forget the essentials
- **PWA support** — install it on any device
- **SSO / OIDC** — single sign-on for your household or team

## Configuration notes

- **Encryption key** — TREK uses an `ENCRYPTION_KEY` to encrypt stored secrets (API keys, MFA, SMTP and OIDC settings) at rest. Runtipi generates one for you (equivalent to `openssl rand -hex 32`) on install. Keep it stable across upgrades; rotating it requires re-entering any stored secrets.
- **TREK's own map services (5.1.0 and newer)** — address search now goes to TREK's Photon server, routing to TREK's Valhalla and place details to TREK's OpenStreetMap copy, with the old public endpoints as silent fallbacks. Each install registers itself for a key in the background on the first search, so search for any place once after updating; nothing to configure. Only request counts are recorded, not searches. Set `TREK_SERVICES=off` through a user-config override to stop the key and go back to the public services, plus `TREK_PLACES_ENABLED=false` to stop place search asking TREK's index, or point `NOMINATIM_URL` at your own geocoder.
- **Sharing place details is off** until an admin turns it on under Admin, Settings. When enabled, only the place detail itself leaves (the field, the value and the place it belongs to) and the Transmission log shows every value sent. No user, trip, note, file or search is included.

Source: <https://github.com/liketrek/TREK> · Docker image: `mauriceboe/trek`
