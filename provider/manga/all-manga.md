---
description: Home sections, search, popular lists, genre and author browsing, chapter lists and page images from All Manga.
icon: book
---

# All Manga

All Manga uses the GraphQL API behind [allmanga.to](https://allmanga.to) for home sections, search, popular lists, tags and details. Chapter pages come from the site's reader, which Cooren opens in a real browser, so `/read` takes several seconds. The site rate limits bursts of requests, so space out calls.

{% hint style="info" %}
Base route: `/manga/allmanga`
{% endhint %}

## Routes

| Route | Description |
| --- | --- |
| `GET /manga/allmanga` | Provider status |
| `GET /manga/allmanga/home` | Home sections |
| `GET /manga/allmanga/search` | Search titles |
| `GET /manga/allmanga/latest` | Recently updated titles |
| `GET /manga/allmanga/popular` | Popular titles by period |
| `GET /manga/allmanga/random` | Random titles |
| `GET /manga/allmanga/tags` | Genre, theme and magazine tags |
| `GET /manga/allmanga/genre/:genre` | Titles in one genre |
| `GET /manga/allmanga/author/:author` | Titles by one author |
| `GET /manga/allmanga/detail` | Full details and chapter list |
| `GET /manga/allmanga/read` | Page images of one chapter |
| `GET /manga/allmanga/image/*` | Image proxy |

Every route except the status route and the image proxy returns `{ "status", "success", "data" }`. Errors return the same envelope with a `message` and `data: null`.

### Title cards

Home, search, latest, popular, random, genre and author return titles in this shape:

```json
{
  "id": "Jy8Bxgx4wSFMeNeeS",
  "title": "Sousou no Frieren",
  "englishTitle": "Frieren: Beyond Journey's End",
  "nativeTitle": "葬送のフリーレン",
  "cover": "http://localhost:3000/manga/allmanga/image/wp.youtube-anime.com/aln.youtube-anime.com/mcovers/m_tbs/AMgSkBPBjeoXCyRZn/014.webp?w=250",
  "score": 8.86,
  "availableChapters": {
    "sub": 158,
    "raw": 0
  }
}
```

* `id` is what `/detail` expects.
* `englishTitle` and `nativeTitle` are empty strings when unknown. `cover` and `score` can be `null`.
* `availableChapters.sub` is the number of English chapters, `raw` the number of untranslated ones.
* `cover` already points at [the image route](#image-proxy).

## Home

### Home sections

`GET /manga/allmanga/home`

Returns a list of sections, each with an `id`, a `title` and `items` (title cards). The first four are always requested: `popular-daily` (15 titles), `latest` (26), `manga-<current year>` (26) and `random` (30). They are followed by the tag sections All Manga features on its home page; their `id` is the tag slug, for example `theme:single_parent` or `young_king_ours-magazine`, and there were 11 of them on 2026-09-29. Empty or failed sections are left out.

**Example**

```bash
curl "http://localhost:3000/manga/allmanga/home"
```

The response below shows two of the 15 sections, each trimmed to one item.

```json
{
  "status": 200,
  "success": true,
  "data": {
    "provider": "AllManga",
    "sections": [
      {
        "id": "latest",
        "title": "Latest Updates",
        "items": [
          {
            "id": "vo4J52L2BLzccYnee",
            "title": "Yumemiru Renaissance",
            "englishTitle": "Dreaming Renaissance",
            "nativeTitle": "夢見るルネサンス",
            "cover": "http://localhost:3000/manga/allmanga/image/cdn.myanimelist.net/images/manga/1/228988.webp",
            "score": null,
            "availableChapters": {
              "sub": 1,
              "raw": 0
            }
          }
        ]
      },
      {
        "id": "theme:single_parent",
        "title": "Single Parent",
        "items": [
          {
            "id": "2zX9g6t8eiM8LMMCF",
            "title": "Hotman",
            "englishTitle": "Hotman",
            "nativeTitle": "ホットマン",
            "cover": "http://localhost:3000/manga/allmanga/image/s4.anilist.co/file/anilistcdn/media/manga/cover/large/bx33608-ky15HmpkPEM7.jpg",
            "score": 7.54,
            "availableChapters": {
              "sub": 167,
              "raw": 0
            }
          }
        ]
      }
    ]
  }
}
```

The home page sends about 15 requests to All Manga at once, so it is the route most likely to lose sections to rate limiting. It returns `502` only when every section fails. With Redis enabled, the result is cached for 30 minutes.

## Search and lists

### Search

`GET /manga/allmanga/search`

Searches titles, 26 per page. Adult titles are excluded. Results include titles with no English chapters: for `frieren`, 10 of the 12 results had `availableChapters.sub: 0`, so check it before linking to a reader.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `q` | Yes | | Search text |
| `page` | No | `1` | Page number |

**Example**

```bash
curl "http://localhost:3000/manga/allmanga/search?q=frieren"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "provider": "AllManga",
    "total": 12,
    "page": 1,
    "results": [
      {
        "id": "Jy8Bxgx4wSFMeNeeS",
        "title": "Sousou no Frieren",
        "englishTitle": "Frieren: Beyond Journey's End",
        "nativeTitle": "葬送のフリーレン",
        "cover": "http://localhost:3000/manga/allmanga/image/wp.youtube-anime.com/aln.youtube-anime.com/mcovers/m_tbs/AMgSkBPBjeoXCyRZn/014.webp?w=250",
        "score": 8.86,
        "availableChapters": {
          "sub": 158,
          "raw": 0
        }
      },
      {
        "id": "8sHYEKgu69sPBBuq4",
        "title": "Sousou no Frieren dj: Ippan Saiin Mahou Otsuyu Dark",
        "englishTitle": "",
        "nativeTitle": "",
        "cover": null,
        "score": null,
        "availableChapters": {
          "sub": 0,
          "raw": 0
        }
      }
    ]
  }
}
```

### Latest

`GET /manga/allmanga/latest`

Returns recently updated titles, 26 per page, in the same shape as search. `total` stops at 2600 for this and other broad lists.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `page` | No | `1` | Page number |

**Example**

```bash
curl "http://localhost:3000/manga/allmanga/latest"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "provider": "AllManga",
    "total": 2600,
    "page": 1,
    "results": [
      {
        "id": "GFWNXAFm4YxYWWS6x",
        "title": "Gokusotsu Kraken",
        "englishTitle": "",
        "nativeTitle": "獄卒クラーケン",
        "cover": "http://localhost:3000/manga/allmanga/image/wp.youtube-anime.com/aln.youtube-anime.com/mcovers/m_tbs/Wy8X6C6ydCXAtN4n2/007.jpg?w=250",
        "score": 6.92,
        "availableChapters": {
          "sub": 54,
          "raw": 0
        }
      }
    ]
  }
}
```

### Popular

`GET /manga/allmanga/popular`

Returns the most popular manga over a period. Adult titles are excluded.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `period` | No | `daily` | `daily` (last day), `weekly` (7 days), `monthly` (30 days) or `all` (all time). Any other value returns `400` |
| `size` | No | `20` | Titles per page, 1 to 100. All Manga returned at most 50 even when more were asked for |
| `page` | No | `1` | Page number |

**Example**

```bash
curl "http://localhost:3000/manga/allmanga/popular?period=weekly&size=5"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "provider": "AllManga",
    "total": 500,
    "page": 1,
    "period": "weekly",
    "results": [
      {
        "id": "JJHbe9N2pe94w7t2S",
        "title": "All-Class Awakening: God Slayer",
        "englishTitle": "All-Class Awakening: God Slayer",
        "nativeTitle": "全职觉醒",
        "cover": "http://localhost:3000/manga/allmanga/image/s4.anilist.co/file/anilistcdn/media/manga/cover/medium/b197224-Pvwfb85Sswpl.jpg",
        "score": null,
        "availableChapters": {
          "sub": 156,
          "raw": 0
        }
      }
    ]
  }
}
```

### Random

`GET /manga/allmanga/random`

Returns 30 random manga. Most have no English chapters: in the call below all 30 had `availableChapters.sub: 0`, so this route is better for discovery than for reading.

**Example**

```bash
curl "http://localhost:3000/manga/allmanga/random"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "provider": "AllManga",
    "results": [
      {
        "id": "uabgyPLqMRGCv8kHe",
        "title": "Uma Musume Pispis☆Spispi Gold Ship",
        "englishTitle": "",
        "nativeTitle": "ウマ娘 ピスピス☆スピスピ ゴルシちゃん",
        "cover": "http://localhost:3000/manga/allmanga/image/s4.anilist.co/file/anilistcdn/media/manga/cover/large/bx170592-2IlKhaVeBPRR.jpg",
        "score": null,
        "availableChapters": {
          "sub": 0,
          "raw": 0
        }
      }
    ]
  }
}
```

## Browse

### Tags

`GET /manga/allmanga/tags`

Returns All Manga's manga tags, 100 per page, with the number of titles for each. `type` is `magazine` for magazines and `genre` for everything else, including themes.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `page` | No | `1` | Page number |

**Example**

```bash
curl "http://localhost:3000/manga/allmanga/tags"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "provider": "AllManga",
    "total": 30766,
    "page": 1,
    "tags": [
      {
        "name": "Isekai",
        "slug": "theme:isekai",
        "type": "genre",
        "count": 537
      },
      {
        "name": "Naver Webtoon",
        "slug": "naver_webtoon-magazine",
        "type": "magazine",
        "count": 1457
      }
    ]
  }
}
```

`total` is only filled on page 1; later pages return `total: 0`. The `slug` values do not work with `/genre/:genre`; see below.

### Genre

`GET /manga/allmanga/genre/:genre`

Returns titles in one genre, 26 per page, in the same order and shape as `/latest`.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `genre` | Yes | Genre name exactly as in `genres[].slug` from `/detail`, URL-encoded, for example `Action` or `Slice%20of%20Life`. Case-sensitive: `action` returns nothing |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `page` | No | `1` | Page number |

Some tag names from `/tags` also work (`Isekai` and `Boys' Love` returned results) and others do not (`Borderline H` and `Naver Webtoon` returned nothing). Tag slugs such as `theme:isekai` return an empty list.

**Example**

```bash
curl "http://localhost:3000/manga/allmanga/genre/Isekai"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "provider": "AllManga",
    "total": 2600,
    "page": 1,
    "results": [
      {
        "id": "GFWNXAFm4YxYWWS6x",
        "title": "Gokusotsu Kraken",
        "englishTitle": "",
        "nativeTitle": "獄卒クラーケン",
        "cover": "http://localhost:3000/manga/allmanga/image/wp.youtube-anime.com/aln.youtube-anime.com/mcovers/m_tbs/Wy8X6C6ydCXAtN4n2/007.jpg?w=250",
        "score": 6.92,
        "availableChapters": {
          "sub": 54,
          "raw": 0
        }
      }
    ]
  }
}
```

### Author

`GET /manga/allmanga/author/:author`

Returns titles by one author, 26 per page, in the same shape as search.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `author` | Yes | Author name exactly as in `authors[].slug` from `/detail`, URL-encoded, for example `Yamada%20Kanehito`. Case-sensitive: `yamada kanehito` returns nothing |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `page` | No | `1` | Page number |

**Example**

```bash
curl "http://localhost:3000/manga/allmanga/author/Yamada%20Kanehito"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "provider": "AllManga",
    "total": 1,
    "page": 1,
    "results": [
      {
        "id": "Jy8Bxgx4wSFMeNeeS",
        "title": "Sousou no Frieren",
        "englishTitle": "Frieren: Beyond Journey's End",
        "nativeTitle": "葬送のフリーレン",
        "cover": "http://localhost:3000/manga/allmanga/image/wp.youtube-anime.com/aln.youtube-anime.com/mcovers/m_tbs/AMgSkBPBjeoXCyRZn/014.webp?w=250",
        "score": 8.86,
        "availableChapters": {
          "sub": 158,
          "raw": 0
        }
      }
    ]
  }
}
```

## Details

### Title details

`GET /manga/allmanga/detail`

Returns metadata and the list of English chapters, newest first. Each chapter `id` is what `/read` expects.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `id` | Yes | | All Manga title id from any list, for example `Jy8Bxgx4wSFMeNeeS` |

**Example**

```bash
curl "http://localhost:3000/manga/allmanga/detail?id=Jy8Bxgx4wSFMeNeeS"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "provider": "AllManga",
    "id": "Jy8Bxgx4wSFMeNeeS",
    "title": "Sousou no Frieren",
    "englishTitle": "Frieren: Beyond Journey's End",
    "nativeTitle": "葬送のフリーレン",
    "cover": "http://localhost:3000/manga/allmanga/image/wp.youtube-anime.com/aln.youtube-anime.com/mcovers/m_tbs/AMgSkBPBjeoXCyRZn/014.webp?w=250",
    "description": "The Demon King has been defeated, and the victorious hero party returns home before disbanding. The...",
    "genres": [
      { "genre": "Adventure", "slug": "Adventure" },
      { "genre": "Demons", "slug": "Demons" }
    ],
    "authors": [
      { "author": "Abe Tsukasa", "slug": "Abe Tsukasa" },
      { "author": "Yamada Kanehito", "slug": "Yamada Kanehito" }
    ],
    "status": "Releasing",
    "totalChapters": 158,
    "rawChapters": 0,
    "airedStart": {
      "year": 2020,
      "month": 3,
      "date": 28
    },
    "airedEnd": {},
    "chapterList": [
      {
        "id": "Jy8Bxgx4wSFMeNeeS:sub:147",
        "number": 147,
        "title": "Chapter 147",
        "lang": "sub"
      },
      {
        "id": "Jy8Bxgx4wSFMeNeeS:sub:146",
        "number": 146,
        "title": "Chapter 146",
        "lang": "sub"
      }
    ]
  }
}
```

* Chapter ids have the form `<titleId>:sub:<number>`. Numbers can be decimal (`Jy8Bxgx4wSFMeNeeS:sub:114.5`).
* `genres[].slug` and `authors[].slug` are the values `/genre/:genre` and `/author/:author` expect. `authors` can include names in Japanese as well.
* `description` is plain text with the HTML removed.
* `airedStart` and `airedEnd` are objects with `year`, `month` and `date`; `airedEnd` is `{}` for ongoing series.

## Read

### Chapter pages

`GET /manga/allmanga/read`

Returns the page images of one chapter in reading order. The API loads the chapter in All Manga's reader in a real browser and collects the image URLs, which is why this route is slow.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `id` | Yes | | Chapter id from `/detail`, in the form `<titleId>:<translation>:<number>`, for example `Jy8Bxgx4wSFMeNeeS:sub:147`. `<translation>` defaults to `sub` and `<number>` to `1` when left out, so `id=Jy8Bxgx4wSFMeNeeS` reads chapter 1. Each part may only contain letters, digits, `_`, `.` and `-` |

**Example**

```bash
curl "http://localhost:3000/manga/allmanga/read?id=Jy8Bxgx4wSFMeNeeS:sub:147"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "provider": "AllManga",
    "id": "Jy8Bxgx4wSFMeNeeS:sub:147",
    "pages": [
      {
        "page": 1,
        "img": "http://localhost:3000/manga/allmanga/image/ytimgf.youtube-anime.com/images9/Jy8Bxgx4wSFMeNeeS/147/sub_1760490481/1.png"
      },
      {
        "page": 2,
        "img": "http://localhost:3000/manga/allmanga/image/ytimgf.youtube-anime.com/images9/Jy8Bxgx4wSFMeNeeS/147/sub_1760490481/2.png"
      }
    ]
  }
}
```

{% hint style="warning" %}
`/read` needs Chrome on the server (see [Browser providers](../../getting-started/configuration.md#browser-providers)) and takes roughly 3 to 10 seconds per chapter. During testing the first call took 7 seconds because it also started the browser, and later chapters took 2.5 to 4 seconds. A chapter number that does not exist takes about 7 seconds to return `404`.
{% endhint %}

Requests for the same chapter that arrive while it is loading share one browser load. With Redis enabled, page lists are cached for 24 hours.

## Images

### Image proxy

`GET /manga/allmanga/image/*`

Serves covers and chapter pages. Every image URL in the responses above already points here, so use them as they are. The part after `/image/` is the upstream URL without `https://`; any query string (such as `?w=250` on covers) is passed on. Covers come from several hosts (`wp.youtube-anime.com`, `s4.anilist.co`, `cdn.myanimelist.net`, `cdn.mangaupdates.com`) and pages from `ytimgf.youtube-anime.com`.

```bash
curl -o 1.png "http://localhost:3000/manga/allmanga/image/ytimgf.youtube-anime.com/images9/Jy8Bxgx4wSFMeNeeS/147/sub_1760490481/1.png"
```

The route requests the image with `Referer: https://allmanga.to/`, follows up to 3 redirects and returns the raw bytes with the upstream `Content-Type` (`image/png`, `image/webp`, `image/jpeg`, ...) and `Cache-Control: public, max-age=604800, immutable`. Errors are plain text, not JSON:

| Status | Body | When |
| --- | --- | --- |
| `400` | `Invalid image URL` or `Invalid image host` | The path is not a valid public hostname and path |
| `403` | `Forbidden image host` | The host resolves to a private or local address |
| `404` and other upstream statuses | `Image unavailable` | The upstream returned an error |
| `502` | `Image unavailable` | The upstream could not be reached or did not return an image |

## Errors

| Status | When |
| --- | --- |
| `400` | `q` or `id` is missing, `period` is not `daily`, `weekly`, `monthly` or `all`, or the chapter id has invalid characters (`Invalid chapter id`) |
| `404` | Unknown title id (`Manga not found`) or a chapter with no pages (`Chapter pages not found`) |
| `502` | The All Manga API returned an error, the reader returned incomplete pages, or every home section failed |
| `503` | All Manga is rate limiting (`AllManga is rate limiting requests, try again shortly`), asks for a captcha, or the reader's Cloudflare check did not complete |
| `504` | The All Manga API or the reader timed out |

## Notes

* Flow: any list gives `id`, `/detail?id=` gives `chapterList[].id`, `/read?id=` gives `pages[].img`.
* All Manga rate limits bursts. When 15 searches were sent at once, 10 succeeded and 5 returned `503`. Send requests one after another and retry a `503` after a short pause.
* List and detail routes answered in 0.6 to 0.9 seconds during testing; `/home` took about 2 seconds.
