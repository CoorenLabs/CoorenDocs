---
description: Anime discovery and metadata from the AniList GraphQL API, with extra artwork from ani.zip.
icon: magnifying-glass
---

# AniList

AniList queries the [AniList GraphQL API](https://graphql.anilist.co) for home feeds, search and full anime detail pages. Spotlight entries and detail pages also pull banner and logo artwork from [ani.zip](https://api.ani.zip). It returns metadata only; pass the AniList `id` to a streaming provider to play something.

{% hint style="info" %}
Base route: `/meta/anilist`
{% endhint %}

## Routes

| Route | Description |
| --- | --- |
| `GET /meta/anilist` | Lists the routes |
| `GET /meta/anilist/home` | Spotlight plus six home sections |
| `GET /meta/anilist/:category` | One home section |
| `GET /meta/anilist/anime/:id` | Full metadata for one anime |
| `GET /meta/anilist/search/:query` | Search anime titles |

Every data route wraps its result the same way:

```json
{
  "success": true,
  "served_cache": false,
  "took_ms": "616.81",
  "data": {}
}
```

`served_cache` is `true` when the result came from Redis, and `took_ms` is a string. Errors return `{ "success": false, "error": "..." }`.

## Home

### Home page

`GET /meta/anilist/home`

Returns every home section in one object: `spotlight` (up to 10 currently airing, trending titles, with descriptions and airing countdowns) and six lists of 20 titles each.

**Example**

```bash
curl "http://localhost:3000/meta/anilist/home"
```

<details>

<summary>Example response</summary>

```json
{
  "success": true,
  "served_cache": false,
  "took_ms": "1171.25",
  "data": {
    "spotlight": [
      {
        "id": 21,
        "title": "ONE PIECE",
        "poster": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx21-ELSYx3yMPcKM.jpg",
        "banner": "https://artworks.thetvdb.com/banners/v4/series/81797/backgrounds/616009a8bd688.jpg",
        "logo": "https://artworks.thetvdb.com/banners/series/81797/icons/5eee2a0450c0e.jpg",
        "description": "Gold Roger was known as the Pirate King, the strongest and most infamous being to have sailed the Grand Line...",
        "season": "Fall 1999",
        "episode": "1181",
        "timeLeft": "96d 1h",
        "status": "Releasing",
        "type": "TV"
      }
    ],
    "recently-added": [
      {
        "id": 178789,
        "title": "Mushoku Tensei: Jobless Reincarnation Season 3",
        "poster": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx178789-hNXjKFzUq7mk.jpg",
        "banner": "https://s4.anilist.co/file/anilistcdn/media/anime/banner/178789-9nHWmoRLlcLu.jpg",
        "logo": "",
        "type": "TV",
        "episodes": 14,
        "status": "Completed"
      }
    ],
    "popular-anime": [
      {
        "id": 16498,
        "title": "Attack on Titan",
        "poster": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx16498-buvcRTBx4NSm.jpg",
        "banner": "https://s4.anilist.co/file/anilistcdn/media/anime/banner/16498-8jpFCOcDmneX.jpg",
        "logo": "",
        "type": "TV",
        "episodes": 25,
        "status": "Completed"
      }
    ],
    "popular-movies": [
      {
        "id": 20954,
        "title": "A Silent Voice",
        "poster": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx20954-sYRfE5jQRtSB.jpg",
        "banner": "https://s4.anilist.co/file/anilistcdn/media/anime/banner/20954-f30bHMXa5Qoe.jpg",
        "logo": "",
        "type": "MOVIE",
        "episodes": 1,
        "status": "Completed"
      }
    ],
    "seasonal-anime": [
      {
        "id": 178789,
        "title": "Mushoku Tensei: Jobless Reincarnation Season 3",
        "poster": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx178789-hNXjKFzUq7mk.jpg",
        "banner": "https://s4.anilist.co/file/anilistcdn/media/anime/banner/178789-9nHWmoRLlcLu.jpg",
        "logo": "",
        "type": "TV",
        "episodes": 14,
        "status": "Completed"
      }
    ],
    "anime-of-all-time": [
      {
        "id": 114129,
        "title": "Gintama: THE VERY FINAL",
        "poster": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx114129-RLgSuh6YbeYx.jpg",
        "banner": "https://s4.anilist.co/file/anilistcdn/media/anime/banner/114129-ZsLDkdwaYeJY.jpg",
        "logo": "",
        "type": "MOVIE",
        "episodes": 1,
        "status": "Completed"
      }
    ],
    "coming-soon": [
      {
        "id": 195516,
        "title": "The Apothecary Diaries Season 3",
        "poster": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx195516-MJpUZlOberqH.jpg",
        "banner": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx195516-MJpUZlOberqH.jpg",
        "logo": "",
        "type": "TV",
        "episodes": null,
        "status": "Not yet released"
      }
    ]
  }
}
```

</details>

In `spotlight`, `episode` is the number of the next episode to air (or the total episode count when nothing is scheduled) and `timeLeft` is the countdown to it, empty when nothing is scheduled.

### Home category

`GET /meta/anilist/:category`

Returns a single section of the home page as an array. It reads the same data as `/home`.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `category` | Yes | One of the values below |

| `category` | Contents |
| --- | --- |
| `spotlight` | Up to 10 currently airing titles sorted by trending, same fields as the home spotlight |
| `recently-added` | 20 anime sorted by trending |
| `popular-anime` | 20 anime sorted by popularity |
| `popular-movies` | 20 anime movies sorted by popularity |
| `seasonal-anime` | 20 anime from the current season, sorted by popularity |
| `anime-of-all-time` | 20 anime sorted by score |
| `coming-soon` | 20 not yet released anime sorted by popularity |

Any other value returns `422`.

**Example**

```bash
curl "http://localhost:3000/meta/anilist/popular-movies"
```

```json
{
  "success": true,
  "served_cache": false,
  "took_ms": "919.00",
  "data": [
    {
      "id": 20954,
      "title": "A Silent Voice",
      "poster": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx20954-sYRfE5jQRtSB.jpg",
      "banner": "https://s4.anilist.co/file/anilistcdn/media/anime/banner/20954-f30bHMXa5Qoe.jpg",
      "logo": "",
      "type": "MOVIE",
      "episodes": 1,
      "status": "Completed"
    },
    {
      "id": 21519,
      "title": "Your Name.",
      "poster": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx21519-SUo3ZQuCbYhJ.png",
      "banner": "https://s4.anilist.co/file/anilistcdn/media/anime/banner/21519-1ayMXgNlmByb.jpg",
      "logo": "",
      "type": "MOVIE",
      "episodes": 1,
      "status": "Completed"
    }
  ]
}
```

Titles that have not aired yet can have `"episodes": null`, as in `coming-soon`:

```json
{
  "id": 195516,
  "title": "The Apothecary Diaries Season 3",
  "poster": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx195516-MJpUZlOberqH.jpg",
  "banner": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx195516-MJpUZlOberqH.jpg",
  "logo": "",
  "type": "TV",
  "episodes": null,
  "status": "Not yet released"
}
```

## Details

### Anime details

`GET /meta/anilist/anime/:id`

Returns full metadata for one anime: titles, artwork, description, airing info, scores, studios, trailer, tags (top 10), relations, up to 25 characters with Japanese voice actors, up to 12 recommendations and AniList's streaming episode list.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Numeric AniList anime id, for example `154587` |

**Example**

```bash
curl "http://localhost:3000/meta/anilist/anime/154587"
```

```json
{
  "success": true,
  "served_cache": false,
  "took_ms": "616.81",
  "data": {
    "id": 154587,
    "title": "Frieren: Beyond Journey’s End",
    "titleRomaji": "Sousou no Frieren",
    "titleNative": "葬送のフリーレン",
    "poster": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx154587-qQTzQnEJJ3oB.jpg",
    "logo": "https://artworks.thetvdb.com/banners/v4/series/424536/clearlogo/696a802a5aa22.png",
    "color": "#bbf1a1",
    "banner": "https://artworks.thetvdb.com/banners/v4/series/424536/backgrounds/64e6cbe29d9c0.jpg",
    "description": "The adventure is over but life goes on for an elf mage just beginning to learn what living is all about...",
    "season": "Fall 2023",
    "episode": "28",
    "totalEpisodes": 28,
    "duration": 24,
    "timeLeft": "",
    "status": "Completed",
    "type": "TV",
    "genres": ["Adventure", "Drama", "Fantasy"],
    "averageScore": 91,
    "meanScore": 91,
    "popularity": 484687,
    "favourites": 56089,
    "source": "MANGA",
    "countryOfOrigin": "JP",
    "startDate": { "year": 2023, "month": 9, "day": 29 },
    "endDate": { "year": 2024, "month": 3, "day": 22 },
    "studios": [
      { "name": "Toho", "isAnimationStudio": false }
    ],
    "trailer": { "id": "tR8YH0G67Rk", "site": "youtube" },
    "synonyms": ["Frieren at the Funeral"],
    "tags": [
      { "name": "Travel", "rank": 96 }
    ],
    "relations": [
      {
        "relationType": "ADAPTATION",
        "id": 118586,
        "title": "Frieren: Beyond Journey’s End",
        "poster": "https://s4.anilist.co/file/anilistcdn/media/manga/cover/large/bx118586-CXKgWikBFQgS.jpg",
        "format": "MANGA",
        "status": "Releasing",
        "episodes": null,
        "type": "MANGA"
      }
    ],
    "characters": [
      {
        "role": "MAIN",
        "id": 176754,
        "name": "Frieren",
        "image": "https://s4.anilist.co/file/anilistcdn/character/large/b176754-PCnpqIOkjhFk.png",
        "voiceActors": [
          {
            "id": 112215,
            "name": "Atsumi Tanezaki",
            "image": "https://s4.anilist.co/file/anilistcdn/staff/large/n112215-kfABGD8W2YSJ.jpg"
          }
        ]
      }
    ],
    "recommendations": [
      {
        "id": 21827,
        "title": "Violet Evergarden",
        "poster": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx21827-ubzq619ZA2E9.png",
        "format": "TV",
        "status": "Completed",
        "episodes": 13,
        "averageScore": 85,
        "season": "WINTER",
        "seasonYear": 2018
      }
    ],
    "streamingEpisodes": [
      {
        "title": "Episode 1 - The Journey's End",
        "thumbnail": "https://img1.ak.crunchyroll.com/i/spire1-tmb/0a540e01e5d00a998a241d1ea23181ad1695991424_full.jpg"
      }
    ]
  }
}
```

For a title that is still airing, `episode` is the next episode number, `timeLeft` counts down to it and `totalEpisodes` can be `null`. For `/meta/anilist/anime/21` (One Piece) the response had `"episode": "1181"`, `"timeLeft": "96d 1h"`, `"totalEpisodes": null` and `"status": "Releasing"`.

## Search

### Search anime

`GET /meta/anilist/search/:query`

Searches anime titles, sorted by popularity and then score.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `query` | Yes | Search text, URL-encoded |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `page` | No | `1` | Page number |
| `perPage` | No | `20` | Results per page. AniList caps this at `50` |

**Example**

```bash
curl "http://localhost:3000/meta/anilist/search/frieren?perPage=2"
```

```json
{
  "success": true,
  "served_cache": false,
  "took_ms": "305.73",
  "data": {
    "pageInfo": {
      "total": 5000,
      "currentPage": 1,
      "lastPage": 2500,
      "hasNextPage": true,
      "perPage": 2
    },
    "results": [
      {
        "id": 154587,
        "title": "Frieren: Beyond Journey’s End",
        "poster": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx154587-qQTzQnEJJ3oB.jpg",
        "banner": "https://s4.anilist.co/file/anilistcdn/media/anime/banner/154587-ivXNJ23SM1xB.jpg",
        "logo": "",
        "format": "TV",
        "status": "Completed",
        "episodes": 28,
        "averageScore": 91,
        "season": "FALL",
        "seasonYear": 2023,
        "color": "#bbf1a1"
      },
      {
        "id": 182255,
        "title": "Frieren: Beyond Journey’s End Season 2",
        "poster": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx182255-butzrqd4I0aC.jpg",
        "banner": "https://s4.anilist.co/file/anilistcdn/media/anime/banner/182255-wyHvp6zJbWsO.jpg",
        "logo": "",
        "format": "TV",
        "status": "Completed",
        "episodes": 10,
        "averageScore": 87,
        "season": "WINTER",
        "seasonYear": 2026,
        "color": "#5dc9f1"
      }
    ]
  }
}
```

A search with no matches still returns `200` with `"results": []`.

## Errors

| Status | When |
| --- | --- |
| `400` | `/anime/:id` with an id that is not a number: `{ "success": false, "error": "Invalid AniList id" }` |
| `404` | `/anime/:id` with an id AniList does not know: `{ "success": false, "error": "Not Found." }` |
| `422` | Unknown `category`, or `page` or `perPage` that is not a number |
| `429` | AniList is rate limiting the server |
| `502` | AniList could not be reached or returned a server error |

## Notes

* Responses are cached in Redis when it is enabled: the home page and categories for 12 hours, detail pages for 24 hours and searches for 6 hours.
* Search `status` values are formatted (`Completed`, `Releasing`, `Not yet released`), while `season` in search results and recommendations is AniList's raw value (`FALL`). The home spotlight and detail pages format it as `Fall 2023`.
* `banner` falls back to AniList's banner and then to the cover image. `logo` is only filled for the spotlight and detail pages, where ani.zip artwork is fetched; elsewhere it is an empty string.
* `pageInfo.total` comes straight from AniList and can change with `perPage`. Use `hasNextPage` to decide whether to request another page.
