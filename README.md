---
description: >-
  Cooren is an open-source scraping API that returns structured data and
  playable links for anime, manga, movies, TV and music.
icon: layer-plus
layout:
  width: default
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
  tags:
    visible: true
---

# Welcome

Cooren is a single HTTP API in front of many content sites. Each site is a provider with its own routes, and every provider returns clean JSON: search results, details, episode and chapter lists, and stream or image links that work in a browser through Cooren's built-in proxy.

It runs on [Bun](https://bun.sh) and [ElysiaJS](https://elysiajs.com). The source lives at [CoorenLabs/CoorenLabs](https://github.com/CoorenLabs/CoorenLabs).

{% hint style="success" %}
Start with the quickstart, then open `/docs` on your running server for the interactive OpenAPI reference.
{% endhint %}

### Start here

<table data-view="cards"><thead><tr><th>Title</th><th>Description</th><th data-card-target data-type="content-ref">Target</th><th data-hidden data-card-cover data-type="image">Cover image</th></tr></thead><tbody><tr><td>Quickstart</td><td>Install the project and run the API locally.</td><td><a href="getting-started/quickstart.md">quickstart.md</a></td><td><a href="https://images.unsplash.com/photo-1639493115941-a70fcef4f715?crop=entropy&#x26;cs=srgb&#x26;fm=jpg&#x26;ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHwxfHxncmFkaWVudCUyMG1lc2h8ZW58MHx8fHwxNzczNDAzNzMwfDA&#x26;ixlib=rb-4.1.0&#x26;q=85">https://images.unsplash.com/photo-1639493115941-a70fcef4f715?crop=entropy&#x26;cs=srgb&#x26;fm=jpg&#x26;ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHwxfHxncmFkaWVudCUyMG1lc2h8ZW58MHx8fHwxNzczNDAzNzMwfDA&#x26;ixlib=rb-4.1.0&#x26;q=85</a></td></tr><tr><td>Configuration</td><td>Set the port, public origin, CORS and Redis cache.</td><td><a href="getting-started/configuration.md">configuration.md</a></td><td><a href="https://images.unsplash.com/photo-1689028294012-5c4c9ff10cb2?crop=entropy&#x26;cs=srgb&#x26;fm=jpg&#x26;ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHw0fHxncmFkaWVudCUyMG1lc2h8ZW58MHx8fHwxNzczNDAzNzMwfDA&#x26;ixlib=rb-4.1.0&#x26;q=85">https://images.unsplash.com/photo-1689028294012-5c4c9ff10cb2?crop=entropy&#x26;cs=srgb&#x26;fm=jpg&#x26;ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHw0fHxncmFkaWVudCUyMG1lc2h8ZW58MHx8fHwxNzczNDAzNzMwfDA&#x26;ixlib=rb-4.1.0&#x26;q=85</a></td></tr><tr><td>Providers</td><td>Browse providers by category and source.</td><td><a href="provider/anime/README.md">README.md</a></td><td><a href="https://images.unsplash.com/photo-1710184713246-91865a6123dc?crop=entropy&#x26;cs=srgb&#x26;fm=jpg&#x26;ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHwzfHxncmFkaWVudCUyMG1lc2h8ZW58MHx8fHwxNzczNDAzNzMwfDA&#x26;ixlib=rb-4.1.0&#x26;q=85">https://images.unsplash.com/photo-1710184713246-91865a6123dc?crop=entropy&#x26;cs=srgb&#x26;fm=jpg&#x26;ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHwzfHxncmFkaWVudCUyMG1lc2h8ZW58MHx8fHwxNzczNDAzNzMwfDA&#x26;ixlib=rb-4.1.0&#x26;q=85</a></td></tr></tbody></table>

### What Cooren covers

| Category | Base route | Providers |
| --- | --- | --- |
| [Anime](provider/anime/README.md) | `/anime` | AnimePahe, ToonStream, Animelok, Miruro, Anivexa, AnimeSaturn, AnimeUnity |
| [Manga](provider/manga/README.md) | `/manga` | All Manga, MangaBall, Atsu, FlameComics, MangaPill |
| [Meta](provider/meta/README.md) | `/meta` | AniList |
| [Movie/TV](provider/movie-tv/README.md) | `/movie-tv` | PrimeSrc |
| [Stream](provider/stream/README.md) | `/stream` | VidCore, VidFast |
| [Music](provider/music/README.md) | `/music` | Tidal |

Two core routes support every provider:

* [Stream Proxy](core/proxy.md) at `/proxy` replays HLS, MP4 and subtitle requests with the headers the source site expects.
* [ID Mappings](core/mappings.md) at `/mappings` converts between AniList, MAL, TMDB, IMDb, Kitsu and AniDB ids.
