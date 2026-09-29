---
description: Music search, metadata, editorial pages and playback manifests from Tidal.
icon: wave-square
---

# Tidal

Tidal calls Tidal's public v1 API (`api.tidal.com`) with the US catalog. It covers search, tracks, albums, artists, playlists, mixes and music videos, Tidal's home, charts and new-release pages, genres and moods. Metadata and 30-second previews work without an account; full-length audio and video need the session id of a Tidal account with a subscription.

{% hint style="info" %}
Base route: `/music/tidal`
{% endhint %}

## Routes

Every resource route answers on both the plural path shown here and the singular path: `/tracks/1550546` and `/track/1550546` are the same route, as are `/album`, `/artist`, `/playlist`, `/mix` and `/video` with any of their sub-routes. This page uses the plural form.

| Route | Description |
| --- | --- |
| `GET /music/tidal` | Lists the main routes |
| `GET /music/tidal/search` | Search tracks, albums, artists, playlists and videos |
| `GET /music/tidal/featured` | Tidal home page |
| `GET /music/tidal/charts` | Top charts page |
| `GET /music/tidal/new` | New releases page |
| `GET /music/tidal/genres` | List genres |
| `GET /music/tidal/genres/:path` | One genre |
| `GET /music/tidal/moods` | List moods |
| `GET /music/tidal/moods/:path` | One mood |
| `GET /music/tidal/recommendations` | Tracks similar to a track, with paging |
| `GET /music/tidal/tracks/:id` | Track details |
| `GET /music/tidal/tracks/:id/stream` | Track details with a preview manifest and, with a session, the full stream |
| `GET /music/tidal/tracks/:id/playbackinfo` | Raw playback info for a track |
| `GET /music/tidal/tracks/:id/radio` | Track radio |
| `GET /music/tidal/albums/:id` | Album details |
| `GET /music/tidal/albums/:id/tracks` | Album tracks |
| `GET /music/tidal/artists/:id` | Artist details |
| `GET /music/tidal/artists/:id/albums` | Artist albums |
| `GET /music/tidal/artists/:id/toptracks` | Artist top tracks |
| `GET /music/tidal/artists/:id/radio` | Artist radio |
| `GET /music/tidal/playlists/:id` | Playlist details |
| `GET /music/tidal/playlists/:id/tracks` | Playlist tracks |
| `GET /music/tidal/mixes/:id` | Mix details |
| `GET /music/tidal/mixes/:id/items` | Mix tracks |
| `GET /music/tidal/videos/:id` | Music video details |
| `GET /music/tidal/videos/:id/stream` | Video details with a preview manifest and, with a session, the full stream |

Every route except `/music/tidal` wraps its result the same way:

```json
{
  "status": 200,
  "success": true,
  "data": {}
}
```

Errors return `{ "status": 404, "success": false, "message": "Track not found or invalid ID", "data": null }`.

### Normalised items

Tracks, albums, artists, playlists, mixes and videos are passed through a cleaner that keeps Tidal's fields and adds a few so every item has the same basics:

| Field | Description |
| --- | --- |
| `artwork` | Full image URL built from Tidal's image id (`cover`, `squareImage`, `picture`, `imageId` and so on). Missing when the item has no image |
| `title`, `name` | Both set, copied from each other |
| `id` | Playlists get their `uuid` copied into `id` |
| `artists` | Always an array; filled from `artist` when Tidal only sends one |
| `album` | Always present |
| `duration`, `isrc`, `explicit` | Always present, defaulting to `0`, `""` and `false` |

When an item has no artists or album, the cleaner adds placeholders, so artists, albums and playlists carry `"artists": [{ "id": 0, "name": "Unknown Artist", "artwork": null }]` and `"album": { "id": 0, "title": "Unknown Album", "artwork": null }`. Treat `id: 0` as empty.

Genres, moods, radio routes and `playbackinfo` return Tidal's data without this cleaning.

### Paging

List routes that take `limit` and `offset` return Tidal's paging fields:

```json
{ "limit": 2, "offset": 1, "totalNumberOfItems": 13, "items": [] }
```

A `limit` or `offset` that is not a number falls back to the default.

## Search

### Search

`GET /music/tidal/search`

Searches the catalog. The result always has the five groups `artists`, `albums`, `playlists`, `tracks` and `videos`, plus `topHit` (the single best match and its group).

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `q` | Yes | | Search text. `query` is accepted as an alias |
| `limit` | No | `20` | Results per group |
| `types` | No | `TRACKS,ALBUMS,ARTISTS,PLAYLISTS,VIDEOS` | Comma-separated groups to search. Groups you leave out come back empty |

**Example**

