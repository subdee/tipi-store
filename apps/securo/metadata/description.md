# Securo

An open-source personal finance manager that runs on your own machine. Accounts, transactions, budgets, goals and assets in one place, with nothing sent to a third party unless you switch a provider on yourself.

## Core features

- **Accounts and transactions** — multi-account tracking with running balances, search, filters and CSV export
- **Import** — OFX, QIF, CAMT and CSV files, with an auto-categorisation rules engine
- **Planning** — budgets, recurring transactions, goals and savings targets with progress tracking
- **Assets** — valuation tracking and growth rules, with live quotes for tickers you add
- **Reports** — net worth and income vs expenses, with per-category sparklines
- **Multi-currency** — automatic FX conversion when an exchange-rate key is configured
- **Multi-user** — workspaces with owner/editor/viewer roles, an admin panel and registration controls
- **Auth** — passwords with TOTP two-factor and brute-force protection, passkeys, and OIDC login for Authentik, Pocket ID and friends
- **AI agents (optional)** — self-hosted LLM chat with tool access over your own data, plus a per-agent knowledge base

## Configuration notes

- **Public registration is off.** The first account is created through the setup screen the first time you open the app, which works regardless of this setting. Open the app and finish setup straight away — until a first user exists, whoever reaches the instance can claim the admin account. To let other people sign up afterwards, turn registration on under **Admin → Settings**.
- **Secrets are generated for you.** The app secret, the database password and the agents MCP secret are all random per install. Upstream ships placeholder defaults for these; changing the app secret later makes stored provider credentials unreadable, so leave it alone once you are running.
- **Expose it with a domain if you want passkeys.** WebAuthn refuses plain HTTP and refuses IP addresses, both by standard rather than by choice of this app. On `http://<host-ip>:8088` passwords and TOTP work but passkeys do not.
- **Trusted proxy hops** — leave at 1 when you reach Securo on the host IP and port. Set it to 2 when you expose it through Runtipi with a domain, so login rate limiting keys on the real client IP instead of lumping every visitor into one bucket.
- **Bank sync is opt-in and provider-based.** SimpleFIN (US and international) needs only the toggle — each connection carries its own setup token from the SimpleFIN Bridge. Enable Banking (~2500 European PSD2 banks) needs an application ID plus its PEM key at `<app data>/secrets/enable_banking_private.pem`. Pluggy (Brazilian banks) is supported upstream but not wired up here.
- **Exchange rates** — a free Open Exchange Rates app ID enables real FX conversion. Without one, cross-currency amounts fall back to 1:1 with a visible warning.
- **AI agents** — off by default. Turning the toggle on activates the feature; the MCP container that gives the agents access to your data runs either way. Add a provider connection under **Settings → AI Agents**: a model server on the Runtipi host is reachable as `http://host.docker.internal:<port>`.
- **Tesouro Direto is disabled** here. It is a Brazilian Treasury bond lookup that polls a Brazilian government CSV endpoint; upstream defaults it on.

## What talks to the internet

The app is private by default in the sense that your financial data stays in your database, but a few things do leave the machine. None of them is analytics — there is no telemetry SDK anywhere in the codebase.

- **Google Fonts** — the UI loads its typeface from `fonts.googleapis.com` on every page load, so Google sees your browser's IP and that you opened Securo.
- **Google's favicon service** — bank and asset logos are `https://www.google.com/s2/favicons?domain=…` URLs your browser fetches. For SimpleFIN-linked banks and for market-tracked assets, that tells Google which institutions and companies appear in your portfolio. Nothing else about the holding is sent, and the feature degrades to a generic icon if the request is blocked.
- **Update check** — the browser asks `api.github.com` for the latest Securo release every six hours. Turn it off in the UI if you would rather it did not.
- **Yahoo Finance** — the backend looks up quotes for tickers you add as assets, which discloses those symbols. Only runs if you track market-priced assets.
- **Your bank-sync provider** — whichever of SimpleFIN or Enable Banking you configure, and only then.

## Data layout

- `${APP_DATA_DIR}/data/postgres` — the database, and the thing to back up
- `${APP_DATA_DIR}/data/attachments` — receipts and invoice documents
- `${APP_DATA_DIR}/data/agent_knowledge` and `data/embedding_models` — agents knowledge base and its local embedding model
- `${APP_DATA_DIR}/secrets` — put the Enable Banking PEM key here; mounted read-only

Redis is a Celery broker only and holds nothing you need to keep.

## Licensing

Securo is licensed under AGPL-3.0.

Source: <https://github.com/securo-finance/securo> · Docker images: `ghcr.io/securo-finance/securo-frontend`, `ghcr.io/securo-finance/securo-backend`
