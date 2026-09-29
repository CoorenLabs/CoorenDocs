---
description: HLS streams and subtitles for movies and TV episodes from the vidfast.vc embed player.
icon: forward
---

# VidFast

VidFast scrapes the [vidfast.vc](https://vidfast.vc) embed player. Cooren opens the player in a real browser, clicks through each of its servers and captures every HLS playlist the player loads, while subtitles come from the site's subtitle API. Sources and subtitles are returned as [stream proxy](../../core/proxy.md) links, so they play without extra headers.

{% hint style="info" %}
Base route: `/stream/vidfast`
{% endhint %}

## Routes

| Route | Description |
| --- | --- |
| `GET /stream/vidfast` | Provider status |
| `GET /stream/vidfast/movie/:id` | Sources for a movie |
| `GET /stream/vidfast/tv/:id/:season/:episode` | Sources for a TV episode |
| `GET /stream/vidfast/watch` | Same as the two routes above, with query parameters |

Source routes return the result object directly, without an envelope. Errors return `{ "error": "..." }`.

## Status

### Provider status

`GET /stream/vidfast`

**Example**

```bash
curl "http://localhost:3000/stream/vidfast"
```

```json
{
  "provider": "vidfast",
  "status": "active",
  "type": "stream",
  "capabilities": ["movie", "tv"]
}
```

This is a fixed response; it does not check that vidfast.vc is reachable.

## Sources

The source routes return this object:

| Field | Description |
| --- | --- |
| `type` | `movie` or `tv` |
| `tmdbId` | The id you sent, as a string |
| `season`, `episode` | Only for TV, as strings |
| `providerName` | `vidfast` |
| `subtitles[].label` | Language name, such as `English` or `Portuguese (BR)` |
| `subtitles[].url` | Subtitle file through `/proxy/fetch` |
| `subtitles[].format` | File format. Every subtitle seen in testing was `srt` |
| `sources[].type` | Always `hls` |
| `sources[].url` | HLS playlist through `/proxy/m3u8-proxy`, with the player's `Referer` attached. Play this URL directly |
| `sources[].quality` | Always `auto`. Master playlists list the actual resolutions |
| `sources[].server` | The player server that produced the stream, such as `vRapid`, `vBlaze` or `Bravo` |
| `isEncrypted` | Always `false` |

Servers that load the same playlist as an earlier server are not listed twice, so `sources` usually has fewer entries than the player has servers.

### Movie sources

`GET /stream/vidfast/movie/:id`

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | TMDB movie id, for example `550`. An IMDb id such as `tt0137523` also works |

**Example**

```bash
curl "http://localhost:3000/stream/vidfast/movie/550"
```

```json
{
  "type": "movie",
  "tmdbId": "550",
  "providerName": "vidfast",
  "subtitles": [
    {
      "label": "Spanish",
      "url": "http://localhost:3000/proxy/fetch?url=https%3A%2F%2Fvidfast.vc%2Fwyzie%2Feu_WWorotE56WcAyo37kpmMw4BhOq3oBgVKIx...",
      "format": "srt"
    },
    {
      "label": "Portuguese (BR)",
      "url": "http://localhost:3000/proxy/fetch?url=https%3A%2F%2Fvidfast.vc%2Fwyzie%2Feu_WWorotE56WcAyo37kpmMw4BhOq3oBgVKIx...",
      "format": "srt"
    }
  ],
  "sources": [
    {
      "type": "hls",
      "url": "http://localhost:3000/proxy/m3u8-proxy?url=https%3A%2F%2Fmoon.zenoak.top%2Fvd%2FOXN3d0w5Q1pyVGNmVFNmZFpLeERhQTo0VTdCeWtX...%2Fmaster.m3u8&headers=%7B%22Referer%22%3A%22https%3A%2F%2Fvidfast.vc%2F%22%7D",
      "quality": "auto",
      "server": "vRapid"
    }
  ],
  "isEncrypted": false
}
```

### TV episode sources

`GET /stream/vidfast/tv/:id/:season/:episode`

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | TMDB TV show id, for example `1399` |
| `season` | Yes | Season number |
| `episode` | Yes | Episode number |

**Example**

```bash
curl "http://localhost:3000/stream/vidfast/tv/1399/1/1"
```

```json
{
  "type": "tv",
  "tmdbId": "1399",
  "season": "1",
  "episode": "1",
  "providerName": "vidfast",
  "subtitles": [
    {
      "label": "English",
      "url": "http://localhost:3000/proxy/fetch?url=https%3A%2F%2Fvidfast.vc%2Fwyzie%2Feu_WWorotE56WcAyo37kpmMw4BhOq3oBgVKIx...",
      "format": "srt"
    }
  ],
  "sources": [
    {
      "type": "hls",
      "url": "http://localhost:3000/proxy/m3u8-proxy?url=https%3A%2F%2Fmoon.zenoak.top%2Fvd%2FU2pISEFyMXRrWHVFNGRoc0ZDV3kyUTpDa2txS0NlVTE2MEMwUV...%2Fmaster.m3u8&headers=%7B%22Referer%22%3A%22https%3A%2F%2Fvidfast.vc%2F%22%7D",
      "quality": "auto",
      "server": "vRapid"
    },
    {
      "type": "hls",
      "url": "http://localhost:3000/proxy/m3u8-proxy?url=https%3A%2F%2Fcdn30092.luxki440das.com%2Fstream2%2Fi-arch-400%2F...%2Findex.m3u8&headers=%7B%22Referer%22%3A%22https%3A%2F%2Fvidfast.vc%2F%22%7D",
      "quality": "auto",
      "server": "Bravo"
    }
  ],
  "isEncrypted": false
}
```

### Watch by query

`GET /stream/vidfast/watch`

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
curl "http://localhost:3000/stream/vidfast/watch?type=movie&id=27205"
```

```json
{
  "type": "movie",
  "tmdbId": "27205",
  "providerName": "vidfast",
  "subtitles": [
    {
      "label": "English",
      "url": "http://localhost:3000/proxy/fetch?url=https%3A%2F%2Fvidfast.vc%2Fwyzie%2Feu_WWorotE56WcAyo37kpmMw4BhOq3oBgVKIx...",
      "format": "srt"
    }
  ],
  "sources": [
    {
      "type": "hls",
      "url": "http://localhost:3000/proxy/m3u8-proxy?url=https%3A%2F%2Fmoon.zenoak.top%2Fvd%2FczN4MEdZVFNJbldWTmlURTVPcmZmZzpRY0Q3OHVO...%2Fmaster.m3u8&headers=%7B%22Referer%22%3A%22https%3A%2F%2Fvidfast.vc%2F%22%7D",
      "quality": "auto",
      "server": "vRapid"
    },
    {
      "type": "hls",
      "url": "http://localhost:3000/proxy/m3u8-proxy?url=https%3A%2F%2Fmoon.zenoak.top%2Fvdb%2FczN4MEdZVFNJbldWTmlURTVPcmZmZzpRY0Q3OHV...%2Fmaster.m3u8&headers=%7B%22Referer%22%3A%22https%3A%2F%2Fvidfast.vc%2F%22%7D",
      "quality": "auto",
      "server": "vBlaze"
    }
  ],
  "isEncrypted": false
}
```

## Errors

| Status | When |
| --- | --- |
| `400` | `/watch` with a `type` other than `movie` or `tv`, or `type=tv` without `s` and `e`: `{ "error": "Invalid parameters." }`. Any route with an id that is not a number or `tt` plus digits, or a season or episode that is not a number: `{ "error": "Invalid id, season or episode" }` |
| `404` | The player reported an error and no server produced a stream, for example an unknown id: `{ "error": "No streams are available for this title" }` |
| `422` | `/watch` without `type` or `id` |
| `502` | The browser failed while loading the player: `{ "error": "Extraction failed: ..." }` |
| `504` | No server produced a stream before the 22 second deadline and none reported an error: `{ "error": "Timed out waiting for streams" }` |

## Notes

* Uncached requests took 14 to 23 seconds in testing, because the browser waits on each server in turn (up to 22 seconds in total). An unknown id took about 12 seconds to return `404`.
* With Redis enabled, results are cached for 10 minutes.
* The server names seen in testing were `vRapid`, `vBlaze`, `Cobra`, `Cine`, `Bravo` and `Horizon`. Not every server has every title.
* The browser opens at most three pages at once across all browser-backed providers, so parallel requests queue.
