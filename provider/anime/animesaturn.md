---
description: Italian anime provider that scrapes AnimeSaturn for search, show details, episode lists and stream sources.
icon: circle-play
---

# AnimeSaturn

AnimeSaturn scrapes [animesaturn.net](https://www.animesaturn.net), an Italian anime streaming site. It covers search, a show page with its episode list, and the stream sources of an episode, decoded from the site's own player. Page requests go through the shared fetcher, which solves a Cloudflare challenge in a browser when one appears.

{% hint style="info" %}
Base route: `/anime/animesaturn`
{% endhint %}

{% hint style="warning" %}
As of 2026-09-29, animesaturn.net and its player host return `502`, so every data route answers `502`. The routes, parameters and response fields below are taken from the source code and could not be checked against live data.
{% endhint %}

## Routes

| Route | Description |
| --- | --- |
| `GET /anime/animesaturn` | Provider info and route list |
| `GET /anime/animesaturn/search/:query` | Search titles |
| `GET /anime/animesaturn/info/:id` | Show details and episode list |
| `GET /anime/animesaturn/watch/:animeId/ep-:number` | Stream sources and downloads for one episode |

## Ids

| Id | Format | Where it comes from |
| --- | --- | --- |
| Anime id | The slug from the site's `/anime/<slug>` URL, letters, digits, `_` and `-` only, for example `one-piece-ita-bz8UJ` | `id` of a search result |
| Episode id | `<animeId>/ep-<number>`, for example `one-piece-ita-bz8UJ/ep-1` | `id` of an entry in `episodes` from `/info` |

## Overview

### Provider info

`GET /anime/animesaturn`

Returns the provider name, a description and the route list. It does not contact the site.

**Example**

```bash
curl "http://localhost:3000/anime/animesaturn"
```

```json
{
  "name": "animesaturn",
  "description": "Italian anime provider backed by AnimeSaturn — search, info and streams.",
  "endpoints": [
    "/anime/animesaturn/search/:query?page=",
    "/anime/animesaturn/info/:id",
    "/anime/animesaturn/watch/:animeId/:episode"
  ]
}
```

## Search

### Search titles

`GET /anime/animesaturn/search/:query`

Searches the site's catalogue filter and returns matching shows.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `query` | Yes | Search text |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `page` | No | `1` | Result page, a positive whole number |

**Response fields**

The response is `{ "results": [...] }`. When the site has no page for the search, `results` is an empty array.

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Anime id, used by `/info/:id` |
| `title` | string | Show title |
| `url` | string | Show page on animesaturn.net |
| `image` | string | Poster URL |
| `type` | string | Type badge shown on the card, when present |

**Example**

```bash
curl "http://localhost:3000/anime/animesaturn/search/one%20piece"
```

Response while the site is down:

```json
{
  "error": "AnimeSaturn responded 502"
}
```

## Details

### Show details

`GET /anime/animesaturn/info/:id`

Returns a show's details and its full episode list.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Anime id from search, for example `one-piece-ita-bz8UJ` |

**Response fields**

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | The anime id you passed |
| `title` | string | Show title |
| `url` | string | Show page on animesaturn.net |
| `image` | string | Poster URL, when found |
| `description` | string | Plot summary, when found |
| `genres` | string[] | Genre names |
| `type` | string | The page's "Tipo" value, when present |
| `status` | string | The page's "Stato" value, when present |
| `totalEpisodes` | number | Number of entries in `episodes` |
| `episodes` | array | Episodes, each with `id` (episode id for `/watch`), `number` and `url` |

**Example**

```bash
curl "http://localhost:3000/anime/animesaturn/info/one-piece-ita-bz8UJ"
```

Response while the site is down:

```json
{
  "error": "AnimeSaturn responded 502"
}
```

## Streams

### Stream sources

`GET /anime/animesaturn/watch/:animeId/ep-:number`

Returns every player server listed on the episode page. For the site's own embedded player, the route fetches and decodes the real stream URL and adds a `proxiedUrl` that goes through the shared [stream proxy](../../core/proxy.md) with the `Referer` the player host requires. Other servers are returned as their embed link.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `animeId` | Yes | Anime id |
| `number` | Yes | Episode part of the id after `ep-`; letters, digits, `.`, `_` and `-` are accepted |

The whole path after `/watch/` is the episode id from `/info`, for example `/watch/one-piece-ita-bz8UJ/ep-1`.

**Response fields**

The response is `{ "results": { "streams": [...], "downloads": [...] } }`. `downloads` is left out when the page lists none.

| Field | Type | Description |
| --- | --- | --- |
| `streams[].server` | string | Server name as shown on the site |
| `streams[].url` | string | Decoded media URL, or the server's embed link when it could not be decoded |
| `streams[].embed` | boolean | `false` for a decoded media URL, otherwise the site's embed flag for that server |
| `streams[].isM3U8` | boolean | Decoded streams only; `true` when `url` is an HLS playlist |
| `streams[].embedUrl` | string | Decoded streams only; the player page the stream came from |
| `streams[].proxiedUrl` | string | Decoded streams only; `url` through the shared stream proxy with the player's `Referer` |
| `downloads[].server` | string | Server name |
| `downloads[].url` | string | Download link |

**Example**

```bash
curl "http://localhost:3000/anime/animesaturn/watch/one-piece-ita-bz8UJ/ep-1"
```

Response while the site is down:

```json
{
  "error": "AnimeSaturn responded 502"
}
```

## Errors

| Status | When |
| --- | --- |
| `400` | `page` is not a positive whole number, the anime id has characters other than letters, digits, `_` and `-`, or the watch path is not `<animeId>/ep-<number>` |
| `404` | `/info`: the show page does not exist or has no title. `/watch`: the episode page does not exist or lists no servers |
| `502` | The site answered with an error status (other than `404`) or did not answer |

```bash
curl "http://localhost:3000/anime/animesaturn/search/naruto?page=0"
```

```json
{
  "error": "page must be a positive integer"
}
```

```bash
curl "http://localhost:3000/anime/animesaturn/info/bad.id"
```

```json
{
  "error": "Invalid anime id"
}
```

```bash
curl "http://localhost:3000/anime/animesaturn/watch/one-piece-ita-bz8UJ"
```

```json
{
  "error": "Episode id must look like {animeId}/ep-{number}"
}
```

The `404` bodies are `{ "error": "Anime not found" }` for `/info` and `{ "error": "No streams found" }` for `/watch`.

## Notes

- Content is in Italian. Titles and descriptions come straight from the site.
- The flow is search, then `/info/:id` for the episode list, then `/watch/<episode id>`.
- Play `proxiedUrl` rather than `url` for decoded streams; the player host checks the `Referer`.
- Nothing is cached; every request goes to the site.
