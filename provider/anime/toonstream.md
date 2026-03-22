---
icon: bolt
---

# ToonStream

## ToonStream

ToonStream exposes home, search, browse, detail, source, and media proxy endpoints.

Use it when you need a richer home layout, movie and series catalogs, and proxied playback support.

{% hint style="info" %}
Base route: `GET /anime/toonstream/...`
{% endhint %}

### Overview

ToonStream supports:

* home page snapshots
* search across movies and series
* paged movie listings
* paged series listings
* movie metadata
* series metadata with seasons and episodes
* direct player source extraction
* HLS, TS, MP4, and generic media proxying

Most responses include a shared envelope:

```ts
type TResponse = {
  success: boolean;
  msg?: string;
  served_cache?: boolean;
  took_ms: number;
};
```

Common item shapes:

```ts
type AnimeCard = {
  type: "movie" | "series";
  slug: string;
  title: string;
  url: string;
  poster: string;
  tmdbRating: number;
};

type Episode = {
  episode_no: number;
  slug: string;
  title: string;
  epXseason: string;
  url: string;
  thumbnail: string;
  ago?: string;
};

type DirectSource = {
  label?: string;
  type: "hls" | "mp4";
  url: string;
  cover?: string;
  thumbnail?: string;
  subtitles?: {
    label: string;
    flag?: string;
    url: string;
  };
  headers?: Record<string, string>;
  proxiedUrl?: string;
};
```

### Root index

#### API index

`GET /anime/toonstream/`

Returns a small index of ToonStream routes, plus a note about proxy usage.

Use it for:

* quick route discovery
* sanity checks during setup
* confirming proxy routes are available

### Home

#### Home snapshot

`GET /anime/toonstream/home`

Returns the scraped home page structure.

The payload includes:

* `main` sections
* `sidebar` sections
* `lastEpisodes`

Example shape:

```json
{
  "success": true,
  "served_cache": false,
  "took_ms": "12.34",
  "data": {
    "main": [
      {
        "label": "Latest Movies",
        "viewMore": "https://toonstream.../movies/",
        "data": [
          {
            "type": "movie",
            "slug": "some-movie-slug",
            "title": "Movie Title",
            "url": "https://toonstream.../movies/some-movie-slug/",
            "poster": "https://...jpg",
            "tmdbRating": 7.8
          }
        ]
      }
    ],
    "sidebar": [
      {
        "label": "Top Rated",
        "data": [
          {
            "type": "series",
            "slug": "some-series-slug",
            "title": "Series Title",
            "url": "https://toonstream.../series/some-series-slug/",
            "poster": "https://...jpg",
            "tmdbRating": 8.4
          }
        ]
      }
    ],
    "lastEpisodes": [
      {
        "slug": "episode-slug",
        "title": "Episode Title",
        "epXseason": "S01E03",
        "url": "https://toonstream.../episode/episode-slug/",
        "thumbnail": "https://...jpg",
        "ago": "2 hours ago"
      }
    ]
  }
}
```

**Caching**

* cache key: `home`
* TTL: 12 hours

### Search

#### Search movies and series

`GET /anime/toonstream/search/:query/:page?`

Searches ToonStream content with pagination.

**Params**

* `query` required
* `page` optional. Default is `1`

Example shape:

```json
{
  "success": true,
  "served_cache": false,
  "took_ms": "55.00",
  "data": {
    "query": "naruto",
    "pagination": {
      "current": 1,
      "start": 1,
      "end": 5
    },
    "data": [
      {
        "type": "series",
        "slug": "naruto",
        "title": "Naruto",
        "url": "https://toonstream.../series/naruto/",
        "poster": "https://...jpg",
        "tmdbRating": 8.3
      }
    ]
  }
}
```

**Notes**

* Spaces in the query are converted to `+`.
* Search results are cached by `query` and `page`.
* Cache TTL is 12 hours.

### Movies

#### Movie listing

`GET /anime/toonstream/movies/:page?`

Returns a paged list of movie cards.

**Params**

* `page` optional. Default is `1`

Example shape:

