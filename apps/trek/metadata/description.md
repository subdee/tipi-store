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

Source: <https://github.com/liketrek/TREK> · Docker image: `mauriceboe/trek`
