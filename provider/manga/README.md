---
description: Manga, manhwa and manhua providers for discovery, search, chapter lists and page images.
icon: book-open
---

# Manga

Manga providers return titles, chapter lists and page image URLs from five upstream sites. Each provider has its own ids, so take the ids for detail and read routes from that provider's search or list routes.

{% hint style="info" %}
Base route: `/manga`
{% endhint %}

## Providers

| Provider | Good for | Page |
| --- | --- | --- |
| All Manga | Popular lists by period, genre and author browsing, a large catalog with scores | [All Manga](all-manga.md) |
| MangaBall | Rich filters, tags, keywords, most-read feeds, chapters in many languages from several groups | [MangaBall](mangaball.md) |
| Atsu | Home carousels, trending and top-rated lists, explore filters, a separate 18+ catalog | [Atsu](atsu.md) |
| FlameComics | FlameComics' own English scanlations, mostly manhwa | [FlameComics](flamecomics.md) |
| MangaPill | Simple search, detail and read, with previous and next chapter ids | [MangaPill](mangapill.md) |

## Response format

Every data route of every manga provider uses the same envelope:

```json
{
  "status": 200,
  "success": true,
  "data": {}
}
```

Errors keep the same shape, with `message` set and `data` set to `null`:

```json
{
  "status": 400,
  "success": false,
  "message": "Query parameter 'q' is required",
  "data": null
}
```

Upstream failures are mapped the same way for every provider:

| Upstream result | Cooren status |
| --- | --- |
| `404` | `404` |
| `429` | `503`, retry shortly |
| Timeout (All Manga, MangaBall, Atsu) | `504` |
| Any other failure, or the site is unreachable | `502` |

Atsu is the only provider that passes an upstream `400` through as `400`, and only on its list and author routes.

## Images

All Manga, MangaBall, Atsu and MangaPill return image URLs that already point at their own `/manga/<provider>/image/*` route, for example:

```txt
http://localhost:3000/manga/mangapill/image/cdn.readdetectiveconan.com/file/mangapill/i/5035.jpeg
```

The part after `/image/` is the upstream URL without `https://`. The route fetches it over HTTPS with the provider's `Referer`, follows up to 3 redirects and returns the image bytes with `Cache-Control: public, max-age=604800, immutable`. Use these URLs as they are in `<img>` tags. They start with your `SERVER_ORIGIN`, so set it to the public URL of the API (see [Configuration](../../getting-started/configuration.md#server_origin)).

FlameComics has no image route. Its covers and pages are direct URLs that load without extra headers.

## Overview route

`GET /manga` lists the providers and their main routes. It does not use the envelope.

```bash
curl "http://localhost:3000/manga"
```

```json
{
  "service": "manga",
  "description": "Unified manga API — provider-isolated route architecture",
  "providers": ["mangaball", "allmanga", "atsu", "flamecomics", "mangapill"],
  "endpoints": {
    "mangaball": [
      "GET /manga/mangaball/home          → Featured titles and banners"
    ],
    "allmanga": [
      "GET /manga/allmanga/home           → Home Page (Popular, Latest, Tags, Random)"
    ],
    "atsu": [
      "--- STANDARD ENDPOINTS ---",
      "GET /manga/atsu/home               → Get All Home Sections combined"
    ],
    "flamecomics": [
      "GET /manga/flamecomics/search              → Search by keyword (?q=query)"
    ],
    "mangapill": [
      "GET /manga/mangapill/search               → Search by keyword (?q=query)"
    ]
  }
}
```

Each provider also answers `GET /manga/<provider>` with a small status object (`provider`, `status`, `description`, `message`).

## Choosing a provider

* For a plain search, detail and reader flow, MangaPill and FlameComics are the simplest: three routes each, and a chapter read answers in about a second.
* For browsing and filtering, MangaBall has the most options (tags, keywords, origin, status, demographic, sort) and chapters in many languages. Its lists include 18+ titles, flagged with `is18plus`.
* For curated home sections and ranked lists, Atsu's carousels, trending, most bookmarked and top-rated routes are fast. Its 18+ titles are only returned by the `/manga/atsu/adult/*` routes.
* All Manga has popular lists by day, week, month or all time, plus genre and author browsing. Its `/read` route drives a real browser and takes 3 to 10 seconds, and the site rate limits bursts of requests.
* Ids do not carry over between providers. Search each provider separately if you want to combine them.
