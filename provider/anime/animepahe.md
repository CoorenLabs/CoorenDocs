---
icon: bolt
---

# AnimePahe

## AnimePahe

AnimePahe exposes search, latest airing, anime metadata, episode lists, and streaming links.

Use it when you need strong search results, full episode lists, and multiple stream variants per episode.

{% hint style="info" %}
Base route: `GET /anime/animepahe/...`
{% endhint %}

### Overview

AnimePahe supports:

* Full-text search
* Latest airing episodes
* Anime metadata
* Complete episode lists
* Stream sources by episode

Provider routes:

* `GET /anime/animepahe/search/:query`
* `GET /anime/animepahe/latest`
* `GET /anime/animepahe/info/:id`
* `GET /anime/animepahe/episodes/:id`
* `GET /anime/animepahe/episode/:id/:session`

### Common data shapes

#### Search item

```ts
interface AnimeSearchItem {
  id: string;
  title: string;
  type: string;
  episodes: number;
  status: string;
  year: number;
  score: number;
  poster: string;
  session: string;
}
```

#### Latest airing item

```ts
interface AiringItem {
  id: string;
  title: string;
  episode: number;
  snapshot: string;
  session: string;
  fansub: string;
  created_at: string;
}
```

#### Episode item

```ts
interface Episode {
  title: string;
  episode: number;
  released: string;
  snapshot: string;
  duration: string;
  filler: boolean;
  session: string;
}
```

#### Anime metadata

```ts
interface AnimeMeta {
  id: string;
  name: string;
  description: string;
  poster: string | null;
  background: string | null;
  aired: string;
  duration: string;
  genres: string[];
  externalLinks: string[];
}
```

#### Stream result

```ts
interface StreamResult {
  id: string;
  title: string;
  url: string;
  directUrl?: string | null;
  quality: string;
  audio: string;
  type?: string;
  downloadUrl?: string | null;
  corsHeaders?: Record<string, string>;
  animeName?: string;
}
```

### Search

#### Full-text search

`GET /anime/animepahe/search/:query`

Searches AnimePahe and returns a curated result list.

**Params**

* `query` required

Example shape:

```json
{
  "results": [
    {
      "id": "xxxxx",
      "title": "Title",
      "type": "TV",
      "episodes": 24,
      "status": "Finished Airing",
      "year": 2015,
      "score": 8.5,
      "poster": "https://i.animepahe.si/posters/....jpg",
      "session": "xxxxx"
    }
  ]
}
```

```bash
curl "http://localhost:3000/anime/animepahe/search/one%20piece"
```

**Notes**

* Uses the AnimePahe API search endpoint internally.
* Search is validated with a schema before transformation.
* Validation failures return `results: []`.
* Poster paths are normalized to absolute URLs.
* `id` is set to the same value as `session`.

Typical flow:

1. Search by title.
2. Pick an anime using `id`.
3. Use that `id` for info and episode routes.

### Latest airing

#### Latest episodes feed

`GET /anime/animepahe/latest`

Returns recently aired episodes.

Example shape:

```json
{
  "results": [
    {
      "id": "anime-session-id",
      "title": "Anime Title",
      "episode": 5,
      "snapshot": "https://i.animepahe.si/screenshots/...jpg",
      "session": "episode-session-id",
      "fansub": "SubsPlease",
      "created_at": "2024-03-01T12:00:00Z"
    }
  ]
}
```

**Notes**

* Uses the airing feed internally.
* Validation or network failures return `results: []`.
* Snapshot URLs are normalized to absolute URLs.
* `id` is anime-level.
* `session` is episode-level.

Use this route for:

* latest episode carousels
* airing feeds
* quick links into the episode stream route

### Anime info

#### Anime metadata

`GET /anime/animepahe/info/:id`

Returns textual and visual metadata for one anime.

**Params**

* `id` required

Example shape:

```json
{
  "id": "anime-id",
  "name": "Anime Title",
  "description": "Full synopsis...",
  "poster": "https://....jpg",
  "background": "https://....jpg",
  "aired": "2015-04-05 to 2015-09-20",
  "duration": "24 min. per ep.",
  "genres": ["Action", "Adventure", "Fantasy"],
  "externalLinks": [
    "https://myanimelist.net/anime/12345",
    "https://anilist.co/anime/67890"
  ]
}
```

