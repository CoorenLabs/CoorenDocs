---
icon: film
---

# Movie/TV

## Movie & TV

The Movie & TV module exposes catalog and source providers behind one route group.

Use it for title discovery, search, and playback source extraction.

{% hint style="info" %}
Base route: `GET /movie-tv/...`
{% endhint %}

### Overview

Current providers:

* `primesrc`
* `yflix`

The root route returns a service-level overview.

#### Movie & TV root

`GET /movie-tv/`

Returns available providers and key routes.

Example shape:

```json
{
  "service": "movie-tv",
  "description": "Unified Movie & TV API — provider-isolated route architecture",
  "providers": ["primesrc", "yflix"],
  "endpoints": {
    "primesrc": [
      "GET /movie-tv/primesrc/movie/:tmdbid",
      "GET /movie-tv/primesrc/tv/:tmdbid/:season/:episode"
    ],
    "yflix": [
      "GET /movie-tv/yflix/home",
      "GET /movie-tv/yflix/search?query=..."
    ]
  }
}
```

### Providers

#### PrimeSrc

PrimeSrc is the source extractor.

Use it when you already have a TMDB ID and need playable hosts.

See [PrimeSrc](primesrc.md) for full details.

#### Yflix

Yflix is the catalog and search provider.

Use it for home feeds and movie or TV title discovery.

See [Yflix](yflix.md) for full details.

### Typical integration

1. Use Yflix for discovery and search.
2. Resolve a TMDB ID in your app.
3. Use PrimeSrc for movie or episode sources.
4. Present returned hosts and subtitles in the player UI.
