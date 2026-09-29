---
description: HLS streams and subtitles for movies and TV episodes from the vidcore.io embed player.
icon: play
---

# VidCore

VidCore scrapes the [vidcore.io](https://vidcore.io) embed player. Cooren opens the player in a real browser, clicks through each of its servers and captures every HLS playlist the player loads, while subtitles come from the site's subtitle API. Sources and subtitles are returned as [stream proxy](../../core/proxy.md) links, so they play without extra headers.

{% hint style="info" %}
Base route: `/stream/vidcore`
{% endhint %}

## Routes

| Route | Description |
| --- | --- |
| `GET /stream/vidcore` | Provider status |
| `GET /stream/vidcore/movie/:id` | Sources for a movie |
| `GET /stream/vidcore/tv/:id/:season/:episode` | Sources for a TV episode |
| `GET /stream/vidcore/watch` | Same as the two routes above, with query parameters |

Every source route wraps its result the same way:

```json
{
  "status": 200,
  "success": true,
  "data": {}
}
```

Errors return `{ "status": 404, "success": false, "message": "...", "data": null }`.

## Status

### Provider status

`GET /stream/vidcore`

**Example**

```bash
curl "http://localhost:3000/stream/vidcore"
```

```json
{
  "provider": "Vidcore",
  "status": "operational",
  "description": "Vidcore is a streaming provider that supplies encrypted video sources and multi-language subtitles.",
  "message": "Vidcore provider is running. Visit /docs for available endpoints."
}
```

This is a fixed response; it does not check that vidcore.io is reachable.

## Sources

The source routes return this object in `data`:

| Field | Description |
| --- | --- |
| `type` | `movie` or `tv` |
| `tmdbId` | The id you sent, as a string |
| `season`, `episode` | Only for TV, as strings |
| `providerName` | `vidcore` |
| `subtitles[].label` | Language name, such as `English` or `Portuguese (BR)` |
| `subtitles[].url` | Subtitle file through `/proxy/fetch` |
| `subtitles[].format` | File format. Every subtitle seen in testing was `srt` |
| `sources[].type` | Always `hls` |
| `sources[].url` | HLS playlist through `/proxy/m3u8-proxy`, with the player's `Referer` attached. Play this URL directly |
| `sources[].quality` | Always `auto`. Master playlists list the actual resolutions |
| `sources[].server` | The player server that produced the stream, such as `Supreme`, `Prime` or `Horizon` |
| `isEncrypted` | Always `false` |

Servers that load the same playlist as an earlier server are not listed twice, so `sources` usually has fewer entries than the player has servers.

### Movie sources

`GET /stream/vidcore/movie/:id`

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | TMDB movie id, for example `550`. An IMDb id such as `tt0137523` also works |

**Example**

```bash
curl "http://localhost:3000/stream/vidcore/movie/550"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "type": "movie",
    "tmdbId": "550",
    "providerName": "vidcore",
    "subtitles": [
      {
        "label": "Spanish",
        "url": "http://localhost:3000/proxy/fetch?url=https%3A%2F%2Fvidcore.io%2Fwyzie%2Feu_WWorotE56WcAyo37kpmMw4BhOq3oBgVKIx...",
        "format": "srt"
      },
      {
        "label": "Portuguese (BR)",
        "url": "http://localhost:3000/proxy/fetch?url=https%3A%2F%2Fvidcore.io%2Fwyzie%2Feu_WWorotE56WcAyo37kpmMw4BhOq3oBgVKIx...",
        "format": "srt"
      }
    ],
    "sources": [
      {
        "type": "hls",
        "url": "http://localhost:3000/proxy/m3u8-proxy?url=https%3A%2F%2Fmoon.zenoak.top%2Fvd%2FOXN3d0w5Q1pyVGNmVFNmZFpLeERhQTo0VTdCeWtX...%2Fmaster.m3u8&headers=%7B%22Referer%22%3A%22https%3A%2F%2Fvidcore.io%2F%22%7D",
        "quality": "auto",
        "server": "Supreme"
      },
      {
        "type": "hls",
        "url": "http://localhost:3000/proxy/m3u8-proxy?url=https%3A%2F%2Fmoon.zenoak.top%2Fvd%2FOXN3d0w5Q1pyVGNmVFNmZFpLeERhQTo0VTdCeWtX...%2Findex-s2160p-v1-a1.m3u8&headers=%7B%22Referer%22%3A%22https%3A%2F%2Fvidcore.io%2F%22%7D",
        "quality": "auto",
        "server": "Prime"
      }
    ],
    "isEncrypted": false
  }
}
```

### TV episode sources

`GET /stream/vidcore/tv/:id/:season/:episode`

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | TMDB TV show id, for example `1399` |
| `season` | Yes | Season number |
| `episode` | Yes | Episode number |

**Example**

```bash
curl "http://localhost:3000/stream/vidcore/tv/1399/1/1"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "type": "tv",
    "tmdbId": "1399",
    "season": "1",
    "episode": "1",
    "providerName": "vidcore",
    "subtitles": [
      {
        "label": "English",
        "url": "http://localhost:3000/proxy/fetch?url=https%3A%2F%2Fvidcore.io%2Fwyzie%2Feu_WWorotE56WcAyo37kpmMw4BhOq3oBgVKIx...",
        "format": "srt"
      }
    ],
    "sources": [
      {
        "type": "hls",
        "url": "http://localhost:3000/proxy/m3u8-proxy?url=https%3A%2F%2Fmoon.zenoak.top%2Fvd%2FU2pISEFyMXRrWHVFNGRoc0ZDV3kyUTpDa2txS0NlVTE2MEMwUV...%2Fmaster.m3u8&headers=%7B%22Referer%22%3A%22https%3A%2F%2Fvidcore.io%2F%22%7D",
        "quality": "auto",
        "server": "Supreme"
      }
    ],
    "isEncrypted": false
  }
}
```

### Watch by query

`GET /stream/vidcore/watch`

The same lookup as the movie and TV routes, with the ids in the query string.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `type` | Yes | | `movie` or `tv` |
| `id` | Yes | | TMDB id, or an IMDb id for movies |
| `s` | For `tv` | | Season number |
| `e` | For `tv` | | Episode number |

**Example**

```bash
curl "http://localhost:3000/stream/vidcore/watch?type=tv&id=1396&s=1&e=1"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "type": "tv",
    "tmdbId": "1396",
    "season": "1",
    "episode": "1",
    "providerName": "vidcore",
    "subtitles": [
      {
        "label": "Portuguese",
        "url": "http://localhost:3000/proxy/fetch?url=https%3A%2F%2Fvidcore.io%2Fwyzie%2Feu_WWorotE56WcAyo37kpmMw4BhOq3oBgVKIx...",
        "format": "srt"
      }
    ],
    "sources": [
      {
        "type": "hls",
        "url": "http://localhost:3000/proxy/m3u8-proxy?url=https%3A%2F%2Fmoon.zenoak.top%2Fvd%2FZVQwNjVpX1NpaU03c19ZRmdlenN0dzpDYmRtcXVw...%2Fmaster.m3u8&headers=%7B%22Referer%22%3A%22https%3A%2F%2Fvidcore.io%2F%22%7D",
        "quality": "auto",
        "server": "Supreme"
      },
      {
        "type": "hls",
        "url": "http://localhost:3000/proxy/m3u8-proxy?url=https%3A%2F%2Fcdn30092.luxki440das.com%2Fstream2%2Fi-arch-400%2Fca061cc077abb...&headers=%7B%22Referer%22%3A%22https%3A%2F%2Fvidcore.io%2F%22%7D",
        "quality": "auto",
        "server": "Horizon"
      }
    ],
    "isEncrypted": false
  }
}
```

## Errors

| Status | When |
| --- | --- |
| `400` | `/watch` without `id` (`TMDB ID is required (?id=...)`), with a `type` other than `movie` or `tv` (`Invalid type. Must be 'movie' or 'tv'`), or `type=tv` without `s` and `e` (`Season (?s=) and Episode (?e=) are required for TV shows`). Any route with an id that is not a number or `tt` plus digits, or a season or episode that is not a number (`Invalid id, season or episode`) |
| `404` | The player reported an error and no server produced a stream, for example an unknown id: `No streams are available for this title` |
| `502` | The browser failed while loading the player: `Extraction failed: ...` |
| `504` | No server produced a stream before the 22 second deadline and none reported an error: `Timed out waiting for streams` |

An unknown id, such as `/stream/vidcore/movie/999999999`, took about 10 seconds to return `404`:

```json
{
  "status": 404,
  "success": false,
  "message": "No streams are available for this title",
  "data": null
}
```

## Notes

* Uncached requests took 13 to 19 seconds in testing, because the browser waits on each server in turn (up to 22 seconds in total). Subtitles are fetched at the same time.
* With Redis enabled, results are cached for 10 minutes.
* The server names seen in testing were `Supreme`, `Prime`, `Orbit`, `Premiere 4K` and `Horizon`. Not every server has every title.
* The browser opens at most three pages at once across all browser-backed providers, so parallel requests queue.
