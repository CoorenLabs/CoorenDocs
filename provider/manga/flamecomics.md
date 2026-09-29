---
description: Search, chapter lists and page images for FlameComics' English scanlations, mostly manhwa.
icon: fire
---

# FlameComics

FlameComics reads the Next.js data routes of [flamecomics.xyz](https://flamecomics.xyz), a scanlation group that publishes English translations of manhwa, manhua and manga. The catalog is limited to FlameComics' own series. Covers and pages are returned as direct URLs; this provider has no image route.

{% hint style="info" %}
Base route: `/manga/flamecomics`
{% endhint %}

## Routes

| Route | Description |
| --- | --- |
| `GET /manga/flamecomics` | Provider status |
| `GET /manga/flamecomics/search` | Search series |
| `GET /manga/flamecomics/detail/:id` | Series details and chapter list |
| `GET /manga/flamecomics/read/:mangaId/:token` | Page images of one chapter |

Every route except the status route returns `{ "status", "success", "data" }`. Errors return the same envelope with a `message` and `data: null`.

## Search

### Search series

`GET /manga/flamecomics/search`

Loads the full FlameComics catalog and returns every series whose title contains the query. Matching ignores case, apostrophes and punctuation, so `READERS VIEWPOINT` finds `Omniscient Reader's Viewpoint`. There is no pagination; a short query such as `the` returned 74 series. No match returns an empty array.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `q` | Yes | | Search text. Leading and trailing spaces are removed; an empty value returns `400` |

**Example**

```bash
curl "http://localhost:3000/manga/flamecomics/search?q=omniscient"
```

```json
{
  "status": 200,
  "success": true,
  "data": [
    {
      "id": "2",
      "title": "Omniscient Reader's Viewpoint",
      "cover": "https://flamecomics.xyz/_next/image?url=https%3A%2F%2Fcdn.flamecomics.xyz%2Fuploads%2Fimages%2Fseries%2F2%2Fthumbnail.png%3F1785331425&w=1920&q=75",
      "type": "Manhwa",
      "status": "Hiatus",
      "url": "https://flamecomics.xyz/series/2"
    }
  ]
}
```

`cover`, `type` and `status` are `null` when the series has none.

## Details

### Series details

`GET /manga/flamecomics/detail/:id`

Returns the series metadata and every chapter, newest first. Each chapter has a `token`, which `/read` needs together with the series id.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Numeric series id from search, for example `2` |

**Example**

```bash
curl "http://localhost:3000/manga/flamecomics/detail/2"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "id": "2",
    "title": "Omniscient Reader's Viewpoint",
    "cover": "https://flamecomics.xyz/_next/image?url=https%3A%2F%2Fcdn.flamecomics.xyz%2Fuploads%2Fimages%2Fseries%2F2%2Fthumbnail.png%3F1785331425&w=1920&q=75",
    "type": "Manhwa",
    "status": "Hiatus",
    "url": "https://flamecomics.xyz/series/2",
    "description": "‘This is a development that I know of.’ The moment he thought that the world had been destroyed, and a new universe had...",
    "altTitles": ["전지적 독자 시점", "Jeonjijeok Dokja Sijeom"],
    "genres": ["Action", "Adventure"],
    "authors": ["Sing-Shong"],
    "artists": ["Sleepy-C", "Redice Studio"],
    "year": 2020,
    "chapters": [
      {
        "id": "11716",
        "number": 311,
        "title": "Dokja’s Fable (Part 13)",
        "token": "364db6fd6bef182e",
        "releaseDate": "2026-05-19T17:59:34.000Z",
        "url": "https://flamecomics.xyz/series/2/364db6fd6bef182e"
      },
      {
        "id": "11689",
        "number": 310,
        "title": "Dokja's Fable (Part 12)",
        "token": "3ac501f3dd9c39cd",
        "releaseDate": "2026-05-14T02:26:28.000Z",
        "url": "https://flamecomics.xyz/series/2/3ac501f3dd9c39cd"
      }
    ]
  }
}
```

`description` is plain text with the HTML removed. `year`, `description`, a chapter `title` and `releaseDate` can be `null`.

## Read

### Chapter pages

`GET /manga/flamecomics/read/:mangaId/:token`

Returns the page image URLs of one chapter in reading order, plus the tokens of the previous and next chapters.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `mangaId` | Yes | Numeric series id, for example `2` |
| `token` | Yes | Chapter `token` from `/detail`, for example `364db6fd6bef182e` |

**Example**

```bash
curl "http://localhost:3000/manga/flamecomics/read/2/364db6fd6bef182e"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "id": "11716",
    "mangaId": "2",
    "mangaTitle": "Omniscient Reader's Viewpoint",
    "number": 311,
    "title": "Dokja’s Fable (Part 13)",
    "token": "364db6fd6bef182e",
    "releaseDate": "2026-05-19T17:59:34.000Z",
    "images": [
      "https://cdn.flamecomics.xyz/uploads/images/series/2/364db6fd6bef182e/ORV-311-00.jpg?1779213574",
      "https://cdn.flamecomics.xyz/uploads/images/series/2/364db6fd6bef182e/ORV-311-01.jpg?1779213574"
    ],
    "prevToken": "3ac501f3dd9c39cd",
    "nextToken": null
  }
}
```

`prevToken` and `nextToken` are `null` at either end of the series. Pass them back to this route with the same `mangaId` to move between chapters.

## Images

Covers point at FlameComics' own `/_next/image` resizer and pages point straight at `cdn.flamecomics.xyz`. Both loaded without a `Referer` during testing, so use them directly. There is no `/manga/flamecomics/image/*` route.

## Errors

| Status | When |
| --- | --- |
| `400` | `q` is missing or empty, or the series id is not numeric |
| `404` | Unknown series id or chapter token (`Not found on FlameComics`), or a chapter with no pages |
| `502` | FlameComics failed, returned an invalid response or could not be reached |
| `503` | FlameComics is rate limiting requests |

## Notes

* Flow: `/search` gives `id`, `/detail/:id` gives `chapters[].token`, `/read/:mangaId/:token` gives `images`.
* Search, detail and read each answered in 0.2 to 1.1 seconds during testing.
* The provider reads the site's Next.js build id from the home page and reuses it. When FlameComics deploys a new build, the next request refreshes the id and retries once.
