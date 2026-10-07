# Databasus

A self-hosted backup tool for databases. Add a database, pick a schedule and a storage destination, and Databasus takes care of dumping, encrypting, uploading, rotating and restoring.

## Core features

- **Databases** — PostgreSQL 14 to 18 (logical dumps and physical, point-in-time backups), MySQL, MariaDB and MongoDB (logical dumps)
- **Storage destinations** — local disk inside the app's data folder, S3 and S3-compatible (Cloudflare R2, MinIO, Backblaze), Google Drive, Dropbox, SFTP, FTP, NAS shares, Azure Blob and any rclone remote
- **Scheduling** — hourly, daily, weekly, monthly or a cron expression, with a configurable backup window
- **Retention** — keep by age, by count, or Grandfather-Father-Son, with per-backup and total size caps
- **Encryption** — every backup is encrypted before it leaves the server, so shared or cloud storage never sees plain data
- **Restore** — download a dump or restore it straight into a target database from the web interface
- **Health checks and verification** — periodic connectivity checks on each database, and optional test restores that prove a backup is usable
- **Notifications** — email, Telegram, Slack, Discord, Mattermost and generic webhooks, on success and on failure
- **Workspaces and users** — group databases, storages and notifiers per project; the first account created administers the instance

## Reaching your databases

Databasus runs on Runtipi's shared `tipi_main_network`. A database that belongs to another Runtipi app lives on that app's private network, so Databasus cannot reach it until the two share a network. The usual pattern:

1. Create a network once on the host: `docker network create backup-net`
2. For each app whose database you want to back up, add a user-config override (`user-config/<store>/<app>/docker-compose.yml`) that attaches the database service to `backup-net` as an external network, then restart the app.
3. Add the same override for Databasus itself.
4. In Databasus, use the database's **container name** (for example `immich_migrated-immich-db-1`) as the host, not the short service name. Short names such as `db` are shared by several apps and resolve to the wrong container.

Because `backup-net` is external, stopping or updating any app leaves it in place.

## Data layout

- `${APP_DATA_DIR}/data` — Databasus's own PostgreSQL (settings, users, schedules, history), its encryption key `secret.key`, and `backups/` when local storage is used. **Back up this folder; without `secret.key` existing backups cannot be decrypted.**

Source: <https://github.com/databasus/databasus> · Docs: <https://databasus.com> · Docker image: `databasus/databasus`
