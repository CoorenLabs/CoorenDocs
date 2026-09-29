---
description: Run Cooren in production with Docker Compose, plain Bun or Vercel.
icon: rocket
---

# Deployment

Docker Compose is the recommended way to run Cooren in production. It bundles everything the providers need: Bun, Chromium for Cloudflare challenges, and Redis for caching.

| Option | Browser providers | Redis cache | Best for |
| --- | --- | --- | --- |
| [Docker Compose](#docker-compose) | Yes | Included | Servers and VPS hosting |
| [Bun](#bun) | Yes, if Chrome is installed | Optional | Existing servers you manage |
| [Vercel](#vercel) | No | Optional | Quick serverless demos |

### Docker Compose

The repository ships a `docker-compose.yml` with two services:

* `api` builds the Cooren image and listens on port 3000.
* `redis` runs Redis 8 as an in-memory cache limited to 256 MB, with persistence turned off.

{% stepper %}
{% step %}
#### Set your public URL

In `.env` next to `docker-compose.yml`:

```bash
SERVER_ORIGIN=https://api.example.com
```

Compose also reads `PORT`, `LOG_LEVEL`, `CORS_ORIGIN` and `CORS_CREDENTIALS` from `.env` or your shell. `NODE_ENV` is always `production` and `REDIS_URL` always points at the bundled Redis.
{% endstep %}

{% step %}
#### Build and start

```bash
docker compose up -d --build
```
{% endstep %}

{% step %}
#### Check it

```bash
curl "https://api.example.com/"
docker compose logs -f api
```

The log shows `[Cache] Enabled (Redis)` once the API is connected to Redis.
{% endstep %}
{% endstepper %}

The compose file also sets two options the browser providers need:

* `shm_size: 1gb`, because Chromium crashes with Docker's default 64 MB of shared memory.
* `init: true`, so leftover browser processes are cleaned up.

### The Docker image

The `Dockerfile` builds on `oven/bun:1.4`, installs Chromium, Xvfb and fonts, sets `CHROME_PATH=/usr/bin/chromium`, and installs production dependencies from the lockfile. It runs `bun src/index.ts` on port 3000.

To run the image without Compose:

```bash
docker build -t cooren .
docker run -d -p 3000:3000 --shm-size=1g --init \
  -e SERVER_ORIGIN=https://api.example.com \
  -e REDIS_URL=redis://your-redis:6379 \
  cooren
```

### Bun

On a server with Bun 1.4 and Chrome installed:

```bash
bun install --production
NODE_ENV=production SERVER_ORIGIN=https://api.example.com bun run start
```

Run it under a process manager such as systemd or pm2, and put a reverse proxy in front of it for HTTPS. On Linux without a display, install Xvfb so the browser providers can start Chrome.

### Vercel

The repository includes a `vercel.json` that runs the app on Vercel's Bun runtime:

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "bunVersion": "1.4.x"
}
```

Vercel detects Elysia automatically and serves the app exported from `src/app.ts`. Set `SERVER_ORIGIN` to your deployment URL in the project's environment variables, and `REDIS_URL` if you have a Redis instance.

{% hint style="warning" %}
Vercel functions have no Chrome, so providers that solve Cloudflare challenges in a browser do not work there: AnimePahe, PrimeSrc, VidCore, VidFast, All Manga chapter pages and parts of Anivexa. Use Docker for full coverage.
{% endhint %}

### Production checklist

* Set `SERVER_ORIGIN` to the public URL; the server will not start in production without it.
* Enable Redis. It cuts latency and keeps you under upstream rate limits.
* Restrict `CORS_ORIGIN` to your own front-end origins if the API is not meant to be public.
* Expect the first request to a Cloudflare-protected provider after a restart to take 8 to 15 seconds while the browser solves the challenge. Later requests reuse the clearance.
