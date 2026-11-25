<div align="center">
  <img src="https://github.com/joaovitoriasilva/endurain/blob/master/frontend/app/public/logo/logo.png?raw=true" width="128" height="128">

# Endurain

<a title="Crowdin" target="_blank" href="https://crowdin.com/project/endurain"><img src="https://badges.crowdin.net/endurain/localized.svg"></a>
![License](https://img.shields.io/github/license/joaovitoriasilva/endurain)
[![GitHub release](https://img.shields.io/github/v/release/joaovitoriasilva/endurain)](https://github.com/joaovitoriasilva/endurain/releases)
[![GitHub stars](https://img.shields.io/github/stars/joaovitoriasilva/endurain.svg?style=social&label=Star)](https://github.com/joaovitoriasilva/endurain/stargazers)

**A self-hosted fitness tracking service**  
Visit Endurain's [Mastodon profile](https://fosstodon.org/@endurain)
and [Discord server](https://discord.gg/6VUjUq2uZR).

  <img src="https://github.com/joaovitoriasilva/endurain/blob/master/screenshot_01.png?raw=true" alt="Endurain Screenshot">
</div>

## Table of Contents

- [Endurain documentation](https://docs.endurain.com)
- [What is Endurain?](#what-is-endurain)
- [Endurain screenshots](https://docs.endurain.com/gallery/)
- [Sponsors](#sponsors)
- [Contributing](#contributing)
- [Help Translate](#help-translate)
- [License](#license)

## What is Endurain?

Endurain is a self-hosted fitness tracking service designed to give users full control over their data and hosting
environment. It's similar to Strava but focused on privacy and customization. Built with:

- **Frontend:** Vue.js, Notivue and Bootstrap CSS
- **Backend:** Python FastAPI, Alembic, SQLAlchemy, stravalib and python-garminconnect for Strava and Garmin Connect
  integration, gpxpy and fitdecode for .gpx and .fit file import respectively
- **Database:** MariaDB or PostgreSQL for efficient data management
- **Observability:** Jaeger for basic tracing and monitoring
- **Integrations:** Supports Strava and Garmin Connect. Manual upload of activities using .gpx and .fit files are also
  supported

To deploy Endurain, a Docker image is available, and a comprehensive example can be found in the "
docker-compose.yml.example" file provided. Configuration is facilitated through environment variables, ensuring
flexibility and ease of customization.

For more information please see the Endurain's [documentation](https://docs.endurain.com).

## Configuration notes

### v0.21.0: back up before updating

v0.21.0 rewrites the schema across 17 revisions, **converting every integer id
to a UUID in a single transaction**. The conversion cannot be undone: restoring
a backup is the only way back to v0.20.x. On the first start it also renames
stored files and import folders to the new UUIDs.

Before taking the update:

```bash
# 1. Confirm v0.20.x finished its data migrations. v0.21.0 only upgrades from
#    v0.20.x with none pending, and the first revision refuses anything else
#    and says why. If any are still running, leave the app on v0.20.x until
#    they finish.

# 2. Stop the app, leaving Postgres up, then dump the database and the data dir
docker stop endurain_<store>-app-1
docker exec endurain_<store>-postgres-1 \
  pg_dump -U endurain -d endurain -Fc -f /tmp/endurain-pre-0.21.dump
docker cp endurain_<store>-postgres-1:/tmp/endurain-pre-0.21.dump .
tar -czf endurain-data-pre-0.21.tar.gz -C <APP_DATA_DIR>/data data

# 3. Take the update. The schema upgrade runs on start, so expect downtime
#    proportional to the size of the database.

# 4. Afterwards, let the planner catch up with the rewritten tables
docker exec endurain_<store>-postgres-1 \
  psql -U endurain -d endurain -c 'VACUUM (ANALYZE);'
```

What changes once it is up:

- **Everyone signs in again**, once. API keys keep working, and any key that had
  no expiry now expires 365 days after the upgrade. Every browser sends one
  "new device" notice at its first sign-in.
- **Third-party clients break.** The API changed throughout — UUID ids, renamed
  scopes, no CSRF token, reorganised routes. The Endurain mobile app must be at
  least 0.11.0, and anything else that talks to the API (GadgetBridge, scripts)
  needs to support v0.21.0 before you upgrade.
- **Activity archives exported before v0.21.0 can no longer be imported.**
  Export again afterwards if you keep archives.
- **A new install no longer creates `admin`/`admin`.** The server log prints a
  one-time setup token, valid 24 hours, and the onboarding page uses it to
  create the first administrator:
  `docker logs endurain_<store>-app-1 | grep -i 'setup token'`.

Environment variables this app handles for you:

- **`TRUSTED_PROXIES` is now required**, because this app runs Endurain behind
  Runtipi's proxy with `BEHIND_PROXY=true`, and v0.21.0 refuses to start when
  that is set without a usable entry. The default trusts the private ranges
  Docker and the proxy use; narrow it with the form field if you want to.
- **`SECRET_KEY` must be at least 32 characters with at least 10 distinct ones**,
  and the example value from Endurain's docs is refused. New installs get a
  64-character key. An existing 32-character key passes; if Endurain does refuse
  yours, replacing it is safe here — this upgrade signs everyone out anyway.
- **`FERNET_KEY` must not be the example value** published in Endurain's `.env.example`
  or docs, or the container refuses to start. A key from fernetkeygen.com is fine.
- **The Geocodes Maps API key is deprecated.** v0.21.0 imports it into
  *Settings > Server > Reverse geocoding* on the first start, shows it read-only
  while the variable is set, and logs one warning per start. Leave the field as
  it is for this release so the import happens — **v0.22.0 refuses to start
  while it is set**, so it will be removed from this app at that point, along
  with reverse geocoding, email and session settings generally moving into the
  admin UI.

### Postgres 18: one-time manual migration required

This app moved from `postgres:17.5` to `postgres:18` to match what Endurain
ships upstream. **Postgres cannot read a data directory written by an older
major version**, so updating this app will leave the database container
failing to start until you migrate the data once, by hand. Nothing is
migrated for you, and there is no automatic path.

Do this *before* taking the app update, while 17.5 is still running. Replace
`<APP_DATA_DIR>` with your app's data directory and `<PROJECT>` with the
compose project name (`docker ps` will show both).

```sh
# 1. Stop only the app, leaving Postgres up so nothing writes during the dump
docker stop <PROJECT>-app-1

# 2. Dump with 18's pg_dump against the running 17 server
mkdir -p /root/endurain-pg18 && chmod 700 /root/endurain-pg18
docker run --rm --network <PROJECT>_<PROJECT>_network \
  -v <APP_DATA_DIR>:/cfg:ro -v /root/endurain-pg18:/dump postgres:18 \
  sh -c 'export PGPASSWORD=$(grep -m1 "^DB_PASSWORD=" /cfg/app.env | cut -d= -f2-); \
         pg_dump -h <PROJECT>-postgres-1 -U endurain -d endurain -Fc -f /dump/endurain.dump'

# 3. Check the dump before destroying anything: expect ~45 tables
docker run --rm -v /root/endurain-pg18:/dump postgres:18 \
  pg_restore -l /dump/endurain.dump | grep -c "TABLE DATA"

# 4. Stop Postgres and MOVE the old cluster aside - do not delete it.
#    Keep it outside the app data directory so it stays out of Runtipi backups.
docker stop <PROJECT>-postgres-1
mv <APP_DATA_DIR>/data/postgres /root/endurain-pg18/postgres-pg17

# 5. Take the app update in Runtipi. Postgres 18 initialises an empty cluster
#    and Endurain builds its own schema on first start.

# 6. Stop the app again, then replace that schema with the dump
docker stop <PROJECT>-app-1
docker run --rm --network <PROJECT>_<PROJECT>_network \
  -v <APP_DATA_DIR>:/cfg:ro -v /root/endurain-pg18:/dump postgres:18 \
  sh -c 'export PGPASSWORD=$(grep -m1 "^DB_PASSWORD=" /cfg/app.env | cut -d= -f2-); \
         pg_restore -h <PROJECT>-postgres-1 -U endurain -d postgres \
                    --clean --create --no-owner /dump/endurain.dump'
docker start <PROJECT>-app-1
```

To roll back at any point, set the image back to `docker.io/postgres:17.5`
through a user-config override, move `postgres-pg17` back to
`<APP_DATA_DIR>/data/postgres`, and restart the app. Keep that directory until
you are satisfied the upgrade took.

Staying on 17.5 is also a valid choice: Postgres 17 is supported until
November 2029 and Endurain does not require 18. Pin it with a user-config
override if you would rather not migrate.

## Sponsors

A huge thank you to the project sponsors! Your support helps keep this project going.

Consider [sponsoring Endurain on GitHub](https://github.com/sponsors/joaovitoriasilva) to ensure continuous development.

## Contributing

Contributions are welcomed! Please open an issue to discuss any changes or improvements before submitting a PR. Check
out the [Contributing Guidelines](CONTRIBUTING.md) for more details.

## Help Translate

Endurain has multi-language support, and you can help translate it into more languages
via [Crowdin](https://crowdin.com/project/endurain).

## License

This project is licensed under the AGPL-3.0 License - see the [LICENSE](LICENSE) file for details.