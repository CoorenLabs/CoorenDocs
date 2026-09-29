---
description: Response formats, status codes, ids and stream links shared across providers.
icon: book
---

# Conventions

Every route returns JSON unless noted: the stream proxy returns media, and AnimePahe's episode route streams newline-delimited JSON.

### Response formats

Each provider keeps the response format it was built with. The provider pages show exact examples.

| Provider | Success | Error |
| --- | --- | --- |
| Manga providers, Tidal, VidCore | `{ "status": 200, "success": true, "data": ... }` | `{ "status": 404, "success": false, "message": "...", "data": null }` |
| AniList, Animelok | `{ "success": true, "data": ... }` | `{ "success": false, "error": "..." }` |
| PrimeSrc | `{ "success": true, "status": 200, "data": [...] }` | `{ "success": false, "status": 404 }` |
| Miruro | The result object itself | `{ "message": "..." }` |
| AnimePahe, Anivexa, VidFast, AnimeSaturn, AnimeUnity | The result object itself | `{ "error": "..." }` |
| ToonStream | `{ "success": true, "data": ... }` | `{ "success": false, "msg": "No Data Scraped!" }` with status `200` |

Some responses also include `served_cache` and `took_ms`, which show whether the result came from the Redis cache and how long the request took.

### Status codes

| Status | Meaning |
| --- | --- |
| `200` | Success |
| `302` | Redirect to a playable stream (Anivexa `/stream/*` routes) |
| `400` | A parameter is missing or invalid |
| `404` | The id, episode or chapter does not exist upstream, or it has no sources |
| `422` | The request failed schema validation, for example a required query parameter is missing or a path parameter has the wrong type |
| `429` | The upstream API is rate limiting, for example AniList |
| `502` | The upstream site failed, timed out or blocked the request |
| `503` | The upstream is rate limiting; retry shortly (manga providers) |
| `504` | The upstream timed out (manga providers) |

A `502` is temporary: retry later. A `404` means the item really is missing.

`422` responses come from Elysia and look like this:

```json
{
  "type": "validation",
  "on": "query",
  "property": "/url",
  "message": "Expected string",
  "summary": "Expected property 'url' to be string but found: undefined"
}
```

### Ids

| Id | Used by | Example |
| --- | --- | --- |
| AniList id | Miruro, Anivexa, Animelok, AniList | `154587` (Frieren) |
| TMDB id | PrimeSrc, VidCore, VidFast | `550` (movie), `1399` (TV) |
| Provider id or slug | AnimePahe, ToonStream, AnimeSaturn, AnimeUnity, manga providers, Tidal | returned by that provider's search |

Use [ID Mappings](../core/mappings.md) to convert between AniList, MAL, TMDB, IMDb, Kitsu and AniDB ids.

### Stream and image links

Many sites only serve video, subtitles or images when a request carries their own `Referer` or `Origin` header, which a browser player cannot set. Responses therefore include links that go through Cooren:

* `proxiedUrl` on streams and subtitles points at the [stream proxy](../core/proxy.md), which adds the right headers and rewrites HLS playlists so every segment also goes through it.
* Manga image URLs point at each provider's `/image/*` route.
* Anivexa uses its own HLS routes for sources that are encrypted or masked.

All of these links start with your `SERVER_ORIGIN`, so set it to the public URL of the API. See [Configuration](configuration.md#server_origin).

### Speed

* Most routes answer in under a second.
* The first request to a Cloudflare-protected provider after a restart takes 8 to 15 seconds while a browser solves the challenge. Later requests reuse the clearance.
* VidCore and VidFast take 15 to 25 seconds because a browser walks every server.
* With Redis enabled, repeat requests are served from the cache.
