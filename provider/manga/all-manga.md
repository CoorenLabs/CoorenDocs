---
icon: book
---

# All Manga

## AllManga

AllManga exposes discovery, search, detail, reading, and image proxy endpoints.

Use it when you need a manga catalog with filters, generated chapter lists, and proxied reader images.

{% hint style="info" %}
Base route: `GET /manga/allmanga/...`
{% endhint %}

### Overview

AllManga supports:

* home sections
* full-text search
* latest updates
* popular lists by period
* random recommendations
* tag and filter discovery
* manga detail with chapter list
* chapter page reading
* image proxying

IDs and conventions:

* manga ID: AllManga `_id`
* chapter ID: `mangaId:translationType:chapterNumber`
* example chapter ID: `12345:sub:12`

Common card shape:

```ts
type AllMangaCard = {
  id: string;
  title: string;
  englishTitle: string;
  nativeTitle: string;
  cover: string;
  score: number;
  availableChapters: {
    sub: number;
    raw: number;
  };
};
```

### Status

#### Provider status

`GET /manga/allmanga/`

Returns a simple health and description payload.

Example shape:

```json
{
  "provider": "AllManga",
  "status": "operational",
  "description": "AllManga is a comprehensive manga database and reading platform...",
  "message": "AllManga provider is running. Visit /docs for available endpoints."
}
```

Use this for:

* health checks
* quick provider discovery
* setup verification

### Home

#### Home sections

`GET /manga/allmanga/home`

Returns a set of curated discovery sections.

Typical sections include:

* popular manga
* latest updates
* current-year manga
* random recommendations
* predefined tag-based lists

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "provider": "AllManga",
    "sections": [
      {
        "id": "popular-daily",
        "title": "Popular Manga (Daily)",
        "items": []
      },
      {
        "id": "latest",
        "title": "Latest Updates",
        "items": []
      },
      {
        "id": "manga-2026",
        "title": "Manga 2026",
        "items": []
      },
      {
        "id": "random",
        "title": "Random Recommendations",
        "items": []
      }
    ]
  }
}
```

**Notes**

* Home mixes multiple internal parsers.
* Tag sections are curated in code.
* Empty sections are omitted.

### Search and filters

#### General search

`GET /manga/allmanga/search?q={query}&page={page}`

Searches manga with pagination.

**Query params**

* `q` required
* `page` optional. Default is `1`

If `q` is missing, the route returns `400`.

Example error:

```json
{
  "status": 400,
  "success": false,
  "message": "Query parameter 'q' is required",
  "data": null
}
```

Example success shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "provider": "AllManga",
    "total": 123,
    "page": 1,
    "results": []
  }
}
```

**Notes**

* Uses the AllManga GraphQL API internally.
* Search defaults to manga-only results.
* Adult and unknown entries are filtered out.
* Default translation type is `sub`.

#### Latest updates

`GET /manga/allmanga/latest?page={page}`

Returns the latest updated manga list.

**Query params**

* `page` optional. Default is `1`

The response shape matches the general search route.

**Notes**

* Internally uses the same search parser with an empty query.

#### Popular

`GET /manga/allmanga/popular?page={page}&size={size}&period={period}`

Returns popular manga with a selectable ranking window.

**Query params**

* `page` optional. Default is `1`
* `size` optional. Default is `20`
* `period` optional. Default is `daily`

Supported `period` values:

* `daily`
* `weekly`
* `monthly`
* `all`

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "provider": "AllManga",
    "total": 500,
    "page": 1,
    "period": "weekly",
    "results": []
  }
}
```

**Notes**

* `daily` maps to 1 day
* `weekly` maps to 7 days
* `monthly` maps to 30 days
* `all` removes the date filter

#### Random recommendations

`GET /manga/allmanga/random`

Returns a random set of manga recommendations.

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "provider": "AllManga",
    "results": []
  }
}
```

#### Tags and genres

`GET /manga/allmanga/tags`