```json
{
  "success": true,
  "served_cache": false,
  "page": 1,
  "took_ms": "23.12",
  "data": {
    "pagination": {
      "current": 1,
      "start": 1,
      "end": 10
    },
    "data": [
      {
        "type": "movie",
        "slug": "some-movie",
        "title": "Some Movie",
        "url": "https://toonstream.../movies/some-movie/",
        "poster": "https://...jpg",
        "tmdbRating": 7.5
      }
    ]
  }
}
```

**Caching**

* cache key: `movies:{page}`
* TTL: 30 days

#### Movie info

`GET /anime/toonstream/movie/info/:slug`

Returns metadata for one movie.

**Params**

* `slug` required

Example shape:

```json
{
  "success": true,
  "served_cache": false,
  "took_ms": "18.25",
  "data": {
    "title": "Movie Title",
    "year": "2023",
    "tmdbRating": 7.8,
    "description": "Full movie description or synopsis.",
    "languages": ["English", "Japanese"],
    "qualities": ["1080p", "720p"],
    "duration": "1h 45min",
    "genres": [
      { "name": "Action", "slug": "action", "url": "https://toonstream.../genre/action/" }
    ],
    "tags": [
      { "name": "Shonen", "url": "https://toonstream.../tag/shonen/" }
    ],
    "casts": [
      { "name": "Some Actor", "url": "https://toonstream.../cast/some-actor/" }
    ]
  }
}
```

**Error behavior**

If scraping fails, the route returns:

* `success: false`
* `msg: "No Data Scraped!"`

**Caching**

* cache key: `movie:info:{slug}`
* TTL: 14 days

#### Movie sources

`GET /anime/toonstream/movie/sources/:slug`

Resolves movie embeds into direct sources and embed URLs.

**Params**

* `slug` required

Example shape:

```json
{
  "success": true,
  "took_ms": "40.01",
  "data": {
    "embeds": [
      "https://toonstream.../embed/ascdn/...",
      "https://toonstream.../embed/rubystm/..."
    ],
    "sources": [
      {
        "label": "1080p",
        "type": "hls",
        "url": "https://ascdn.../playlist.m3u8",
        "cover": "https://...jpg",
        "thumbnail": "https://...jpg",
        "subtitles": [
          { "label": "EN", "flag": "us", "url": "https://...vtt" }
        ],
        "headers": { "Referer": "https://..." },
        "proxiedUrl": "https://your-server.../anime/toonstream/m3u8-proxy?url=..."
      }
    ]
  }
}
```

**How it works**

1. Loads the movie page.
2. Scrapes ToonStream iframe URLs.
3. Follows those URLs to player iframes.
4. Extracts supported sources from AS-CDN or RubyStream.
5. Optionally rewrites source URLs to proxy routes.

**Caching**

* iframe cache key: `movie:iframes:{slug}`
* direct sources are also cached internally by source URL

### Series

#### Series listing

`GET /anime/toonstream/series/:page?`

Returns a paged list of series cards.

**Params**

* `page` optional. Default is `1`

The response shape matches the movie listing, but items use `type: "series"`.

**Caching**

* cache key: `series:{page}`
* TTL: 30 days

#### Series info

`GET /anime/toonstream/series/info/:slug`

Returns metadata for one series, including seasons and episodes.

**Params**

* `slug` required

Example shape:

```json
{
  "success": true,
  "served_cache": false,
  "took_ms": "60.50",
  "data": {
    "title": "Series Title",
    "year": "2022",
    "tmdbRating": 8.4,
    "totalSeasons": 3,
    "totalEpisodes": 24,
    "description": "Series synopsis...",
    "languages": ["English", "Japanese"],
    "qualities": ["1080p", "720p"],
    "runtime": "24 min per ep",
    "genres": [
      { "name": "Action", "slug": "action", "url": "https://..." }
    ],
    "tags": [
      { "name": "Shonen", "url": "https://..." }
    ],
    "casts": [
      { "name": "Some Actor", "url": "https://..." }
    ],
    "seasons": [
      {
        "label": "Season 1",
        "season_no": 1,
        "episodes": [
          {
            "episode_no": 1,
            "slug": "series-s1e1",
            "title": "Episode 1",
            "epXseason": "S01E01",
            "url": "https://toonstream.../episode/series-s1e1/",
            "thumbnail": "https://...jpg"
          }
        ]
      }
    ]
  }
}
```

