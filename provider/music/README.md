---
description: Music search, metadata, editorial pages and playback manifests.
icon: music
---

# Music

Music providers return tracks, albums, artists, playlists and editorial pages, plus playback manifests for previews and, with an account session, full-length streams.

{% hint style="info" %}
Base route: `/music`
{% endhint %}

## Providers

| Provider | Good for | Page |
| --- | --- | --- |
| Tidal | Search, full catalog metadata, home, charts and new-release pages, genres, moods, radio, 30-second previews and session-based full streams | [Tidal](tidal.md) |

## Overview route

`GET /music` lists the providers and a few of their routes.

```bash
curl "http://localhost:3000/music"
```

```json
{
  "service": "music",
  "description": "Unified Music API — provider-isolated route architecture",
  "providers": ["tidal"],
  "endpoints": {
    "tidal": [
      "GET /music/tidal/search?q=...         → Search music",
      "GET /music/tidal/tracks/:id           → Track details",
      "GET /music/tidal/featured             → Featured highlights"
    ]
  }
}
```

## Choosing a provider

* Tidal is the only music provider. It calls Tidal's API directly, so most routes answer in under a second.
* Metadata, artwork and 30-second previews need no account. Full-length audio and video need a Tidal session id, sent as the `x-tidal-sessionid` header or the `sessionId` query parameter.
