# openGym

A self-hosted gym and body-weight tracker you actually own. No account on someone else's server, no subscription, no telemetry.

## Core features

- **Weekly plan** — a routine per weekday, built from a library of 1,324 exercises with animated demos, plus your own custom ones
- **Guided workouts** — it starts today's session, pre-fills your weights from last time, runs the rest timer, detects PRs and keeps the screen awake while you train
- **Progression that follows a rule** — linear, Greyskull LP, double progression or added time, per routine and overridable per exercise, with stalls and deloads handled for you
- **Supersets, warm-up sets, timed exercises and cardio** — including reps-per-side and bodyweight exercises that progress in reps instead of load
- **Body weight & stats** — weight chart with a goal line, estimated 1RM per exercise, activity heatmap and a muscle map showing balance, fatigue and strength
- **Passkeys, not passwords** — Face ID / Touch ID / fingerprint login, one profile per person, synced across devices
- **Imports and exports** — bring history in from FitNotes, Strong, Hevy or an Apple Health export; take everything out again as JSON
- **Installable** — full PWA, so it can go on a phone home screen; push notifications for rest timers and workout-day reminders work out of the box

## Configuration notes

- **Public URL and passkey hostname** — passkeys are bound to the exact host you open openGym with. Set the public URL to the full address including scheme and port (no trailing slash), and the passkey hostname to just its host part. A mismatch between the two is the single most common reason sign-in fails. Browsers also require HTTPS for passkeys, with `localhost` as the only exception, so reaching openGym over plain HTTP on a LAN IP will not let you register one — put it behind your reverse proxy or Runtipi's exposed domain first.
- **Admin dashboard** — off by default, and a fresh instance has open signup and no admin. To enable it, register your passkey profile first, look up your id in `${APP_DATA_DIR}/data/app/db.json` (`users[].id`), enter it in the Admin user IDs field and restart the app.
- **Exercise media** — on first start a one-shot `media` container downloads roughly 140 MB of exercise images and animations from [hasaneyldrm/exercises-dataset](https://github.com/hasaneyldrm/exercises-dataset) into `${APP_DATA_DIR}/data/media`. It runs alongside the app rather than blocking it, so openGym is usable straight away and the exercise images and animations appear once the download finishes, a few minutes in. Later starts skip it.

## Data layout

- `${APP_DATA_DIR}/data/app` — profiles, public passkeys, per-user plans and workouts, body weight, the activity log and the session-cookie secret. **Back this up and you have backed up everything** — passkey private keys never touch the server, they stay on your devices.
- `${APP_DATA_DIR}/data/media` — the downloaded exercise images and animations, safe to delete and re-fetch

Source: <https://gitlab.com/DuarteSantos8/opengym> · Docker images: `registry.gitlab.com/duartesantos8/opengym/web`, `registry.gitlab.com/duartesantos8/opengym/api`
