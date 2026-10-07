<div align="center">
  <h1>Zerobyte</h1>
  <h3>Powerful backup automation for your remote storage<br />Encrypt, compress, and protect your data with ease</h3>
  <a href="https://github.com/nicotsx/zerobyte/blob/main/LICENSE">
    <img src="https://img.shields.io/github/license/nicotsx/zerobyte" />
  </a>
  <br />
  <figure>
    <img src="https://github.com/nicotsx/zerobyte/blob/main/screenshots/backup-details.webp?raw=true" alt="Demo" />
    <figcaption>
      <p align="center">
        Backup management with scheduling and monitoring
      </p>
    </figcaption>
  </figure>
</div>

## Intro

Zerobyte is a backup automation tool that helps you save your data across multiple storage backends. Built on top of
Restic, it provides an modern web interface to schedule, manage, and monitor encrypted backups of your remote storage.

### Features

- &nbsp; **Automated backups** with encryption, compression and retention policies powered by Restic
- &nbsp; **Flexible scheduling** For automated backup jobs with fine-grained retention policies
- &nbsp; **End-to-end encryption** ensuring your data is always protected
- &nbsp; **Multi-protocol support**: Backup from NFS, SMB, WebDAV, SFTP, or local directories

## Configuration notes

- **Allowed webhook origins** — Zerobyte only calls HTTP endpoints whose origin is on this list. Gotify, generic webhooks, self-hosted ntfy servers and backup pre/post webhooks all need their origin here (scheme, host and port, for example `https://gotify.example.com`). Slack, Discord, Pushover, Telegram and the public ntfy.sh do not. Leave empty if you use none of these.
- **What Zerobyte can see** — the whole Runtipi folder is mounted read-write at `/runtipi` inside the container, so `/runtipi/app-data`, `/runtipi/media` and `/runtipi/backups` can be added as directory volumes. Anything outside it (other disks, an rclone config, an SSH key for SFTP repositories) needs an extra mount through a user-config override.

## Data layout

- `${APP_DATA_DIR}/var/lib/zerobyte/data` — the configuration database and `restic.pass`, the encryption password for every repository. **Without `restic.pass` no backup can be read back; keep a copy of it somewhere else.**