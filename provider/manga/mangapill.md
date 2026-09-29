---
description: Search, chapter lists and page images from MangaPill.
icon: pills
---

# MangaPill

MangaPill scrapes [mangapill.com](https://mangapill.com), a free English reader with a large catalogue of manga, manhwa and manhua. It is a small provider with three routes: search, detail with the full chapter list, and read. Pages and covers are served through Cooren's image route.

{% hint style="info" %}
Base route: `/manga/mangapill`
{% endhint %}

## Routes

| Route | Description |
| --- | --- |
| `GET /manga/mangapill` | Provider status |
| `GET /manga/mangapill/search` | Search titles |
| `GET /manga/mangapill/detail/:id` | Title details and chapter list |
| `GET /manga/mangapill/read/:chapterId` | Page images of one chapter |
| `GET /manga/mangapill/image/*` | Image proxy |

Every route except the status route and the image proxy returns `{ "status", "success", "data" }`. Errors return the same envelope with a `message` and `data: null`.

## Search

### Search titles

`GET /manga/mangapill/search`

Returns the titles from MangaPill's quick search. There is no pagination; a broad query such as `one piece` returned 50 results. A query with no matches returns an empty array.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `q` | Yes | | Search text. Leading and trailing spaces are removed; an empty value returns `400` |

**Example**

```bash
curl "http://localhost:3000/manga/mangapill/search?q=frieren"
```

```json
{
  "status": 200,
  "success": true,
  "data": [
    {
      "id": "5035",
      "title": "Sousou no Frieren",
      "altTitle": "Frieren - Beyond Journey's End",
      "cover": "http://localhost:3000/manga/mangapill/image/cdn.readdetectiveconan.com/file/mangapill/i/5035.jpeg",
      "type": "manga",
      "status": "publishing",
      "year": 2020,
      "url": "https://mangapill.com/manga/5035/sousou-no-frieren"
    }
  ]
}
```

`altTitle`, `cover`, `type`, `status` and `year` are `null` when MangaPill does not show them.

## Details

### Title details

`GET /manga/mangapill/detail/:id`

Returns the title's metadata and every chapter, newest first. Each chapter `id` is what `/read` expects.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Numeric MangaPill id from search, for example `5035` |

**Example**

```bash
curl "http://localhost:3000/manga/mangapill/detail/5035"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "id": "5035",
    "title": "Sousou no Frieren",
    "altTitle": "Frieren - Beyond Journey's End",
    "cover": "http://localhost:3000/manga/mangapill/image/cdn.readdetectiveconan.com/file/mangapill/i/5035.jpeg",
    "type": "manga",
    "status": "publishing",
    "year": 2020,
    "description": "Frieren is a member of the hero's party that defeated the demon king. Both a magician and an elf, those are the things t...",
    "genres": ["Adventure", "Drama"],
    "url": "https://mangapill.com/manga/5035",
    "chapters": [
      {
        "id": "5035-10147000",
        "number": 147,
        "title": "Chapter 147",
        "url": "https://mangapill.com/chapters/5035-10147000/sousou-no-frieren-chapter-147"
      },
      {
        "id": "5035-10146000",
        "number": 146,
        "title": "Chapter 146",
        "url": "https://mangapill.com/chapters/5035-10146000/sousou-no-frieren-chapter-146"
      }
    ]
  }
}
```

`number` is parsed from the chapter name and keeps decimals (`110.5` for `Chapter 110.5`). It is `null` when the name has no chapter number.

## Read

### Chapter pages

`GET /manga/mangapill/read/:chapterId`

Returns the page image URLs of one chapter in reading order, plus the ids of the previous and next chapters.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `chapterId` | Yes | Chapter id from `/detail`, in the form `<mangaId>-<number>`, for example `5035-10147000` |

**Example**

```bash
curl "http://localhost:3000/manga/mangapill/read/5035-10147000"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "id": "5035-10147000",
    "mangaId": "5035",
    "title": "Sousou no Frieren Chapter 147",
    "number": 147,
    "pages": [
      "http://localhost:3000/manga/mangapill/image/cdn.readdetectiveconan.com/file/mangap/5035/10147000/0199e56b-d6b8-7d16-ab07-d0e924724b44/1.jpeg",
      "http://localhost:3000/manga/mangapill/image/cdn.readdetectiveconan.com/file/mangap/5035/10147000/0199e56b-d6b8-7d16-ab07-d0e924724b44/2.jpeg"
    ],
    "url": "https://mangapill.com/chapters/5035-10147000",
    "prevChapterId": "5035-10146000",
    "nextChapterId": null
  }
}
```

`prevChapterId` is `null` on the first chapter and `nextChapterId` is `null` on the latest one.

## Images

### Image proxy

`GET /manga/mangapill/image/*`

Serves MangaPill covers and pages. The `cover` and `pages` URLs in the responses above already point here, so use them as they are. The part after `/image/` is the upstream URL without `https://`; any query string is passed on.

```bash
curl -o 1.jpeg "http://localhost:3000/manga/mangapill/image/cdn.readdetectiveconan.com/file/mangap/5035/10147000/0199e56b-d6b8-7d16-ab07-d0e924724b44/1.jpeg"
```

The route requests the image with `Referer: https://mangapill.com/`, follows up to 3 redirects and returns the raw bytes with the upstream `Content-Type` (for example `image/jpeg`) and `Cache-Control: public, max-age=604800, immutable`. Errors are plain text, not JSON:

| Status | Body | When |
| --- | --- | --- |
| `400` | `Invalid image URL` or `Invalid image host` | The path is not a valid public hostname and path |
| `403` | `Forbidden image host` | The host resolves to a private or local address |
| `404` and other upstream statuses | `Image unavailable` | The upstream returned an error |
| `502` | `Image unavailable` | The upstream could not be reached or did not return an image |

## Errors

| Status | When |
| --- | --- |
| `400` | `q` is missing or empty, the manga id is not numeric, or the chapter id is not in the `<mangaId>-<number>` form |
| `404` | MangaPill has no page for that id (`Not found on MangaPill`) |
| `502` | MangaPill failed, returned an unexpected page or a chapter without pages, or could not be reached |
| `503` | MangaPill is rate limiting requests |

## Notes

* Flow: `/search` gives `id`, `/detail/:id` gives `chapters[].id`, `/read/:chapterId` gives `pages`.
* Search, detail and read each answered in 0.3 to 1.3 seconds during testing.
* Requests go through Cooren's Cloudflare-aware fetcher, which retries with a browser fingerprint or a browser solve if MangaPill starts challenging requests.
