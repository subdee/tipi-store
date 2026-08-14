# Aeterna

*"What words would you leave behind?"*

Aeterna is a lightweight, self-hosted dead man's switch. You write messages for the people you care about, and you check in regularly. If you stop checking in, Aeterna delivers those messages by email.

## Core features

- **Email delivery** — messages and attachments are sent automatically when you miss your check-ins
- **Heartbeat check-ins** — confirm you're around from the web UI or with a one-click link in the reminder email
- **File attachments** — documents, photos or instructions travel with the message, and are deleted from the server right after delivery
- **Webhooks** — trigger home automation or custom scripts when a switch fires
- **Encrypted at rest** — messages and attachments are stored AES-256-GCM encrypted and only decrypted at delivery time

## Configuration notes

- **Public URL** — set this to the exact URL you use to reach Aeterna, without a trailing slash. It's used for the check-in links inside the reminder emails and as the allowed CORS origin, so a wrong value means the links in your emails point nowhere. Update it (and restart the app) if you later put Aeterna behind a domain.
- **SMTP** — email is not configured through Runtipi. After the first start, open Aeterna's **Settings** page and enter your SMTP details there, then send a test email. Without working SMTP, no check-in reminders and no deliveries go out.
- **Master password** — set on first launch in the setup screen shown by the app; keep the recovery key it gives you.
- **Encryption key** — a 32-byte AES key is generated on first start and stored in `${APP_DATA_DIR}/data/secrets/encryption_key`. Back it up together with the database: without that exact file, existing messages and attachments cannot be decrypted.

## Data layout

- `${APP_DATA_DIR}/data/app` — SQLite database and uploaded attachments
- `${APP_DATA_DIR}/data/secrets` — the encryption key

Source: <https://github.com/alpyxn/aeterna> · Docker images: `ghcr.io/alpyxn/aeterna-frontend`, `ghcr.io/alpyxn/aeterna-backend`