```bash
curl "http://localhost:3000/music/tidal/search?q=get%20lucky&limit=2&types=TRACKS"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "artists": { "limit": 2, "offset": 0, "totalNumberOfItems": 0, "items": [] },
    "albums": { "limit": 2, "offset": 0, "totalNumberOfItems": 0, "items": [] },
    "playlists": { "limit": 2, "offset": 0, "totalNumberOfItems": 0, "items": [] },
    "tracks": {
      "limit": 2,
      "offset": 0,
      "totalNumberOfItems": 282,
      "items": [
        {
          "id": 20115564,
          "title": "Get Lucky",
          "duration": 370,
          "isrc": "USQX91300108",
          "explicit": false,
          "audioQuality": "LOSSLESS",
          "artists": [
            {
              "id": 8847,
              "name": "Daft Punk",
              "type": "MAIN",
              "picture": "65d23ddc-7b3b-4b17-bc84-b02832ddc204",
              "artwork": "https://resources.tidal.com/images/65d23ddc/7b3b/4b17/bc84/b02832ddc204/750x750.jpg"
            }
          ],
          "album": {
            "id": 20115556,
            "title": "Random Access Memories",
            "cover": "b66a5c40-c34d-4507-a0dc-5f98e46fdd20",
            "artwork": "https://resources.tidal.com/images/b66a5c40/c34d/4507/a0dc/5f98e46fdd20/640x640.jpg"
          },
          "artwork": "https://resources.tidal.com/images/b66a5c40/c34d/4507/a0dc/5f98e46fdd20/640x640.jpg",
          "name": "Get Lucky"
        }
      ]
    },
    "videos": { "limit": 2, "offset": 0, "totalNumberOfItems": 0, "items": [] },
    "topHit": {
      "value": {
        "id": 20115564,
        "title": "Get Lucky",
        "artwork": "https://resources.tidal.com/images/b66a5c40/c34d/4507/a0dc/5f98e46fdd20/640x640.jpg",
        "name": "Get Lucky"
      },
      "type": "TRACKS"
    }
  }
}
```

