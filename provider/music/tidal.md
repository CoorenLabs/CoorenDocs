---
icon: wave-square
---

# Tidal

## Tidal

Tidal exposes music search, metadata, editorial discovery, and playback manifest endpoints.

Use it when you need tracks, albums, artists, playlists, mixes, videos, and stream metadata from one provider.

{% hint style="info" %}
Base route: `GET /music/tidal/...`
{% endhint %}

### Overview

Tidal supports:

* global search
* featured, charts, and new pages
* genres and moods
* track recommendations
* track metadata and streaming info
* album metadata and track lists
* artist metadata, albums, top tracks, and radio
* playlist metadata and tracks
* mix metadata and items
* video metadata and streaming info

Most resource routes support both singular and plural aliases:

* `/track/:id` and `/tracks/:id`
* `/album/:id` and `/albums/:id`
* `/artist/:id` and `/artists/:id`
* `/playlist/:id` and `/playlists/:id`
* `/mix/:id` and `/mixes/:id`
* `/video/:id` and `/videos/:id`

Common response envelope:

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

`GET /music/tidal/`

Returns provider status and a quick route index.

Example shape:

```json
{
  "provider": "Tidal",
  "status": "operational",
  "description": "High-fidelity music streaming API with comprehensive metadata",
  "endpoints": [
    "GET /tidal/search?q=...&limit=20   → Search everything",
    "GET /tidal/tracks/:id             → Track details & metadata",
    "GET /tidal/tracks/:id/stream      → DASH preview & full audio",
    "GET /tidal/albums/:id             → Album details",
    "GET /tidal/artists/:id            → Artist details",
    "GET /tidal/featured               → Home spotlights",
    "GET /tidal/videos/:id/stream      → High-quality video"
  ],
  "note": "All resource endpoints support both singular and plural"
}
```

### Search and discovery

#### Global search

`GET /music/tidal/search`

Searches across multiple Tidal resource types.

**Query params**

* `q` required
* `query` optional alias for `q`
* `limit` optional. Default is `20`
* `types` optional. Default is `TRACKS,ALBUMS,ARTISTS,PLAYLISTS,VIDEOS`

If `q` is missing, the route returns `400`.

Example error:

```json
{ "status": 400, "success": false, "message": "Query parameter 'q' is required", "data": null }
```

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "topHit": { "value": {} },
    "tracks": { "items": [], "totalNumberOfItems": 50 },
    "albums": { "items": [], "totalNumberOfItems": 20 },
    "artists": { "items": [], "totalNumberOfItems": 15 },
    "playlists": { "items": [], "totalNumberOfItems": 10 },
    "videos": { "items": [], "totalNumberOfItems": 5 }
  }
}
```

**Metadata cleanup**

Search results are normalized before return.

That cleanup:

* resolves artwork fields into one `artwork` URL
* normalizes nested album data
* ensures `artists` is always an array
* fills sensible fallback values for missing artist or album data
* prefers `uuid` as `id` when needed

#### Editorial pages

These routes expose Tidal editorial page layouts:

**Featured**

`GET /music/tidal/featured`

**Query params**

* `deviceType` optional. Default is `PHONE`

**Charts**

`GET /music/tidal/charts`

**New releases**

`GET /music/tidal/new`

These routes clean nested page modules before return.

Use them for:

* home screens
* charts carousels
* new release sections

#### Genres and moods

**List genres**

`GET /music/tidal/genres`

**Genre page**

`GET /music/tidal/genres/:path`

**Params**

* `path` required

**List moods**

`GET /music/tidal/moods`

**Mood page**

`GET /music/tidal/moods/:path`

**Params**

* `path` required

These routes return Tidal page-style collections for that segment.

#### Recommendations

`GET /music/tidal/recommendations`

Returns similar tracks for a seed track.

**Query params**

* `trackId` required, or `id`
* `limit` optional. Default is `50`

Use this for:

* related tracks
* autoplay queues
* simple radio generation

### Tracks

Track routes work with both `/track/:id` and `/tracks/:id`.

#### Track details

`GET /music/tidal/tracks/:id`

Returns cleaned track metadata.

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "id": 123456,
    "title": "Song Title",
    "name": "Song Title",
    "duration": 210,
    "isrc": "US1234567890",
    "explicit": false,
    "artwork": "https://resources.tidal.com/images/.../640x640.jpg",
    "album": {
      "id": 98765,
      "title": "Album Name",
      "artwork": "https://resources.tidal.com/images/.../640x640.jpg"
    },
    "artists": [
      {
        "id": 54321,
        "name": "Main Artist",
        "artwork": "https://resources.tidal.com/images/.../640x640.jpg"
      }
    ]
  }
}
```

Invalid or missing track IDs return `404`.

