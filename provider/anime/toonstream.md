---
description: Browse ToonStream's cartoons and anime, including Hindi, Tamil and Telugu dubs, and get HLS sources for movies and episodes.
icon: bolt
---

# ToonStream

ToonStream scrapes [toonstream.us](https://toonstream.us), a catalog of anime and Western cartoons with many Hindi, Tamil, Telugu and English dubs. It covers the home page, search, paged movie and series listings, details with every season and episode, and playable sources. Sources are resolved from the site's embedded players into HLS links, each with a `proxiedUrl` that plays through the API's [stream proxy](../../core/proxy.md).

{% hint style="info" %}
Base route: `/anime/toonstream`
{% endhint %}

## Routes

| Route | Description |
| --- | --- |
| `GET /anime/toonstream` | Lists the provider's routes |
| `GET /anime/toonstream/home` | Home page sections and latest episodes |
| `GET /anime/toonstream/search/:query/:page?` | Search movies and series |
| `GET /anime/toonstream/movies/:page?` | Browse movies |
| `GET /anime/toonstream/movies/info/:slug` | Movie details |
| `GET /anime/toonstream/movies/sources/:slug` | Movie sources |
| `GET /anime/toonstream/series/:page?` | Browse series |
| `GET /anime/toonstream/series/info/:slug` | Series details with seasons and episodes |
| `GET /anime/toonstream/episode/sources/:slug` | Episode sources |

### Response format

Every route except the index wraps its result:

```json
{
  "success": true,
  "served_cache": false,
  "took_ms": "518.95",
  "data": {}
}
```

* `served_cache` is `true` when the result came from Redis. The two sources routes do not include it.
* `took_ms` is a string. The episode sources route does not include it.
* The listing routes (`movies`, `series`) also include `page`.

When nothing could be scraped, for example an unknown slug, the route still answers `200`:

```json
{
  "success": false,
  "took_ms": "575.44",
  "msg": "No Data Scraped!"
}
```

Check `success` rather than the status code.

## Overview

### Provider index

`GET /anime/toonstream`

**Example**

```bash
curl "http://localhost:3000/anime/toonstream"
```

```json
{
  "name": "toonstream-api",
  "version": "0.1",
  "endpoints": [
    "/anime/toonstream/home",
    "/anime/toonstream/search/{query}/{page}",
    "----------------------",
    "/anime/toonstream/movies/{page}",
    "/anime/toonstream/movies/info/{slug}",
    "/anime/toonstream/movies/sources/{slug}",
    "----------------------",
    "/anime/toonstream/series/{page}",
    "/anime/toonstream/series/info/{slug}",
    "/anime/toonstream/episode/sources/{slug}?season={season}&episode={episode}"
  ]
}
```

### Home

`GET /anime/toonstream/home`

Returns the home page sections (`main`), the sidebar sections (`sidebar`) and the latest episodes (`lastEpisodes`). Each section has a `label`, a list of cards and, when the site links one, a `viewMore` URL.

**Example**

```bash
curl "http://localhost:3000/anime/toonstream/home"
```

```json
{
  "success": true,
  "served_cache": false,
  "took_ms": "730.61",
  "data": {
    "main": [
      {
        "label": "Random series",
        "data": [
          {
            "type": "series",
            "title": "Iron Man",
            "slug": "iron-man",
            "poster": "https://image.tmdb.org/t/p/w780/zOTJT7JbzSrMBX2OCGPqUnkQA4y.jpg",
            "url": "https://toonstream.us/series/iron-man",
            "tmdbRating": 7.2
          }
        ]
      },
      {
        "label": "Anime Series",
        "viewMore": "https://toonstream.us/category/anime-series",
        "data": [
          {
            "type": "series",
            "title": "LIAR GAME",
            "slug": "liar-game",
            "poster": "https://image.tmdb.org/t/p/w780/9npgB8fyf7qN8F4ngkuY2eHczxD.jpg",
            "url": "https://toonstream.us/series/liar-game",
            "tmdbRating": 9
          }
        ]
      }
    ],
    "sidebar": [],
    "lastEpisodes": [
      {
        "title": "LIAR GAME",
        "slug": "liar-game-1x23",
        "url": "https://toonstream.us/episode/liar-game-1x23/",
        "epXseason": "1x23",
        "ago": "",
        "thumbnail": "https://image.tmdb.org/t/p/w780/9npgB8fyf7qN8F4ngkuY2eHczxD.jpg"
      }
    ]
  }
}
```

In testing the home page had six `main` sections (Random series, Anime Series, Animated Series, Anime Movies, Animated Movies, Random Movies) of 18 cards each, an empty `sidebar`, and 18 `lastEpisodes`. `ago` was always an empty string. A `lastEpisodes` slug goes straight into [Episode sources](#episode-sources).

## Search

### Search movies and series

`GET /anime/toonstream/search/:query/:page?`

Returns matching movie and series cards. `type` tells you which info route to call next.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `query` | Yes | Search text, URL-encoded |
| `page` | No | Page number, an integer of at least `1`. Defaults to `1` |

**Example**

```bash
curl "http://localhost:3000/anime/toonstream/search/naruto"
```

```json
{
  "success": true,
  "served_cache": false,
  "took_ms": "518.95",
  "data": {
    "query": "naruto",
    "pagination": {
      "current": 1,
      "start": 1,
      "end": 1
    },
    "data": [
      {
        "type": "movie",
        "title": "Naruto Shippuden the Movie",
        "slug": "naruto-shippuden-the-movie",
        "poster": "https://image.tmdb.org/t/p/w780/vDkct38sSFSWJIATlfJw0l3QOIR.jpg",
        "url": "https://toonstream.us/movies/naruto-shippuden-the-movie",
        "tmdbRating": 7.4
      },
      {
        "type": "series",
        "title": "Naruto",
        "slug": "naruto",
        "poster": "https://image.tmdb.org/t/p/w780/vauCEnR7CiyBDzRCeElKkCaXIYu.jpg",
        "url": "https://toonstream.us/series/naruto",
        "tmdbRating": 8.3
      }
    ]
  }
}
```

`pagination.end` is the last page. A search with no matches, or a page past the end, returns `success: true` with an empty `data` array.

## Movies

### Browse movies

`GET /anime/toonstream/movies/:page?`

Returns one page of movie cards, 12 per page, in the site's order.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `page` | No | Page number, an integer of at least `1`. Defaults to `1` |

**Example**

```bash
curl "http://localhost:3000/anime/toonstream/movies/2"
```

```json
{
  "success": true,
  "served_cache": false,
  "page": 2,
  "took_ms": "543.23",
  "data": {
    "pagination": {
      "current": 2,
      "start": 1,
      "end": 35
    },
    "data": [
      {
        "type": "movie",
        "title": "Hotel Transylvania 4: Transformania",
        "slug": "hotel-transylvania-transformania",
        "poster": "https://image.tmdb.org/t/p/w780/teCy1egGQa0y8ULJvlrDHQKnxBL.jpg",
        "url": "https://toonstream.us/movies/hotel-transylvania-transformania",
        "tmdbRating": 7
      }
    ]
  }
}
```

### Movie info

`GET /anime/toonstream/movies/info/:slug`

Returns a movie's details. `casts` holds both directors and voice cast, told apart by their URL (`/director/` or `/cast/`).

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `slug` | Yes | Movie slug from search, home or the movie listing |

**Example**

```bash
curl "http://localhost:3000/anime/toonstream/movies/info/the-last-naruto-the-movie"
```

```json
{
  "success": true,
  "served_cache": false,
  "took_ms": "607.12",
  "data": {
    "title": "The Last: Naruto the Movie",
    "year": "2014",
    "tmdbRating": 7.7,
    "description": "Two years after the events of the Fourth Great Ninja War, the moon that Hagoromo Otsutsuki created long ago to seal away the Gedo Statue begins to descend towards the world...",
    "languages": ["Hindi [Fan Dub]", "English", "Japanese"],
    "qualities": ["1080p FHD", "720p HD", "480p"],
    "duration": "1h 52m",
    "genres": [
      {
        "name": "Action",
        "url": "https://toonstream.us/category/action/",
        "slug": "action"
      }
    ],
    "tags": [
      {
        "name": "The Last: Naruto the Movie",
        "url": "https://toonstream.us/tag/the-last-naruto-the-movie/"
      }
    ],
    "casts": [
      {
        "name": "Hirofumi Masuda",
        "url": "https://toonstream.us/director/hirofumi-masuda/"
      },
      {
        "name": "Akira Ishida",
        "url": "https://toonstream.us/cast/akira-ishida/"
      }
    ]
  }
}
```

### Movie sources

`GET /anime/toonstream/movies/sources/:slug`

Returns every embedded player on the movie page (`embeds`) and the ones the API could turn into direct streams (`sources`). See [Source fields](#source-fields).

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `slug` | Yes | Movie slug |

**Example**

```bash
curl "http://localhost:3000/anime/toonstream/movies/sources/death-note-relight-2-ls-successors"
```

```json
{
  "success": true,
  "took_ms": "2891.69",
  "data": {
    "embeds": [
      "https://rubystm.com/e/cemnmsgo7cig.html",
      "https://filesforever.link/embed/fz3yf7g",
      "https://cloudy.upns.one/#bsfbiz",
      "https://vidmoly.net/embed-q433cwq8wzis.html",
      "https://abyssplayer.com/-H6kB6B-f",
      "https://as-cdn26.top/video/23f8361e3b539eb12866b699f4da27dd",
      "https://turbonewvid.com/t/6aa594fe86626"
    ],
    "sources": [
      {
        "label": "Ruby",
        "type": "hls",
        "url": "https://ozovxg2t3b2c8m8g.streamruby.net/hls2/04/00492/cemnmsgo7cig_,l,n,h,x,.urlset/master.m3u8?t=AahXQ809vwE8Y4za6rBLWedzpgYQactvxFtjOA-Y1hA&s=1790686137&e=32400&v=1871233949&i=103.41&sp=0&fr=cemnmsgo7cig",
        "cover": "https://img.streamruby.com//cemnmsgo7cig_xt.jpg",
        "thumbnail": "https://ozovxg2t3b2c8m8g.streamruby.net/vtt/04/00492/cemnmsgo7cig_sli.vtt",
        "subtitles": {
          "label": "English",
          "url": "https://ozovxg2t3b2c8m8g.streamruby.net/vtt/04/00492/cemnmsgo7cig_eng.vtt"
        },
        "headers": {
          "Origin": "https://rubystm.com",
          "Referer": "https://rubystm.com/"
        },
        "proxiedUrl": "http://localhost:3000/proxy/m3u8-proxy?url=https%3A%2F%2Fozovxg2t3b2c8m8g.streamruby.net%2Fhls2%2F04%2F00492%2Fcemnmsgo7cig_%2Cl%2Cn%2Ch%2Cx%2C.urlset%2Fmaster.m3u8%3Ft%3DAahXQ809vwE8Y4za6rBLWedzpgYQactvxFtjOA-Y1hA%26s%3D1790686137%26e%3D32400%26v%3D1871233949%26i%3D103.41%26sp%3D0%26fr%3Dcemnmsgo7cig&headers=%7B%22Origin%22%3A%22https%3A%2F%2Frubystm.com%22%2C%22Referer%22%3A%22https%3A%2F%2Frubystm.com%2F%22%7D"
      }
    ]
  }
}
```

Some movies only have players the API cannot resolve. For example `the-last-naruto-the-movie` returned five `embeds` and an empty `sources` array.

## Series

### Browse series

`GET /anime/toonstream/series/:page?`

Returns one page of series cards, 12 per page. The response has the same shape as [Browse movies](#browse-movies), with `type: "series"`.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `page` | No | Page number, an integer of at least `1`. Defaults to `1` |

**Example**

```bash
curl "http://localhost:3000/anime/toonstream/series"
```

```json
{
  "success": true,
  "served_cache": false,
  "page": 1,
  "took_ms": "618.33",
  "data": {
    "pagination": {
      "current": 1,
      "start": 1,
      "end": 49
    },
    "data": [
      {
        "type": "series",
        "title": "LIAR GAME",
        "slug": "liar-game",
        "poster": "https://image.tmdb.org/t/p/w780/9npgB8fyf7qN8F4ngkuY2eHczxD.jpg",
        "url": "https://toonstream.us/series/liar-game",
        "tmdbRating": 9
      }
    ]
  }
}
```

### Series info

`GET /anime/toonstream/series/info/:slug`

Returns a series' details and every season with its episodes. Each episode `slug` is what [Episode sources](#episode-sources) takes. `runtime` is the episode length.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `slug` | Yes | Series slug from search, home or the series listing |

**Example**

```bash
curl "http://localhost:3000/anime/toonstream/series/info/grand-blue-dreaming"
```

```json
{
  "success": true,
  "served_cache": false,
  "took_ms": "915.24",
  "data": {
    "title": "Grand Blue Dreaming",
    "year": "2018",
    "tmdbRating": 7.8,
    "description": "A college student joins the local diving club after meeting some rowdy upperclassmen. New adventures in booze and the ocean await.",
    "languages": ["Hindi", "Japanese"],
    "qualities": ["1080p FHD", "720p HD", "480p"],
    "runtime": "24min",
    "genres": [
      {
        "name": "Animation",
        "url": "https://toonstream.us/category/animation/",
        "slug": "animation"
      }
    ],
    "tags": [
      {
        "name": "Grand Blue Dreaming",
        "url": "https://toonstream.us/tag/grand-blue-dreaming/"
      }
    ],
    "casts": [],
    "totalSeasons": 3,
    "totalEpisodes": 29,
    "seasons": [
      {
        "label": "Season 1",
        "season_no": 1,
        "episodes": [
          {
            "episode_no": 1,
            "slug": "grand-blue-dreaming-1x1",
            "title": "S 1 | E 1",
            "epXseason": "1x1",
            "url": "https://toonstream.us/episode/grand-blue-dreaming-1x1/",
            "thumbnail": "https://image.tmdb.org/t/p/w780/81SzeqvZXXQDfHgQ6i0efTz5WAS.jpg"
          }
        ]
      },
      {
        "label": "Season 2",
        "season_no": 2,
        "episodes": [
          {
            "episode_no": 1,
            "slug": "grand-blue-dreaming-2x1",
            "title": "S 2 | E 1",
            "epXseason": "2x1",
            "url": "https://toonstream.us/episode/grand-blue-dreaming-2x1/",
            "thumbnail": "https://image.tmdb.org/t/p/w780/1RdR5KOTpeFWdfFc3tpgeviu4zM.jpg"
          }
        ]
      }
    ]
  }
}
```

### Episode sources

`GET /anime/toonstream/episode/sources/:slug`

Returns the embedded players and resolved sources for one episode. Pass either the full episode slug (`grand-blue-dreaming-2x1`), or the series slug with `season` and `episode`; the API then builds `<slug>-<season>x<episode>`. Both query parameters must be present to be used.

`hash` is the AS-CDN video id found among the embeds, or `null` when there is no AS-CDN player.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `slug` | Yes | Episode slug from series info or home, or a series slug when `season` and `episode` are given |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `season` | No | none | Season number, `0` or more |
| `episode` | No | none | Episode number, `0` or more |

**Example**

```bash
curl "http://localhost:3000/anime/toonstream/episode/sources/grand-blue-dreaming?season=2&episode=1"
```

```json
{
  "success": true,
  "data": {
    "hash": "d5b8786f4dea41ac9a605b5a068a8069",
    "embeds": [
      "https://rubystm.com/e/p1adftjtf97n.html",
      "https://filesforever.link/embed/ahbyz4u",
      "https://vidmoly.net/embed-02zr8qio3qld.html",
      "https://abyssplayer.com/iBblt37Oz",
      "https://as-cdn26.top/video/d5b8786f4dea41ac9a605b5a068a8069",
      "https://turbonewvid.com/t/6a0465c77f60d"
    ],
    "sources": [
      {
        "label": "Ruby",
        "type": "hls",
        "url": "https://rap7c5roebejl0.streamruby.net/hls2/01/00469/p1adftjtf97n_,l,n,h,x,.urlset/master.m3u8?t=3h-ZRtWIhsO0kYl91ftSaGt3TZ-UMAQhETn2_f9n4Bk&s=1790686604&e=32400&v=1871250007&i=103.41&sp=0",
        "cover": "https://img.streamruby.com//p1adftjtf97n_xt.jpg",
        "thumbnail": "https://rap7c5roebejl0.streamruby.net/vtt/01/00469/p1adftjtf97n_sli.vtt",
        "headers": {
          "Origin": "https://rubystm.com",
          "Referer": "https://rubystm.com/"
        },
        "proxiedUrl": "http://localhost:3000/proxy/m3u8-proxy?url=https%3A%2F%2Frap7c5roebejl0.streamruby.net%2Fhls2%2F01%2F00469%2Fp1adftjtf97n_%2Cl%2Cn%2Ch%2Cx%2C.urlset%2Fmaster.m3u8%3Ft%3D3h-ZRtWIhsO0kYl91ftSaGt3TZ-UMAQhETn2_f9n4Bk%26s%3D1790686604%26e%3D32400%26v%3D1871250007%26i%3D103.41%26sp%3D0&headers=%7B%22Origin%22%3A%22https%3A%2F%2Frubystm.com%22%2C%22Referer%22%3A%22https%3A%2F%2Frubystm.com%2F%22%7D"
      }
    ]
  }
}
```

The same episode by its full slug:

```bash
curl "http://localhost:3000/anime/toonstream/episode/sources/grand-blue-dreaming-2x1"
```

### Source fields

| Field | Description |
| --- | --- |
| `label` | The player the source came from: `Ruby`, `Multi Audio` (AS-CDN) or `Turbo` |
| `type` | `hls` or `mp4` |
| `url` | The direct stream. It needs `headers` to play |
| `cover` | Poster image, when the player has one |
| `thumbnail` | Seek-preview thumbnails (WebVTT), when available |
| `subtitles` | One subtitle track, `{ "label", "url" }`, English when available. Omitted when there are none |
| `headers` | Headers the host requires, such as `Origin` and `Referer` |
| `proxiedUrl` | The stream through the API's [stream proxy](../../core/proxy.md) with `headers` attached. Use this in a browser player |

Only Ruby (rubystm, streamruby), AS-CDN and Turbo (emturbovid, turbovidhls, turboviplay) players are resolved. Other players, such as filesforever, vidmoly or abyssplayer, only appear in `embeds`. Ruby playlists usually carry several audio tracks (for example Hindi, Tamil, Telugu and English).

{% hint style="warning" %}
On 2026-09-29 the AS-CDN host (`as-cdn26.top`) answered HTTP 523, so every source observed came from Ruby. The `Multi Audio` source shape above is taken from the source code.
{% endhint %}

## Errors

| Status | When |
| --- | --- |
| `200` with `success: false` | Unknown slug, or ToonStream failed or returned nothing usable (`"msg": "No Data Scraped!"`) |
| `422` | `page` is not an integer of at least `1`, or `season` or `episode` is not a number of at least `0` |

`422` bodies come from Elysia's validation, for example:

```json
{
  "type": "validation",
  "on": "property",
  "property": "root",
  "message": "Expected number to be greater or equal to 1",
  "summary": "Expected number to be greater or equal to 1",
  "found": 0
}
```

## Notes

* Typical flow: `search` or `home` → `movies/info/:slug` or `series/info/:slug` → `movies/sources/:slug` or `episode/sources/:slug` → play `proxiedUrl`.
* When a player host fails with a 5xx error or times out, the API skips that host for 5 minutes. The first request that hits a down host can take several seconds (about 7 seconds in testing); later ones are fast.
* With Redis enabled (`REDIS_URL`), results are cached: home, search and listings for 12 hours, movie info for 14 days, series info for 3 days, the list of embeds on a page for 1 day, and resolved sources for 2 hours (AS-CDN), 8 hours (Ruby) or 12 hours (Turbo). Requests that fail are not cached.
* `tmdbRating` is a number; `year` is a string.
