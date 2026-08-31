# Donetick

An open-source task and chore manager for yourself, your household or a group of friends. Self-hosted, no account on someone else's server.

## Core features

- **Natural language task creation** — type "take the trash out every Monday and Tuesday at 6:15 pm" and Donetick works out the due date and the recurrence
- **Advanced scheduling** — daily, weekly, monthly, yearly, specific days or months, or adaptive scheduling that learns from past completions; recurrence can run from the previous due date or from the actual completion date
- **Shared with others** — create a circle, then assign or share tasks, with automatic assignee rotation by turns, at random, or to whoever has done the least
- **Subtasks, labels and priorities** — nestable subtasks that reset when a recurring task completes, shared labels across the group, and five priority levels
- **Things** — track a number, a flag or a piece of text that is not a task, and mark tasks done automatically when it changes to a given value
- **NFC tags** — write a tag and stick it on the thing, scanning it completes the task
- **Points and analytics** — a points system for completions plus breakdowns by label and status
- **Notifications** — Telegram, Discord, Pushover, the mobile app, or your own endpoint through the webhook system
- **Auth** — local accounts with TOTP MFA, or any OIDC provider (Authentik, Authelia, Keycloak)
- **Integrations** — a REST API, an external API for long-lived tokens, and an official Home Assistant integration that creates a to-do list per user

## Configuration notes

- **JWT secret** — generated for you on install. Donetick refuses to start if it is shorter than 32 characters or a known weak value, and changing it later invalidates every session, so everyone has to sign in again.
- **Public URL** — optional, and only needed for links that Donetick generates itself: password reset emails and the OAuth2 redirect. The app works over LAN without it.
- **Disable new signups** — a fresh instance has open registration. Create your accounts first, then turn this on and restart so nobody else can register.
- **Email, OIDC, S3 photo storage** — not exposed as fields here, but every setting in the app's [`selfhosted.yaml`](https://github.com/donetick/donetick/blob/main/config/selfhosted.yaml) has a matching `DT_`-prefixed environment variable (for example `oauth2.client_id` becomes `DT_OAUTH2_CLIENT_ID`), which you can add to the app's env file in Runtipi.

## Data layout

- `${APP_DATA_DIR}/data/donetick.db` — the SQLite database holding users, circles, tasks, history and points. **Back this file up and you have backed up everything.**

Source: <https://github.com/donetick/donetick> · Docker image: `donetick/donetick`
