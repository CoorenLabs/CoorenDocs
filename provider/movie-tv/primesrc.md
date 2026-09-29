---
description: Direct MP4 links for movies and TV episodes by TMDB id, extracted from the file hosts PrimeSrc lists.
icon: clapperboard
---

# PrimeSrc

PrimeSrc uses [primesrc.me](https://primesrc.me), which lists the file hosts that carry a given movie or episode. Cooren asks it for that list by TMDB id, resolves each host it has an extractor for and returns direct stream links with a ready-to-play `proxiedUrl`. The link lookup is behind Cloudflare, so the first request after a restart solves a challenge in a browser.

{% hint style="info" %}
Base route: `/movie-tv/primesrc`
{% endhint %}

## Routes

| Route | Description |
| --- | --- |
| `GET /movie-tv/primesrc` | Lists the routes |
| `GET /movie-tv/primesrc/movie/:tmdbid` | Sources for a movie |
| `GET /movie-tv/primesrc/tv/:tmdbid/:season/:episode` | Sources for a TV episode |

## Supported hosts

PrimeSrc lists many hosts (Filemoon, Voe, Mixdrop, Luluvdoo, Streamwish and others). Cooren only resolves the ones it has an extractor for and skips the rest.

| Host | Stream type | Notes |
| --- | --- | --- |
| Streamtape | `mp4` | Returned sources for every title tested |
| Dood | `mp4` | Skipped when the file has been removed from the host. Every Dood file listed for the tested titles had been removed, so none came back |
| PrimeVid | `hls` | Has an extractor, but PrimeSrc did not list PrimeVid for any tested title |

## Response format

Both source routes return the same envelope. `data` holds one entry per host link that produced a stream, so the same host can appear several times with different files.

```json
{
  "success": true,
  "status": 200,
  "data": [
    {
      "name": "Streamtape",
      "sources": [],
      "subtitles": []
    }
  ]
}
```

| Field | Description |
| --- | --- |
| `name` | Host name |
| `sources[].type` | `mp4` or `hls` |
| `sources[].url` | Direct link on the host. It only plays with the `headers` below |
| `sources[].dub` | Always `Original Audio` |
| `sources[].headers` | Headers the host expects |
| `sources[].quality` | Vertical resolution, such as `1080`, taken from the file name. Missing when the file name has none |
| `sources[].sizeBytes` | File size in bytes, when PrimeSrc reports it |
| `sources[].proxiedUrl` | The same stream through the [stream proxy](../../core/proxy.md) with the headers attached. Use this in a browser player |
| `subtitles` | Always empty for Streamtape and Dood |

## Sources

### Movie sources

`GET /movie-tv/primesrc/movie/:tmdbid`

Returns every playable host link for one movie.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `tmdbid` | Yes | Numeric TMDB movie id, for example `550` |

**Example**

```bash
curl "http://localhost:3000/movie-tv/primesrc/movie/550"
```

```json
{
  "success": true,
  "status": 200,
  "data": [
    {
      "name": "Streamtape",
      "sources": [
        {
          "type": "mp4",
          "url": "https://861134162.tapecontent.net/radosgw/0ZqD3GkxGxUbPpV/N4rdS3broKG_.../Fight.Club.10th.Anniversary.Edition.1999.1080p.BrRip.x264.YIFY.mp4?stream=1",
          "dub": "Original Audio",
          "headers": {
            "origin": "https://streamta.site",
            "referer": "https://streamta.site/"
          },
          "quality": 1080,
          "sizeBytes": 1825361101,
          "proxiedUrl": "http://localhost:3000/proxy/mp4-proxy?url=https%3A%2F%2F861134162.tapecontent.net%2Fradosgw%2F...&headers=%7B%22origin%22%3A%22https%3A%2F%2Fstreamta.site%22%2C%22referer%22%3A%22https%3A%2F%2Fstreamta.site%2F%22%7D"
        }
      ],
      "subtitles": []
    },
    {
      "name": "Streamtape",
      "sources": [
        {
          "type": "mp4",
          "url": "https://861113629.tapecontent.net/radosgw/ZK0M0JP2dvhKgQ/zpw5KmGoytbm6.../Fight.Club.1999.REPACK.720p.BluRay.x264.AAC-%5BYTS.MX%5D.mp4?stream=1",
          "dub": "Original Audio",
          "headers": {
            "origin": "https://streamta.site",
            "referer": "https://streamta.site/"
          },
          "quality": 720,
          "sizeBytes": 1288490189,
          "proxiedUrl": "http://localhost:3000/proxy/mp4-proxy?url=https%3A%2F%2F861113629.tapecontent.net%2Fradosgw%2F...&headers=%7B%22origin%22%3A%22https%3A%2F%2Fstreamta.site%22%2C%22referer%22%3A%22https%3A%2F%2Fstreamta.site%2F%22%7D"
        }
      ],
      "subtitles": []
    }
  ]
}
```

### TV episode sources

`GET /movie-tv/primesrc/tv/:tmdbid/:season/:episode`

Returns every playable host link for one episode. The response has the same format as the movie route.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `tmdbid` | Yes | Numeric TMDB TV show id, for example `1399` |
| `season` | Yes | Season number |
| `episode` | Yes | Episode number |

**Example**

```bash
curl "http://localhost:3000/movie-tv/primesrc/tv/1399/1/1"
```

```json
{
  "success": true,
  "status": 200,
  "data": [
    {
      "name": "Streamtape",
      "sources": [
        {
          "type": "mp4",
          "url": "https://868418907.tapecontent.net/radosgw/ZVg6PMVb1gtqWQR/46RomZWSAc-u.../E01.720p.BluRay.500MB.ShAaNiG.com.mkv.mp4?stream=1",
          "dub": "Original Audio",
          "headers": {
            "origin": "https://streamta.site",
            "referer": "https://streamta.site/"
          },
          "quality": 720,
          "sizeBytes": 559939584,
          "proxiedUrl": "http://localhost:3000/proxy/mp4-proxy?url=https%3A%2F%2F868418907.tapecontent.net%2Fradosgw%2F...&headers=%7B%22origin%22%3A%22https%3A%2F%2Fstreamta.site%22%2C%22referer%22%3A%22https%3A%2F%2Fstreamta.site%2F%22%7D"
        }
      ],
      "subtitles": []
    }
  ]
}
```

## Errors

| Status | When |
| --- | --- |
| `404` | No supported host produced a stream. This covers unknown TMDB ids, episodes that do not exist, titles that only have unsupported hosts and upstream failures. The body is `{ "success": false, "status": 404 }` |
| `422` | `tmdbid`, `season` or `episode` is not a number |

## Notes

* The first request after a restart took about 14 seconds because a browser solved the Cloudflare challenge on primesrc.me. Later requests took 6 to 12 seconds, because host links are looked up one at a time with a short pause between them.
* If the link lookup is challenged again mid-request, the remaining hosts are skipped and only the sources found so far are returned.
* With Redis enabled, results are cached for 3 hours and misses for 10 minutes.
* PrimeSrc only takes TMDB ids. To start from an AniList, MAL or IMDb id, convert it with [ID Mappings](../../core/mappings.md).
