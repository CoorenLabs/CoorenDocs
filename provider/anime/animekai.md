---
icon: bolt
---

# Animekai

## Animekai

Animekai exposes search, browse, schedule, info, and streaming endpoints.

Use it when you need a richer anime catalog with filters, relations, and episode-level stream data.

{% hint style="info" %}
Base route: `GET /anime/animekai/...`
{% endhint %}

### Overview

Animekai supports:

* Full-text search
* Spotlight items
* Airing schedule
* Search suggestions
* Recent and discovery lists
* Browse by type
* Browse by genre
* Anime details with episodes
* Stream sources
* Episode server lists

Most listing endpoints return this shape:

```ts
interface AnimeKaiPagedResult<T> {
  currentPage: number;
  hasNextPage: boolean;
  totalPages: number;
  results: T[];
}
```

Common item shape:

```ts
type AnimeKaiSearchItem = {
  id: string;
  title: string;
  url: string;
  image?: string;
  japaneseTitle?: string | null;
  type?: string;
  sub?: number;
  dub?: number;
  episodes?: number;
};
```

### Search

#### Full-text search

`GET /anime/animekai/search/:query`

Searches anime by title.

**Params**

* `query` required. URL-encoded search text.
* `page` optional. Default is `1`.

**Returns**

`AnimeKaiPagedResult<AnimeKaiSearchItem>`

```bash
curl "http://localhost:3000/anime/animekai/search/one%20piece?page=1"
```

**Notes**

* Pagination is 1-based.
* If `page <= 0`, it is coerced to `1`.
* The provider normalizes the query for the upstream site.

### Spotlight and schedule

#### Spotlight

`GET /anime/animekai/spotlight`

Returns featured titles from the home carousel.

Example shape:

```json
{
  "results": [
    {
      "id": "anime-slug",
      "title": "Title",
      "japaneseTitle": "Japanese Title",
      "banner": "https://...banner.jpg",
      "url": "https://animekai.site/watch/anime-slug",
      "type": "TV",
      "genres": ["action", "fantasy"],
      "releaseDate": "2024",
      "quality": "HD",
      "sub": 12,
      "dub": 12,
      "description": "..."
    }
  ]
}
```

#### Airing schedule

`GET /anime/animekai/schedule/:date`

Returns the airing schedule for a specific day.

**Params**

* `date` required. Format: `YYYY-MM-DD`

Example shape:

```json
[
  {
    "id": "anime-slug",
    "title": "Title",
    "japaneseTitle": "Japanese Title",
    "airingTime": "12:00 AM",
    "airingEpisode": "5"
  }
]
```

**Notes**

* The provider converts the date into a timestamp.
* It applies an upstream timezone offset of `5.5`.
* If the upstream request fails, it returns an empty array.

### Search suggestions

#### Typeahead suggestions

`GET /anime/animekai/suggestions/:query`

Returns lightweight suggestions for autocomplete UI.

**Params**

* `query` required. Partial search text.

Example shape:

```json
{
  "results": [
    {
      "id": "anime-slug",
      "title": "Title",
      "url": "https://animekai.site/watch/anime-slug",
      "japaneseTitle": "Japanese Title",
      "image": "https://...poster.jpg",
      "type": "TV",
      "year": "2020",
      "sub": 12,
      "dub": 12,
      "episodes": 24
    }
  ]
}
```

### Recent and discovery lists

All endpoints below:

* accept `page`
* default `page` to `1`
* coerce `page <= 0` to `1`
* return `AnimeKaiPagedResult<AnimeKaiSearchItem>`

#### Recently updated episodes

`GET /anime/animekai/recent-episodes`

Returns freshly updated series.

#### Recently added series

`GET /anime/animekai/recent-added`

Returns newly added titles.

#### Latest completed series

`GET /anime/animekai/completed`

Returns recently completed shows.

#### New releases

`GET /anime/animekai/new-releases`

Returns newly released titles.

### Browse by type

These endpoints return `AnimeKaiPagedResult<AnimeKaiSearchItem>`.

Each accepts `page`.

#### Movies

`GET /anime/animekai/movies`

