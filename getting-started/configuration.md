---
description: Every environment variable Cooren reads, with defaults.
icon: sliders
---

# Configuration

Cooren is configured with environment variables. Copy `.env.example` to `.env` to start; every variable is optional in development.

### Variables

| Variable | Default | Description |
| --- | --- | --- |
| `PORT` | `3000` | Port the server listens on. |
| `NODE_ENV` | `development` | `development` or `production`. Shown on the `/` route. |
| `SERVER_ORIGIN` | `http://localhost:PORT` | Public URL of this API. Every proxied stream, subtitle and image link in a response is built from it. Required when `NODE_ENV` is `production`. |
| `LOG_LEVEL` | `info` | `debug`, `info`, `warn`, `error` or `silent`. |
| `CORS_ORIGIN` | `*` | `*` allows every origin. Otherwise a comma-separated list of allowed origins. |
| `CORS_CREDENTIALS` | `false` | Set to `true` to allow cookies on cross-origin requests. |
| `REDIS_URL` | not set | Redis connection URL, for example `redis://localhost:6379`. Turns caching on. |

### Example `.env`

```bash
PORT=3000
NODE_ENV=development
LOG_LEVEL=info
SERVER_ORIGIN=http://localhost:3000
CORS_ORIGIN=*
CORS_CREDENTIALS=false
REDIS_URL=
```

### SERVER\_ORIGIN

Stream links in responses point back at your own server, for example `http://localhost:3000/proxy/m3u8-proxy?url=...`. Set `SERVER_ORIGIN` to the exact URL clients use to reach the API, including the scheme and any port, and without a trailing slash.

{% hint style="warning" %}
If `SERVER_ORIGIN` is wrong, search and details still work but every `proxiedUrl` and image link points at the wrong host.
{% endhint %}

In production the server refuses to start without it:

```
Error: SERVER_ORIGIN must be set in production
```

### Caching

Without `REDIS_URL` every request goes to the upstream site. With it, provider results are cached for minutes to days depending on how often the data changes, for example 10 minutes for MangaBall details and 24 hours for ID mappings. Caching makes repeat requests fast and keeps you under upstream rate limits, such as AniList's.

* Caching needs the Bun runtime.
* If Redis is unreachable at startup, the server logs a warning and runs without a cache.
* Cloudflare clearance cookies are cached too, so a browser solve is shared between requests.

### Browser providers

Some providers solve Cloudflare challenges in a real Chrome window, controlled by `puppeteer-real-browser`. Chrome is found automatically. Set `CHROME_PATH` to use a specific Chrome or Chromium binary; the Docker image sets it to `/usr/bin/chromium`. On Linux servers without a display, the browser runs inside Xvfb, which the Docker image installs.

### CORS

* `CORS_ORIGIN=*` reflects any origin.
* `CORS_ORIGIN=https://app.example.com,https://admin.example.com` allows only those origins.
* `CORS_CREDENTIALS=true` adds `Access-Control-Allow-Credentials: true`. Use it with an explicit origin list, not `*`.
