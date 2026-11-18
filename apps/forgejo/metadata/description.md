# Forgejo

A lightweight, self-hosted software forge. Easy to install, low maintenance, 100% free software.

## Core features

- Git hosting over HTTPS and SSH, with mirrors of repositories hosted elsewhere
- Pull requests with code review, issues, labels, milestones and project boards
- Wiki, releases and a package registry (container, npm, PyPI, Maven and more)
- Forgejo Actions: GitHub-Actions-compatible CI (needs a separately installed runner)
- Organisations, teams, and fine-grained access control

## Configuration notes

> [!IMPORTANT]
> **Finish the install wizard straight after installing.** On first visit Forgejo shows an
> install page; the database settings are already filled in. Create the administrator account
> in the *Administrator account settings* section at the bottom of that page. Until the wizard
> is completed, anyone who can reach the app can complete it and become the admin.

- **Registration is disabled** (`FORGEJO__service__DISABLE_REGISTRATION=true`). Add users from
  *Site administration → User accounts*. Set a user-config override to change it.
- **SSH port**: Git over SSH is published on the host port you choose (default `2222`), and
  clone URLs show `ssh://git@<your-domain>:<port>/…`. A Cloudflare Tunnel only carries the web
  UI, so SSH clones need a route that reaches that port directly, such as Tailscale, plus a
  firewall rule allowing it.
- **Image source**: images are pulled from `data.forgejo.org`, Forgejo's own registry, not
  from `codeberg.org`. That keeps installs and updates working if Codeberg is unreachable.
- **Database**: PostgreSQL 18. Data lives under `app-data/…/forgejo/data/`.
