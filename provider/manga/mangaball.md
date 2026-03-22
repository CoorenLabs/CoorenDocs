---
icon: book
---

# MangaBall

## MangaBall

MangaBall exposes discovery, browse, search, detail, reading, and image proxy endpoints.

Use it when you need deeper manga filtering, author and tag exploration, and proxied chapter images.

{% hint style="info" %}
Base route: `GET /manga/mangaball/...`
{% endhint %}

### Overview

MangaBall supports:

* recommendations and home feeds
* latest, popular, and recently added lists
* origin and status browsing
* simple and advanced search
* person and author discovery
* tag and keyword exploration
* manga detail with chapter translations
* chapter reading
* image proxying

Common title shape:

```ts
type MangaballTitle = {
  _id: string;
  title: string;
  alternateTitle?: string;
  thumbnail?: string;
  background?: string;
  tags?: { tag: string; id_tags: string }[];
  authors?: { authors: string; id_authors: string }[];
  status?: string;
  slug?: string;
  description?: string;
  updated_at?: string;
  languageFlag?: string;
  stats_count?: unknown;
};
```

Most routes return this envelope:

```ts
type ApiEnvelope<T> = {
  status: number;
  success: boolean;
  data: T | null;
  message?: string;
};
```

### Status

#### Provider status

`GET /manga/mangaball/`

Returns a basic provider status payload.

Example shape:

```json
{
  "provider": "Mangaball",
  "status": "operational",
  "description": "Mangaball is a popular online manga reading platform...",
  "message": "Mangaball provider is running. Visit /docs for available endpoints."
}
```

### Browse and discovery

Most browse routes return:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "data": [],
    "pagination": {}
  }
}
```

#### Recommendations

`GET /manga/mangaball/recommendation`

Returns recommended manga.

**Query params**

* `limit` optional. Default is `12`

#### Home

`GET /manga/mangaball/home`

Returns a featured home feed.

#### Latest updated

`GET /manga/mangaball/latest`

Returns recently updated titles.

**Query params**

* `page` optional. Default is `1`
* `limit` optional. Default is `24`

#### For you

`GET /manga/mangaball/foryou`

Returns recently-read-based suggestions.

**Query params**

* `time` optional. Default is `day`
* `limit` optional. Default is `12`

#### Recent chapters read

`GET /manga/mangaball/recent`

Returns a recently read chapters feed.

**Query params**

* `time` optional. Default is `day`
* `limit` optional. Default is `12`

#### Popular

`GET /manga/mangaball/popular`

Returns popular manga.

**Query params**

* `limit` optional. Default is `24`

#### By origin

`GET /manga/mangaball/origin`

Returns titles filtered by origin.

**Query params**

* `origin` optional. Default is `all`

Typical values include:

* `manga`
* `manhwa`
* `manhua`
* `comics`

#### Recently added

`GET /manga/mangaball/added`

Returns recently added titles.

**Query params**

* `page` optional. Default is `1`
* `limit` optional. Default is `24`

#### New chapters

`GET /manga/mangaball/new-chap`

Returns titles with recently updated chapters.

**Query params**

* `page` optional. Default is `1`
* `limit` optional. Default is `24`

### Language and status browsing

These routes use the advanced filter layer internally.

#### By original language

All routes below accept:

* `page` optional. Default is `1`
* `limit` optional. Default is `24`

**Manga**

`GET /manga/mangaball/manga`

Forces `originalLang = "jp"`.

**Manhwa**

`GET /manga/mangaball/manhwa`

Forces `originalLang = "kr"`.

**Manhua**

`GET /manga/mangaball/manhua`

Forces `originalLang = "zh"`.

**Comics**

`GET /manga/mangaball/comics`

Forces `originalLang = "en"`.

#### By publication status

All routes below accept:

* `page` optional. Default is `1`
* `limit` optional. Default is `24`

**Ongoing**

`GET /manga/mangaball/ongoing`

**Completed**

`GET /manga/mangaball/completed`

**On hold**

`GET /manga/mangaball/on-hold`

**Cancelled**

`GET /manga/mangaball/cancelled`

**Hiatus**

`GET /manga/mangaball/hiatus`

### Search and filters

#### Simple search

`GET /manga/mangaball/search`

Searches by title.

**Query params**

* `q` required
* `page` optional. Default is `1`
* `limit` optional. Default is `24`

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

**Notes**

* Search sorting is fixed internally to `updated_chapters_desc`.

#### Advanced filters

`GET /manga/mangaball/filters`

Runs a fully configurable filtered search.

**Basic query params**

* `q` optional. Default is `""`
* `sort` optional. Default is `updated_chapters_desc`
* `page` optional. Default is `1`
* `limit` optional. Default is `10`

**Tag filters**

* `tag_included` optional. Repeatable
* `tag_included_mode` optional. Default is `and`
* `tag_excluded` optional. Repeatable
* `tag_excluded_mode` optional. Default is `and`

**Other filters**

* `demographic` optional. Default is `any`
* `person` optional. Default is `any`
* `original_lang` optional. Default is `any`
* `status` optional. Default is `any`
* `translated_lang` optional. Repeatable

Example:

```bash
curl "http://localhost:3000/manga/mangaball/filters?sort=popular_desc&tag_included=123&tag_included=456&original_lang=jp&status=ongoing&page=1&limit=20"
```

#### Person search

`GET /manga/mangaball/person-search`

Searches authors or people by name.

**Query params**

* `q` required

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "data": [
      {
        "id_person": "12345",
        "name": "Author Name"
      }
    ]
  }
}
```

