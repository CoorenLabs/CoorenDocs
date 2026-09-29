---
description: Metadata providers for anime discovery, home feeds and detail pages.
icon: circle-info
---

# Meta

Meta providers return titles, artwork and metadata, not streams. Use them to build home pages, search and detail screens, then pass the ids to a streaming provider.

{% hint style="info" %}
Base route: `/meta`
{% endhint %}

## Providers

| Provider | Good for | Page |
| --- | --- | --- |
| AniList | Anime home feed, search, full detail pages with characters, relations and recommendations | [AniList](anilist.md) |

## Overview route

`GET /meta` lists the providers and their main routes.

```bash
curl "http://localhost:3000/meta"
```

```json
{
  "service": "meta",
  "description": "Meta providers — anime/movie discovery via AniList and external data sources",
  "providers": ["anilist"],
  "endpoints": {
    "anilist": [
      "GET /meta/anilist/home              → Home page (spotlight + sections)",
      "GET /meta/anilist/anime/:id         → Full anime metadata",
      "GET /meta/anilist/search/:query     → AniList search"
    ]
  }
}
```

## Choosing a provider

* AniList is the only meta provider. Its ids are AniList ids, which Miruro, Anivexa and Animelok accept directly.
* To reach TMDB-based providers such as [PrimeSrc](../movie-tv/primesrc.md) or [VidCore](../stream/vidcore.md), convert the AniList id with [ID Mappings](../../core/mappings.md).
