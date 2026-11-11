# NVIDIA Vibe Coder

Telegram AI coding assistant powered by NVIDIA NIM and Cloudflare Workers.

## Features

- Telegram webhook with a path secret
- NVIDIA NIM OpenAI-compatible Chat Completions API
- Per-user conversation history and model selection in Cloudflare KV
- Repository metadata lookup through GitHub API
- GitHub Actions deployment to Cloudflare Workers
- Health endpoint and webhook setup endpoint

## Requirements

- Node.js 22+
- Cloudflare account and API token
- Telegram bot token from [@BotFather](https://t.me/BotFather)
- NVIDIA API key from [NVIDIA Build](https://build.nvidia.com/)
- Cloudflare KV namespace

## 1. Create KV namespace

```bash
npx wrangler kv namespace create SESSIONS
```

Copy the returned ID into `wrangler.toml`, replacing `REPLACE_WITH_KV_NAMESPACE_ID`.

## 2. Configure secrets

```bash
npx wrangler secret put TELEGRAM_BOT_TOKEN
npx wrangler secret put NVIDIA_API_KEY
npx wrangler secret put WEBHOOK_SECRET
npx wrangler secret put GITHUB_TOKEN
```

Use a long random value for `WEBHOOK_SECRET`. Optional `GITHUB_TOKEN` enables the `/repo owner/name` command. Set `ADMIN_IDS` only if you add admin-only commands in a future release.

For local development, copy `.dev.vars.example` to `.dev.vars` and fill in values. Never commit real secrets.

## 3. Deploy

```bash
npm install
npm run typecheck
npx wrangler deploy
```

## 4. Set Telegram webhook

After deployment, call the protected setup endpoint:

```bash
curl -X POST "https://YOUR-WORKER.YOUR-SUBDOMAIN.workers.dev/admin/set-webhook" \
  -H "Authorization: Bearer YOUR_WEBHOOK_SECRET"
```

The endpoint registers the webhook URL with Telegram. The secret is also required in the webhook URL path.

## 5. GitHub Actions

Add these repository Actions secrets:

- `CLOUDFLARE_API_TOKEN` — token with permission to edit Workers and access KV resources
- `CLOUDFLARE_ACCOUNT_ID` — Cloudflare account ID

Push to `main` or run **Deploy Worker** manually.

## Commands

- `/start`, `/help` — help
- `/model` — current model
- `/setmodel <model-id>` — select NVIDIA model
- `/reset` — clear chat history
- `/repo <owner/name>` — inspect a public or token-accessible GitHub repository

Send any normal text to chat with the coding assistant.

## Endpoints

- `GET /` — service information
- `GET /health` — health check
- `POST /admin/set-webhook` — register Telegram webhook (Bearer secret)
- `POST /telegram/webhook/<WEBHOOK_SECRET>` — Telegram updates

## Security notes

- Keep all API tokens in Worker Secrets or GitHub Actions Secrets.
- Do not expose the NVIDIA or GitHub token to Telegram users.
- Restrict bot usage at the network/application layer before deploying it to a public audience if it is intended to be private.
- This initial version reads repository metadata only. It does not modify files, create commits, or open pull requests.
