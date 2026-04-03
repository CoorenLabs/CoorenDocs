---
icon: book
---

# MangaFire

## MangaFire

MangaFire exposes discovery, search, metadata, chapter listing, and reading endpoints.

Use it when you need a high-quality manga provider with extensive language support, volume metadata, and curated home sections.

{% hint style="info" %}
Base route: `GET /manga/mangafire/...`
{% endhint %}

### Overview

MangaFire supports:

*   Home sections (trending, most viewed, recently updated)
*   Full-text search with pagination
*   Comprehensive manga metadata
*   Multi-language chapter lists
*   Volume metadata and covers
*   Reading endpoints for images

IDs and conventions:

*   Manga ID: A slug containing the title and a unique hash
*   Chapter ID: A numeric ID for reading

---

### Home

`GET /manga/mangafire/home`

Returns curated sections for the homepage.

**Typical sections include:**

*   `releasingManga`: Trending/Featured series
*   `mostViewedManga`: Ranked lists for `day`, `week`, and `month`
*   `recentlyUpdatedManga`: Latest additions and updates
*   `newReleaseManga`: Freshly indexed titles

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "releasingManga": [
      {
        "id": "manga-slug.id123",
        "status": "Releasing",
        "name": "Manga Title",
        "description": "Cleaned description text...",
        "currentChapter": "Chap 100 - Vol 10",
        "genres": ["Action", "Comedy"],
        "poster": "https://.../images/cover.jpg"
      }
    ],
    "mostViewedManga": {
      "day": [],
      "week": [],
      "month": []
    },
    "recentlyUpdatedManga": [],
    "newReleaseManga": []
  }
}
```

---

### Search and Discovery

#### General Search
`GET /manga/mangafire/search?q={query}&page={page}`

Searches for manga titles.

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "currentPage": 1,
    "totalPages": 5,
    "results": [
      {
        "id": "manga-slug.id123",
        "title": "Manga Title",
        "poster": "https://.../images/cover.jpg",
        "type": "Manga",
        "chapters": [
          {
            "url": "...",
            "title": "...",
            "chapter": "Chap 100",
            "releaseDate": null
          }
        ]
      }
    ]
  }
}
```

#### Latest Updates
`GET /manga/mangafire/latest?page={page}`

Returns the global list of latest updated manga. The response shape matches the general search route.

#### Browse by Category
`GET /manga/mangafire/category/{category}?page={page}`

Browse specific categories (e.g., `manga`, `manhwa`, `one-shot`). The response shape matches the general search route but includes the requested `category`.

#### Browse by Genre
`GET /manga/mangafire/genre/{genre}?page={page}`

Browse manga filtered by genre slug (e.g., `action`, `adventure`). The response shape matches the general search route but includes the requested `genre`.

---

### Manga Metadata

#### Manga Detail
`GET /manga/mangafire/detail/{id}`

Returns detailed metadata for a specific manga.

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "id": "manga-slug.id123",
    "title": "Manga Title",
    "altTitles": "Original Title; Alt Title...",
    "poster": "https://.../images/cover.jpg",
    "status": "Completed",
    "type": "Manga",
    "description": "Cleaned description text...",
    "author": "Author Name",
    "published": "Jan 01, 2020 to Jan 01, 2022",
    "genres": [
      "Action",
      "Comedy"
    ],
    "rating": "9.00",
    "similarManga": [
      {
        "id": "similar-manga.id456",
        "name": "Similar Manga Title",
        "poster": "https://.../images/cover.jpg"
      }
    ]
  }
}
```

#### Volumes
`GET /manga/mangafire/volumes/{id}?lang={lang}`

Returns volume list with cover images for a specific language (default `en`).

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "mangaId": "manga-slug.id123",
    "volumes": [
      {
        "volume": "10",
        "url": "/read/manga-slug.id123/en/volume-10",
        "image": "https://.../images/cover.jpg"
      }
    ]
  }
}
```

---

### Chapters and Reading

#### Chapter List
`GET /manga/mangafire/chapters/{id}?lang={lang}`

This endpoint operates in two modes:

**1. Language Check (Omit `lang`)**

Returns a list of available languages and their chapter counts.

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": [
    {
      "id": "EN",
      "title": "English",
      "chapters": "252 Chapters"
    },
    {
      "id": "ES",
      "title": "Spanish",
      "chapters": "30 Chapters"
    }
  ]
}
```

**2. Chapter List (Provide `lang`)**

Returns the full list of chapters for that language.

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": [
    {
      "number": "100.5",
      "title": "",
      "chapterId": "1234568",
      "language": "en",
      "releaseDate": null
    },
    {
      "number": "100",
      "title": "",
      "chapterId": "1234567",
      "language": "en",
      "releaseDate": null
    }
  ]
}
```

#### Read Chapter
`GET /manga/mangafire/read/{id}`

Returns an array of direct image URLs for the specified numeric chapter ID.

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": [
    "https://foo.mfcdn.nl/images/page1.jpg",
    "https://foo.mfcdn.nl/images/page2.jpg"
  ]
}
```

---

### Typical Workflow

1.  **Search**: Use `/manga/mangafire/search?q=query` to find the manga ID (`manga-slug.id123`).
2.  **Check Languages**: Call `/manga/mangafire/chapters/manga-slug.id123` to see available translations.
3.  **Get Chapters**: Call `/manga/mangafire/chapters/manga-slug.id123?lang=en` to get the list of numeric chapter IDs.
4.  **Read**: Call `/manga/mangafire/read/1234567` to get the image URLs for display in a reader.
