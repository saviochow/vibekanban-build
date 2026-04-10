# vibekanban-build

Builds and publishes [Vibe Kanban](https://github.com/BloopAI/vibe-kanban) Docker images to GHCR, ready to deploy as a Portainer stack.

## Why this exists

Vibe Kanban's frontend has one build-time argument (`VITE_RELAY_API_BASE_URL`) that gets compiled into the JS bundle. This means images are deployment-specific — there's no generic image you can pull. This repo automates the build for your own domain.

## Fork it

Click **Fork** at the top right, then:

1. **Set your relay URL** — either:
   - **Option A (simple)**: edit `build.env` and change `VITE_RELAY_API_BASE_URL`, commit and push
   - **Option B (no commit)**: go to your fork's Settings → Secrets and variables → Actions → Variables → add `VITE_RELAY_API_BASE_URL`. This overrides `build.env` without modifying the file.

2. **Wait for the first build** — triggered automatically on push, or manually from Actions tab → "Build and publish images" → Run workflow.

3. **Make the images public** — after the first successful build:
   - Go to `https://github.com/users/<your-username>/packages`
   - For each package (`vibekanban-app`, `vibekanban-relay`): Package settings → Change visibility → Public
   - Otherwise Portainer needs auth to pull.

4. **Deploy with Portainer** — edit `portainer-stack.yml` to point at your fork's images:
   ```yaml
   image: ghcr.io/<your-username>/vibekanban-app:latest
   image: ghcr.io/<your-username>/vibekanban-relay:latest
   ```
   Then paste it into Portainer's stack editor and set the env vars listed below.

## What the workflow does

- **Hourly cron** polls upstream [BloopAI/vibe-kanban](https://github.com/BloopAI/vibe-kanban) for new commits
- **Skips the build** if an image tagged with the current upstream SHA already exists
- **Rebuilds** when upstream changes, on push to `main`, or on manual trigger
- **Tags** each build with both `:latest` and `:<sha>`

## Build-time variables

| Variable | Required | What it does |
|----------|----------|-------------|
| `VITE_RELAY_API_BASE_URL` | **Yes** | Compiled into the frontend JS bundle. Must match your relay URL. |
| `SENTRY_DSN_REMOTE` | No | Enables Sentry error tracking |
| `POSTHOG_API_KEY` | No | Enables PostHog analytics |
| `POSTHOG_API_ENDPOINT` | No | PostHog endpoint URL |

Set these in `build.env` (committed) or as repo variables (runtime override).

## Portainer deployment — runtime env vars

These go into Portainer's "Environment variables" section when creating the stack:

```
# Secrets — generate with `openssl rand -base64 48` / `32`
VIBEKANBAN_REMOTE_JWT_SECRET=
ELECTRIC_ROLE_PASSWORD=
DB_PASSWORD=

# URLs — must match your VITE_RELAY_API_BASE_URL build-time value
APP_URL=https://vk.example.com
RELAY_URL=https://vk-relay.example.com

# GitHub OAuth (login)
GITHUB_OAUTH_CLIENT_ID=
GITHUB_OAUTH_CLIENT_SECRET=

# GitHub App (optional — enables issues/webhooks/PR integration)
GITHUB_APP_ID=
GITHUB_APP_PRIVATE_KEY=
GITHUB_APP_WEBHOOK_SECRET=
GITHUB_APP_SLUG=
```

## Reverse proxy

The stack exposes the app on `:8081` and the relay on `:8082` on your Docker host. Point your reverse proxy (HAProxy, Caddy, Traefik, nginx, etc.) at these:

- `<APP_URL host>` → `http://<docker-host>:8081`
- `<RELAY_URL host>` → `http://<docker-host>:8082` (must forward WebSocket upgrades)

## Related

- [vibekanban-deploy](https://github.com/gabrielbelli/vibekanban-deploy) — public deployment kit with scripts, docs, and architecture diagrams
- [vibe-kanban](https://github.com/BloopAI/vibe-kanban) — upstream source

## Licence

BSD 2-Clause. See [LICENSE](LICENSE).

This repo contains only build automation. It does not redistribute Vibe Kanban source or binaries — the workflow clones vibekanban at build time directly from [BloopAI/vibe-kanban](https://github.com/BloopAI/vibe-kanban), which is licensed under Apache 2.0.