Returns available tags, genres, and magazines with counts.

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "provider": "AllManga",
    "tags": [
      {
        "name": "Action",
        "slug": "action",
        "type": "genre",
        "count": 1200
      },
      {
        "name": "Shounen Jump (Weekly)",
        "slug": "shounen_jump_weekly_-magazine",
        "type": "magazine",
        "count": 300
      }
    ]
  }
}
```

Use this route to build:

* filter pills
* dropdowns
* genre and magazine selectors

#### Search by genre

`GET /manga/allmanga/genre/:genre?page={page}`

Filters manga by a genre slug.

**Params**

* `genre` required
* `page` optional. Default is `1`

The response shape matches the search route.

**Notes**

* Internally injects `genres: [genre]` into the search args.

#### Search by author

`GET /manga/allmanga/author/:author?page={page}`

Filters manga by an author slug.

**Params**

* `author` required
* `page` optional. Default is `1`

The response shape matches the search route.

**Notes**

* Internally injects `authors: [author]` into the search args.

### Manga detail and chapters

#### Manga detail

`GET /manga/allmanga/detail?id={id}`

Returns metadata and a generated chapter list for one manga.

**Query params**

* `id` required

If `id` is missing, the route returns `400`.

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "provider": "AllManga",
    "id": "12345",
    "title": "Manga Title",
    "englishTitle": "English Title",
    "nativeTitle": "Original Title",
    "cover": "https://.../manga/allmanga/image/...",
    "description": "Cleaned description text...",
    "genres": [
      { "genre": "Action", "slug": "action" },
      { "genre": "Romance", "slug": "romance" }
    ],
    "authors": [
      { "author": "Some Author", "slug": "some_author" }
    ],
    "status": "Ongoing",
    "totalChapters": 45,
    "rawChapters": 0,
    "airedStart": "2020-01-01",
    "airedEnd": null,
    "chapterList": [
      {
        "id": "12345:sub:45",
        "number": 45,
        "title": "Chapter 45",
        "lang": "sub"
      },
      {
        "id": "12345:sub:44",
        "number": 44,
        "title": "Chapter 44",
        "lang": "sub"
      }
    ]
  }
}
```

**How it works**

* Loads GraphQL metadata for the manga.
* Scrapes the manga HTML page for extra fields.
* Extracts description and cover.
* Extracts genre and author slugs from page links.
* Derives `totalChapters` from `availableChapters.sub`.
* Builds `chapterList` in descending order.

Chapter IDs are generated like:

```txt
mangaId:sub:chapterNumber
```

#### Read chapter

`GET /manga/allmanga/read?id={chapterId}`

Returns all page images for one chapter.

**Query params**

* `id` required

Expected `id` format:

```txt
mangaId:translationType:chapterNumber
```

Example:

```txt
12345:sub:12
```

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "provider": "AllManga",
    "id": "12345:sub:12",
    "pages": [
      {
        "page": 1,
        "img": "https://.../manga/allmanga/image/..."
      },
      {
        "page": 2,
        "img": "https://.../manga/allmanga/image/..."
      }
    ]
  }
}
```

**Notes**

* Splits the chapter ID into manga ID, translation type, and chapter number.
* Reconstructs the image URLs from `pictureUrlHead` and `pictureUrls`.
* Forces absolute HTTPS image URLs.
* Wraps all page images with the image proxy route.

Recommended flow:

1. Call `/manga/allmanga/detail?id={mangaId}`
2. Read `chapterList`
3. Pick a chapter ID
4. Call `/manga/allmanga/read?id={chapterId}`
5. Render `pages[].img` in the reader

### Image proxy

#### Proxy image

`GET /manga/allmanga/image/*`

Proxies AllManga CDN images through your server.

Use the proxied image URLs returned by the provider.

Do not call the upstream CDN directly from the client.

**Behavior**

1. Reconstructs the target URL from the wildcard path.
2. Fetches the image with a valid image accept header.
3. Sends the request with `Referer: https://allmanga.to/`.
4. Returns raw image bytes.
5. Sets a safe cache header.

Returned headers include:

* `Content-Type` from upstream, or `image/jpeg`
* `Cache-Control: public, max-age=86400`

If proxying fails, the route returns an error envelope with the right status code.

### Error model

Successful responses use:

```json
{
  "status": 200,
  "success": true,
  "data": {}
}
```

Errors use:

```json
{
  "status": 400,
  "success": false,
  "message": "Error text",
  "data": null
}
```

Common failures:

* missing `q` on search
* missing `id` on detail or read
* upstream GraphQL or site errors
* missing chapter page data

### Typical workflow

#### Discovery flow

1. Load `/manga/allmanga/home`
2. Use `/manga/allmanga/search?q={query}&page=1` for search
3. Use `/manga/allmanga/tags` for filter UI
4. Use `/manga/allmanga/genre/{slug}?page=1` or `/manga/allmanga/author/{slug}?page=1`
5. Use `/manga/allmanga/popular?period=daily|weekly|monthly|all`

#### Reading flow

1. User opens a manga card
2. Load `/manga/allmanga/detail?id={mangaId}`
3. Render metadata and `chapterList`
4. User picks a chapter
5. Load `/manga/allmanga/read?id={chapterId}`
6. Render the returned page images

AllManga is a good fit when you want one provider to cover both manga discovery and the full reading flow.