#### TV series

`GET /anime/animekai/tv`

#### OVA

`GET /anime/animekai/ova`

#### ONA

`GET /anime/animekai/ona`

#### Specials

`GET /anime/animekai/specials`

### Genres

#### Genre list

`GET /anime/animekai/genres`

Returns all supported genres.

Example shape:

```json
{
  "results": [
    "action",
    "adventure",
    "comedy",
    "fantasy",
    "romance",
    "sci-fi"
  ]
}
```

**Notes**

* Genres are normalized to lowercase.
* Use this route to build filter dropdowns.

#### Browse by genre

`GET /anime/animekai/genre/:genre`

Returns anime for a specific genre.

**Params**

* `genre` required
* `page` optional. Default is `1`

**Returns**

`AnimeKaiPagedResult<AnimeKaiSearchItem>`

{% hint style="warning" %}
If `genre` is missing, the provider throws before fetching upstream data.
{% endhint %}

### Anime info

#### Full anime details

`GET /anime/animekai/info?id=:id`

Returns metadata, episodes, recommendations, and relations.

**Params**

* `id` required

**Validation**

* Missing `id` returns `400`
* Unknown anime returns `404`

Example request:

```bash
curl "http://localhost:3000/anime/animekai/info?id=some-anime-slug"
```

Example shape:

```ts
type AnimeKaiInfo = {
  id: string;
  title: string;
  japaneseTitle?: string | null;
  image?: string;
  description?: string;
  type?: string;
  url?: string;
  totalEpisodes?: number;
  status?: string;
  season?: string;
  duration?: string;
  malId?: string;
  anilistId?: string;
  hasSub?: boolean;
  hasDub?: boolean;
  subOrDub?: "sub" | "dub" | "both";
  genres?: string[];
  recommendations?: AnimeKaiRelatedItem[];
  relations?: AnimeKaiRelatedItem[];
  episodes: {
    id: string;
    number: number;
    title: string;
    isFiller: boolean;
    isSubbed: boolean;
    isDubbed: boolean;
    url: string;
  }[];
};
```

**Notes**

* `id` may include extra parts after `$`.
* The provider extracts the anime slug before the first `$`.
* `subOrDub` shows audio availability.

### Streaming and servers

Episode routes depend on the full `episodeId`.

Typical format:

```txt
anime-slug$ep=1$token=SOME_TOKEN
```

#### Stream sources

`GET /anime/animekai/watch/:episodeId`

Returns stream sources for one episode.

**Params**

* `episodeId` required
* `dub` optional
  * `"true"` or `"1"` selects dub
  * anything else selects softsub

**Validation**

* Missing `episodeId` returns `400`

**Behavior**

* The anime slug is derived from the part before the first `$`.
* The route delegates to `AnimeKai.streams(animeSlug, episodeId, subOrDub)`.

#### Episode servers

`GET /anime/animekai/servers/:episodeId`

Returns available hosts and intro or outro timing data.

**Params**

* `episodeId` required
* `dub` optional
  * `"true"` or `"1"` selects dub
  * anything else selects softsub

**Validation**

* Missing `episodeId` returns `400`

Example shape:

```json
{
  "servers": [
    {
      "name": "Server name",
      "url": "https://...",
      "intro": { "start": 0, "end": 90 },
      "outro": { "start": 1200, "end": 1300 }
    }
  ]
}
```

**Notes**

* Results come from `AnimeKai.fetchEpisodeServers(episodeId, subOrDub)`.
* Intro and outro offsets are useful for player skip controls.

### Error handling

If the upstream site changes, you may see:

* empty `results`
* empty schedule arrays
* zeroed pagination values

Known route-level validation:

* `/info` requires `id`
* `/watch/:episodeId` requires `episodeId`
* `/servers/:episodeId` requires `episodeId`

Upstream failures are logged internally and usually return safe fallback payloads.

### When to use Animekai

Animekai is a good fit when you need:

* richer browse flows
* spotlight and schedule data
* genre and type filters
* recommendation and relation blocks
* episode server metadata

Use it alongside other anime providers if you want broader source coverage.
