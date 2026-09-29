---
description: Italian anime provider that uses AnimeUnity for search, show details, episode lists and VixCloud HLS streams.
icon: earth-europe
---

# AnimeUnity

AnimeUnity uses [animeunity.so](https://www.animeunity.so), an Italian anime streaming site. It searches the site's archive, reads show details and the full episode list from the site's JSON API, and resolves an episode to a VixCloud HLS playlist plus a download link when the player offers one.

{% hint style="info" %}
Base route: `/anime/animeunity`
{% endhint %}

{% hint style="warning" %}
animeunity.so only serves visitors from Italy. From anywhere else Cloudflare answers `403` and every data route returns `502`, which was the case on 2026-09-29. Run the API from an Italian IP to use this provider. The response fields below are taken from the source code and could not be checked against live data.
{% endhint %}

## Routes

| Route | Description |
| --- | --- |
| `GET /anime/animeunity` | Provider info and route list |
| `GET /anime/animeunity/search/:query` | Search titles |
| `GET /anime/animeunity/info/:id` | Show details and episode list |
| `GET /anime/animeunity/watch/:animeId/:epId` | HLS stream and download link for one episode |

## Ids

| Id | Format | Where it comes from |
| --- | --- | --- |
| Anime id | Numeric site id, for example `2998`. `/info` also accepts the id followed by the slug, as in `2998-one-piece` | `id` of a search result |
| Episode id | `<animeId>/<epId>`, both numeric. Only `epId` is used, so `/watch/<epId>` on its own is accepted too | `id` of an entry in `episodes` from `/info` |

## Overview

### Provider info

`GET /anime/animeunity`

Returns the provider name, a description and the route list. It does not contact the site.

**Example**

```bash
curl "http://localhost:3000/anime/animeunity"
```

```json
{
  "name": "animeunity",
  "description": "Italian anime provider backed by AnimeUnity — search, info and vixcloud streams.",
  "endpoints": [
    "/anime/animeunity/search/:query",
    "/anime/animeunity/info/:id",
    "/anime/animeunity/watch/:animeId/:epId"
  ]
}
```

## Search

### Search titles

`GET /anime/animeunity/search/:query`

Searches the site's archive by title and returns matching shows.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `query` | Yes | Search text |

**Response fields**

The response is `{ "results": [...] }`. When the archive page is missing or has no records, `results` is an empty array.

| Field | Type | Description |
| --- | --- | --- |
| `id` | number | Anime id, used by `/info/:id` |
| `title` | string | Title, falling back to the English and then the Italian title |
| `url` | string | Show page on animeunity.so, `/anime/<id>-<slug>` |
| `image` | string | Poster URL |
| `type` | string | Type as given by the site |
| `score` | string | Site score |

**Example**

```bash
curl "http://localhost:3000/anime/animeunity/search/one%20piece"
```

Response from outside Italy:

```json
{
  "error": "www.animeunity.so responded 403"
}
```

## Details

### Show details

`GET /anime/animeunity/info/:id`

Returns a show's details and every episode. The episode list is read from the site's API in pages of 120 and merged, so long shows need several upstream requests.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Numeric anime id, optionally followed by `-` and the slug, for example `2998` |

**Response fields**

| Field | Type | Description |
| --- | --- | --- |
| `id` | number | Anime id |
| `title` | string | Title, falling back to the English and then the Italian title |
| `url` | string | Show page on animeunity.so |
| `image` | string | Poster URL |
| `description` | string | Plot summary |
| `genres` | string[] | Genre names |
| `status` | string | Airing status as given by the site |
| `totalEpisodes` | number | Episode count reported by the site |
| `episodes` | array | Episodes, each with `id` (episode id for `/watch`), `number` and `url` |

**Example**

```bash
curl "http://localhost:3000/anime/animeunity/info/2998"
```

Response from outside Italy:

```json
{
  "error": "www.animeunity.so responded 403"
}
```

## Streams

### Stream sources

`GET /anime/animeunity/watch/:animeId/:epId`

Looks up the episode's VixCloud player, reads the playlist address, token and expiry from it, and returns one HLS stream. When the player allows 1080p, `&h=1` is added to the playlist URL. If the player exposes a download link, it is returned with the quality read from the file name.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `animeId` | No | Numeric anime id; accepted for readability but not used |
| `epId` | Yes | Numeric episode id |

**Response fields**

The response is `{ "results": { "streams": [...], "downloads": [...] } }`. `downloads` is left out when the player has no download link.

| Field | Type | Description |
| --- | --- | --- |
| `streams[].url` | string | VixCloud HLS playlist URL with `token` and `expires` |
| `streams[].quality` | string | Always `auto` |
| `streams[].isM3U8` | boolean | Always `true` |
| `downloads[].url` | string | Download link from the player |
| `downloads[].quality` | string | For example `1080p`, read from the file name, or `unknown` |

**Example**

```bash
curl "http://localhost:3000/anime/animeunity/watch/<animeId>/<epId>"
```

Use the `id` of an episode from `/info` as `epId`. Response from outside Italy:

```json
{
  "error": "www.animeunity.so responded 403"
}
```

## Errors

| Status | When |
| --- | --- |
| `400` | The anime id is not numeric, or the watch path is not `<animeId>/<epId>` or `<epId>` with numeric ids |
| `404` | `/info`: the site has no show with that id. `/watch`: the episode has no player or the player has no playable stream |
| `502` | animeunity.so or the VixCloud player host answered with an error status (including the Cloudflare `403` outside Italy), did not answer, or returned invalid JSON |

```bash
curl "http://localhost:3000/anime/animeunity/info/one-piece"
```

```json
{
  "error": "Anime id must be numeric (e.g., 2998)"
}
```

```bash
curl "http://localhost:3000/anime/animeunity/watch/2998/abc"
```

```json
{
  "error": "Episode id must look like {animeId}/{epId}"
}
```

The `404` bodies are `{ "error": "Anime not found" }` for `/info` and `{ "error": "No streams found" }` for `/watch`.

## Notes

- Content is in Italian. Titles fall back from the main title to the English and Italian ones.
- The flow is search, then `/info/:id` for the episode list, then `/watch/<episode id>`.
- Stream URLs carry an expiry, so request them again rather than storing them.
- Responses are not cached; every request goes to the site.