#### Titles by person

`GET /manga/mangaball/person/:id_person`

Returns titles for one person or author.

**Params**

* `id_person` required

**Query params**

* `page` optional. Default is `1`

### Tags and keywords

#### All tags

`GET /manga/mangaball/tags`

Returns grouped tags by category.

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "data": {
      "genre": [
        { "id_tags": "123", "name": "Action", "slug": "action" }
      ],
      "theme": [],
      "origin": []
    }
  }
}
```

#### Tag statistics

`GET /manga/mangaball/tags-detail`

Returns aggregate tag stats and top lists.

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "tags_info": {
      "total_tags": "200",
      "total_title": "50.0K",
      "top_genre": "Action",
      "avg/tag": "250",
      "manga": "30.0K",
      "manhwa": "10.0K",
      "manhua": "5.0K",
      "comics": "5.0K"
    },
    "all_tags": []
  }
}
```

#### Titles by tag

`GET /manga/mangaball/tags/:id_tags`

Returns titles for one tag.

**Params**

* `id_tags` required

**Query params**

* `page` optional. Default is `1`

#### Titles by keyword

`GET /manga/mangaball/keyword/:id_keyword`

Returns titles for one keyword.

**Params**

* `id_keyword` required

**Query params**

* `page` optional. Default is `1`

### Manga detail and chapters

#### Manga detail

`GET /manga/mangaball/detail/:slug`

Returns full manga metadata and chapter translations.

**Params**

* `slug` required

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "title": "Manga Title",
    "title_alter": ["Alt Title 1", "Alt Title 2"],
    "thumbnail": "https://.../manga/mangaball/image/...",
    "status": "Ongoing",
    "genres": [
      { "name": "Action", "id_tags": "123" },
      { "name": "Romance", "id_tags": "456" }
    ],
    "keywords": [
      { "name": "Dungeon", "id_keywords": "789" }
    ],
    "likes": "12K",
    "views": "200K",
    "chapters": {
      "total_chapters": 120,
      "all_chapters": [
        {
          "chapter": "120",
          "translations": [
            {
              "id_chapter": "abcd1234",
              "language": "en",
              "group": {
                "id_tags": "groupId",
                "name": "Scan Group Name",
                "icon": "https://.../manga/mangaball/image/..."
              }
            }
          ]
        }
      ]
    }
  }
}
```

**How it works**

* Scrapes the title detail HTML page.
* Extracts title, alt titles, status, genres, keywords, likes, and views.
* Extracts cover and background images.
* Reads `title_id` from the page.
* Calls the chapter API to fill the chapter list.

**Error behavior**

If the detail page or chapter fetch fails, the route returns `404`.

### Read chapter

#### Chapter images

`GET /manga/mangaball/read/:id_chapter`

Returns chapter metadata and proxied page images.

**Params**

* `id_chapter` required

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "title_id": "12345",
    "chapter_id": "abcd1234",
    "chapter_number": "12",
    "chapter_volume": "1",
    "chapter_language": "en",
    "images": [
      "https://.../manga/mangaball/image/hostname/path/page1.jpg",
      "https://.../manga/mangaball/image/hostname/path/page2.jpg"
    ]
  }
}
```

**How it works**

* Loads the chapter detail HTML page.
* Scans script blocks for the chapter metadata variables.
* Extracts the `chapterImages` JSON payload.
* Wraps every image URL through the image proxy route.

### Image proxy

#### Proxy image

`GET /manga/mangaball/image/*`

Proxies images from MangaBall hosts and CDNs.

Always use the proxied image URLs returned by the API.

**Behavior**

* If the wildcard path starts with a hostname, the route tries `https://{path}` directly.
* Otherwise, it tries a fallback host list.
* On success, it returns raw image bytes.
* On failure, it returns `404`.

Returned headers include:

* upstream `Content-Type`, or `image/jpeg`
* `Cache-Control: public, max-age=86400`

### Error model

Success responses use:

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

* missing `q` on search or person search
* upstream API failures
* missing detail or chapter pages
* image proxy misses

### Typical workflow

#### Discovery flow

1. Load `/manga/mangaball/home`
2. Add `/manga/mangaball/recommendation`, `/latest`, and `/popular`
3. Offer language filters with `/manga`, `/manhwa`, `/manhua`, and `/comics`
4. Offer status filters with `/ongoing`, `/completed`, `/on-hold`, `/cancelled`, and `/hiatus`
5. Use `/tags` and `/tags-detail` for richer filter UI

#### Search flow

1. Use `/manga/mangaball/search?q=...` for simple title search
2. Use `/manga/mangaball/filters?...` for advanced filtering
3. Use `/person-search?q=...` to find authors
4. Use `/person/{id_person}?page=1` to list a person’s titles

#### Reading flow

1. Open `/manga/mangaball/detail/{slug}`
2. Read `chapters.all_chapters`
3. Pick a translation entry and take `id_chapter`
4. Call `/manga/mangaball/read/{id_chapter}`
5. Render the returned `images[]`

MangaBall is a good fit when you want stronger browsing and filtering than a simpler manga source.
