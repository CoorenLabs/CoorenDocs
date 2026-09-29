---
description: HLS streams and subtitles for movies and TV episodes, captured from embed players in a real browser.
icon: tower-broadcast
---

# Stream

Stream providers open a movie or TV embed player in a browser, switch through each of its servers and capture the HLS playlists the player loads. Every stream and subtitle comes back already routed through the [stream proxy](../../core/proxy.md), so it plays in any HLS player.

{% hint style="info" %}
Base route: `/stream`
{% endhint %}

## Providers

| Provider | Good for | Page |
| --- | --- | --- |
| VidCore | HLS streams from vidcore.io with 18 to 31 subtitle languages, `{ status, success, data }` envelope | [VidCore](vidcore.md) |
| VidFast | HLS streams from vidfast.vc with the same subtitles, plain response object | [VidFast](vidfast.md) |

Both providers take the same ids and routes and return the same data. They differ in the site they scrape, the server names they report and the response envelope.

## Overview route

`GET /stream` lists the providers and their routes.

```bash
curl "http://localhost:3000/stream"
```

```json
{
  "service": "stream",
  "description": "Unified direct streaming API — provider-isolated route architecture",
  "providers": ["vidcore", "vidfast"],
  "endpoints": {
    "vidcore": [
      "GET /stream/vidcore                                 → Provider Status and info",
      "GET /stream/vidcore/watch?type=movie&id=550         → Fetch extracted sources & subs (Param based)",
      "GET /stream/vidcore/watch?type=tv&id=550&s=1&e=1    → Fetch extracted sources & subs (Param based)",
      "GET /stream/vidcore/movie/:id                       → Fetch extracted sources & subs (Path based)",
      "GET /stream/vidcore/tv/:id/:season/:episode         → Fetch extracted sources & subs (Path based)",
      "NOTE: Returns cleanly extracted HLS streams routed through the internal proxy."
    ],
    "vidfast": [
      "GET /stream/vidfast                                 → Provider Status and info",
      "GET /stream/vidfast/watch?type=movie&id=550         → Fetch extracted sources & subs (Param based)",
      "GET /stream/vidfast/watch?type=tv&id=550&s=1&e=1    → Fetch extracted sources & subs (Param based)",
      "GET /stream/vidfast/movie/:id                       → Fetch extracted sources & subs (Path based)",
      "GET /stream/vidfast/tv/:id/:season/:episode         → Fetch extracted sources & subs (Path based)",
      "NOTE: Returns cleanly extracted HLS streams routed through the internal proxy."
    ]
  }
}
```

## Choosing a provider

* Both took 14 to 23 seconds per uncached request in testing, because a browser tries every server in the player. Show a loading state, and enable Redis so repeats within 10 minutes are instant.
* VidCore usually returned two or three sources per title; VidFast returned one to three. When one returns nothing, try the other.
* Pick VidCore if you want the same `{ status, success, data }` envelope as Tidal and the manga providers. VidFast returns the result object on its own.
* For whole MP4 files with a known size instead of HLS, use [PrimeSrc](../movie-tv/primesrc.md).