Track items in search have the same fields as [track details](#track-details); they are trimmed here.

## Discovery

### Home, charts and new releases

`GET /music/tidal/featured`

`GET /music/tidal/charts`

`GET /music/tidal/new`

Return Tidal's editorial pages: the home page (`featured`), the top charts page (`charts`) and the new releases page (`new`). A page has `rows`, each row has `modules`, and each module has a `type` (`PLAYLIST_LIST`, `TRACK_LIST`, `ALBUM_LIST`, `VIDEO_LIST`), a `title` and its items in `pagedList.items`. Items are normalised.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `deviceType` | No | `PHONE` | Layout to request. `PHONE`, `TABLET`, `BROWSER`, `DESKTOP` and `TV` all worked; any other value returns `404` |

**Example**

```bash
curl "http://localhost:3000/music/tidal/featured"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "selfLink": null,
    "id": "eyJwIjoiY2E5MWZhZGQtZTdhZC00NTgyLWEzYzgtYmUzYjllNmY0MTIzIiwicFYiOjJ9",
    "title": "Home",
    "rows": [
      {
        "modules": [
          {
            "id": "eyJwIjoiY2E5MWZhZGQtZTdhZC00NTgyLWEzYzgt...",
            "type": "PLAYLIST_LIST",
            "title": "The Hits",
            "description": "",
            "showMore": {
              "title": "View all",
              "apiPath": "pages/single-module-page/ca91fadd-e7ad-4582-a3c8-be3b9e6f412..."
            },
            "pagedList": {
              "dataApiPath": "pages/data/dba67408-bd39-46fb-99ac-f2593aa716bf",
              "limit": 15,
              "offset": 0,
              "totalNumberOfItems": 10,
              "items": [
                {
                  "uuid": "edf3b7d2-cb42-41d7-93c0-afa2a395521b",
                  "title": "TIDAL's Top Hits",
                  "type": "EDITORIAL",
                  "url": "https://tidal.com/browse/playlist/edf3b7d2-cb42-41d7-93c0-afa2a395521b",
                  "image": "73a09fe5-4d89-48da-939d-b41c97d61728",
                  "squareImage": "baf1bba0-e74f-471a-90ae-c8f623d376e3",
                  "duration": 19215,
                  "numberOfTracks": 100,
                  "id": "edf3b7d2-cb42-41d7-93c0-afa2a395521b",
                  "artwork": "https://resources.tidal.com/images/baf1bba0/e74f/471a/90ae/c8f623d376e3/640x640.jpg",
                  "name": "TIDAL's Top Hits"
                }
              ]
            }
          }
        ]
      },
      {
        "modules": [
          {
            "id": "eyJwIjoiY2E5MWZhZGQtZTdhZC00NTgyLWEzYzgt...",
            "type": "TRACK_LIST",
            "title": "New Tracks",
            "pagedList": {
              "dataApiPath": "pages/data/6eebf48e-fc39-4480-ad57-bdb217164971",
              "limit": 5,
              "offset": 0,
              "totalNumberOfItems": 257,
              "items": [
                {
                  "id": 564082204,
                  "title": "Patient Zero",
                  "duration": 226,
                  "artists": [
                    {
                      "id": 3557299,
                      "name": "Taylor Swift",
                      "type": "MAIN",
                      "artwork": "https://resources.tidal.com/images/58fac352/8af0/45b8/90e1/7183f2a6d04b/750x750.jpg"
                    }
                  ],
                  "album": {
                    "id": 564082191,
                    "title": "The Life of a Showgirl: The Encore",
                    "cover": "39e3dad3-33fb-47f2-87f8-8b6f8c55734c",
                    "artwork": "https://resources.tidal.com/images/39e3dad3/33fb/47f2/87f8/8b6f8c55734c/640x640.jpg"
                  },
                  "artwork": "https://resources.tidal.com/images/39e3dad3/33fb/47f2/87f8/8b6f8c55734c/640x640.jpg",
                  "name": "Patient Zero"
                }
              ]
            }
          }
        ]
      }
    ]
  }
}
```

`/charts` returned a page titled `Top` (modules `Top playlists`, `Top Tracks`, `Top Albums`, `The Hits`, `Top Artists`) and `/new` a page titled `New` (modules `New Arrivals`, `New Tracks`, `New Albums`, `New Music Videos`):

```bash
curl "http://localhost:3000/music/tidal/new"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "selfLink": null,
    "id": "eyJwIjoiNDdhZTMzMTctOTlhYi00YzgzLWJhNzQtN2UyMTlmNjg0MmY4IiwicFYiOjMwfQ==",
    "title": "New",
    "rows": [
      {
        "modules": [
          {
            "id": "eyJwIjoiNDdhZTMzMTctOTlhYi00YzgzLWJhNzQt...",
            "type": "ALBUM_LIST",
            "title": "New Albums",
            "pagedList": {
              "dataApiPath": "pages/data/ebf18430-2ad1-4603-a7ec-37bd285c363e",
              "limit": 12,
              "offset": 0,
              "totalNumberOfItems": 201,
              "items": [
                {
                  "id": 564082191,
                  "title": "The Life of a Showgirl: The Encore",
                  "duration": 3348,
                  "numberOfTracks": 16,
                  "releaseDate": "2026-09-25",
                  "artwork": "https://resources.tidal.com/images/39e3dad3/33fb/47f2/87f8/8b6f8c55734c/640x640.jpg",
                  "name": "The Life of a Showgirl: The Encore"
                }
              ]
            }
          }
        ]
      }
    ]
  }
}
```

### Genres and moods

`GET /music/tidal/genres`

`GET /music/tidal/moods`

List Tidal's genres (20 were returned) and moods (5 were returned). Use `path` from an entry with the single genre or mood route.

**Example**

```bash
curl "http://localhost:3000/music/tidal/genres"
```

```json
{
  "status": 200,
  "success": true,
  "data": [
    {
      "name": "Hip Hop / Rap",
      "path": "Hiphop",
      "hasPlaylists": true,
      "hasArtists": false,
      "hasAlbums": true,
      "hasTracks": true,
      "hasVideos": true,
      "image": "658519e4-8aea-41a8-86f9-e3fbfb31a630"
    },
    {
      "name": "R&B / Soul",
      "path": "Funk",
      "hasPlaylists": true,
      "hasArtists": false,
      "hasAlbums": true,
      "hasTracks": true,
      "hasVideos": true,
      "image": "b100bf56-2285-48ad-a608-386a7254fe8c"
    }
  ]
}
```

Moods have the same fields. The paths returned were `relax`, `party`, `workout`, `love` and `concentrate`.

`GET /music/tidal/genres/:path`

`GET /music/tidal/moods/:path`

Return a single genre or mood entry.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `path` | Yes | The `path` from the list, for example `Hiphop` or `relax`. Case-sensitive |

**Example**

```bash
curl "http://localhost:3000/music/tidal/moods/relax"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "name": "Relax",
    "path": "relax",
    "hasPlaylists": true,
    "hasArtists": false,
    "hasAlbums": false,
    "hasTracks": false,
    "hasVideos": false,
    "image": "b589ddb1-ef3a-457e-9e00-84b475196ee2"
  }
}
```

An unknown path returns `404` with Tidal's message, such as `GENRES [NotAGenre] not found`.

### Recommendations

`GET /music/tidal/recommendations`

Returns tracks similar to a track (Tidal's track radio), with paging and normalised items.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `trackId` | Yes | | Tidal track id. `id` is accepted as an alias |
| `limit` | No | `50` | Tracks per page |
| `offset` | No | `0` | Tracks to skip |

**Example**

```bash
curl "http://localhost:3000/music/tidal/recommendations?trackId=1550546&limit=2"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "limit": 2,
    "offset": 0,
    "totalNumberOfItems": 95,
    "items": [
      {
        "id": 1550546,
        "title": "One More Time",
        "duration": 320,
        "trackNumber": 1,
        "volumeNumber": 1,
        "url": "http://www.tidal.com/track/1550546",
        "isrc": "GBDUW0000053",
        "explicit": false,
        "audioQuality": "LOSSLESS",
        "artwork": "https://resources.tidal.com/images/7d3b9810/5634/400c/ad89/50609e0ce800/640x640.jpg",
        "name": "One More Time"
      }
    ]
  }
}
```

## Tracks

### Track details

`GET /music/tidal/tracks/:id`

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Tidal track id, for example `1550546` |

**Example**

```bash
curl "http://localhost:3000/music/tidal/tracks/1550546"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "id": 1550546,
    "title": "One More Time",
    "duration": 320,
    "replayGain": -6.83,
    "peak": 0.979767,
    "allowStreaming": true,
    "streamReady": true,
    "payToStream": false,
    "adSupportedStreamReady": true,
    "djReady": true,
    "stemReady": false,
    "streamStartDate": "2005-01-24T00:00:00.000+0000",
    "premiumStreamingOnly": false,
    "trackNumber": 1,
    "volumeNumber": 1,
    "version": null,
    "popularity": 88,
    "copyright": "℗ 2001 Daft Life Limited",
    "bpm": 123,
    "key": "G",
    "keyScale": "MAJOR",
    "url": "http://www.tidal.com/track/1550546",
    "isrc": "GBDUW0000053",
    "editable": false,
    "explicit": false,
    "audioQuality": "LOSSLESS",
    "audioModes": ["STEREO"],
    "mediaMetadata": { "tags": ["LOSSLESS"] },
    "upload": false,
    "accessType": "PUBLIC",
    "spotlighted": false,
    "ai": false,
    "artist": {
      "id": 8847,
      "name": "Daft Punk",
      "handle": null,
      "type": "MAIN",
      "picture": "c92cf3f5-066f-4f0a-87d0-c2bebff46d36"
    },
    "artists": [
      {
        "id": 8847,
        "name": "Daft Punk",
        "handle": null,
        "type": "MAIN",
        "picture": "c92cf3f5-066f-4f0a-87d0-c2bebff46d36",
        "artwork": "https://resources.tidal.com/images/c92cf3f5/066f/4f0a/87d0/c2bebff46d36/750x750.jpg",
        "title": "Daft Punk",
        "duration": 0,
        "isrc": "",
        "explicit": false,
        "artists": [{ "id": 0, "name": "Unknown Artist", "artwork": null }],
        "album": { "id": 0, "title": "Unknown Album", "artwork": null }
      }
    ],
    "album": {
      "id": 1550545,
      "title": "Discovery",
      "cover": "7d3b9810-5634-400c-ad89-50609e0ce800",
      "vibrantColor": "#dca60f",
      "videoCover": null,
      "artwork": "https://resources.tidal.com/images/7d3b9810/5634/400c/ad89/50609e0ce800/640x640.jpg",
      "name": "Discovery",
      "duration": 0,
      "isrc": "",
      "explicit": false,
      "artists": [{ "id": 0, "name": "Unknown Artist", "artwork": null }],
      "album": { "id": 0, "title": "Unknown Album", "artwork": null }
    },
    "mixes": { "TRACK_MIX": "001862b4e8a51e878d8ca2a5f06f01" },
    "artwork": "https://resources.tidal.com/images/7d3b9810/5634/400c/ad89/50609e0ce800/640x640.jpg",
    "name": "One More Time"
  }
}
```

`mixes.TRACK_MIX` is a mix id you can pass to the [mix routes](#mixes).

### Track stream

`GET /music/tidal/tracks/:id/stream`

Returns the track details plus two playback objects:

* `preview`: a 30-second preview at `LOW` quality. It works without a session.
* `audio`: the full-length track at `audioQuality`. It needs a Tidal session; without one it is `{ "error": "Session ID required for full audio", "status": 401 }`.

Both hold Tidal's playback info with the base64 `manifest` and a `manifestDecoded` copy. For tracks the manifest is a DASH MPD (`manifestMimeType` `application/dash+xml`) whose segment URLs you can play with a DASH player.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Tidal track id |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `audioQuality` | No | `HI_RES` | Quality of the full-length `audio`: `LOW`, `HIGH`, `LOSSLESS`, `HI_RES` or `HI_RES_LOSSLESS`. The preview always uses `LOW` |
| `sessionId` | No | | Tidal session id for the full-length `audio`. The `x-tidal-sessionid` header does the same and takes priority |

**Headers**

| Name | Required | Description |
| --- | --- | --- |
| `x-tidal-sessionid` | No | Tidal session id for the full-length `audio` |

**Example**

```bash
curl "http://localhost:3000/music/tidal/tracks/1550546/stream"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "id": 1550546,
    "title": "One More Time",
    "duration": 320,
    "isrc": "GBDUW0000053",
    "audioQuality": "LOSSLESS",
    "artwork": "https://resources.tidal.com/images/7d3b9810/5634/400c/ad89/50609e0ce800/640x640.jpg",
    "name": "One More Time",
    "preview": {
      "trackId": 1550546,
      "assetPresentation": "PREVIEW",
      "audioMode": "STEREO",
      "audioQuality": "LOW",
      "manifestMimeType": "application/dash+xml",
      "manifestHash": "nHQAxK2oQxxZDoX9h/qA8t6pafVLGkf5ReQlScouI3M=",
      "manifest": "PD94bWwgdmVyc2lvbj0nMS4wJyBlbmNvZGluZz0nVVRGLTgnPz48TVBEIHht...",
      "albumReplayGain": -6.83,
      "albumPeakAmplitude": 1,
      "trackReplayGain": -5.83,
      "trackPeakAmplitude": 0.979767,
      "previewReason": "FULL_REQUIRES_SUBSCRIPTION",
      "manifestDecoded": "<?xml version='1.0' encoding='UTF-8'?><MPD xmlns=\"urn:mpeg:dash:schema:mpd:2011\" xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\" xmlns:xlink=\"http://www.w3.org/1999/xlink\" xmlns:cenc=\"urn:mpeg:c..."
    },
    "audio": { "error": "Session ID required for full audio", "status": 401 }
  }
}
```

The track fields at the top of `data` are the same as [track details](#track-details); they are trimmed here.

With a session:

```bash
curl -H "x-tidal-sessionid: <your session id>" "http://localhost:3000/music/tidal/tracks/1550546/stream?audioQuality=LOSSLESS"
```

{% hint style="warning" %}
Full-length playback with a valid session could not be verified on 2026-09-29. With an invalid session id, `audio` was `{ "error": "Asset is not ready for playback", "status": 401 }` and `preview` still worked.
{% endhint %}

### Playback info

`GET /music/tidal/tracks/:id/playbackinfo`

Returns Tidal's playback info for the track without a session, so it is always a preview (`assetPresentation` `PREVIEW`). Unlike `/stream`, there is no `manifestDecoded` and no track metadata.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Tidal track id |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `audioQuality` | No | `HI_RES` | `LOW`, `HIGH`, `LOSSLESS`, `HI_RES` or `HI_RES_LOSSLESS`. Previews are capped at `LOSSLESS`. An unknown value returns `404` |

**Example**

```bash
curl "http://localhost:3000/music/tidal/tracks/1550546/playbackinfo"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "trackId": 1550546,
    "assetPresentation": "PREVIEW",
    "audioMode": "STEREO",
    "audioQuality": "LOSSLESS",
    "manifestMimeType": "application/dash+xml",
    "manifestHash": "+B1SqQMhwatwtVHt4cKaqpntbhCkChjcaLWG00CrEnw=",
    "manifest": "PD94bWwgdmVyc2lvbj0nMS4wJyBlbmNvZGluZz0nVVRGLTgnPz48TVBEIHht...",
    "albumReplayGain": -6.83,
    "albumPeakAmplitude": 1,
    "trackReplayGain": -5.83,
    "trackPeakAmplitude": 0.979767,
    "bitDepth": 16,
    "sampleRate": 44100,
    "previewReason": "FULL_REQUIRES_SUBSCRIPTION"
  }
}
```

### Track radio

`GET /music/tidal/tracks/:id/radio`

Returns the first 10 tracks of the track's radio, as Tidal sends them (not normalised). Use [recommendations](#recommendations) for paging and normalised items.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Tidal track id |

**Example**

```bash
curl "http://localhost:3000/music/tidal/tracks/1550546/radio"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "limit": 10,
    "offset": 0,
    "totalNumberOfItems": 95,
    "items": [
      {
        "id": 1550546,
        "title": "One More Time",
        "duration": 320,
        "trackNumber": 1,
        "volumeNumber": 1,
        "url": "http://www.tidal.com/track/1550546",
        "isrc": "GBDUW0000053",
        "explicit": false,
        "audioQuality": "LOSSLESS",
        "artist": {
          "id": 8847,
          "name": "Daft Punk",
          "type": "MAIN",
          "picture": "c92cf3f5-066f-4f0a-87d0-c2bebff46d36"
        }
      }
    ]
  }
}
```

## Albums

### Album details

`GET /music/tidal/albums/:id`

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Tidal album id, for example `20115556` |

**Example**

```bash
curl "http://localhost:3000/music/tidal/albums/20115556"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "id": 20115556,
    "title": "Random Access Memories",
    "duration": 4479,
    "streamReady": true,
    "allowStreaming": true,
    "numberOfTracks": 13,
    "numberOfVideos": 0,
    "numberOfVolumes": 1,
    "releaseDate": "2013-05-20",
    "copyright": "(P)  2013 Daft Life Limited under exclusive license to Columbia Records, a Division of Sony Music En...",
    "type": "ALBUM",
    "version": null,
    "url": "http://www.tidal.com/album/20115556",
    "cover": "b66a5c40-c34d-4507-a0dc-5f98e46fdd20",
    "vibrantColor": "#efdd83",
    "videoCover": null,
    "explicit": false,
    "upc": "886443927087",
    "popularity": 84,
    "audioQuality": "LOSSLESS",
    "audioModes": ["STEREO"],
    "mediaMetadata": { "tags": ["LOSSLESS"] },
    "artist": {
      "id": 8847,
      "name": "Daft Punk",
      "handle": null,
      "type": "MAIN",
      "picture": "c92cf3f5-066f-4f0a-87d0-c2bebff46d36"
    },
    "artists": [
      {
        "id": 8847,
        "name": "Daft Punk",
        "type": "MAIN",
        "picture": "c92cf3f5-066f-4f0a-87d0-c2bebff46d36",
        "artwork": "https://resources.tidal.com/images/c92cf3f5/066f/4f0a/87d0/c2bebff46d36/750x750.jpg"
      }
    ],
    "artwork": "https://resources.tidal.com/images/b66a5c40/c34d/4507/a0dc/5f98e46fdd20/640x640.jpg",
    "name": "Random Access Memories",
    "isrc": "",
    "album": { "id": 0, "title": "Unknown Album", "artwork": null }
  }
}
```

### Album tracks

`GET /music/tidal/albums/:id/tracks`

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Tidal album id |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `limit` | No | `50` | Tracks per page |
| `offset` | No | `0` | Tracks to skip |

**Example**

```bash
curl "http://localhost:3000/music/tidal/albums/20115556/tracks?limit=2&offset=1"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "limit": 2,
    "offset": 1,
    "totalNumberOfItems": 13,
    "items": [
      {
        "id": 20115558,
        "title": "The Game of Love",
        "duration": 322,
        "trackNumber": 2,
        "volumeNumber": 1,
        "url": "http://www.tidal.com/track/20115558",
        "isrc": "USQX91300102",
        "explicit": false,
        "audioQuality": "LOSSLESS",
        "artwork": "https://resources.tidal.com/images/b66a5c40/c34d/4507/a0dc/5f98e46fdd20/640x640.jpg",
        "name": "The Game of Love"
      }
    ]
  }
}
```

## Artists

### Artist details

`GET /music/tidal/artists/:id`

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Tidal artist id, for example `8847` |

**Example**

```bash
curl "http://localhost:3000/music/tidal/artists/8847"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "id": 8847,
    "name": "Daft Punk",
    "artistTypes": ["ARTIST", "CONTRIBUTOR"],
    "url": "http://www.tidal.com/artist/8847",
    "picture": "65d23ddc-7b3b-4b17-bc84-b02832ddc204",
    "selectedAlbumCoverFallback": null,
    "popularity": 91,
    "artistRoles": [{ "categoryId": -1, "category": "Artist" }],
    "mixes": { "ARTIST_MIX": "0002737adb8d077f4591c4cdb7ba60" },
    "handle": null,
    "userId": null,
    "spotlighted": false,
    "artwork": "https://resources.tidal.com/images/65d23ddc/7b3b/4b17/bc84/b02832ddc204/750x750.jpg",
    "title": "Daft Punk",
    "duration": 0,
    "isrc": "",
    "explicit": false,
    "artists": [{ "id": 0, "name": "Unknown Artist", "artwork": null }],
    "album": { "id": 0, "title": "Unknown Album", "artwork": null }
  }
}
```

### Artist albums

`GET /music/tidal/artists/:id/albums`

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Tidal artist id |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `limit` | No | `50` | Albums per page |
| `offset` | No | `0` | Albums to skip |

**Example**

```bash
curl "http://localhost:3000/music/tidal/artists/8847/albums?limit=2"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "limit": 2,
    "offset": 0,
    "totalNumberOfItems": 16,
    "items": [
      {
        "id": 328631591,
        "title": "Random Access Memories (Drumless Edition)",
        "duration": 4475,
        "numberOfTracks": 13,
        "releaseDate": "2013-05-20",
        "type": "ALBUM",
        "url": "http://www.tidal.com/album/328631591",
        "explicit": false,
        "audioQuality": "LOW",
        "artwork": "https://resources.tidal.com/images/3ca5a58c/5ce7/4343/a8cd/dca24fcba071/640x640.jpg",
        "name": "Random Access Memories (Drumless Edition)",
        "isrc": ""
      }
    ]
  }
}
```

### Artist top tracks

`GET /music/tidal/artists/:id/toptracks`

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Tidal artist id |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `limit` | No | `10` | Tracks per page |
| `offset` | No | `0` | Tracks to skip |

**Example**

```bash
curl "http://localhost:3000/music/tidal/artists/8847/toptracks?limit=2"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "limit": 2,
    "offset": 0,
    "totalNumberOfItems": 216,
    "items": [
      {
        "id": 1550546,
        "title": "One More Time",
        "duration": 320,
        "trackNumber": 1,
        "popularity": 88,
        "url": "http://www.tidal.com/track/1550546",
        "isrc": "GBDUW0000053",
        "explicit": false,
        "audioQuality": "LOSSLESS",
        "artwork": "https://resources.tidal.com/images/7d3b9810/5634/400c/ad89/50609e0ce800/640x640.jpg",
        "name": "One More Time"
      }
    ]
  }
}
```

### Artist radio

`GET /music/tidal/artists/:id/radio`

Returns the first 10 tracks of the artist's radio, as Tidal sends them (not normalised).

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Tidal artist id |

**Example**

```bash
curl "http://localhost:3000/music/tidal/artists/8847/radio"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "limit": 10,
    "offset": 0,
    "totalNumberOfItems": 99,
    "items": [
      {
        "id": 1550546,
        "title": "One More Time",
        "duration": 320,
        "url": "http://www.tidal.com/track/1550546",
        "isrc": "GBDUW0000053",
        "audioQuality": "LOSSLESS",
        "artist": { "id": 8847, "name": "Daft Punk", "type": "MAIN" }
      }
    ]
  }
}
```

## Playlists

### Playlist details

`GET /music/tidal/playlists/:id`

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Tidal playlist UUID, for example `abd35446-6a67-4f2b-9f92-0636742c63cb` |

**Example**

```bash
curl "http://localhost:3000/music/tidal/playlists/abd35446-6a67-4f2b-9f92-0636742c63cb"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "uuid": "abd35446-6a67-4f2b-9f92-0636742c63cb",
    "title": "Daft Punk Essentials",
    "numberOfTracks": 25,
    "numberOfVideos": 0,
    "creator": { "id": 0 },
    "description": "Daft Punk called it quits in 2021 as one of the most beloved acts of the 21st century. They emerged ...",
    "duration": 8038,
    "lastUpdated": "2023-09-03T20:38:45.303+0000",
    "created": "2014-09-30T15:27:05.510+0000",
    "type": "EDITORIAL",
    "publicPlaylist": true,
    "url": "http://www.tidal.com/playlist/abd35446-6a67-4f2b-9f92-0636742c63cb",
    "image": "cc169813-ac53-4bfe-9164-304de2b6f1c6",
    "popularity": 0,
    "squareImage": "2a1eae8e-49e9-4d79-bb00-956a242eddd6",
    "customImageUrl": null,
    "promotedArtists": [{ "id": 8847, "name": "Daft Punk", "handle": null, "type": "MAIN", "picture": null }],
    "lastItemAddedAt": "2023-09-03T20:37:45.344+0000",
    "id": "abd35446-6a67-4f2b-9f92-0636742c63cb",
    "artwork": "https://resources.tidal.com/images/2a1eae8e/49e9/4d79/bb00/956a242eddd6/640x640.jpg",
    "name": "Daft Punk Essentials",
    "isrc": "",
    "explicit": false,
    "artists": [{ "id": 0, "name": "Unknown Artist", "artwork": null }],
    "album": { "id": 0, "title": "Unknown Album", "artwork": null }
  }
}
```

### Playlist tracks

`GET /music/tidal/playlists/:id/tracks`

Each entry keeps Tidal's wrapper (`item`, `type`, `cut`) and also has the track's fields copied to the top level, so you can read `id`, `title` and `artwork` directly.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Tidal playlist UUID |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `limit` | No | `50` | Items per page |
| `offset` | No | `0` | Items to skip |

**Example**

```bash
curl "http://localhost:3000/music/tidal/playlists/abd35446-6a67-4f2b-9f92-0636742c63cb/tracks?limit=2"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "limit": 2,
    "offset": 0,
    "totalNumberOfItems": 25,
    "items": [
      {
        "item": {
          "id": 1550546,
          "title": "One More Time",
          "duration": 320,
          "url": "http://www.tidal.com/track/1550546",
          "isrc": "GBDUW0000053",
          "dateAdded": "2014-10-21T21:23:17.140+0000",
          "index": 1171,
          "itemUuid": "a7494e4a-8962-4f3c-8e8e-c87c9c59f816",
          "artwork": "https://resources.tidal.com/images/7d3b9810/5634/400c/ad89/50609e0ce800/640x640.jpg",
          "name": "One More Time"
        },
        "type": "track",
        "cut": null,
        "id": 1550546,
        "title": "One More Time",
        "duration": 320,
        "url": "http://www.tidal.com/track/1550546",
        "isrc": "GBDUW0000053",
        "dateAdded": "2014-10-21T21:23:17.140+0000",
        "index": 1171,
        "itemUuid": "a7494e4a-8962-4f3c-8e8e-c87c9c59f816",
        "artwork": "https://resources.tidal.com/images/7d3b9810/5634/400c/ad89/50609e0ce800/640x640.jpg",
        "name": "One More Time"
      }
    ]
  }
}
```

## Mixes

### Mix details

`GET /music/tidal/mixes/:id`

Returns a mix's header: title, subtitle, colours and images. Mix ids come from the `mixes` field of tracks (`TRACK_MIX`) and artists (`ARTIST_MIX`).

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Tidal mix id, for example `001862b4e8a51e878d8ca2a5f06f01` |

**Example**

```bash
curl "http://localhost:3000/music/tidal/mixes/001862b4e8a51e878d8ca2a5f06f01"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "id": "001862b4e8a51e878d8ca2a5f06f01",
    "title": "One More Time",
    "subTitle": "Daft Punk",
    "description": null,
    "graphic": {
      "type": "SQUARES_GRID",
      "text": "One More Time",
      "images": [{ "id": "dummy-placeholder", "vibrantColor": "#FFFFFF", "type": "ARTIST" }]
    },
    "images": {
      "SMALL": {
        "width": 320,
        "height": 320,
        "url": "https://images.tidal.com/0/EMACGMACIKABKKAB/CAEQBhokN2QzYjk4MTAvNTYzNC80MDBjL2FkODkvNTA2MDllMGNlODAw..."
      },
      "LARGE": {
        "width": 1500,
        "height": 1500,
        "url": "https://images.tidal.com/0/ENwLGNwLIO4FKO4F/CAEQBhokN2QzYjk4MTAvNTYzNC80MDBjL2FkODkvNTA2MDllMGNlODAw..."
      }
    },
    "sharingImages": null,
    "mixType": "TRACK_MIX",
    "mixNumber": null,
    "contentBehavior": "UNRESTRICTED",
    "master": false,
    "titleColor": "#DCB958",
    "subTitleColor": "#DCB958",
    "descriptionColor": null,
    "shortSubtitle": "Created by TIDAL",
    "artwork": "https://images.tidal.com/0/ENwLGNwLIO4FKO4F/CAEQBhokN2QzYjk4MTAvNTYzNC80MDBjL2FkODkvNTA2MDllMGNlODAw...",
    "name": "One More Time",
    "duration": 0,
    "isrc": "",
    "explicit": false,
    "artists": [{ "id": 0, "name": "Unknown Artist", "artwork": null }],
    "album": { "id": 0, "title": "Unknown Album", "artwork": null }
  }
}
```

`images` also has a `MEDIUM` size, and `detailImages` repeats the same three sizes.

### Mix items

`GET /music/tidal/mixes/:id/items`

Returns the tracks in a mix. Entries have the same wrapper as [playlist tracks](#playlist-tracks), without `cut`.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Tidal mix id |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `limit` | No | `50` | Items per page |
| `offset` | No | `0` | Items to skip |

Tidal ignored `limit` and `offset` for the tested mix and always returned all 95 items.

**Example**

```bash
curl "http://localhost:3000/music/tidal/mixes/001862b4e8a51e878d8ca2a5f06f01/items"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "limit": 95,
    "offset": 0,
    "totalNumberOfItems": 95,
    "items": [
      {
        "item": {
          "id": 1550546,
          "title": "One More Time",
          "duration": 320,
          "url": "http://www.tidal.com/track/1550546",
          "isrc": "GBDUW0000053",
          "artwork": "https://resources.tidal.com/images/7d3b9810/5634/400c/ad89/50609e0ce800/640x640.jpg",
          "name": "One More Time"
        },
        "type": "track",
        "id": 1550546,
        "title": "One More Time",
        "duration": 320,
        "url": "http://www.tidal.com/track/1550546",
        "isrc": "GBDUW0000053",
        "artwork": "https://resources.tidal.com/images/7d3b9810/5634/400c/ad89/50609e0ce800/640x640.jpg",
        "name": "One More Time"
      }
    ]
  }
}
```

## Videos

### Video details

`GET /music/tidal/videos/:id`

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Tidal video id, for example `44187439` |

**Example**

```bash
curl "http://localhost:3000/music/tidal/videos/44187439"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "id": 44187439,
    "title": "Around the World",
    "volumeNumber": 0,
    "trackNumber": 0,
    "releaseDate": "2005-05-06T00:00:00.000+0000",
    "imagePath": null,
    "imageId": "30bfadb8-e012-4baa-a712-1a62ab404f7d",
    "vibrantColor": "#e3b4c3",
    "duration": 242,
    "quality": "MP4_1080P",
    "streamReady": true,
    "allowStreaming": true,
    "explicit": false,
    "popularity": 57,
    "type": "Music Video",
    "adsUrl": null,
    "adsPrePaywallOnly": true,
    "artist": {
      "id": 8847,
      "name": "Daft Punk",
      "handle": null,
      "type": "MAIN",
      "picture": "c92cf3f5-066f-4f0a-87d0-c2bebff46d36"
    },
    "artists": [
      {
        "id": 8847,
        "name": "Daft Punk",
        "type": "MAIN",
        "picture": "c92cf3f5-066f-4f0a-87d0-c2bebff46d36",
        "artwork": "https://resources.tidal.com/images/c92cf3f5/066f/4f0a/87d0/c2bebff46d36/750x750.jpg"
      }
    ],
    "album": { "id": 0, "title": "Unknown Album", "artwork": null },
    "artwork": "https://resources.tidal.com/images/30bfadb8/e012/4baa/a712/1a62ab404f7d/640x640.jpg",
    "name": "Around the World",
    "isrc": ""
  }
}
```

### Video stream

`GET /music/tidal/videos/:id/stream`

Works like [track stream](#track-stream): video details plus a `LOW` quality `preview` that works without a session, and the full-length video, which needs a session. The full video is returned under the `audio` key, like tracks; without a session it is `{ "error": "Session ID required for full video", "status": 401 }`. The video manifest decodes to JSON with HLS playlist URLs (`application/vnd.apple.mpegurl`).

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Tidal video id |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `quality` | No | `HIGH` | Video quality of the full-length stream, sent to Tidal as `videoquality`. The preview always uses `LOW` |
| `sessionId` | No | | Tidal session id for the full-length video. The `x-tidal-sessionid` header does the same and takes priority |

**Headers**

| Name | Required | Description |
| --- | --- | --- |
| `x-tidal-sessionid` | No | Tidal session id for the full-length video |

**Example**

```bash
curl "http://localhost:3000/music/tidal/videos/44187439/stream"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "id": 44187439,
    "title": "Around the World",
    "duration": 242,
    "quality": "MP4_1080P",
    "artwork": "https://resources.tidal.com/images/30bfadb8/e012/4baa/a712/1a62ab404f7d/640x640.jpg",
    "name": "Around the World",
    "preview": {
      "videoId": 44187439,
      "streamType": "ON_DEMAND",
      "assetPresentation": "PREVIEW",
      "videoQuality": "LOW",
      "manifestMimeType": "application/vnd.tidal.emu",
      "manifestHash": "IHlbEMJ6vj+gPYDJi1lL5mm9EsZ5DZUbJCHvU18nCZk=",
      "manifest": "eyJtaW1lVHlwZSI6ImFwcGxpY2F0aW9uL3ZuZC5hcHBsZS5tcGVndXJsIiwi...",
      "manifestDecoded": "{\"mimeType\":\"application/vnd.apple.mpegurl\",\"urls\":[\"https://im-fa.manifest.tidal.com/1/manifests/CAESCDQ0MTg3NDM5IhZkNmNucGx1ZWhNZlE1M2c0d0pNTnFBIhZE...\"]}"
    },
    "audio": { "error": "Session ID required for full video", "status": 401 }
  }
}
```

## Errors

| Status | When |
| --- | --- |
| `400` | `/search` without `q` (`Query parameter 'q' is required`), or `/recommendations` without `trackId` (`Query parameter 'trackId' is required`) |
| `404` | The id or path does not exist. Track, album, artist, playlist, mix and video routes use their own message, such as `Track not found or invalid ID` or `Album not found`; the others pass on Tidal's message, such as `Resource not found`. An unknown `deviceType` or `audioQuality` also returns `404` |
| `502` | Tidal could not be reached or returned a server error |
| `504` | Tidal did not answer within 15 seconds |

Other `4xx` statuses from Tidal are passed through with Tidal's message.

## Notes

* Requests go straight to Tidal's API with no browser, so most routes answer in under a second.
* The session id only changes the `audio` part of the `/stream` routes. Every other route, including `playbackinfo`, ignores it.
* All lookups use the US catalog, so availability follows Tidal's US region.
* Image ids such as `cover`, `picture` and `squareImage` are Tidal image UUIDs; use the `artwork` URL the cleaner adds instead of building image URLs yourself.
