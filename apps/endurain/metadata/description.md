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