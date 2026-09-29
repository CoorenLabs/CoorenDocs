---
description: Install Cooren, run it locally and make your first requests.
icon: bolt
---

# Quickstart

This page gets the API running on your machine in a few minutes. To run it in production, see [Deployment](deployment.md).

### Requirements

* [Git](https://git-scm.com)
* [Bun](https://bun.sh/docs/installation) 1.4 or newer
* Google Chrome or Chromium, for the providers that pass Cloudflare challenges in a real browser (AnimePahe, PrimeSrc, All Manga chapter pages, VidCore, VidFast and parts of Anivexa)

{% hint style="info" %}
Prefer containers? `docker compose up -d --build` starts the API with Chromium and Redis already set up. See [Deployment](deployment.md#docker-compose).
{% endhint %}

### Run the API

{% stepper %}
{% step %}
#### Clone the repository

```bash
git clone https://github.com/CoorenLabs/CoorenLabs.git
cd CoorenLabs
```
{% endstep %}

{% step %}
#### Install dependencies

```bash
bun install
```
{% endstep %}

{% step %}
#### Create your environment file

```bash
cp .env.example .env
```

The defaults work for local development. See [Configuration](configuration.md) for every variable.
{% endstep %}

{% step %}
#### Start the server

```bash
bun run dev
```

`dev` restarts the server when you change a file. Use `bun run start` to run it without watching.
{% endstep %}
{% endstepper %}

### Check that it works

The root route returns the API status:

```bash
curl "http://localhost:3000/"
```

```json
{
  "name": "Cooren API",
  "version": "3.0.0",
  "repo": "https://github.com/CoorenLabs/CoorenLabs.git",
  "environment": "development",
  "about": "Cooren is a high-performance, scalable scraping engine designed to collect, organize, and deliver structured data from across the world of anime, movies, manga, and music, all in one unified ecosystem",
  "status": "operational"
}
```

Then open [http://localhost:3000/docs](http://localhost:3000/docs) for the interactive OpenAPI reference. The raw spec is at `/docs/json`.

### Make a first request

Search AniList through the meta provider:

```bash
curl "http://localhost:3000/meta/anilist/search/frieren"
```

Each category also has an overview route that lists its providers and their endpoints:

```bash
curl "http://localhost:3000/anime/"
```

```json
{
  "service": "anime",
  "description": "Unified anime API — provider-isolated route architecture",
  "providers": ["animepahe", "toonstream", "animesaturn", "animeunity", "animelok", "miruro", "anivexa"],
  "endpoints": {
    "animepahe": [
      "GET /anime/animepahe/search/:query         → Search titles"
    ]
  }
}
```

### Route groups

| Prefix | Purpose |
| --- | --- |
| `/anime` | Anime providers |
| `/manga` | Manga providers |
| `/meta` | Metadata and discovery (AniList) |
| `/movie-tv` | Movie and TV providers |
| `/stream` | Direct stream extractors for movies and TV |
| `/music` | Music providers |
| `/proxy` | [Stream proxy](../core/proxy.md) for HLS, MP4 and files |
| `/mappings` | [ID mappings](../core/mappings.md) between anime databases |
| `/docs` | OpenAPI reference |

### Project layout

| Path | Contents |
| --- | --- |
| `src/index.ts` | Starts the server |
| `src/app.ts` | Builds the Elysia app: CORS, OpenAPI docs and every route group |
| `src/core` | Configuration, logging, Redis cache, the stream proxy, ID mappings, and the Cloudflare, browser and TLS helpers in `src/core/lib` |
| `src/providers/<category>/<provider>` | One folder per provider with its routes, scraper and types |
| `src/providers/origins.ts` | The base URL of every upstream site |

### Useful scripts

| Command | What it does |
| --- | --- |
| `bun run dev` | Start the server and restart on file changes |
| `bun run start` | Start the server |
| `bun run typecheck` | Type-check the project |
| `bun run lint` | Lint the project |
