---
icon: music
---

# Music

## Music

The music module currently exposes one provider: Tidal.

Use it for music search, metadata, editorial pages, and playback manifest access.

{% hint style="info" %}
Base route: `GET /music/...`
{% endhint %}

### Overview

Current providers:

* `tidal`

The root route returns a small provider index.

#### Music root

`GET /music/`

Returns the music service overview and provider list.

Example shape:

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

### Provider

#### Tidal

Tidal is the current music provider.

It supports:

* global search
* track, album, artist, playlist, mix, and video metadata
* recommendations and radio
* editorial pages
* genres and moods
* playback manifest access

See [Tidal](tidal.md) for the full route reference.
