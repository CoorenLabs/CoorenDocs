---
icon: clapperboard
---

# Yflix

## Yflix

Yflix exposes catalog and search routes for movies and TV.

Use it when you need a browse-first provider for discovery and lightweight search.

{% hint style="info" %}
Base route: `GET /movie-tv/yflix/...`
{% endhint %}

### Overview

Yflix supports:

* a home feed
* search across movies and TV
* optional type filtering

It is best used as the discovery side of the Movie & TV stack.

### Status

#### Provider root

`GET /movie-tv/yflix/`

Returns provider metadata and route hints.

Example shape:

```json
{
  "provider": "yflix",
  "type": "movie/tv",
  "endpoints": [
    "/yflix/home",
    "/yflix/search/:query"
  ]
}
```

### Home

#### Home feed

`GET /movie-tv/yflix/home`

Scrapes the Yflix home page and returns structured sections.

Example shape:

```json
{
  "success": true,
  "data": {
    "sections": [
      {
        "title": "Trending Movies",
        "items": []
      },
      {
        "title": "Trending TV",
        "items": []
      }
    ]
  }
}
```

**How it works**

* loads the Yflix home HTML
* passes the page into the home parser
* returns normalized sections for client rendering

Use it for:

* home pages
* featured rows
* trending sections

### Search

#### Search movies and TV

`GET /movie-tv/yflix/search`

Searches Yflix titles with optional type filtering.

**Query params**

* `query` required
* `page` optional. Default is `1`
* `type` optional. Allowed values: `movie`, `tv`

Example shape:

```json
{
  "success": true,
  "query": "Original Query",
  "page": 1,
  "type": "movie",
  "data": []
}
```

Failure shape:

```json
{
  "success": false,
  "query": "Original Query",
  "page": 1
}
```

**How it works**

1. Normalizes the query by replacing spaces with `+`.
2. URL-encodes the final query.
3. Builds the Yflix browser URL.
4. Adds `page` if `page > 1`.
5. Adds the type filter when present.
6. Loads the HTML and parses the result cards.

**Notes**

* leave `type` empty to search both movies and TV
* use `type=movie` for movie-only results
* use `type=tv` for TV-only results

### Typical workflow

1. Load `/movie-tv/yflix/home` for discovery.
2. Use `/movie-tv/yflix/search?query=...` for search.
3. Resolve the selected title to TMDB in your app.
4. Use PrimeSrc for final playback source extraction.

Yflix is a good fit when you want a simple catalog layer to pair with a dedicated source extractor.
