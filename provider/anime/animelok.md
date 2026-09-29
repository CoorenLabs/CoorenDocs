---
description: Get episode lists and sub and dub embed players from Animelok by AniList id.
icon: circle-play
---

# Animelok

Animelok reads [animelok.cc](https://animelok.cc) and is keyed by AniList id, so you can use it directly with ids from [Miruro](miruro.md) or any AniList search. It returns paged episode lists with titles, air dates and thumbnails, and the embed players for each episode, split into sub and dub. The players are iframe pages, not direct video files, so load them in an `<iframe>`.

{% hint style="info" %}
Base route: `/anime/animelok`
{% endhint %}

## Routes

| Route | Description |
| --- | --- |
| `GET /anime/animelok` | Lists the provider's routes |
| `GET /anime/animelok/episodes/:anilistId` | Episode list, paged |
| `GET /anime/animelok/stream/:anilistId/:episode` | Sub and dub embed players for one episode |

### Response format

Successful responses wrap the result:

```json
{
  "success": true,
  "served_cache": false,
  "took_ms": "189.84",
  "data": {}
}
```

`served_cache` is `true` when the result came from Redis, and `took_ms` is a string. Errors are `{ "success": false, "error": "..." }`.

## Overview

### Provider index

`GET /anime/animelok`

**Example**

```bash
curl "http://localhost:3000/anime/animelok"
```

```json
{
  "name": "animelok",
  "version": "1.0",
  "description": "Anime provider backed by animelok.cc — episode lists and embed stream sources keyed by AniList ID.",
  "endpoints": [
    "/anime/animelok/episodes/:anilistId?page=&pageSize=",
    "/anime/animelok/stream/:anilistId/:episode?quality="
  ]
}
```

## Episodes

### Episode list

`GET /anime/animelok/episodes/:anilistId`

Returns one page of episodes and the total episode count. Pages start at `0`. A page past the end returns an empty `episodes` array.

`name` is the episode title, or `Episode <n>` when there is none; `title` is the raw title and can be `null`. `thumbnail`, `image` and `img` hold the same URL and are left out when the episode has no image.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `anilistId` | Yes | AniList id, for example `154587` for Frieren |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `page` | No | `0` | Zero-based page number. Negative values are treated as `0` |
| `pageSize` | No | `30` | Episodes per page. `0` uses the default |

**Example**

```bash
curl "http://localhost:3000/anime/animelok/episodes/154587?page=1&pageSize=2"
```

```json
{
  "success": true,
  "served_cache": false,
  "took_ms": "1278.28",
  "data": {
    "episodes": [
      {
        "number": 3,
        "name": "Killing Magic",
        "title": "Killing Magic",
        "airdate": "2023-09-29",
        "thumbnail": "https://artworks.thetvdb.com/banners/v4/episode/9993738/screencap/6516dbf654876.jpg",
        "image": "https://artworks.thetvdb.com/banners/v4/episode/9993738/screencap/6516dbf654876.jpg",
        "img": "https://artworks.thetvdb.com/banners/v4/episode/9993738/screencap/6516dbf654876.jpg"
      },
      {
        "number": 4,
        "name": "The Land Where Souls Rest",
        "title": "The Land Where Souls Rest",
        "airdate": "2023-09-29",
        "thumbnail": "https://artworks.thetvdb.com/banners/v4/episode/9993739/screencap/6516dc5905a4d.jpg",
        "image": "https://artworks.thetvdb.com/banners/v4/episode/9993739/screencap/6516dc5905a4d.jpg",
        "img": "https://artworks.thetvdb.com/banners/v4/episode/9993739/screencap/6516dc5905a4d.jpg"
      }
    ],
    "total": 28,
    "page": 1
  }
}
```

The whole list is fetched once and paged by the API, so changing `page` or `pageSize` does not cost another upstream request when Redis is enabled.

## Streams

### Episode players

`GET /anime/animelok/stream/:anilistId/:episode`

Returns the embed players for one episode, split into `sub` and `dub`. Each track has:

| Field | Description |
| --- | --- |
| `embeds` | Player URLs with a `server` name, for example `HD-1`, `HD-2` or `multi` |
| `best` | The first embed URL, or `null` when the track has none |
| `hash` | The AS-CDN video id when one of the embeds is an AS-CDN player, otherwise `null` |
| `servers` | Always an empty array |

Players that are not specific to sub or dub, such as multi-language players, appear in both tracks. `episodeNumber` echoes the episode you asked for.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `anilistId` | Yes | AniList id |
| `episode` | Yes | Episode number |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `quality` | No | `1080p` | Echoed back as `preferQuality`. It does not filter the players |

**Example**

```bash
curl "http://localhost:3000/anime/animelok/stream/154587/1"
```

```json
{
  "success": true,
  "served_cache": false,
  "took_ms": "315.36",
  "data": {
    "episodeNumber": 1,
    "preferQuality": "1080p",
    "sub": {
      "hash": null,
      "servers": [],
      "embeds": [
        {
          "url": "https://flixcloud.cc/e/rjyccuaq8h5n?v=1",
          "server": "HD-1"
        },
        {
          "url": "https://flixcloud.cc/e/rjyccuaq8h5n?v=2",
          "server": "HD-2"
        }
      ],
      "best": "https://flixcloud.cc/e/rjyccuaq8h5n?v=1"
    },
    "dub": {
      "hash": null,
      "servers": [],
      "embeds": [
        {
          "url": "https://flixcloud.cc/e/rjyccuaq8h5n?v=1&a=1",
          "server": "HD-1"
        },
        {
          "url": "https://flixcloud.cc/e/rjyccuaq8h5n?v=2&a=1",
          "server": "HD-2"
        }
      ],
      "best": "https://flixcloud.cc/e/rjyccuaq8h5n?v=1&a=1"
    }
  }
}
```

For some titles the tracks also carry multi-language players and an AS-CDN hash. The `sub` track of Naruto (`/anime/animelok/stream/20/1`), trimmed:

```json
{
  "hash": "36660e59856b4de58a219bcf4e27eba3",
  "servers": [],
  "embeds": [
    {
      "url": "https://flixcloud.cc/e/bo2qdw3m3kjf?v=1",
      "server": "HD-1"
    },
    {
      "url": "https://animesalt.cx/as-cdn/clone/multi-lang-plyr.php?data=W3sibGFuZ3VhZ2UiOiJIaW5kaSIsImxpbmsiOi...",
      "server": "multi"
    },
    {
      "url": "https://as-cdn26.top/video/36660e59856b4de58a219bcf4e27eba3",
      "server": "multi"
    }
  ],
  "best": "https://flixcloud.cc/e/bo2qdw3m3kjf?v=1"
}
```

## Errors

| Status | When |
| --- | --- |
| `404` | Animelok has no such AniList id or episode, or no players for it (`{ "success": false, "error": "Not found on animelok" }`) |
| `422` | `page` or `pageSize` is not a number |
| `502` | Animelok failed or returned an unexpected response; `error` holds the reason, for example `animelok responded 503 for /watch/154587` |

## Notes

* Only the AniList id is needed; there is no search route. Find ids with [Miruro](miruro.md) search, or read them from AnimePahe's `externalLinks`.
* For the Reanime players (`flixcloud.cc`), the dub URL is the sub URL with `a=1` added; both tracks point at the same video id.
* With Redis enabled (`REDIS_URL`), episode lists are cached for 6 hours and players for 2 hours. Empty results are not cached.
