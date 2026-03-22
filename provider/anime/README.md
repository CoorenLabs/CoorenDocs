---
icon: tv
---

# Anime

## Anime

The anime module exposes multiple providers behind one route group.

Use it for anime search, browse flows, metadata, schedules, and streaming sources.

{% hint style="info" %}
Base route: `GET /anime/...`
{% endhint %}

### Overview

Current providers:

* `animepahe`
* `animekai`
* `toonstream`

The root anime group is split by provider.

Each provider focuses on a slightly different use case:

* AnimePahe for search, latest episodes, and stream variants
* Animekai for rich catalog browsing, schedules, and relations
* ToonStream for home feeds, movie or series browsing, and proxy-backed playback

### Providers

#### AnimePahe

AnimePahe is a good fit when you need:

* rich search results
* latest airing episodes
* full episode lists
* multiple stream variants per episode

See [AnimePahe](animepahe.md) for the full route reference.

#### Animekai

Animekai is a good fit when you need:

* spotlight and airing schedule data
* browse by type and genre
* recommendations and relations
* server lists and intro or outro timing

See [Animekai](animekai.md) for the full route reference.

#### ToonStream

ToonStream is a good fit when you need:

* a home-style feed
* movie and series browsing
* direct source extraction
* media proxy routes for playback

See [ToonStream](toonstream.md) for the full route reference.

### Typical integration

1. Use Animekai or ToonStream for discovery-heavy experiences.
2. Use AnimePahe when you want stronger episode and stream coverage.
3. Pick the provider that best matches your UI flow.
4. Mix providers if you want broader source coverage.