**Implementation notes**

* Scrapes the main series page for metadata.
* Derives a `postId` from the page body classes.
* Loads each season through an internal AJAX endpoint.
* Builds `seasons` and `episodes` from the returned HTML.

**Caching**

* cache key: `series:info:{slug}`
* TTL: 3 days

#### Episode sources

`GET /anime/toonstream/episode/sources/:slug`

Resolves a series episode into embed URLs and direct sources.

**Params**

* `slug` required

The behavior matches movie source extraction:

* scrape ToonStream iframes
* resolve player iframes
* extract AS-CDN or RubyStream sources
* optionally proxify the final URLs

Example shape:

```json
{
  "success": true,
  "data": {
    "embeds": [
      "https://toonstream.../embed/ascdn/...",
      "https://toonstream.../embed/rubystm/..."
    ],
    "sources": []
  }
}
```

**Caching**

* cache key: `episode:iframes:{slug}`

### Media proxy routes

Some ToonStream sources require custom headers or a trusted origin.

These proxy routes help you serve those files through your own API.

Relevant environment variables:

* `SERVER_ORIGIN` required
* `PROXIFY` optional
* `ALLOWED_ORIGINS` optional

Size limits:

* m3u8: 5 MB
* ts segments: 50 MB
* generic fetch: 50 MB
* mp4: 20 GB

If a file exceeds its limit, the proxy returns `413`.

Most proxy routes accept:

* `url` required
* `headers` optional as encoded JSON

#### M3U8 proxy

`GET /anime/toonstream/m3u8-proxy?url={url}&headers={encodedHeaders}`

Fetches an HLS playlist, rewrites media URLs to proxy routes, and returns the rewritten playlist.

**Behavior**

* rewrites nested playlists back to `m3u8-proxy`
* rewrites TS media segments to `ts-segment`
* rewrites other media references to `fetch`
* resolves relative URLs against the original playlist URL
* adds `Connection: keep-alive`

Use this for HLS players that cannot reach the source directly.

#### TS segment proxy

`GET /anime/toonstream/ts-segment?url={url}&headers={encodedHeaders}`

Proxies TS segments used by HLS playback.

**Behavior**

* forwards the raw upstream stream
* preserves or defaults the content type
* sets `Cache-Control: public, max-age=86400`
* adds `Connection: keep-alive`

#### MP4 proxy

`GET /anime/toonstream/mp4-proxy?url={url}&headers={encodedHeaders}`

Range-aware proxy for MP4 playback.

**Behavior**

* forwards the client `Range` header
* preserves partial content responses
* returns `accept-ranges: bytes`
* keeps scrub and seek working in players

#### Generic fetch proxy

`GET /anime/toonstream/fetch?url={url}&headers={encodedHeaders}`

Proxies other files like captions, keys, images, or auxiliary media.

**Behavior**

* forwards the upstream status and body
* preserves the upstream content type when possible
* defaults to `application/octet-stream`

### Security notes

`SERVER_ORIGIN` is required.

The router throws at startup if it is missing.

Proxy routes also:

* abort upstream fetches when the client disconnects
* enforce file size limits
* trust caller-supplied target URLs and headers

For production, add your own:

* host allowlists
* auth
* rate limiting

### Typical workflow

1. Use `/anime/toonstream/home` for a home screen.
2. Use `/anime/toonstream/search/:query/:page?` for search.
3. Use `/anime/toonstream/movies/:page?` or `/anime/toonstream/series/:page?` for browsing.
4. Use detail routes for metadata.
5. Use source routes for playback URLs.
6. Use proxy routes when the player needs safer access to upstream media.

ToonStream is a good fit when you want both catalog browsing and proxy-backed playback inside one provider.
