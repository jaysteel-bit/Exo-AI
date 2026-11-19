# Exo AI: fork notes

Exo AI is Exo Enterprise's agent orchestrator for Flow OS, forked from
[n8n-io/nod](https://github.com/n8n-io/nod) (Apache-2.0). Keep `LICENSE` and
`NOTICE` as they are; Apache-2.0 requires both.

- **Repo:** `jaysteel-bit/Exo-AI`, branch `exo-main` (default). Remote `upstream` = n8n-io/nod.
- **Role:** the agent orchestrator layer for all of Flow OS. It reaches Flow OS through the Flow API
  (`flow-api.exoflowos.com`) and the `flow` CLI / MCP tools in the flow-os repo (`cli/`).
- **Surface in Flow OS:** the first tab on Routines (`| Exo AI | My Flows | Exa Gallery |`), as a native webview
  pane. Nod refuses framing (`frame-ancestors 'none'`) and signs in with a cookie, so it cannot be an iframe.

## What this fork changes

| File                                  | Change                                                             |
| ------------------------------------- | ------------------------------------------------------------------ |
| `.github/workflows/exo-ai-images.yml` | Builds server/browser/workspace images on GitHub runners → GHCR    |
| `deployment/exo/compose.exo.yaml`     | Points the upstream self-host stack at those images; loopback only |
| `REBRAND.md`                          | This file                                                          |

The visual rebrand (name, logo, favicon, title) is not done yet. When it is, list each file here and keep upstream
component names so merges stay clean.

Upstream's own workflows are still in `.github/workflows/`. Its nightly schedules (`ci.yml`, `evals.yml`) will run on
this repo and may fail without n8n's secrets. Disable them in the repo's Actions settings if they get noisy.

## Deployment (Contabo)

Target: `/opt/exo-ai` on Contabo (`vmi3405304`), loopback `127.0.0.1:4310`, public host `ai.exoflowos.com` through the
`exo-contabo` Cloudflare tunnel with Cloudflare Access in front. Exo AI keeps its own token sign-in behind Access.

**Prerequisite: the Contabo RAM upgrade.** Before it, the box had 7.8 GB total and ~2.7 GB available, with no swap,
next to Exo Base, Team chat, HeyForm, Hermes and the software factory. The stack's default caps are server 2 GB,
browser 3 GB, workspace 2 GB (`NOD_SERVER_MEMORY`, `NOD_BROWSER_MEMORY`, `NOD_WORKSPACE_MEMORY` in `.env`).

```sh
git clone https://github.com/jaysteel-bit/Exo-AI.git /opt/exo-ai   # compose files only; never build here
cd /opt/exo-ai/deployment/self-host
COMPOSE_PROJECT_NAME=exo-ai sh setup.sh                                  # writes .env with token, vault key, db password
# in .env: NOD_PUBLIC_URL=https://ai.exoflowos.com
docker compose -f compose.yaml -f ../exo/compose.exo.yaml --env-file .env pull
docker compose -f compose.yaml -f ../exo/compose.exo.yaml --env-file .env up -d --no-build --wait
```

Back up `.env` (it holds `NOD_VAULT_KEY`; saved credentials can't be read without it) to the workspace key store.

## Local development (Windows)

`pnpm start` works on Windows with Node ≥ 24.21 and Docker Desktop, plus `NOD_VAULT_KEY` set in `.env` (Windows has
no keychain fallback). Portable Node lives in `.tools/node` (git-excluded).
