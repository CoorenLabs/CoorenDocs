---
description: Anime providers for search, discovery, episode lists and playable streams.
icon: tv
---

# Anime

The anime group holds seven providers, each under its own route. Some are keyed by the site's own ids and slugs (AnimePahe, ToonStream, AnimeSaturn, AnimeUnity); the others take AniList ids (Miruro, Animelok, Anivexa), so an id from one of them works in the others.

{% hint style="info" %}
Base route: `/anime`
{% endhint %}

## Providers

| Provider | Good for | Page |
| --- | --- | --- |
| AnimePahe | Search, latest releases, full episode lists, Japanese and English-dub HLS streams at 360p to 1080p with MP4 download links | [AnimePahe](animepahe.md) |
| ToonStream | Anime and Western cartoons with Hindi, Tamil, Telugu and English dubs; home feed, movie and series browsing, HLS sources | [ToonStream](toonstream.md) |
| Animelok | Paged episode lists and sub and dub embed players by AniList id | [Animelok](animelok.md) |
| Miruro | AniList search, filters, trending, schedule and full details, plus episode lists with skip times and streams from several sites by AniList id | [Miruro](miruro.md) |
| Anivexa | Episodes and streams from a dozen sites by AniList id, and id mappings | [Anivexa](anivexa.md) |
| AnimeSaturn | Italian-language anime: search, details with episodes, streams | [AnimeSaturn](animesaturn.md) |
| AnimeUnity | Italian-language anime: search, details with episodes, streams | [AnimeUnity](animeunity.md) |

{% hint style="warning" %}
On 2026-09-29 AnimeSaturn's site was down (the API returns `502`), and AnimeUnity only serves Italian IP addresses (from elsewhere the API returns `502`).
{% endhint %}

## Anime index

`GET /anime`

Lists the providers and a one-line summary of each provider's routes.

```bash
curl "http://localhost:3000/anime"
```

```json
{
  "service": "anime",
  "description": "Unified anime API — provider-isolated route architecture",
  "providers": [
    "animepahe",
    "toonstream",
    "animesaturn",
    "animeunity",
    "animelok",
    "miruro",
    "anivexa"
  ],
  "endpoints": {
    "animepahe": [
      "GET /anime/animepahe/search/:query         → Search titles",
      "GET /anime/animepahe/latest                → Latest updated titles",
      "GET /anime/animepahe/info/:id              → Full title details",
      "GET /anime/animepahe/episodes/:id          → Episode list",
      "GET /anime/animepahe/episode/:id/:session  → Stream results"
    ]
  }
}
```

Each provider also answers at its own root, for example `GET /anime/miruro`, with its route list.

## Choosing a provider

* **Discovery and metadata:** Miruro. It covers search, autocomplete, filters, trending, popular, upcoming, the airing schedule, characters, relations and recommendations, all from AniList.
* **Direct streams with sub and dub:** Miruro's `watch` route groups streams from several sites into sub, dub and soft-sub tracks. AnimePahe gives Japanese and English-dub HLS at three qualities plus MP4 download links.
* **Already have an AniList id:** Miruro, Anivexa and Animelok take it directly. Animelok returns embed players for an `<iframe>`, not direct streams.
* **Western cartoons and Indian-language dubs:** ToonStream.
* **Italian-language anime:** AnimeUnity, from an Italian IP. AnimeSaturn when its site is back up.
* **Browser playback:** use the `proxiedUrl` on AnimePahe, ToonStream and Miruro streams. It plays through the [stream proxy](../../core/proxy.md) with the headers the host needs.
* **Speed:** AnimePahe's first request after a restart takes several seconds while a browser solves Cloudflare's challenge; later requests are fast. Miruro's AniList-backed routes share AniList's limit of about 30 requests per minute.
