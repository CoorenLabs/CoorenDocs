---
description: Movie and TV episode sources looked up by TMDB id.
icon: film
---

# Movie/TV

Movie/TV providers take a TMDB id and return playable sources for a movie or a TV episode. They do not offer search or browsing, so resolve the TMDB id in your app first.

{% hint style="info" %}
Base route: `/movie-tv`
{% endhint %}

## Providers

| Provider | Good for | Page |
| --- | --- | --- |
| PrimeSrc | Direct MP4 files from Streamtape and Dood, with file size and quality | [PrimeSrc](primesrc.md) |

## Overview route

`GET /movie-tv` lists the providers and their routes.

```bash
curl "http://localhost:3000/movie-tv"
```

```json
{
  "service": "movie-tv",
  "description": "Unified Movie & TV API — provider-isolated route architecture",
  "providers": ["primesrc"],
  "endpoints": {
    "primesrc": [
      "GET /movie-tv/primesrc/movie/:tmdbid   → Get movie sources",
      "GET /movie-tv/primesrc/tv/:tmdbid/:season/:episode → Get TV episode sources"
    ]
  }
}
```

## Choosing a provider

* PrimeSrc returns whole MP4 files with a known size and resolution, which suits downloads and simple `<video>` players. It answered in 6 to 14 seconds in testing.
* For HLS streams with subtitles, use the [Stream](../stream/README.md) providers VidCore and VidFast, which take the same TMDB ids but took 14 to 23 seconds per request in testing.
* To turn an AniList, MAL or IMDb id into a TMDB id, use [ID Mappings](../../core/mappings.md).
