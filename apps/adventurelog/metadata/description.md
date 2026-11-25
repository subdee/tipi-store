<div align="center">
  <h1>AdventureLog</h1>
  <p>
    The ultimate travel companion for the modern-day explorer.
  </p>
</div>

## ⭐ About the Project

Starting from a simple idea of tracking travel locations, AdventureLog has grown into a full-fledged travel companion.
With AdventureLog, you can log your adventures, keep track of where you've been on the world map, plan your next trip
collaboratively, and share your experiences with friends and family.

AdventureLog was created to solve a problem: the lack of a modern, open-source, user-friendly travel companion. Many
existing travel apps are either too complex, too expensive, or too closed-off to be useful for the average traveler.
AdventureLog aims to be the opposite: simple, beautiful, and open to everyone.

<!-- Features -->

### 🎯 Features

- **Track Your Adventures** 🌍: Log your adventures and keep track of where you've been on the world map.
    - Locations can store a variety of information, including the location, date, and description.
    - Locations can be sorted into custom categories for easy organization.
    - Locations can be marked as private or public, allowing you to share your adventures with friends and family.
    - Keep track of the countries and regions you've visited with the world travel book.
    - Upload trails and activities to your locations to remember your experiences with detailed maps and stats.
- **Plan Your Next Trip** 📃: Take the guesswork out of planning your next adventure with an easy-to-use itinerary
  planner.
    - Itineraries can be created for any number of days and can include multiple destinations.
    - Itineraries include many planning features like flight information, notes, checklists, and links to external
      resources.
    - Itineraries can be shared with friends and family for collaborative planning.
- **Share Your Experiences** 📸: Share your adventures with friends and family and collaborate on trips together.
    - Locations and itineraries can be shared via a public link or directly with other AdventureLog users.
    - Collaborators can view and edit shared itineraries (collections), making planning a breeze.

## Configuration notes

- **v0.14.0 fixes several privately reported security issues,** so taking it is strongly recommended.
  Database migrations run automatically when the containers start — **back up first**, as upstream asks.
- **The admin account** is `admin` with the password from the form field, created on the very first start
  only. Changing the field later does not change the password. From v0.14.0 an empty or weak value (such as
  `admin`) is replaced by a generated password, printed once in the backend log:
  `docker logs adventurelog_<store>-backend-1 | grep -A3 "Admin username"`.
- **Registration is open by default.** Upstream's `DISABLE_REGISTRATION` defaults to `False`, and this app
  does not override it, so anyone who can reach the instance can create an account. Set
  `DISABLE_REGISTRATION=True` through a user-config override if you expose it, and
  `DISABLE_REGISTRATION_MESSAGE` for the text people see.
- **Two new privacy switches in v0.14.0.** `ALLOW_PUBLIC_ACCESS=False` keeps everything behind sign-in —
  public and unlisted travel stories, public places, profiles and guest links.
  `INTEGRATIONS_ALLOW_PRIVATE_NETWORKS` is on by default, which is what lets AdventureLog reach an Immich,
  Wanderer or Endurain on the same host; turn it off if strangers can sign up, so no user can point an
  integration at your own network.
- **Uploads.** `BODY_SIZE_LIMIT` now sets the bundled nginx limit as well as the frontend's, and this app
  passes `Infinity` to match upstream. It used to cap bodies at 100 KB, which was small enough to refuse
  photo uploads.
- **Background work needs no extra containers.** Since v0.14.0 geocoding, backups, collection archives,
  PDFs, restores and imports run in a worker inside the backend container, replacing the old cron jobs, and
  caching uses a bundled memcached. To use Redis instead, add `CACHE_BACKEND=redis` and a `REDIS_URL`
  through an override and run your own Redis.
- **ARM** — AdventureLog needs PostGIS, and the official `postgis/postgis` images are amd64 only. This app
  uses `imresamu/postgis`, the multi-arch rebuild, so the same configuration works on both architectures.
- **API clients must be updated for v0.14.0.** Every endpoint moved under `/api/`, browser sign-in under
  `/api/auth/browser/v1/`, and error bodies changed shape. Interactive docs are served at `/api/docs/` on
  your own instance.
- **Fixing older data.** v0.14.0 ships management commands for records saved by earlier versions —
  `activity_elevation_fix` re-measures elevation, and `fix_location_countries` and
  `reassign_territory_locations` correct places filed under the wrong country. Run them with
  `docker exec adventurelog_<store>-backend-1 python manage.py <command>`.

## Data layout

- `${APP_DATA_DIR}/data/postgres` — the PostGIS database: locations, collections, trips and visits.
  **This is the one to back up.**
- `${APP_DATA_DIR}/adventurelog_media` — uploaded images, attachments and GPX tracks.

<!-- Roadmap -->

## 🧭 Roadmap

The AdventureLog Roadmap can be found [here](https://github.com/users/seanmorley15/projects/5)
