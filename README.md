# vibekanban-build

Personal image builder for self-hosted Vibe Kanban. Publishes Docker images to GHCR with hardcoded build-time settings, ready to deploy as a Portainer stack.

## What this does

- GitHub Actions clones [vibekanban](https://github.com/BloopAI/vibe-kanban) on every push + weekly
- Builds `app` and `relay` images with personal build args baked in
- Pushes to `ghcr.io/gabrielbelli/vibekanban-app` and `ghcr.io/gabrielbelli/vibekanban-relay`
- `portainer-stack.yml` is ready to paste into Portainer

## Build-time settings

Edit `build.env` to change baked-in values. Changes trigger a rebuild.

Currently set:
- `VITE_RELAY_API_BASE_URL=https://vk-relay.gabrielbelli.com` (hardcoded into frontend bundle)

## Triggering a build

- **Automatic**: push to `main`, or weekly cron (Mondays at 04:00 UTC)
- **Manual**: Actions tab → "Build and publish images" → Run workflow (optionally pin a vibekanban ref)

## Deploying

1. Wait for the build workflow to finish (or trigger manually)
2. In Portainer: Stacks → Add stack → paste `portainer-stack.yml`
3. Set the env vars listed below
4. Deploy

## Required env vars in Portainer

```
# Secrets — generate with `openssl rand -base64 48` / `32`
VIBEKANBAN_REMOTE_JWT_SECRET=
ELECTRIC_ROLE_PASSWORD=
DB_PASSWORD=

# URLs
APP_URL=https://vk.gabrielbelli.com
RELAY_URL=https://vk-relay.gabrielbelli.com

# GitHub OAuth (login)
GITHUB_OAUTH_CLIENT_ID=
GITHUB_OAUTH_CLIENT_SECRET=

# GitHub App (issues/webhooks/PR integration)
GITHUB_APP_ID=
GITHUB_APP_PRIVATE_KEY=
GITHUB_APP_WEBHOOK_SECRET=
GITHUB_APP_SLUG=
```

## HAProxy

- `vk.gabrielbelli.com` → `http://<docker-host>:8081`
- `vk-relay.gabrielbelli.com` → `http://<docker-host>:8082` (must forward WebSocket upgrades)

## Related repos

- [vibekanban-deploy](https://github.com/gabrielbelli/vibekanban-deploy) — public deployment kit with docs and scripts
- [vibe-kanban](https://github.com/BloopAI/vibe-kanban) — upstream source