#### Track streaming

`GET /music/tidal/tracks/:id/stream`

Returns playback metadata for preview audio and optional full audio.

**Query params**

* `audioQuality` optional. Default is `HI_RES`
* `sessionId` optional

**Headers**

* `x-tidal-sessionid` optional alternative to `sessionId`

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "preview": {
      "manifest": "BASE64_STRING",
      "manifestDecoded": "#EXTM3U\n..."
    },
    "audio": {
      "manifest": "BASE64_STRING",
      "manifestDecoded": "#EXTM3U\n..."
    }
  }
}
```

**Behavior**

* always fetches track metadata
* always fetches preview playback info
* fetches full audio only when a valid session ID is provided
* decodes base64 manifests into extra `manifestDecoded` fields

{% hint style="warning" %}
This route returns playback metadata, not raw audio bytes.
{% endhint %}

If no valid session is provided:

* preview still works
* full audio returns an embedded `401` error in the `audio` section

#### Playback info only

`GET /music/tidal/tracks/:id/playbackinfo`

Returns the raw Tidal playback info payload for the track.

**Query params**

* `audioQuality` optional. Default is `HI_RES`

Use this when you want the raw Tidal playback JSON instead of the combined stream payload.

#### Track radio

`GET /music/tidal/tracks/:id/radio`

Returns a track-based station or radio result.

### Albums

Album routes work with both `/album/:id` and `/albums/:id`.

#### Album details

`GET /music/tidal/albums/:id`

Returns cleaned album metadata.

#### Album tracks

`GET /music/tidal/albums/:id/tracks`

Returns the album track list.

**Query params**

* `limit` optional. Default is `50`

Example shape:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "items": [],
    "totalNumberOfItems": 12
  }
}
```

### Artists

Artist routes work with both `/artist/:id` and `/artists/:id`.

#### Artist details

`GET /music/tidal/artists/:id`

Returns cleaned artist metadata.

#### Artist albums

`GET /music/tidal/artists/:id/albums`

**Query params**

* `limit` optional. Default is `50`

#### Artist top tracks

`GET /music/tidal/artists/:id/toptracks`

**Query params**

* `limit` optional. Default is `10`

#### Artist radio

`GET /music/tidal/artists/:id/radio`

Returns an artist-based radio result.

Artist collection routes clean returned `items[]` before responding.

### Playlists

Playlist routes work with both `/playlist/:id` and `/playlists/:id`.

#### Playlist details

`GET /music/tidal/playlists/:id`

Returns playlist metadata.

#### Playlist tracks

`GET /music/tidal/playlists/:id/tracks`

**Query params**

* `limit` optional. Default is `50`

Returns cleaned playlist track entries.

### Mixes

Mix routes work with both `/mix/:id` and `/mixes/:id`.

#### Mix details

`GET /music/tidal/mixes/:id`

Returns mix metadata.

#### Mix items

`GET /music/tidal/mixes/:id/items`

**Query params**

* `limit` optional. Default is `50`

Returns cleaned mix items.

### Videos

Video routes work with both `/video/:id` and `/videos/:id`.

#### Video details

`GET /music/tidal/videos/:id`

Returns cleaned video metadata.

#### Video streaming

`GET /music/tidal/videos/:id/stream`

Returns preview and optional full video playback metadata.

**Query params**

* `quality` optional. Default is `HIGH`
* `sessionId` optional

**Headers**

* `x-tidal-sessionid` optional alternative to `sessionId`

The behavior mirrors track streaming:

* preview is always fetched
* full playback requires a valid session ID
* manifests are returned both encoded and decoded

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

* missing `q` on search
* missing `trackId` on recommendations
* invalid IDs on resource routes
* invalid or missing session IDs for post-paywall playback

### Typical workflow

#### Search to playback

1. Call `/music/tidal/search?q=artist+song&types=TRACKS`
2. Pick a track ID from `data.tracks.items`
3. Load `/music/tidal/tracks/:id`
4. Load `/music/tidal/tracks/:id/stream?audioQuality=HIGH&sessionId=...`
5. Use the returned manifests in your own player pipeline

#### Browse to album playback

1. Call `/music/tidal/featured`
2. Pick an album ID from returned modules
3. Load `/music/tidal/albums/:id`
4. Load `/music/tidal/albums/:id/tracks`
5. Pick a track and call the track stream route

#### Other resource flows

* artists → `/artists/:id`, `/artists/:id/toptracks`
* playlists → `/playlists/:id`, `/playlists/:id/tracks`
* mixes → `/mixes/:id/items`
* videos → `/videos/:id`, `/videos/:id/stream`

Tidal is a good fit when you want one music provider to cover search, discovery, metadata, and playback manifests.