```bash
curl "http://localhost:3000/anime/animepahe/info/XXXXX"
```

**Implementation details**

* Scrapes the anime page HTML.
* Extracts the title, synopsis, poster, background, aired range, duration, genres, and external links.
* Synopsis line breaks are preserved from upstream `<br>` tags.
* External links are normalized to absolute URLs.

{% hint style="warning" %}
If parsing fails or the anime is missing, the route returns `{ "error": "Anime not found" }` as a payload-level error.
{% endhint %}

### Episodes

#### Full episode list

`GET /anime/animepahe/episodes/:id`

Returns all episodes for one anime.

**Params**

* `id` required

Example shape:

```json
{
  "results": [
    {
      "title": "Episode 1 Title",
      "episode": 1,
      "released": "2020-01-01T12:00:00.000Z",
      "snapshot": "https://i.animepahe.si/screenshots/...jpg",
      "duration": "24 min",
      "filler": false,
      "session": "episode-session-id"
    }
  ]
}
```

**Notes**

* Uses the release API internally.
* Fetches all pages if the upstream response has multiple pages.
* Concatenates all results before transforming them.
* Sorts episodes in ascending order.
* Snapshot URLs are normalized.
* `filler` is `true` when the upstream field is `1`.

Typical use:

* build an episode selector
* map episode sessions to stream requests

### Streaming

#### Stream sources

`GET /anime/animepahe/episode/:id/:session`

Returns stream options for one episode.

**Params**

* `id` required. Anime-level identifier
* `session` required. Episode-level session

**Response format**

This route streams NDJSON, not one JSON array.

Each line is one complete `StreamResult` object.

Example:

```
{"id":"...","title":"jpn / 720p", ...}
{"id":"...","title":"eng / 1080p", ...}
```

Example yielded shape:

```json
{
  "id": "animeId--720--jpn",
  "title": "jpn / 720p",
  "url": "https://kwik.cx/...",
  "directUrl": "https://...m3u8-or-similar...",
  "quality": "720",
  "audio": "jpn",
  "downloadUrl": "https://kwik.cx/mp4/...?...file=Title_-_Sub_-_720p_-_Episode_1.mp4",
  "corsHeaders": {
    "Referer": "https://kwik.cx/"
  }
}
```

**How it works**

* Loads the play page for the anime and episode session.
* Scrapes audio, quality, and kwik source links.
* Scrapes matching download links when available.
* Streams each resolved result back line by line.

**Direct and HLS extraction**

1. It tries direct source extraction first.
2. If needed, it falls back to HLS extraction.
3. It builds a download-friendly URL when possible.

**Client handling**

Parse the response line by line.

Do not expect a single JSON array payload.

Typical flow:

1. Call `/anime/animepahe/episodes/:id`
2. Pick an episode
3. Read `episode.session`
4. Call `/anime/animepahe/episode/:id/:session`
5. Collect stream variants for a quality or audio picker

### ID and session rules

AnimePahe uses two identifiers:

* anime-level `id`
* episode-level `session`

General rule:

* use `id` for `/info/:id`
* use `id` for `/episodes/:id`
* use both `id` and `session` for `/episode/:id/:session`

Search and latest results already expose the values you need.

### Error handling

Search, latest, and episodes:

* upstream validation failures return empty arrays
* network failures return safe fallback payloads

Info:

* parsing failures return `{ "error": "Anime not found" }`
* treat that as an application-level error

Streams:

* failures for one quality or audio variant do not stop the whole stream
* broken variants are skipped

{% hint style="info" %}
Like any scraping provider, AnimePahe can break when upstream HTML or API structures change.
{% endhint %}

### When to use AnimePahe

AnimePahe is a good fit when you need:

* rich search results with year, score, and status
* a latest-airing feed
* full episode lists with filler flags
* multiple stream variants per episode
* direct and download-friendly playback URLs

It works well alongside other anime providers if you want broader source coverage.
