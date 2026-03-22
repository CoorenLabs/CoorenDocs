---
icon: clapperboard
---

# PrimeSrc

## PrimeSrc

PrimeSrc exposes source extraction routes for TMDB-backed movies and TV episodes.

Use it when you already know the TMDB ID and need direct host results.

{% hint style="info" %}
Base route: `GET /movie-tv/primesrc/...`
{% endhint %}

### Overview

PrimeSrc focuses on link extraction.

It:

* loads upstream server lists
* resolves supported hosts
* extracts sources and subtitles
* caches both hits and misses

Currently supported hosts:

* `PrimeVid`
* `Streamtape`
* `Dood`

Common response shape:

```ts
type ServerSource = {
  name: "PrimeVid" | "Streamtape" | "Dood";
  sources: any[];
  subtitles?: any[];
};

type PrimesrcResponse<T> = {
  success: boolean;
  status: number;
  data?: T;
};
```

### Status

#### Provider root

`GET /movie-tv/primesrc/`

Returns provider metadata and route hints.

Example shape:

```json
{
  "name": "Primesrc",
  "endpoints": [
    "/primesrc/movie/{tmdbId}",
    "/primesrc/tv/{tmdbId}/{season}/{episode}"
  ]
}
```

### Movie sources

#### Movie by TMDB ID

`GET /movie-tv/primesrc/movie/:tmdbid`

Resolves streaming sources for one movie.

**Params**

* `tmdbid` required. Numeric TMDB movie ID

Example shape:

```json
{
  "success": true,
  "status": 200,
  "data": [
    {
      "name": "PrimeVid",
      "sources": [],
      "subtitles": []
    },
    {
      "name": "Streamtape",
      "sources": []
    },
    {
      "name": "Dood",
      "sources": []
    }
  ]
}
```

If no supported result exists, the route returns:

```json
{
  "success": false,
  "status": 404
}
```

**How it works**

1. Calls the upstream movie server endpoint with the TMDB ID.
2. Reads the returned `servers[]`.
3. Skips unsupported hosts.
4. Resolves each supported server link.
5. Runs the correct extractor for that host.
6. Collects `sources` and optional `subtitles`.

**Caching**

* cache key: `primesrc:source:movie:{tmdbid}`
* hit TTL: 3 hours
* miss TTL: 10 minutes using a cached `NOT_FOUND` marker

### TV episode sources

#### Episode by TMDB ID

`GET /movie-tv/primesrc/tv/:tmdbid/:season/:episode`

Resolves streaming sources for one TV episode.

**Params**

* `tmdbid` required. Numeric TMDB show ID
* `season` required. Season number
* `episode` required. Episode number

The response shape matches the movie route.

**How it works**

1. Calls the upstream TV server endpoint with TMDB ID, season, and episode.
2. Uses the same server selection pipeline as the movie route.
3. Extracts supported hosts into `ServerSource[]`.

**Caching**

* cache key: `primesrc::source:tv:{tmdbid}:{season}:{episode}`
* hit TTL: 3 hours
* miss TTL: 10 minutes using `NOT_FOUND`

### Usage notes

PrimeSrc is not a browse provider.

You should pair it with a catalog source and a TMDB mapping flow.

Typical flow:

1. Find a title in your catalog provider.
2. Resolve its TMDB ID.
3. Call the PrimeSrc movie or TV route.
4. Present each host as a player option.

### Error model

Success responses use:

```json
{
  "success": true,
  "status": 200,
  "data": []
}
```

Misses return:

```json
{
  "success": false,
  "status": 404
}
```

Common failure cases:

* unsupported hosts only
* upstream server list empty
* extractor failure for all hosts
* cached `NOT_FOUND` result

PrimeSrc is a good fit when you need actual streaming hosts from a known TMDB movie or episode.
