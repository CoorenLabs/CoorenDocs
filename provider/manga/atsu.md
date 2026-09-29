---
description: Home carousels, ranked lists, filtered browsing, chapter lists and page images from Atsu, with a separate 18+ catalog.
icon: book-bookmark
---

# Atsu

Atsu uses the JSON API of [atsu.moe](https://atsu.moe), a reader for manga, manhwa, manhua and OEL comics, and its search index for the explore routes. It is good for home carousels, ranked lists (trending, most bookmarked, top rated) and filtered browsing. Adult titles are kept apart: the regular routes return only non-adult titles, and every list route has an `/adult` twin that returns only adult titles.

{% hint style="info" %}
Base route: `/manga/atsu`
{% endhint %}

## Routes

| Route | Description |
| --- | --- |
| `GET /manga/atsu` | Provider status |
| `GET /manga/atsu/home` | All home carousels |
| `GET /manga/atsu/trending` | Trending titles |
| `GET /manga/atsu/most-bookmarked` | Most bookmarked titles in a time frame |
| `GET /manga/atsu/hot-updates` | Recently updated titles |
| `GET /manga/atsu/top-rated` | Top rated titles |
| `GET /manga/atsu/popular` | Popular titles |
| `GET /manga/atsu/recently-added` | Recently added titles |
| `GET /manga/atsu/explore` | Filter by genres, types and statuses |
| `GET /manga/atsu/genre/:slug` | Titles in one genre |
| `GET /manga/atsu/author/:slug` | Titles by one author or artist |
| `GET /manga/atsu/filters` | Genre, type and status values for filtering |
| `GET /manga/atsu/detail/:id` | Full details and chapter list |
| `GET /manga/atsu/info/:id` | Lightweight chapter list |
| `GET /manga/atsu/read` | Page images of one chapter |
| `GET /manga/atsu/image/*` | Image proxy |
| `GET /manga/atsu/adult/home` | Adult home carousels |
| `GET /manga/atsu/adult/trending` | Adult trending titles |
| `GET /manga/atsu/adult/most-bookmarked` | Adult most bookmarked titles |
| `GET /manga/atsu/adult/hot-updates` | Adult recently updated titles |
| `GET /manga/atsu/adult/top-rated` | Adult top rated titles |
| `GET /manga/atsu/adult/popular` | Adult popular titles |
| `GET /manga/atsu/adult/recently-added` | Adult recently added titles |
| `GET /manga/atsu/adult/explore` | Filter adult titles |
| `GET /manga/atsu/adult/genre/:slug` | Adult titles in one genre |
| `GET /manga/atsu/adult/author/:slug` | All titles by one author, adult included |

Every route except the status route and the image proxy returns `{ "status", "success", "data" }`. Errors return the same envelope with a `message` and `data: null`.

### List items

Home, ranked lists, explore, genre and author routes all return titles in this shape. Every image URL already points at [the image route](#image-proxy).

```json
{
  "id": "v8Kbg",
  "title": "Sakamoto Days",
  "thumbnail": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/n6oo6UH4grMnLAow.jpg",
  "images": {
    "small": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/n6oo6UH4grMnLAow-small.avif",
    "medium": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/n6oo6UH4grMnLAow-medium.avif",
    "large": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/n6oo6UH4grMnLAow-large.avif"
  },
  "type": "Manga",
  "isAdult": false
}
```

`type` is one of `Manga`, `Manwha` (Atsu's spelling), `Manhua`, `OEL` or `Other`. In explore, genre and author results, `images.large` is the same poster as `thumbnail`.

## Home

### Home carousels

`GET /manga/atsu/home`

Returns every carousel from Atsu's home page as an object keyed by section. Each section has a `title` and 20 `items`. The keys seen during testing were `trending-carousel`, `most-bookmarked`, `hot-updates`, `recently-updated`, `top-rated`, `popular` and `recently-added`.

**Example**

```bash
curl "http://localhost:3000/manga/atsu/home"
```

The response below shows two of the seven sections, each trimmed to one item.

```json
{
  "status": 200,
  "success": true,
  "data": {
    "trending-carousel": {
      "title": "Trending",
      "items": [
        {
          "id": "v8Kbg",
          "title": "Sakamoto Days",
          "thumbnail": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/n6oo6UH4grMnLAow.jpg",
          "images": {
            "small": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/n6oo6UH4grMnLAow-small.avif",
            "medium": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/n6oo6UH4grMnLAow-medium.avif",
            "large": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/n6oo6UH4grMnLAow-large.avif"
          },
          "type": "Manga",
          "isAdult": false
        }
      ]
    },
    "top-rated": {
      "title": "Top Rated",
      "items": [
        {
          "id": "CM0wz",
          "title": "BERSERK",
          "thumbnail": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/CjNJG6RuRRmsbQJ6.jpg",
          "images": {
            "small": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/CjNJG6RuRRmsbQJ6-small.avif",
            "medium": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/CjNJG6RuRRmsbQJ6-medium.avif",
            "large": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/CjNJG6RuRRmsbQJ6-large.avif"
          },
          "type": "Manga",
          "isAdult": false
        }
      ]
    }
  }
}
```

## Ranked lists

### Trending, hot updates, top rated, popular, recently added

* `GET /manga/atsu/trending`
* `GET /manga/atsu/hot-updates`
* `GET /manga/atsu/top-rated`
* `GET /manga/atsu/popular`
* `GET /manga/atsu/recently-added`

Each returns one page of 40 titles from the matching Atsu list. `hot-updates` is Atsu's recently updated list. A page past the end returns an empty `items` array.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `page` | No | `0` | Page number, starting at `0`. Negative or invalid values become `0` |
| `types` | No | `Manga,Manwha,Manhua,OEL` | Comma-separated types to include: `Manga`, `Manwha`, `Manhua`, `OEL`, `Other`. Case-sensitive; an unknown value returns `400` |

**Example**

```bash
curl "http://localhost:3000/manga/atsu/trending?types=Manga"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "page": 0,
    "items": [
      {
        "id": "v8Kbg",
        "title": "Sakamoto Days",
        "thumbnail": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/n6oo6UH4grMnLAow.jpg",
        "images": {
          "small": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/n6oo6UH4grMnLAow-small.avif",
          "medium": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/n6oo6UH4grMnLAow-medium.avif",
          "large": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/n6oo6UH4grMnLAow-large.avif"
        },
        "type": "Manga",
        "isAdult": false
      }
    ]
  }
}
```

### Most bookmarked

`GET /manga/atsu/most-bookmarked`

Returns one page of 40 titles ranked by bookmarks in a time frame. Same response as the other ranked lists.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `page` | No | `0` | Page number, starting at `0` |
| `timeframe` | No | `7` | Days to rank over: `1`, `7` or `30`. A non-numeric value returns `400`; other numbers such as `5` or `365` returned an empty list |
| `types` | No | `Manga,Manwha,Manhua,OEL` | Sent to Atsu, but this list returned mixed types even with `types=Manga` during testing |

**Example**

```bash
curl "http://localhost:3000/manga/atsu/most-bookmarked?timeframe=30"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "page": 0,
    "items": [
      {
        "id": "YMv6",
        "title": "The Warrior's Ballad",
        "thumbnail": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/74nHdvEdSb3ivehi.jpg",
        "images": {
          "small": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/74nHdvEdSb3ivehi-small.avif",
          "medium": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/74nHdvEdSb3ivehi-medium.avif",
          "large": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/74nHdvEdSb3ivehi-large.avif"
        },
        "type": "Manwha",
        "isAdult": false
      }
    ]
  }
}
```

## Browse

### Filters

`GET /manga/atsu/filters`

Returns the values the explore and genre routes accept. Use a genre's `slug` (a numeric id) in `genres` or `/genre/:slug`, and a type's or status's `slug` in `types` or `statuses`.

**Example**

```bash
curl "http://localhost:3000/manga/atsu/filters"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "genres": [
      { "name": "Action", "slug": "39" },
      { "name": "Adult", "slug": "46" }
    ],
    "types": [
      { "name": "Manga", "slug": "Manga" },
      { "name": "Manhwa", "slug": "Manwha" },
      { "name": "Manhua", "slug": "Manhua" },
      { "name": "OEL", "slug": "OEL" },
      { "name": "Other", "slug": "Other" }
    ],
    "statuses": [
      { "name": "Ongoing", "slug": "Ongoing" },
      { "name": "Completed", "slug": "Completed" },
      { "name": "Hiatus", "slug": "Hiatus" },
      { "name": "Canceled", "slug": "Canceled" }
    ]
  }
}
```

The 21 genres returned on 2026-09-29 were: Action `39`, Adult `46`, Adventure `37`, Boys Love `180`, Comedy `6`, Drama `31`, Fantasy `36`, Girls Love `4`, Hentai `10`, Historical `45`, Horror `44`, Martial Arts `29`, Mystery `32`, Psychological `18`, Romance `9`, Sci-Fi `1`, Slice of Life `7`, Smut `41`, Supernatural `22`, Thriller `19`, Tragedy `5`.

### Explore

`GET /manga/atsu/explore`

Returns one page of 40 titles matching the filters, sorted by views. Unknown filter values return an empty list rather than an error.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `genres` | No | | Comma-separated genre ids from `/filters`, for example `39,36`. A title must have every listed genre |
| `types` | No | `Manga,Manwha,Manhua,OEL` | Comma-separated types. A title must be one of them. `Other` titles only appear when you ask for `Other` |
| `statuses` | No | any | Comma-separated statuses: `Ongoing`, `Completed`, `Hiatus`, `Canceled`. A title must have one of them |
| `page` | No | `0` | Page number, starting at `0` |

**Example**

```bash
curl "http://localhost:3000/manga/atsu/explore?genres=39,36&types=Manwha&statuses=Completed"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "page": 0,
    "items": [
      {
        "id": "fX0YJ",
        "title": "The Greatest Estate Developer",
        "thumbnail": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/QX9jgNKJH6iFeeVd.jpg",
        "images": {
          "small": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/QX9jgNKJH6iFeeVd-small.avif",
          "medium": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/QX9jgNKJH6iFeeVd-medium.avif",
          "large": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/QX9jgNKJH6iFeeVd.jpg"
        },
        "type": "Manwha",
        "isAdult": false
      }
    ]
  }
}
```

### Genre

`GET /manga/atsu/genre/:slug`

Shortcut for `/explore?genres=<slug>` with the default types.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `slug` | Yes | Numeric genre id from `/filters` or from `genres[].slug` in `/detail`, for example `9` (Romance). A name such as `romance` returns an empty list |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `page` | No | `0` | Page number, starting at `0` |

**Example**

```bash
curl "http://localhost:3000/manga/atsu/genre/9"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "page": 0,
    "items": [
      {
        "id": "OaKBx",
        "title": "Revenge of the Baskerville Bloodhound",
        "thumbnail": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/J8DNwkNsNPfdwobD.jpg",
        "images": {
          "small": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/J8DNwkNsNPfdwobD-small.avif",
          "medium": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/J8DNwkNsNPfdwobD-medium.avif",
          "large": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/J8DNwkNsNPfdwobD.jpg"
        },
        "type": "Manwha",
        "isAdult": false
      }
    ]
  }
}
```

### Author

`GET /manga/atsu/author/:slug`

Returns the titles of one author or artist. This route drops adult titles; `/adult/author/:slug` returns all of them.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `slug` | Yes | Author slug from `authors[].slug` in `/detail`, for example `kamome-shirahama` |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `type` | No | any role | `Author` or `Artist` to limit by role. Case-sensitive; any other value returns `400` |
| `page` | No | `0` | Page number, starting at `0` |

**Example**

```bash
curl "http://localhost:3000/manga/atsu/author/kamome-shirahama?type=Artist"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "author": "Kamome Shirahama",
    "role": "Artist",
    "page": 0,
    "items": [
      {
        "id": "2VgNt",
        "title": "Witch Hat Atelier",
        "thumbnail": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/N3HO30V76uWZV9iG.jpg",
        "images": {
          "small": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/N3HO30V76uWZV9iG-small.avif",
          "medium": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/N3HO30V76uWZV9iG-medium.avif",
          "large": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/N3HO30V76uWZV9iG-large.avif"
        },
        "type": "Manga",
        "isAdult": false
      }
    ]
  }
}
```

`role` is the `type` you passed, or `Any`. An unknown slug returns `404` with `Author not found`.

## Details

### Title details

`GET /manga/atsu/detail/:id`

Returns full metadata and every chapter release, newest first. When several scanlation groups released the same chapter, each release is its own entry, so the list can be longer than `totalChapters` (Witch Hat Atelier has `totalChapters: 99` and 350 entries).

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Atsu title id from any list route, for example `2VgNt` |

**Example**

```bash
curl "http://localhost:3000/manga/atsu/detail/2VgNt"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "id": "2VgNt",
    "title": "Witch Hat Atelier",
    "englishTitle": null,
    "altTitles": ["고깔모자의 아틀리에", "Asrama Topi Lancip"],
    "synopsis": "In a world where everyone takes wonders like magic spells and dragons for granted, Coco is a girl with a simple dream: s...",
    "type": "Manga",
    "isAdult": false,
    "status": "Ongoing",
    "genres": [
      { "genre": "Adventure", "slug": "37" },
      { "genre": "Fantasy", "slug": "36" }
    ],
    "authors": [
      { "author": "Kamome Shirahama", "slug": "kamome-shirahama", "role": "Author" },
      { "author": "Kamome Shirahama", "slug": "kamome-shirahama", "role": "Artist" }
    ],
    "scanlators": [
      { "id": "cmgzkynrz5nvxm191t8wd2zk5", "name": "Alpha", "score": 30, "myVote": null }
    ],
    "poster": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/N3HO30V76uWZV9iG.jpg",
    "banner": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/banners/2VgNt.jpg",
    "rating": 8.47980857142857,
    "views": "16.3M",
    "totalChapters": 99,
    "chapters": [
      {
        "id": "prIIkU",
        "title": "Chapter 99",
        "number": 99,
        "pages": 17,
        "createdAt": 1788464782000,
        "scanId": "cmrsjj6s100002eqvm319u8nr",
        "scanlator": "Thunder"
      }
    ]
  }
}
```

`createdAt` is a Unix time in milliseconds. `scanlator` is the group name matched from `scanlators`, or `Unknown`. `views` is a display string. This route works for adult titles too (`isAdult: true`).

### Chapter info

`GET /manga/atsu/info/:id`

A lighter chapter list: title, type and chapters, oldest first, without dates or scanlator names.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Atsu title id, for example `2VgNt` |

**Example**

```bash
curl "http://localhost:3000/manga/atsu/info/2VgNt"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "id": "2VgNt",
    "title": "Witch Hat Atelier",
    "type": "Manga",
    "chapters": [
      {
        "id": "vJly4Skj",
        "title": "Chapter 1",
        "number": 1,
        "pages": 66,
        "scanId": "cmgzkynrz5nvxm191t8wd2zk5"
      }
    ]
  }
}
```

## Read

### Chapter pages

`GET /manga/atsu/read`

Returns the page images of one chapter in reading order. Page numbers start at `0`.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `mangaId` | Yes | | Atsu title id, for example `2VgNt` |
| `chapterId` | Yes | | Chapter `id` from `/detail` or `/info`, for example `prIIkU` |

**Example**

```bash
curl "http://localhost:3000/manga/atsu/read?mangaId=2VgNt&chapterId=prIIkU"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "id": "prIIkU",
    "title": "Chapter 99",
    "pages": [
      {
        "img": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/pages/cmrsjj6s100002eqvm319u8nr/prIIkU/2fcc8efe01d30477.avif",
        "page": 0
      },
      {
        "img": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/pages/cmrsjj6s100002eqvm319u8nr/prIIkU/c259750f44584825.avif",
        "page": 1
      }
    ]
  }
}
```

## Adult routes

The routes under `/manga/atsu/adult` mirror the list routes above with the same parameters, defaults and response shapes:

| Route | Same as |
| --- | --- |
| `GET /manga/atsu/adult/home` | `/home` |
| `GET /manga/atsu/adult/trending` | `/trending` |
| `GET /manga/atsu/adult/most-bookmarked` | `/most-bookmarked` |
| `GET /manga/atsu/adult/hot-updates` | `/hot-updates` |
| `GET /manga/atsu/adult/top-rated` | `/top-rated` |
| `GET /manga/atsu/adult/popular` | `/popular` |
| `GET /manga/atsu/adult/recently-added` | `/recently-added` |
| `GET /manga/atsu/adult/explore` | `/explore` |
| `GET /manga/atsu/adult/genre/:slug` | `/genre/:slug` |
| `GET /manga/atsu/adult/author/:slug` | `/author/:slug` |

Home, ranked lists, explore and genre return only titles with `isAdult: true`. The adult author route returns every title by the author, adult or not; for example `takahiro` returned 6 titles there and 4 on the regular route. There is no adult version of `/filters`, `/detail`, `/info` or `/read`: the regular routes work for adult titles.

**Example**

```bash
curl "http://localhost:3000/manga/atsu/adult/popular"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "page": 0,
    "items": [
      {
        "id": "xurMP",
        "title": "Imaizumi Brings All the Gals to His House",
        "thumbnail": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/dbikxTTDry9Vn6uc.jpg",
        "images": {
          "small": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/dbikxTTDry9Vn6uc-small.avif",
          "medium": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/dbikxTTDry9Vn6uc-medium.avif",
          "large": "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/posters/dbikxTTDry9Vn6uc-large.avif"
        },
        "type": "Manga",
        "isAdult": true
      }
    ]
  }
}
```

## Images

### Image proxy

`GET /manga/atsu/image/*`

Serves Atsu posters, banners and pages. Every image URL in the responses above already points here, so use them as they are. The part after `/image/` is the upstream URL without `https://`; a path on `atsu.moe` is fetched from `cdn.atsu.moe`.

```bash
curl -o page.avif "http://localhost:3000/manga/atsu/image/cdn.atsu.moe/static/pages/cmrsjj6s100002eqvm319u8nr/prIIkU/2fcc8efe01d30477.avif"
```

The route requests the image with `Referer: https://atsu.moe/`, follows up to 3 redirects and returns the raw bytes with the upstream `Content-Type` (`image/avif`, `image/jpeg`, ...) and `Cache-Control: public, max-age=604800, immutable`. Errors are plain text, not JSON:

| Status | Body | When |
| --- | --- | --- |
| `400` | `Invalid image URL` or `Invalid image host` | The path is not a valid public hostname and path |
| `403` | `Forbidden image host` | The host resolves to a private or local address |
| `404` and other upstream statuses | `Image unavailable` | The upstream returned an error |
| `502` | `Image unavailable` | The upstream could not be reached or did not return an image |

## Errors

| Status | When |
| --- | --- |
| `400` | `mangaId` or `chapterId` is missing on `/read`, or Atsu rejected a parameter on a ranked list or author route (`Atsu rejected the request parameters`), for example an unknown `types` value, a non-numeric `timeframe` or an unknown author `type` |
| `404` | Unknown title id, chapter id or author slug (`Manga not found`, `Author not found`, `Not found on Atsu`) |
| `502` | Atsu failed or returned an invalid response |
| `503` | Atsu is rate limiting requests |
| `504` | Atsu timed out |

## Notes

* Flow: any list route gives `id`, `/detail/:id` or `/info/:id` gives `chapters[].id`, `/read?mangaId=&chapterId=` gives `pages`.
* All `page` parameters start at `0`, not `1`.
* Every route answered in 0.3 to 0.4 seconds during testing.
