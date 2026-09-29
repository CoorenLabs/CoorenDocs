---
description: AniList-backed anime search, discovery and details, plus episode lists and multi-provider streams from Miruro.
icon: compass
---

# Miruro

Miruro combines two sources. Search, discovery lists, the airing schedule and details (characters, relations, recommendations) come from the [AniList](https://anilist.co) GraphQL API, with banner and logo artwork from ani.zip. Episode lists with skip times and the streams for each episode come from the [miruro.to](https://www.miruro.to) catalog API, which aggregates several sites, such as AnimePahe, KickAssAnime, Anikoto, Aniwaves and Icarus, into sub, dub and soft-sub tracks. Every id is an AniList id, and every stream and subtitle carries a `proxiedUrl` that plays through the API's [stream proxy](../../core/proxy.md).

{% hint style="info" %}
Base route: `/anime/miruro`
{% endhint %}

## Routes

| Route | Description |
| --- | --- |
| `GET /anime/miruro` | Lists the provider's routes |
| `GET /anime/miruro/search/:query` | Search anime |
| `GET /anime/miruro/suggestions/:query` | Short search results for autocomplete |
| `GET /anime/miruro/filter` | Browse by genre, tag, year, season, format and status |
| `GET /anime/miruro/spotlight` | Ten trending titles with descriptions and artwork |
| `GET /anime/miruro/trending` | Trending anime |
| `GET /anime/miruro/popular` | Most popular anime of all time |
| `GET /anime/miruro/upcoming` | Most anticipated anime that have not aired |
| `GET /anime/miruro/recent` | Anime airing now, newest first |
| `GET /anime/miruro/schedule` | Next episodes to air |
| `GET /anime/miruro/info/:id` | Full details for one anime |
| `GET /anime/miruro/characters/:id` | Characters and voice actors, paged |
| `GET /anime/miruro/relations/:id` | Sequels, prequels, source material and other related media |
| `GET /anime/miruro/recommendations/:id` | Community recommendations, paged |
| `GET /anime/miruro/episodes/:id` | Episode list with skip times |
| `GET /anime/miruro/watch/:provider/:anilistId/:category/:slug` | Streams and subtitles for one episode |

Successful responses are the result object itself. Errors are `{ "message": "..." }`.

Paged routes return this envelope around their list:

| Field | Description |
| --- | --- |
| `page` | Current page |
| `perPage` | Items per page |
| `total` | Total matches as reported by AniList. AniList caps this, so large result sets show `5000` |
| `hasNextPage` | Whether another page exists |

## Overview

### Provider index

`GET /anime/miruro`

**Example**

```bash
curl "http://localhost:3000/anime/miruro"
```

```json
{
  "name": "miruro",
  "version": "1.0",
  "description": "Anime provider backed by AniList metadata and the Miruro catalog API.",
  "endpoints": [
    "/search/:query?page=&perPage= → Search anime",
    "/suggestions/:query           → Lightweight search suggestions",
    "/filter                       → Advanced filter",
    "/spotlight                    → Spotlight anime",
    "/trending                     → Trending anime",
    "/popular                      → Popular anime",
    "/upcoming                     → Upcoming anime",
    "/recent                       → Recently updated/airing anime",
    "/schedule                     → Anime schedule",
    "/info/:id                     → Full anime info",
    "/characters/:id               → Anime characters",
    "/relations/:id                → Anime relations",
    "/recommendations/:id          → Anime recommendations",
    "/episodes/:id                 → Anime episodes",
    "/watch/:provider/:anilistId/:category/:slug → Watch stream sources ('all' matches any provider/category)"
  ]
}
```

## Search

### Search anime

`GET /anime/miruro/search/:query`

Returns anime matching the query, best match first. Adult titles are excluded. Each result is an AniList media object; the discovery routes below return the same object.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `query` | Yes | Search text, URL-encoded |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `page` | No | `1` | Page number |
| `perPage` | No | `20` | Results per page |

**Example**

```bash
curl "http://localhost:3000/anime/miruro/search/frieren?perPage=2"
```

```json
{
  "page": 1,
  "perPage": 2,
  "total": 5000,
  "hasNextPage": true,
  "results": [
    {
      "id": 154587,
      "title": {
        "romaji": "Sousou no Frieren",
        "english": "Frieren: Beyond Journey’s End",
        "native": "葬送のフリーレン"
      },
      "coverImage": {
        "large": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/medium/bx154587-qQTzQnEJJ3oB.jpg",
        "extraLarge": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx154587-qQTzQnEJJ3oB.jpg"
      },
      "bannerImage": "https://s4.anilist.co/file/anilistcdn/media/anime/banner/154587-ivXNJ23SM1xB.jpg",
      "format": "TV",
      "season": "FALL",
      "seasonYear": 2023,
      "episodes": 28,
      "duration": 24,
      "status": "FINISHED",
      "averageScore": 91,
      "meanScore": 91,
      "popularity": 484687,
      "favourites": 56089,
      "genres": ["Adventure", "Drama", "Fantasy"],
      "source": "MANGA",
      "countryOfOrigin": "JP",
      "isAdult": false,
      "studios": {
        "nodes": [
          {
            "name": "MADHOUSE",
            "isAnimationStudio": true
          }
        ]
      },
      "nextAiringEpisode": null,
      "startDate": {
        "year": 2023,
        "month": 9,
        "day": 29
      },
      "endDate": {
        "year": 2024,
        "month": 3,
        "day": 22
      }
    }
  ]
}
```

A search with no matches returns `200` with `"total": 0` and an empty `results` array.

### Suggestions

`GET /anime/miruro/suggestions/:query`

Returns up to 8 short results for a search box. `title` is the English title, or the romaji title when there is no English one.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `query` | Yes | Search text, URL-encoded |

**Example**

```bash
curl "http://localhost:3000/anime/miruro/suggestions/frieren"
```

```json
{
  "suggestions": [
    {
      "id": 154587,
      "title": "Frieren: Beyond Journey’s End",
      "title_romaji": "Sousou no Frieren",
      "poster": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/medium/bx154587-qQTzQnEJJ3oB.jpg",
      "format": "TV",
      "status": "FINISHED",
      "year": 2023,
      "episodes": 28
    },
    {
      "id": 209939,
      "title": "Sousou no Frieren 3rd Season",
      "title_romaji": "Sousou no Frieren 3rd Season",
      "poster": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/medium/bx209939-g2Njkml1rheG.jpg",
      "format": "TV",
      "status": "NOT_YET_RELEASED",
      "year": 2027,
      "episodes": null
    }
  ]
}
```

## Discovery

### Filter

`GET /anime/miruro/filter`

Browses AniList with any combination of filters. Adult titles are excluded. Results use the same media object as [Search anime](#search-anime).

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `genre` | No | none | An AniList genre, for example `Action` |
| `tag` | No | none | An AniList tag, for example `Time Skip` |
| `year` | No | none | Season year, for example `2024` |
| `season` | No | none | `WINTER`, `SPRING`, `SUMMER` or `FALL` (any case) |
| `format` | No | none | `TV`, `TV_SHORT`, `MOVIE`, `SPECIAL`, `OVA`, `ONA` or `MUSIC` (any case) |
| `status` | No | none | `FINISHED`, `RELEASING`, `NOT_YET_RELEASED`, `CANCELLED` or `HIATUS` (any case) |
| `sort` | No | `POPULARITY_DESC` | `SCORE_DESC`, `POPULARITY_DESC`, `TRENDING_DESC`, `START_DATE_DESC`, `FAVOURITES_DESC` or `UPDATED_AT_DESC`. Must be upper case; any other value falls back to the default |
| `page` | No | `1` | Page number |
| `perPage` | No | `20` | Results per page. `per_page` also works |

An unknown `season`, `format` or `status` returns `400`.

**Example**

```bash
curl "http://localhost:3000/anime/miruro/filter?genre=Action&year=2024&season=spring&format=tv&status=finished&sort=SCORE_DESC&perPage=2"
```

```json
{
  "page": 1,
  "perPage": 2,
  "total": 5000,
  "hasNextPage": true,
  "results": [
    {
      "id": 136804,
      "title": {
        "romaji": "Kono Subarashii Sekai ni Shukufuku wo! 3",
        "english": "KONOSUBA -God's blessing on this wonderful world! 3",
        "native": "この素晴らしい世界に祝福を！３"
      },
      "coverImage": {
        "large": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/medium/bx136804-7FVftG67FPBc.jpg",
        "extraLarge": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx136804-7FVftG67FPBc.jpg"
      },
      "bannerImage": "https://s4.anilist.co/file/anilistcdn/media/anime/banner/136804-yHC8D64UTiRA.jpg",
      "format": "TV",
      "season": "SPRING",
      "seasonYear": 2024,
      "episodes": 11,
      "duration": 24,
      "status": "FINISHED",
      "averageScore": 82,
      "meanScore": 82,
      "popularity": 181076,
      "favourites": 4327,
      "genres": ["Action", "Adventure", "Comedy", "Ecchi", "Fantasy"],
      "source": "LIGHT_NOVEL",
      "countryOfOrigin": "JP",
      "isAdult": false,
      "studios": {
        "nodes": [
          {
            "name": "Drive",
            "isAnimationStudio": true
          }
        ]
      },
      "nextAiringEpisode": null,
      "startDate": {
        "year": 2024,
        "month": 4,
        "day": 10
      },
      "endDate": {
        "year": 2024,
        "month": 6,
        "day": 19
      }
    }
  ]
}
```

```bash
curl "http://localhost:3000/anime/miruro/filter?season=autumn"
```

```json
{
  "message": "Invalid season; expected one of WINTER, SPRING, SUMMER, FALL"
}
```

### Spotlight

`GET /anime/miruro/spotlight`

Returns ten titles sorted by trending, then popularity, for a hero banner. Each result is the search media object plus `description` and `logo`. When ani.zip has artwork, `bannerImage` is replaced by its fanart and `logo` is its clear logo; otherwise `logo` is missing.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `allowAll` | No | `false` | `true` includes titles from every country. By default only Japanese titles are returned |

**Example**

```bash
curl "http://localhost:3000/anime/miruro/spotlight"
```

```json
{
  "results": [
    {
      "id": 178789,
      "title": {
        "romaji": "Mushoku Tensei III: Isekai Ittara Honki Dasu",
        "english": "Mushoku Tensei: Jobless Reincarnation Season 3",
        "native": "無職転生Ⅲ ～異世界行ったら本気だす～"
      },
      "coverImage": {
        "large": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/medium/bx178789-hNXjKFzUq7mk.jpg",
        "extraLarge": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx178789-hNXjKFzUq7mk.jpg"
      },
      "bannerImage": "https://artworks.thetvdb.com/banners/series/371310/backgrounds/5fef65bea1666.jpg",
      "format": "TV",
      "season": "SUMMER",
      "seasonYear": 2026,
      "episodes": 14,
      "duration": 24,
      "status": "FINISHED",
      "averageScore": 86,
      "meanScore": 86,
      "popularity": 164503,
      "favourites": 6162,
      "genres": ["Adventure", "Drama", "Ecchi", "Fantasy"],
      "source": "LIGHT_NOVEL",
      "countryOfOrigin": "JP",
      "isAdult": false,
      "studios": {
        "nodes": [
          {
            "name": "Studio Bind",
            "isAnimationStudio": true
          }
        ]
      },
      "nextAiringEpisode": null,
      "startDate": {
        "year": 2026,
        "month": 7,
        "day": 4
      },
      "endDate": {
        "year": 2026,
        "month": 9,
        "day": 27
      },
      "description": "The third season of <i>Mushoku Tensei: Isekai Ittara Honki Dasu</i>.",
      "logo": "https://artworks.thetvdb.com/banners/v4/series/371310/clearlogo/611e10939affb.png"
    }
  ]
}
```

`description` can contain HTML tags such as `<i>` and `<br>`.

### Trending, popular, upcoming and recent

`GET /anime/miruro/trending`

`GET /anime/miruro/popular`

`GET /anime/miruro/upcoming`

`GET /anime/miruro/recent`

Paged lists of search media objects. Adult titles are excluded.

| Route | What it returns |
| --- | --- |
| `trending` | Sorted by current trending score |
| `popular` | Sorted by total AniList users |
| `upcoming` | Titles not yet released, sorted by popularity |
| `recent` | Titles currently airing, newest start date first |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `page` | No | `1` | Page number |
| `perPage` | No | `20` | Results per page |
| `allowAll` | No | `false` | `true` includes titles from every country. By default only Japanese titles are returned |

**Example**

```bash
curl "http://localhost:3000/anime/miruro/trending?perPage=1"
```

```json
{
  "page": 1,
  "perPage": 1,
  "total": 5000,
  "hasNextPage": true,
  "results": [
    {
      "id": 178789,
      "title": {
        "romaji": "Mushoku Tensei III: Isekai Ittara Honki Dasu",
        "english": "Mushoku Tensei: Jobless Reincarnation Season 3",
        "native": "無職転生Ⅲ ～異世界行ったら本気だす～"
      },
      "coverImage": {
        "large": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/medium/bx178789-hNXjKFzUq7mk.jpg",
        "extraLarge": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx178789-hNXjKFzUq7mk.jpg"
      },
      "bannerImage": "https://s4.anilist.co/file/anilistcdn/media/anime/banner/178789-9nHWmoRLlcLu.jpg",
      "format": "TV",
      "season": "SUMMER",
      "seasonYear": 2026,
      "episodes": 14,
      "duration": 24,
      "status": "FINISHED",
      "averageScore": 86,
      "meanScore": 86,
      "popularity": 164503,
      "favourites": 6162,
      "genres": ["Adventure", "Drama", "Ecchi", "Fantasy"],
      "source": "LIGHT_NOVEL",
      "countryOfOrigin": "JP",
      "isAdult": false,
      "studios": {
        "nodes": [
          {
            "name": "Studio Bind",
            "isAnimationStudio": true
          }
        ]
      },
      "nextAiringEpisode": null,
      "startDate": {
        "year": 2026,
        "month": 7,
        "day": 4
      },
      "endDate": {
        "year": 2026,
        "month": 9,
        "day": 27
      }
    }
  ]
}
```

On 2026-09-29 the first results were Shingeki no Kyojin (`16498`) for `popular`, Kusuriya no Hitorigoto 3rd Season (`195516`) for `upcoming` and Umayuru: Full Gate! (`215835`) for `recent`.

### Schedule

`GET /anime/miruro/schedule`

Returns the next episodes to air, soonest first. Each result is the search media object plus `next_episode`, `airingAt` (Unix seconds) and `timeUntilAiring` (seconds).

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `page` | No | `1` | Page number |
| `perPage` | No | `20` | Results per page |

**Example**

```bash
curl "http://localhost:3000/anime/miruro/schedule?perPage=2"
```

```json
{
  "page": 1,
  "perPage": 2,
  "total": 5000,
  "hasNextPage": true,
  "results": [
    {
      "id": 207809,
      "title": {
        "romaji": "Sora wa Akai Kawa no Hotori",
        "english": "Red River",
        "native": "天は赤い河のほとり"
      },
      "coverImage": {
        "large": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/medium/bx207809-cpS7CAyjN7iP.jpg",
        "extraLarge": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx207809-cpS7CAyjN7iP.jpg"
      },
      "bannerImage": null,
      "format": "TV",
      "season": "SUMMER",
      "seasonYear": 2026,
      "episodes": 24,
      "duration": 23,
      "status": "RELEASING",
      "averageScore": 62,
      "meanScore": 63,
      "popularity": 10993,
      "favourites": 109,
      "genres": ["Action", "Adventure", "Drama", "Fantasy", "Romance"],
      "source": "MANGA",
      "countryOfOrigin": "JP",
      "isAdult": false,
      "studios": {
        "nodes": [
          {
            "name": "Tatsunoko Production",
            "isAnimationStudio": true
          }
        ]
      },
      "nextAiringEpisode": {
        "episode": 13,
        "airingAt": 1790699700,
        "timeUntilAiring": 13413
      },
      "startDate": {
        "year": 2026,
        "month": 7,
        "day": 8
      },
      "endDate": {
        "year": null,
        "month": null,
        "day": null
      },
      "next_episode": 13,
      "airingAt": 1790699700,
      "timeUntilAiring": 13413
    }
  ]
}
```

## Details

### Anime info

`GET /anime/miruro/info/:id`

Returns everything AniList has on one anime in one response: titles, description, artwork, tags, studios, trailer, the first 25 characters (with Japanese voice actors) and staff, relations, the top 10 recommendations, external and streaming links, and score and status statistics. `bannerImage` and `logo` come from ani.zip when it has them. `idMal` is the MyAnimeList id.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | AniList id |

**Example**

```bash
curl "http://localhost:3000/anime/miruro/info/154587"
```

<details>

<summary>Response (trimmed)</summary>

```json
{
  "id": 154587,
  "idMal": 52991,
  "title": {
    "romaji": "Sousou no Frieren",
    "english": "Frieren: Beyond Journey’s End",
    "native": "葬送のフリーレン"
  },
  "description": "The adventure is over but life goes on for an elf mage just beginning to learn what living is all about...",
  "coverImage": {
    "large": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/medium/bx154587-qQTzQnEJJ3oB.jpg",
    "extraLarge": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx154587-qQTzQnEJJ3oB.jpg",
    "color": "#bbf1a1"
  },
  "bannerImage": "https://artworks.thetvdb.com/banners/v4/series/424536/backgrounds/64e6cbe29d9c0.jpg",
  "format": "TV",
  "season": "FALL",
  "seasonYear": 2023,
  "episodes": 28,
  "duration": 24,
  "status": "FINISHED",
  "averageScore": 91,
  "meanScore": 91,
  "popularity": 484687,
  "favourites": 56089,
  "trending": 23,
  "genres": ["Adventure", "Drama", "Fantasy"],
  "tags": [
    {
      "name": "Travel",
      "rank": 96,
      "isMediaSpoiler": false
    }
  ],
  "source": "MANGA",
  "countryOfOrigin": "JP",
  "isAdult": false,
  "hashtag": "#フリーレン #frieren",
  "synonyms": ["Frieren at the Funeral", "장송의 프리렌"],
  "siteUrl": "https://anilist.co/anime/154587",
  "trailer": {
    "id": "tR8YH0G67Rk",
    "site": "youtube",
    "thumbnail": "https://i.ytimg.com/vi/tR8YH0G67Rk/hqdefault.jpg"
  },
  "studios": {
    "nodes": [
      {
        "id": 245,
        "name": "Toho",
        "isAnimationStudio": false,
        "siteUrl": "https://anilist.co/studio/245"
      }
    ]
  },
  "nextAiringEpisode": null,
  "startDate": {
    "year": 2023,
    "month": 9,
    "day": 29
  },
  "endDate": {
    "year": 2024,
    "month": 3,
    "day": 22
  },
  "characters": {
    "edges": [
      {
        "role": "MAIN",
        "node": {
          "id": 176754,
          "name": {
            "full": "Frieren",
            "native": "フリーレン"
          },
          "image": {
            "large": "https://s4.anilist.co/file/anilistcdn/character/large/b176754-PCnpqIOkjhFk.png"
          }
        },
        "voiceActors": [
          {
            "id": 112215,
            "name": {
              "full": "Atsumi Tanezaki",
              "native": "種﨑敦美"
            },
            "image": {
              "large": "https://s4.anilist.co/file/anilistcdn/staff/large/n112215-kfABGD8W2YSJ.jpg"
            },
            "languageV2": "Japanese"
          }
        ]
      }
    ]
  },
  "staff": {
    "edges": [
      {
        "role": "Original Story",
        "node": {
          "id": 122202,
          "name": {
            "full": "Kanehito Yamada",
            "native": "山田鐘人"
          },
          "image": {
            "large": "https://s4.anilist.co/file/anilistcdn/staff/large/n122202-lxoDudr9vaul.jpg"
          }
        }
      }
    ]
  },
  "relations": {
    "edges": [
      {
        "relationType": "SOURCE",
        "node": {
          "id": 118586,
          "title": {
            "romaji": "Sousou no Frieren",
            "english": "Frieren: Beyond Journey’s End",
            "native": "葬送のフリーレン"
          },
          "coverImage": {
            "large": "https://s4.anilist.co/file/anilistcdn/media/manga/cover/medium/bx118586-CXKgWikBFQgS.jpg"
          },
          "format": "MANGA",
          "type": "MANGA",
          "status": "RELEASING",
          "episodes": null,
          "meanScore": 87
        }
      }
    ]
  },
  "recommendations": {
    "nodes": [
      {
        "rating": 1191,
        "mediaRecommendation": {
          "id": 21827,
          "title": {
            "romaji": "Violet Evergarden",
            "english": "Violet Evergarden",
            "native": "ヴァイオレット・エヴァーガーデン"
          },
          "coverImage": {
            "large": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/medium/bx21827-ubzq619ZA2E9.png"
          },
          "format": "TV",
          "episodes": 13,
          "status": "FINISHED",
          "meanScore": 85,
          "averageScore": 85
        }
      }
    ]
  },
  "externalLinks": [
    {
      "url": "https://frieren-anime.jp/",
      "site": "Official Site",
      "type": "INFO"
    }
  ],
  "streamingEpisodes": [
    {
      "title": "Episode 1 - The Journey's End",
      "thumbnail": "https://img1.ak.crunchyroll.com/i/spire1-tmb/0a540e01e5d00a998a241d1ea23181ad1695991424_full.jpg",
      "url": "http://www.crunchyroll.com/watch/G2XU04E88/the-journeys-end",
      "site": "Crunchyroll"
    }
  ],
  "stats": {
    "scoreDistribution": [
      {
        "score": 10,
        "amount": 1159
      }
    ],
    "statusDistribution": [
      {
        "status": "CURRENT",
        "amount": 73311
      }
    ]
  },
  "logo": "https://artworks.thetvdb.com/banners/v4/series/424536/clearlogo/696a802a5aa22.png"
}
```

</details>

### Characters

`GET /anime/miruro/characters/:id`

Returns characters, main roles first, with their details and voice actors in every language.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | AniList id |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `page` | No | `1` | Page number |
| `perPage` | No | `25` | Characters per page |

**Example**

```bash
curl "http://localhost:3000/anime/miruro/characters/154587?perPage=1"
```

```json
{
  "page": 1,
  "perPage": 1,
  "total": 500,
  "hasNextPage": true,
  "characters": [
    {
      "role": "MAIN",
      "node": {
        "id": 176754,
        "name": {
          "full": "Frieren",
          "native": "フリーレン",
          "userPreferred": "Frieren"
        },
        "image": {
          "large": "https://s4.anilist.co/file/anilistcdn/character/large/b176754-PCnpqIOkjhFk.png",
          "medium": "https://s4.anilist.co/file/anilistcdn/character/medium/b176754-PCnpqIOkjhFk.png"
        },
        "description": "Frieren is the protagonist of _Sousou no Frieren_ and [Fern](https://anilist.co/character/183965)'s ...",
        "gender": "Female",
        "dateOfBirth": {
          "year": null,
          "month": null,
          "day": null
        },
        "age": "1000+",
        "favourites": 22493,
        "siteUrl": "https://anilist.co/character/176754"
      },
      "voiceActors": [
        {
          "id": 112215,
          "name": {
            "full": "Atsumi Tanezaki",
            "native": "種﨑敦美"
          },
          "image": {
            "large": "https://s4.anilist.co/file/anilistcdn/staff/large/n112215-kfABGD8W2YSJ.jpg"
          },
          "languageV2": "Japanese"
        }
      ]
    }
  ]
}
```

Character descriptions are AniList Markdown.

### Relations

`GET /anime/miruro/relations/:id`

Returns every related anime, manga and music entry with its AniList relation type. Frieren's six relations were `SOURCE`, `SEQUEL`, `SIDE_STORY` (twice), `CHARACTER` and `OTHER`.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | AniList id |

**Example**

```bash
curl "http://localhost:3000/anime/miruro/relations/154587"
```

```json
{
  "id": 154587,
  "title": {
    "romaji": "Sousou no Frieren",
    "english": "Frieren: Beyond Journey’s End"
  },
  "relations": [
    {
      "relationType": "SOURCE",
      "node": {
        "id": 118586,
        "title": {
          "romaji": "Sousou no Frieren",
          "english": "Frieren: Beyond Journey’s End",
          "native": "葬送のフリーレン"
        },
        "coverImage": {
          "large": "https://s4.anilist.co/file/anilistcdn/media/manga/cover/medium/bx118586-CXKgWikBFQgS.jpg"
        },
        "bannerImage": "https://s4.anilist.co/file/anilistcdn/media/manga/banner/118586-R1c7mc72oPvS.jpg",
        "format": "MANGA",
        "type": "MANGA",
        "status": "RELEASING",
        "episodes": null,
        "chapters": null,
        "meanScore": 87,
        "averageScore": 87,
        "popularity": 85727,
        "startDate": {
          "year": 2020,
          "month": 4,
          "day": 28
        }
      }
    }
  ]
}
```

### Recommendations

`GET /anime/miruro/recommendations/:id`

Returns community recommendations, highest rated first. `rating` is the number of users who agreed with the recommendation.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | AniList id |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `page` | No | `1` | Page number |
| `perPage` | No | `10` | Recommendations per page |

**Example**

```bash
curl "http://localhost:3000/anime/miruro/recommendations/154587?perPage=1"
```

```json
{
  "page": 1,
  "perPage": 1,
  "total": 500,
  "hasNextPage": true,
  "recommendations": [
    {
      "rating": 1191,
      "mediaRecommendation": {
        "id": 21827,
        "title": {
          "romaji": "Violet Evergarden",
          "english": "Violet Evergarden",
          "native": "ヴァイオレット・エヴァーガーデン"
        },
        "coverImage": {
          "large": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/medium/bx21827-ubzq619ZA2E9.png",
          "extraLarge": "https://s4.anilist.co/file/anilistcdn/media/anime/cover/large/bx21827-ubzq619ZA2E9.png"
        },
        "bannerImage": "https://s4.anilist.co/file/anilistcdn/media/anime/banner/21827-ROucgYiiiSpR.jpg",
        "format": "TV",
        "episodes": 13,
        "status": "FINISHED",
        "meanScore": 85,
        "averageScore": 85,
        "popularity": 549894,
        "genres": ["Drama", "Fantasy", "Slice of Life"],
        "startDate": {
          "year": 2018
        }
      }
    }
  ]
}
```

## Episodes

### Episode list

`GET /anime/miruro/episodes/:id`

Returns every episode from the Miruro catalog, with synopsis, thumbnail, air date, length in seconds, a filler flag and skip times for openings and endings. The top-level `id` is Miruro's own id for the anime.

Each episode `id` is a ready-made watch path: append it to `/anime/miruro/` to get every stream for that episode, for example `/anime/miruro/watch/all/154587/all/1`.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | AniList id |

**Example**

```bash
curl "http://localhost:3000/anime/miruro/episodes/154587"
```

```json
{
  "id": "o2Eqmv4w0JYgQJWQFi9CDDdH-PgffNpm",
  "anilistId": 154587,
  "episodes": [
    {
      "id": "watch/all/154587/all/1",
      "number": 1,
      "title": "The Journey's End",
      "description": "The world celebrates the Demon King's defeat at the hands of the Hero and his companions. Now that their great adventure is over, what will Frieren the mage do next?",
      "image": "https://image.tmdb.org/t/p/original/7SfqQVmW125ugsktoRVTnRXpAkd.jpg",
      "airDate": "2023-09-29",
      "duration": 1800,
      "filler": false,
      "skipTimes": [
        {
          "type": "op",
          "start": 3.221,
          "end": 93.221
        },
        {
          "type": "ed",
          "start": 1417.135,
          "end": 1507.135
        }
      ]
    }
  ]
}
```

`skipTimes[].type` values seen include `op`, `ed` and `mixed_op`. `start` and `end` are in seconds.

## Streams

### Watch

`GET /anime/miruro/watch/:provider/:anilistId/:category/:slug`

Returns the streams and subtitles for one episode, grouped by track (`sub`, `dub`, `ssub`) and then by source site. Use `all` for `provider` or `category` to get everything; the episode list's `id` does this.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `provider` | Yes | A source site such as `animepahe`, `kickassanime`, `anikoto`, `aniwaves` or `icarus`, or `all` for every site |
| `anilistId` | Yes | AniList id |
| `category` | Yes | `sub` (hard-subbed), `dub`, `ssub` (soft subs as separate subtitle files), or `all` |
| `slug` | Yes | Anything that ends with the episode number: `1`, `episode-1` and `frieren-episode-1` all mean episode 1 |

**Example**

```bash
curl "http://localhost:3000/anime/miruro/watch/kickassanime/154587/dub/frieren-episode-1"
```

```json
{
  "anilistId": 154587,
  "episode": 1,
  "tracks": [
    {
      "track": "dub",
      "providers": [
        {
          "provider": "kickassanime",
          "subtitles": [
            {
              "language": "en",
              "label": "English",
              "file": "https://subst.krussdomi.com/67d0c079169c31976b8d7970/67e0fa641967208e6c20882f.vtt",
              "format": "vtt",
              "default": true,
              "proxiedUrl": "http://localhost:3000/proxy/fetch?url=https%3A%2F%2Fsubst.krussdomi.com%2F67d0c079169c31976b8d7970%2F67e0fa641967208e6c20882f.vtt&headers=%7B%22Referer%22%3A%22https%3A%2F%2Fkrussdomi.com%2F%22%2C%22Origin%22%3A%22https%3A%2F%2Fkrussdomi.com%22%7D"
            }
          ],
          "thumbnails": {
            "vtt": "https://subst.krussdomi.com/67d0c079169c31976b8d7970/preview-4EC9L.vtt",
            "sprite": null
          },
          "downloads": [],
          "servers": [
            {
              "server": "Vid",
              "headers": {
                "Referer": "https://krussdomi.com/"
              },
              "streams": [
                {
                  "url": "https://hls.krussdomi.com/manifest/67d0c079169c31976b8d7970/master.m3u8",
                  "format": "hls",
                  "quality": null,
                  "resolution": null,
                  "codec": null,
                  "embed": null,
                  "proxiedUrl": "http://localhost:3000/proxy/m3u8-proxy?url=https%3A%2F%2Fhls.krussdomi.com%2Fmanifest%2F67d0c079169c31976b8d7970%2Fmaster.m3u8&headers=%7B%22Referer%22%3A%22https%3A%2F%2Fkrussdomi.com%2F%22%2C%22Origin%22%3A%22https%3A%2F%2Fkrussdomi.com%22%7D"
                }
              ],
              "embed": {
                "url": "https://krussdomi.com/cat-player/player?id=67d0c079169c31976b8d7970&source=vidstream&ln=en-US"
              }
            }
          ]
        }
      ]
    }
  ]
}
```

With `all/154587/all/1` the same episode returned three tracks: `sub` from AnimePahe and Aniwaves; `dub` from AnimePahe, KickAssAnime, Anikoto, Aniwaves and Icarus; and `ssub` from KickAssAnime, Anikoto and Icarus.

**Fields**

| Field | Description |
| --- | --- |
| `tracks[].track` | `sub`, `dub` or `ssub` |
| `providers[].provider` | The source site |
| `providers[].subtitles` | Subtitle files with `language`, `label`, `file`, `format`, `default` and a `proxiedUrl` through `/proxy/fetch`. Empty for hard-subbed sources |
| `providers[].thumbnails` | Seek-preview thumbnails (`vtt`, `sprite`), or `null` |
| `providers[].downloads` | Download links, for example `{ "url", "quality", "size", "fansub" }` for AnimePahe. Often empty |
| `servers[].headers` | Headers the host requires |
| `servers[].streams` | Playable streams. Can be empty when the server only offers an `embed` player |
| `streams[].format` | `hls` for HLS playlists. Every stream seen in testing was `hls`; any other value is proxied as MP4 |
| `streams[].quality`, `resolution`, `codec` | Filled in by some sources (AnimePahe gives `"360p"`, `{ "width": 640, "height": 360 }`, `"h264"`), otherwise `null` |
| `streams[].proxiedUrl` | The stream through the [stream proxy](../../core/proxy.md): `/proxy/m3u8-proxy` for HLS, `/proxy/mp4-proxy` otherwise |
| `servers[].embed` | The source's own player page, `{ "url" }`, or `null` |

The proxied links carry the server's `headers`. When a server only lists a `Referer`, the API adds the matching `Origin` too, as in the example above.

## Errors

| Status | When |
| --- | --- |
| `400` | `filter`: `season`, `format` or `status` is not one of the allowed values |
| `404` | `info`, `characters`, `relations`, `recommendations`: AniList has no anime with that id, or the AniList request failed. `episodes`: the id is not in the Miruro catalog, or the catalog request failed. `watch`: the episode, provider or category has no streams, `slug` does not end in a number, or the catalog request failed |
| `500` | `search`, `suggestions`, `filter`, `spotlight`, `trending`, `popular`, `upcoming`, `recent`, `schedule`: the AniList request failed |

Each error has a fixed message, for example `{ "message": "Anime info not found" }`, `{ "message": "Episodes not found" }`, `{ "message": "Stream sources not found" }` or `{ "message": "Search failed" }`.

## Notes

* All ids are AniList ids. Other providers keyed by AniList id, such as [Animelok](animelok.md) and [Anivexa](anivexa.md), take the same ids.
* AniList allows about 30 requests per minute per IP (its `X-RateLimit-Limit` header read `30` on 2026-09-29) and answers `429` beyond that. Miruro does not pass the `429` on: while the server is rate limited, the AniList-backed list routes return `500` and the id routes return `404`, even for valid ids. Wait a minute and retry. `episodes` and `watch` use the Miruro catalog and are not affected.
* Typical flow: `search` → `episodes/:id` → `watch/...` using the episode `id` → play a `proxiedUrl`.
* With Redis enabled (`REDIS_URL`), the lookup from AniList id to Miruro catalog id is cached for 7 days. Other Miruro routes are not cached.
