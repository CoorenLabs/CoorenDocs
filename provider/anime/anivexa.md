---
description: Multi-provider anime aggregator that turns an AniList id into episode lists and stream sources from twelve sites.
icon: layer-group
---

# Anivexa

Anivexa takes an AniList id, matches it on twelve anime sites (MKissa, ReAnime, Anikoto, AnimeGG, 2DHive, AnimeNoSub, AniZone, AniWaves, AniBD, Senshi, KickAssAnime and AnimeOnsen) and returns every site's episode list in one response. Each episode carries a ready-made watch id that you pass back to get that site's streams for the sub or dub version. It also ships HLS helper routes for the two sites whose playlists cannot be played directly, and a captcha page for MKissa.

{% hint style="info" %}
Base route: `/anime/anivexa`
{% endhint %}

## Routes

| Route | Description |
| --- | --- |
| `GET /anime/anivexa` | Service info, provider names and route list |
| `GET /anime/anivexa/map/:anilistId` | External ids, synonyms and franchise entries for an AniList id |
| `GET /anime/anivexa/episodes/:anilistId` | Episode lists from all twelve providers |
| `GET /anime/anivexa/episodes/:provider[/:provider...]/:anilistId` | Episode lists from the providers you name |
| `GET /anime/anivexa/watch/:provider/:anilistId/:audio/:provider-:episode` | Stream sources for one episode on one provider |
| `GET /anime/anivexa/stream/reanime/:anilistId/:audio/:episode` | `302` redirect to a playable ReAnime HLS playlist |
| `GET /anime/anivexa/stream/2dhive/:anilistId/:audio/:episode` | `302` redirect to a playable 2DHive HLS playlist |
| `GET /anime/anivexa/hls/flixcloud/playlist.m3u8` | Decodes and rewrites a Flixcloud (ReAnime) HLS playlist |
| `GET /anime/anivexa/hls/flixcloud/segment.ts` | Unwraps one Flixcloud video segment |
| `GET /anime/anivexa/hls/senshi/playlist.m3u8` | Decrypts and rewrites a Senshi HLS playlist |
| `GET /anime/anivexa/captcha/mkissa` | HTML page that solves the MKissa captcha and retries a watch request |

## How the routes fit together

1. Call `/episodes/:anilistId` (or the filtered form) with an AniList id, for example `154587` for Frieren.
2. Pick a provider key, then `episodes.sub` or `episodes.dub`, then an episode.
3. The episode `id` is a path relative to the base route, such as `watch/kaa/154587/sub/kaa-1`. Request `http://localhost:3000/anime/anivexa/` followed by that id.
4. Play the `url` of a stream from the watch response with the `referer` or `headers` it lists, or play its `proxiedUrl` when there is one.

Sub and dub are separate lists and separate watch paths. A provider that has no dub returns an empty `dub` array, and asking its watch route for `dub` returns `404`.

## Providers

These are the provider names used in URLs and in response keys. "Lists" shows which audio lists were filled for Frieren (`154587`), and "Watch returns" is what `/watch` returned for episode 1.

| Provider | Site | Lists | Watch returns |
| --- | --- | --- | --- |
| `mkissa` | mkissa.to | `sub`, `dub`, `raw` (always empty) | `sources` of embed pages (`Sw`, `Fm-Hls`, `Sup`, `Uni`, `Mp4`, `Ok`); some carry a direct `extractedUrl`. May ask for a captcha |
| `reanime` | reanime.to | `sub`, `dub` | Flixcloud HLS (`HD-1`, `HD-2`) with a `proxiedUrl` through `/hls/flixcloud`, ASS subtitles, intro and outro. Uses its own response shape |
| `anikoto` | anikototv.to | `sub`, `dub` | MegaPlay HLS (`Vidstream-2`, `Vidstream-1 beta`, `HD-1`) and their embed pages, VTT subtitles, intro, outro and download links |
| `animegg` | animegg.org | `sub`, `dub` (24 sub, 9 dub for Frieren) | Direct MP4 files at 360p, 480p, 720p and 1080p, plus an embed page |
| `2dhive` | 2dhive.com | `sub`, `dub` | BabaStream embed and MP4, MegaPlay HLS and embed |
| `animenosub` | animenosub.to | `sub` | Vidmoly HLS (`Omega`) and Vtube, FileMoon and StreamWish embeds, plus `RAW - ...` entries |
| `anizone` | anizone.to | `sub` | One HLS stream with many ASS subtitle tracks and a storyboard VTT |
| `aniwaves` | aniwaves.ru | `sub`, `dub` | Vidplay HLS plus `Vidplay`, `BYFMS` and `DGHG` embeds |
| `anibd` | epeng.animeapps.top | `sub` | One HLS stream (`SR`) and one embed (`SB`) |
| `senshi` | senshi.to | `sub`, `dub` | HLS with a `proxiedUrl` through `/hls/senshi`, VTT subtitles and subtitle font files |
| `kaa` | kaa.lt (KickAssAnime) | `sub`, `dub` | HLS from krussdomi.com (`VidStreaming`, `BirdStream`) with a `Referer` header |
| `animeonsen` | animeonsen.xyz | `sub` | A DASH manifest; the stream and its subtitles need the `Authorization` header given in the response |

Matching is done per site, so a provider can miss a title another provider has. In testing, `animenosub` found no match for Attack on Titan (`16498`), Demon Slayer (`101922`) or Naruto (`20`), and `animeonsen` was not confident for One Piece (`21`) or Naruto. Those providers come back with an `error` field instead of episodes.

## Overview

### Service info

`GET /anime/anivexa`

Returns the service name, version, provider names (the values you can use as `:provider`) and the route list.

**Example**

```bash
curl "http://localhost:3000/anime/anivexa"
```

```json
{
  "service": "anivexa",
  "description": "Anivexa — multi-provider anime episode & stream API (AniList-ID-based)",
  "version": "2.2.1",
  "providers": ["mkissa", "reanime", "anikoto", "animegg", "2dhive", "animenosub", "anizone", "aniwaves", "anibd", "senshi", "kaa", "animeonsen"],
  "endpoints": [
    "GET /anime/anivexa/map/:anilistId",
    "GET /anime/anivexa/episodes/:anilistId",
    "GET /anime/anivexa/episodes/:provider[/:provider...]/:anilistId?map=true|false",
    "GET /anime/anivexa/watch/:provider/:anilistId/sub|dub/:provider-:episode",
    "GET /anime/anivexa/stream/reanime/:anilistId/sub|dub/:episode",
    "GET /anime/anivexa/stream/2dhive/:anilistId/sub|dub/:episode",
    "GET /anime/anivexa/captcha/mkissa?next=/watch/mkissa/:anilistId/sub|dub/mkissa-:episode"
  ]
}
```

## Mappings

### Map ids

`GET /anime/anivexa/map/:anilistId`

Returns the ids other databases use for the same show (MyAnimeList, AniDB, Kitsu, TMDB, TheTVDB, IMDb and more), its synonyms, and related franchise entries from AniList. The same `mappings` object is included in `/episodes` responses.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `anilistId` | Yes | Numeric AniList anime id |

**Example**

```bash
curl "http://localhost:3000/anime/anivexa/map/154587"
```

```json
{
  "mappings": {
    "id": 154587,
    "title": "Frieren: Beyond Journey’s End",
    "type": "TV",
    "format": "TV",
    "episodes": 28,
    "malId": 52991,
    "aniId": 154587,
    "anidbId": 17617,
    "animePlanetId": "frieren-beyond-journeys-end",
    "kitsuId": 46474,
    "animeCountdownId": 1990194,
    "anisearchId": 17669,
    "notifyMoeId": null,
    "simklId": 1990194,
    "imdbId": "tt22248376",
    "themoviedbId": 209867,
    "thetvdbId": 424536,
    "livechartId": 11376,
    "annId": 26334,
    "animescheduleId": null,
    "animethemesId": null,
    "animefillerlistId": null,
    "franchiseAnchor": "tvdb:424536",
    "franchiseId": 405283420,
    "defaultTvdbSeason": "1",
    "tmdbSeason": "1",
    "episodeOffset": null,
    "tmdbOffset": null,
    "malIds": null,
    "aniskip": null,
    "animefillerlist": null,
    "synonyms": ["Frieren at the Funeral", "장송의 프리렌"],
    "franchise": [
      {
        "relation": "SOURCE",
        "anilistId": 118586,
        "title": "Sousou no Frieren",
        "type": "MANGA",
        "format": "MANGA"
      },
      {
        "relation": "SIDE_STORY",
        "anilistId": 173310,
        "title": "Sousou no Frieren Anthology",
        "type": "MANGA",
        "format": "MANGA"
      }
    ]
  }
}
```

## Episodes

### All providers

`GET /anime/anivexa/episodes/:anilistId`

Queries all twelve providers in parallel and returns one key per provider next to `page`, `type` (`"all"`) and `mappings`. Each provider key holds either `{ meta, episodes }` or `{ error }` when that site failed or had no match. The call only fails as a whole when the AniList id does not exist.

This is the slowest route. Uncached calls took 4 to 9 seconds in testing, and a slow site can push it to about 15 to 20 seconds, because each provider gets up to 20 seconds before it is reported as an error.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `anilistId` | Yes | Numeric AniList anime id |

**Example**

```bash
curl "http://localhost:3000/anime/anivexa/episodes/21"
```

The real response has a key for each of the twelve providers; three are shown.

<details>

<summary>Response</summary>

```json
{
  "page": 1,
  "type": "all",
  "mappings": {
    "id": 21,
    "title": "ONE PIECE",
    "type": "TV",
    "format": "TV",
    "episodes": null,
    "malId": 21,
    "aniId": 21,
    "anidbId": 69,
    "animePlanetId": "one-piece",
    "kitsuId": 12,
    "animeCountdownId": 38636,
    "anisearchId": 2227,
    "notifyMoeId": null,
    "simklId": 38636,
    "imdbId": "tt0388629",
    "themoviedbId": 37854,
    "thetvdbId": 81797,
    "livechartId": 321,
    "annId": 836,
    "animescheduleId": null,
    "animethemesId": null,
    "animefillerlistId": null,
    "franchiseAnchor": "tvdb:81797",
    "franchiseId": 1125119618,
    "defaultTvdbSeason": null,
    "tmdbSeason": null,
    "episodeOffset": null,
    "tmdbOffset": null,
    "malIds": null,
    "aniskip": null,
    "animefillerlist": null,
    "synonyms": ["ワンピース"],
    "franchise": [
      {
        "relation": "SIDE_STORY",
        "anilistId": 466,
        "title": "ONE PIECE: Taose! Kaizoku Ganzack",
        "type": "ANIME",
        "format": "OVA"
      }
    ]
  },
  "mkissa": {
    "meta": {
      "id": "ReooPAxPMsHM4KPMY",
      "title": "ONE PIECE"
    },
    "episodes": {
      "sub": [
        {
          "id": "watch/mkissa/21/sub/mkissa-1",
          "audio": "sub",
          "number": 1,
          "title": "I`m Luffy! The Man Who`s Gonna Be King of the Pirates!",
          "duration": 25,
          "filler": false,
          "uncensored": false,
          "description": "Alvida pirates plunder a ship only to find a barrel containing a strange boy names Luffy...",
          "image": "https://artworks.thetvdb.com/banners/v4/episode/361887/screencap/604df7d3ecf3a.jpg",
          "airDate": "1999-10-20"
        }
      ],
      "dub": [
        {
          "id": "watch/mkissa/21/dub/mkissa-1",
          "audio": "dub",
          "number": 1,
          "title": "I`m Luffy! The Man Who`s Gonna Be King of the Pirates!",
          "duration": 25,
          "filler": false,
          "uncensored": false,
          "description": "Alvida pirates plunder a ship only to find a barrel containing a strange boy names Luffy...",
          "image": "https://artworks.thetvdb.com/banners/v4/episode/361887/screencap/604df7d3ecf3a.jpg",
          "airDate": "1999-10-20"
        }
      ],
      "raw": []
    }
  },
  "kaa": {
    "meta": {
      "id": "one-piece-0948",
      "title": "One Piece",
      "source": "kaa",
      "matchScore": 1
    },
    "episodes": {
      "sub": [
        {
          "id": "watch/kaa/21/sub/kaa-1",
          "audio": "sub",
          "number": 1,
          "title": "I`m Luffy! The Man Who`s Gonna Be King of the Pirates!",
          "duration": 1500,
          "filler": false,
          "uncensored": false,
          "description": "Alvida pirates plunder a ship only to find a barrel containing a strange boy names Luffy...",
          "image": "https://artworks.thetvdb.com/banners/v4/episode/361887/screencap/604df7d3ecf3a.jpg",
          "airDate": "1999-10-20"
        }
      ],
      "dub": [
        {
          "id": "watch/kaa/21/dub/kaa-1",
          "audio": "dub",
          "number": 1,
          "title": "I`m Luffy! The Man Who`s Gonna Be King of the Pirates!",
          "duration": 1500,
          "filler": false,
          "uncensored": false,
          "description": "Alvida pirates plunder a ship only to find a barrel containing a strange boy names Luffy...",
          "image": "https://artworks.thetvdb.com/banners/v4/episode/361887/screencap/604df7d3ecf3a.jpg",
          "airDate": "1999-10-20"
        }
      ]
    }
  },
  "animeonsen": {
    "error": "AnimeOnsen match not confident for AniList 21"
  }
}
```

</details>

**Episode fields**

| Field | Type | Description |
| --- | --- | --- |
| `id` | string | Watch path relative to the base route: `watch/<provider>/<anilistId>/<audio>/<provider>-<number>` |
| `sourceNumber` | number or string | The site's own episode number (`animegg`, `animenosub`, `anizone`, `aniwaves`, `animeonsen`); `aniwaves` and `animeonsen` return it as a string |
| `audio` | string | `sub` or `dub` |
| `number` | number | Episode number in AniList numbering. Can be `0` or a half number such as `13.5` or `1004.5` for specials on some sites |
| `title` | string or null | Episode title, mostly from ani.zip metadata |
| `titleJapanese`, `titleRomanji` | string or null | `reanime` only |
| `duration` | number or null | Seconds for most providers; minutes for `mkissa`; `null` on `anikoto` |
| `score` | null | `reanime` only |
| `filler` | boolean | Filler flag |
| `recap` | boolean | `reanime`, `senshi` and `aniwaves` only |
| `uncensored` | boolean | Always `false` in tests |
| `description` | string or null | Episode synopsis |
| `image` | string or null | Episode thumbnail |
| `airDate` | string or null | `YYYY-MM-DD` |
| `sourceLink` | string | `anibd` only, the site's own episode link key |

`meta` is provider-specific. It usually holds the site's own id or slug and title; the title-matching providers also return `matchScore`, `numbering` and `episodeOffset`. When a site keeps counting from a previous season, `numbering` is `"offset"`, `episodeOffset` is the shift, `number` is the AniList-relative number and `sourceNumber` is the site's number. Only `"local"` and `"standard"` appeared in tests.

### Selected providers

`GET /anime/anivexa/episodes/:provider[/:provider...]/:anilistId`

Same as the route above but only for the providers you list, separated by slashes. `type` is `"filtered"`. Provider names are case-insensitive. Unknown names are ignored and listed in `_unknownProviders`; if none of the names is valid the route returns `400`.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `provider` | Yes | One or more provider names from the table above, for example `kaa/senshi` |
| `anilistId` | Yes | Numeric AniList anime id |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `map` | No | `true` | Set to `false` to leave out the `mappings` object |

**Example**

```bash
curl "http://localhost:3000/anime/anivexa/episodes/KAA/kickassanime/154587?map=false"
```

```json
{
  "page": 1,
  "type": "filtered",
  "kaa": {
    "meta": {
      "id": "sousou-no-frieren-2d15",
      "title": "Frieren: Beyond Journey's End",
      "source": "kaa",
      "matchScore": 1
    },
    "episodes": {
      "sub": [
        {
          "id": "watch/kaa/154587/sub/kaa-1",
          "audio": "sub",
          "number": 1,
          "title": "The Journey`s End",
          "duration": 1560,
          "filler": false,
          "uncensored": false,
          "description": "The world celebrates the Demon King's defeat at the hands of the Hero and his companions. ...",
          "image": "https://artworks.thetvdb.com/banners/v4/episode/9350138/screencap/6516db4c3a389.jpg",
          "airDate": "2023-09-29"
        }
      ],
      "dub": [
        {
          "id": "watch/kaa/154587/dub/kaa-1",
          "audio": "dub",
          "number": 1,
          "title": "The Journey`s End",
          "duration": 1560,
          "filler": false,
          "uncensored": false,
          "description": "The world celebrates the Demon King's defeat at the hands of the Hero and his companions. ...",
          "image": "https://artworks.thetvdb.com/banners/v4/episode/9350138/screencap/6516db4c3a389.jpg",
          "airDate": "2023-09-29"
        }
      ]
    }
  },
  "_unknownProviders": ["kickassanime"]
}
```

```bash
curl "http://localhost:3000/anime/anivexa/episodes/kickassanime/154587"
```

```json
{
  "error": "No valid providers specified",
  "unknown": ["kickassanime"]
}
```

## Watch

### Stream sources

`GET /anime/anivexa/watch/:provider/:anilistId/:audio/:provider-:episode`

Returns the stream sources one provider has for one episode. Build the path from an episode `id` returned by `/episodes`. The provider name appears twice and both must match, so `watch/kaa/154587/sub/senshi-1` returns `404`.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `provider` | Yes | Provider name, lowercase (`kaa`, not `kickassanime`) |
| `anilistId` | Yes | Numeric AniList anime id |
| `audio` | Yes | `sub` or `dub` |
| `provider-episode` | Yes | The provider name again, a hyphen and a whole episode number, for example `kaa-1` |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `captchaToken` | No | | `mkissa` only. A solved MKissa Turnstile token; `turnstileToken` is accepted too. See [MKissa captcha](#mkissa-captcha) |
| `captchaProvider` | No | `turnstile1` | `mkissa` only. Captcha provider name sent with the token |

The same token can be sent in the `x-captcha-token` or `cf-turnstile-response` header, and the provider in `x-captcha-provider`.

Most providers return `anilistId`, `episode`, `audio` and a `streams` array. Depending on the provider there are also `malId`, `providerEpisode`, `title`, `intro`, `outro`, `subtitles`, `downloads` and `headers`. Each stream has a `url`, a `type` (`hls`, `mp4`, `dash` or `embed`), a `server` name, and the `referer` or `headers` needed to play it; most also have `priority` and `isActive`. `embed` entries are web pages meant for an iframe, not media files. `mkissa` and `reanime` use their own shapes, shown below.

**Example: Senshi**

```bash
curl "http://localhost:3000/anime/anivexa/watch/senshi/154587/sub/senshi-1"
```

```json
{
  "anilistId": 154587,
  "malId": 52991,
  "episode": 1,
  "audio": "sub",
  "intro": { "start": 0, "end": 0 },
  "outro": { "start": 0, "end": 0 },
  "streams": [
    {
      "url": "https://s-90.bcdn1.se/i/9f90e474-ae25-4b9e-8563-227e682d505f/master.txt?token=AWq7w6n3jyXCdzdvUbGsGXMC...",
      "proxiedUrl": "http://localhost:3000/anime/anivexa/hls/senshi/playlist.m3u8?url=https%3A%2F%2Fs-90.bcdn1.se%2Fi%2F9f90e474-...",
      "type": "hls",
      "server": "Senshi",
      "referer": "https://senshi.to/",
      "quality": "1080p",
      "subtitles": [
        {
          "url": "https://s.anicdn.se/uploads/attachments/9f90e474-ae25-4b9e-8563-227e682d505f/sub_en.vtt",
          "label": "English",
          "srclang": "en",
          "default": true
        }
      ],
      "fonts": ["https://fs.anicdn.se/uploads/assets/d2Vsb3ZlYm9vYmllc2RvbnR3ZQ/Aver.ttf"],
      "priority": 5,
      "isActive": true
    }
  ],
  "subtitles": [
    {
      "url": "https://s.anicdn.se/uploads/attachments/9f90e474-ae25-4b9e-8563-227e682d505f/sub_en.vtt",
      "label": "English",
      "srclang": "en",
      "default": true
    }
  ],
  "downloads": [],
  "headers": {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/149.0.0.0 Safari/537.36",
    "Referer": "https://senshi.to/",
    "Origin": "https://senshi.to"
  }
}
```

**Example: KickAssAnime**

```bash
curl "http://localhost:3000/anime/anivexa/watch/kaa/154587/sub/kaa-1"
```

```json
{
  "anilistId": 154587,
  "episode": 1,
  "audio": "sub",
  "streams": [
    {
      "url": "https://hls.krussdomi.com/manifest/67d0c079169c31976b8d7970/master.m3u8",
      "type": "hls",
      "server": "VidStreaming",
      "headers": { "Referer": "https://krussdomi.com/" },
      "priority": 1,
      "isActive": true
    },
    {
      "url": "https://hls.krussdomi.com/manifest/ZjczZjBkNzU1NDliNTliNmNiYzlhMmVlNDk5N2Q5MGY6.../master.m3u8",
      "type": "hls",
      "server": "BirdStream",
      "headers": { "Referer": "https://krussdomi.com/" },
      "priority": 1,
      "isActive": false
    }
  ]
}
```

**Example: ReAnime**

ReAnime returns the first working stream at the top level (`stream_url`, `proxiedUrl`, `subtitles`, `intro`, `outro`) and every decrypted server in `streams`. `stream_url` is a Flixcloud playlist that players cannot read directly, so play `proxiedUrl`. For Frieren, sub and dub pointed at the same multi-audio video, and the dub `proxiedUrl` ended in `audio=1` so the English track is selected. `failedServers` lists servers that could not be decrypted.

```bash
curl "http://localhost:3000/anime/anivexa/watch/reanime/154587/sub/reanime-1"
```

```json
{
  "anime": "Frieren: Beyond Journey’s End",
  "slug": "frieren-beyond-journey-s-end-yw2a3j",
  "ep": 1,
  "audio": "sub",
  "server": "HD-2",
  "stream_url": "https://fetch8.flixcloud.cc/_v7/60aa496b-cea9-45a8-a4f4-8395990c0d06/master.m3u8?token=eyJhbGciOiJIUzI1NiIs...",
  "proxiedUrl": "http://localhost:3000/anime/anivexa/hls/flixcloud/playlist.m3u8?url=https%3A%2F%2Ffetch8.flixcloud.cc%2F_v7%2F60aa496b-...&key=Aan7yK6PJzOhcWrSygwwB1niiQY9QbJ5LxOnwAWrqs8&audio=0",
  "streams": [
    {
      "server": "HD-2",
      "audio": "sub",
      "index": 0,
      "url": "https://fetch8.flixcloud.cc/_v7/60aa496b-cea9-45a8-a4f4-8395990c0d06/master.m3u8?token=eyJhbGciOiJIUzI1NiIs...",
      "proxiedUrl": "http://localhost:3000/anime/anivexa/hls/flixcloud/playlist.m3u8?url=https%3A%2F%2Ffetch8.flixcloud.cc%2F_v7%2F60aa496b-...&key=Aan7yK6PJzOhcWrSygwwB1niiQY9QbJ5LxOnwAWrqs8&audio=0",
      "type": "hls",
      "embed": "https://flixcloud.cc/e/rjyccuaq8h5n?v=2",
      "subtitles": [
        {
          "url": "https://vault-94.fallencdn.top/subtitles/60aa496b-cea9-45a8-a4f4-8395990c0d06/60aa496b-cea9-45a8-a4f4-8395990c0d06_eng_3.ass",
          "language": "English (Full Subtitles [9volt])",
          "format": "ass",
          "default": true
        }
      ],
      "thumbnails_vtt": "https://fetch8.flixcloud.cc/thumbnails_vtt/60aa496b-cea9-45a8-a4f4-8395990c0d06",
      "video_title": "[LostYears] Frieren Beyond Journey's End - S01E01 (WEB 1080p x265 10-bit AAC Opus) [0F7F64F6].mkv",
      "intro": { "start": 0, "end": 90, "title": "Opening" },
      "outro": { "start": 1459, "end": 1549, "title": "Ending" }
    }
  ],
  "subtitles": [
    {
      "url": "https://vault-94.fallencdn.top/subtitles/60aa496b-cea9-45a8-a4f4-8395990c0d06/60aa496b-cea9-45a8-a4f4-8395990c0d06_eng_3.ass",
      "language": "English (Full Subtitles [9volt])",
      "format": "ass",
      "default": true
    }
  ],
  "thumbnails_vtt": "https://fetch8.flixcloud.cc/thumbnails_vtt/60aa496b-cea9-45a8-a4f4-8395990c0d06",
  "video_title": "[LostYears] Frieren Beyond Journey's End - S01E01 (WEB 1080p x265 10-bit AAC Opus) [0F7F64F6].mkv",
  "intro": { "start": 0, "end": 90, "title": "Opening" },
  "outro": { "start": 1459, "end": 1549, "title": "Ending" },
  "embeds": [
    { "name": "HD-2", "type": "sub", "url": "https://flixcloud.cc/e/rjyccuaq8h5n?v=2" }
  ],
  "allServers": [
    { "name": "HD-2", "type": "sub", "embed": "https://flixcloud.cc/e/rjyccuaq8h5n?v=2" }
  ],
  "failedServers": []
}
```

**Example: MKissa**

MKissa returns `sources` instead of `streams`. Each source is an embed page (`type` is `"iframe"`) sorted by `priority`. When the server could pull the file out of the embed, `extractedUrl` and `extractedType` are set.

```bash
curl "http://localhost:3000/anime/anivexa/watch/mkissa/154587/sub/mkissa-1"
```

```json
{
  "anilistId": 154587,
  "mkissaId": "ReHMC7TQnch3C6z8j",
  "episode": 1,
  "audio": "sub",
  "intro": null,
  "outro": null,
  "sources": [
    {
      "name": "Sw",
      "url": "https://streamwish.to/e/26qc49bl6muh",
      "extractedUrl": null,
      "extractedType": null,
      "type": "iframe",
      "priority": 5.5,
      "headers": {
        "Referer": "https://mkissa.to",
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36"
      },
      "downloads": {
        "sourceName": "Streamwish",
        "downloadUrl": "https://streamwish.to/26qc49bl6muh"
      }
    },
    {
      "name": "Mp4",
      "url": "https://mp4upload.com/embed-b4a5678plwzf.html",
      "extractedUrl": "https://a4.mp4upload.com:183/d/xkx3o2vez3b4quuotsrrqzsbctr2way2qxp57wrhjgsy.../video.mp4",
      "extractedType": "mp4",
      "type": "iframe",
      "priority": 4,
      "headers": {
        "Referer": "https://www.mp4upload.com/",
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36"
      },
      "downloads": null
    }
  ]
}
```

<details>

<summary>More watch examples (Anikoto, AnimeGG, AnimeOnsen, 2DHive)</summary>

```bash
curl "http://localhost:3000/anime/anivexa/watch/anikoto/154587/sub/anikoto-1"
```

```json
{
  "anilistId": 154587,
  "malId": 52991,
  "episode": 1,
  "audio": "sub",
  "streams": [
    {
      "url": "https://fetch.nexabloom.top/anime/bb6d2babd7797d94d8f4a8600bc9b44e/b7d51fb7e838ee9b60dcdb34b953bc07/master.m3u8?token=...",
      "type": "hls",
      "server": "Vidstream-2",
      "embedUrl": "https://megaplay.buzz/stream/s-2/107257/sub",
      "referer": "https://megaplay.buzz/",
      "subtitles": [
        {
          "url": "https://4driv.onehpanddreaming.site/anime/bb6d2babd7797d94d8f4a8600bc9b44e/b7d51fb7e838ee9b60dcdb34b953bc07/subtitles/...",
          "label": "English",
          "srclang": "en",
          "default": true,
          "source": "Vidstream-2"
        }
      ],
      "subtitleType": "softsub",
      "variant": "modern",
      "intro": { "start": 0, "end": 89 },
      "outro": { "start": 1460, "end": 1549 },
      "priority": 5,
      "isActive": true
    }
  ],
  "subtitles": [
    {
      "url": "https://4driv.onehpanddreaming.site/anime/bb6d2babd7797d94d8f4a8600bc9b44e/b7d51fb7e838ee9b60dcdb34b953bc07/subtitles/...",
      "label": "English",
      "srclang": "en",
      "default": true,
      "source": "Vidstream-2"
    }
  ],
  "downloads": [
    { "url": "https://pahe.nekostream.site/BbKSj", "label": "Kiwi" }
  ],
  "headers": {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36",
    "Referer": "https://megaplay.buzz/"
  }
}
```

```bash
curl "http://localhost:3000/anime/anivexa/watch/animegg/154587/sub/animegg-1"
```

```json
{
  "anilistId": 154587,
  "episode": 1,
  "providerEpisode": 1,
  "audio": "sub",
  "title": "Sousou no Frieren Episode 1",
  "streams": [
    {
      "url": "https://www.animegg.org/play/445674/video.mp4?for=101790686022063",
      "type": "mp4",
      "quality": "360p",
      "backup": "https://www.mp4upload.com/embed-wk2q0xewz8oa.html",
      "audio": "sub",
      "server": "Animegg",
      "embed": "https://www.animegg.org/embed/131519",
      "referer": "https://www.animegg.org/",
      "priority": 1,
      "isActive": true
    }
  ]
}
```

```bash
curl "http://localhost:3000/anime/anivexa/watch/animeonsen/154587/sub/animeonsen-1"
```

```json
{
  "anilistId": 154587,
  "episode": 1,
  "providerEpisode": 1,
  "audio": "sub",
  "intro": { "start": 1, "end": 90 },
  "outro": null,
  "streams": [
    {
      "url": "https://cdn.animeonsen.xyz/video/mp4-dash/LQf5VCVYgecMGZob/1/manifest.mpd",
      "type": "dash",
      "server": "AnimeOnsen",
      "referer": "https://www.animeonsen.xyz/",
      "headers": { "Authorization": "Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6ImRlZmF1bHQifQ..." },
      "subtitles": [
        {
          "url": "https://api.animeonsen.xyz/v4/subtitles/LQf5VCVYgecMGZob/ar-ME/1",
          "label": "العربية",
          "srclang": "ar-ME",
          "default": false,
          "headers": { "Authorization": "Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6ImRlZmF1bHQifQ..." }
        }
      ],
      "priority": 5,
      "isActive": true
    }
  ]
}
```

```bash
curl "http://localhost:3000/anime/anivexa/watch/2dhive/154587/sub/2dhive-1"
```

```json
{
  "anilistId": 154587,
  "episode": 1,
  "audio": "sub",
  "streams": [
    {
      "server": "BabaStream",
      "url": "https://babastream.top/embed/52991/1/sub",
      "type": "embed"
    },
    {
      "server": "BabaStream",
      "url": "https://scontent-iad6-1.xx.fbcdn.net/m1/v/t0.71334-6/An-IRYpqZPGFHDM_h15lR5XwWIVXQOf4AOZqQM6c...",
      "type": "mp4",
      "embed": "https://babastream.top/embed/52991/1/sub",
      "referer": "https://babastream.top/"
    },
    {
      "server": "MegaPlay",
      "url": "https://fetch.nexabloom.top/anime/bb6d2babd7797d94d8f4a8600bc9b44e/b7d51fb7e838ee9b60dcdb34b953bc07/master.m3u8?token=...",
      "type": "hls",
      "variant": "modern",
      "embed": "https://megaplay.buzz/stream/mal/52991/1/sub",
      "referer": "https://megaplay.buzz/",
      "subtitles": [
        {
          "file": "https://4driv.onehpanddreaming.site/anime/bb6d2babd7797d94d8f4a8600bc9b44e/b7d51fb7e838ee9b60dcdb34b953bc07/subtitles/ara-6.vtt",
          "label": "Arabic",
          "kind": "captions"
        }
      ],
      "intro": { "start": 0, "end": 89 },
      "outro": { "start": 1460, "end": 1549 }
    },
    {
      "server": "MegaPlay",
      "url": "https://megaplay.buzz/stream/mal/52991/1/sub",
      "type": "embed"
    }
  ]
}
```

</details>

## Stream redirects

These two routes skip the JSON and answer with a `302` redirect to a playlist you can hand straight to a player.

### ReAnime stream

`GET /anime/anivexa/stream/reanime/:anilistId/:audio/:episode`

Redirects to the `proxiedUrl` of the first working ReAnime server, which is served by `/hls/flixcloud/playlist.m3u8`.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `anilistId` | Yes | Numeric AniList anime id |
| `audio` | Yes | `sub` or `dub` |
| `episode` | Yes | Whole episode number (just the number, no provider prefix) |

**Example**

```bash
curl -s -D - -o /dev/null "http://localhost:3000/anime/anivexa/stream/reanime/154587/sub/1"
```

```text
HTTP/1.1 302 Found
Access-Control-Allow-Origin: *
Location: http://localhost:3000/anime/anivexa/hls/flixcloud/playlist.m3u8?url=https%3A%2F%2Ffetch8.flixcloud.cc%2F_v7%2F60aa496b-cea9-45a8-a4f4-8395990c0d06%2Fmaster.m3u8%3Ftoken%3D...&key=Aan7yK6PJzOhcWrSygwwB1niiQY9QbJ5LxOnwAWrqs8&audio=0
Cache-Control: no-store
```

### 2DHive stream

`GET /anime/anivexa/stream/2dhive/:anilistId/:audio/:episode`

Redirects to the MegaPlay HLS playlist through the shared [stream proxy](../../core/proxy.md), with the needed `Referer` already attached. Falls back to the BabaStream file when MegaPlay has nothing.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `anilistId` | Yes | Numeric AniList anime id |
| `audio` | Yes | `sub` or `dub` |
| `episode` | Yes | Whole episode number |

**Example**

```bash
curl -s -D - -o /dev/null "http://localhost:3000/anime/anivexa/stream/2dhive/154587/sub/1"
```

```text
HTTP/1.1 302 Found
Access-Control-Allow-Origin: *
Location: http://localhost:3000/proxy/m3u8-proxy?url=https%3A%2F%2Ffetch.nexabloom.top%2Fanime%2Fbb6d2babd7797d94d8f4a8600bc9b44e%2Fb7d51fb7e838ee9b60dcdb34b953bc07%2Fmaster.m3u8%3Ftoken%3D...&headers=%7B%22Referer%22%3A%22https%3A%2F%2Fmegaplay.buzz%2F%22%7D
Cache-Control: no-store
```

## HLS helpers

You normally do not build these URLs yourself. They appear as `proxiedUrl` in ReAnime and Senshi watch responses and inside the playlists these routes return. Every rewritten line points back at these routes or at the shared [stream proxy](../../core/proxy.md), so a player only ever talks to your API.

### Flixcloud playlist

`GET /anime/anivexa/hls/flixcloud/playlist.m3u8`

Fetches a Flixcloud playlist (used by ReAnime), decodes it when it is obfuscated, and rewrites nested playlists to this route and segments to `/hls/flixcloud/segment.ts`. Only `https` URLs on `flixcloud.cc`, `rundowncdn.top` and `stronghole.site` (and their subdomains) are accepted.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `url` | Yes | | Upstream playlist URL |
| `key` | No | | Base64url key used to decode the playlist, taken from the watch response |
| `audio` | No | | Index of the audio track to mark as default (`0` is Japanese, `1` is English for the tested show) |

**Example**

```bash
curl "http://localhost:3000/anime/anivexa/hls/flixcloud/playlist.m3u8?url=https%3A%2F%2Ffetch8.flixcloud.cc%2F_v7%2F60aa496b-cea9-45a8-a4f4-8395990c0d06%2Fmaster.m3u8%3Ftoken%3D...&key=Aan7yK6PJzOhcWrSygwwB1niiQY9QbJ5LxOnwAWrqs8&audio=0"
```

```text
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="audio",LANGUAGE="jpn",NAME="Native",DEFAULT=YES,AUTOSELECT=YES,URI="http://localhost:3000/anime/anivexa/hls/flixcloud/playlist.m3u8?url=https%3A%2F%2Ffetch8.flixcloud.cc%2F_v7%2FiToCUPVCuJLcef4h2-jkbmBCcq5g2Lsdp0c%2Faudio%2Fnative.m3u8&key=Aan7yK6PJzOhcWrSygwwB1niiQY9QbJ5LxOnwAWrqs8"
#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="audio",LANGUAGE="eng",NAME="English",DEFAULT=NO,AUTOSELECT=NO,URI="http://localhost:3000/anime/anivexa/hls/flixcloud/playlist.m3u8?url=https%3A%2F%2Ffetch8.flixcloud.cc%2F_v7%2FiToCUPVCuJLcef4h2-jkbmBCcq5g2Lsdp0c%2Faudio%2Fenglish.m3u8&key=Aan7yK6PJzOhcWrSygwwB1niiQY9QbJ5LxOnwAWrqs8"
#EXT-X-STREAM-INF:BANDWIDTH=5184,RESOLUTION=1920x1080,AUDIO="audio",CODECS="avc1.64001f,mp4a.40.2"
http://localhost:3000/anime/anivexa/hls/flixcloud/playlist.m3u8?url=https%3A%2F%2Ffetch8.flixcloud.cc%2F_v7%2FiToCUPVCuJLcef4h2-jkbmBCcq5g2Lsdp0c%2Fvideo.m3u8&key=Aan7yK6PJzOhcWrSygwwB1niiQY9QbJ5LxOnwAWrqs8
```

The nested media playlist lists segments through the segment route:

```text
#EXTM3U
#EXT-X-VERSION:6
#EXT-X-TARGETDURATION:6
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-PLAYLIST-TYPE:VOD
#EXT-X-INDEPENDENT-SEGMENTS
#EXTINF:6.006000,
http://localhost:3000/anime/anivexa/hls/flixcloud/segment.ts?url=https%3A%2F%2Flock97.stronghole.site%2F_v7%2F60aa496b-cea9-45a8-a4f4-8395990c0d06%2Fseg-0-f1-v1-a0.png
#EXTINF:6.006000,
http://localhost:3000/anime/anivexa/hls/flixcloud/segment.ts?url=https%3A%2F%2Flock97.stronghole.site%2F_v7%2F60aa496b-cea9-45a8-a4f4-8395990c0d06%2Fseg-1-f1-v1-a0.webp
```

### Flixcloud segment

`GET /anime/anivexa/hls/flixcloud/segment.ts`

Fetches one Flixcloud segment. Flixcloud hides MPEG-TS segments behind a PNG or WebP header and a byte mask; this route strips both and streams plain `video/mp2t`. Segment responses are cacheable for a day.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `url` | Yes | | Upstream segment URL on one of the Flixcloud hosts |

**Example**

```bash
curl -s -D - -o segment.ts "http://localhost:3000/anime/anivexa/hls/flixcloud/segment.ts?url=https%3A%2F%2Flock97.stronghole.site%2F_v7%2F60aa496b-cea9-45a8-a4f4-8395990c0d06%2Fseg-0-f1-v1-a0.png"
```

```text
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Content-Type: video/mp2t
Cache-Control: public, max-age=86400
Transfer-Encoding: chunked
```

### Senshi playlist

`GET /anime/anivexa/hls/senshi/playlist.m3u8`

Fetches a Senshi playlist, decrypts it when it is encrypted (`EM3U8v1:` prefix), and rewrites nested playlists to this route and segments to the shared [stream proxy](../../core/proxy.md) with Senshi's `Referer` and `Origin`. Only `https` URLs on `bcdn*.se` hosts are accepted.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `url` | Yes | | Upstream playlist URL from a Senshi watch response |

**Example**

```bash
curl "http://localhost:3000/anime/anivexa/hls/senshi/playlist.m3u8?url=https%3A%2F%2Fs-90.bcdn1.se%2Fi%2F9f90e474-ae25-4b9e-8563-227e682d505f%2Fmaster.txt%3Ftoken%3D..."
```

```text
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="group_audio",NAME="Japanese",DEFAULT=YES,LANGUAGE="ja",CHANNELS="2",URI="http://localhost:3000/anime/anivexa/hls/senshi/playlist.m3u8?url=https%3A%2F%2Fs-90.bcdn1.se%2Fi%2F9f90e474-ae25-4b9e-8563-227e682d505f%2Faudio%2F0_ja%2Fplaylist.txt%3Ftoken%3D..."
#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="group_audio",NAME="English",DEFAULT=NO,LANGUAGE="en",CHANNELS="2",URI="http://localhost:3000/anime/anivexa/hls/senshi/playlist.m3u8?url=https%3A%2F%2Fs-90.bcdn1.se%2Fi%2F9f90e474-ae25-4b9e-8563-227e682d505f%2Faudio%2F1_en%2Fplaylist.txt%3Ftoken%3D..."

#EXT-X-STREAM-INF:BANDWIDTH=3476000,RESOLUTION=1920x1080,CODECS="avc1.640032,mp4a.40.2",AUDIO="group_audio"
http://localhost:3000/anime/anivexa/hls/senshi/playlist.m3u8?url=https%3A%2F%2Fs-90.bcdn1.se%2Fi%2F9f90e474-ae25-4b9e-8563-227e682d505f%2Fvideo%2F1080%2Fplaylist.txt%3Ftoken%3D...
```

The nested media playlist points its segments at the shared proxy:

```text
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-TARGETDURATION:10
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-PLAYLIST-TYPE:VOD
#EXTINF:10.010011,
http://localhost:3000/proxy/ts-segment?url=https%3A%2F%2Fs-90.bcdn1.se%2Fi%2F9f90e474-ae25-4b9e-8563-227e682d505f%2Fc%2FdmlkZW8vMTA4MC9zZWdtZW50XzAwMC50c3B5SZRjmQANUz1jGQxBVnU.jpg&headers=%7B%22Referer%22%3A%22https%3A%2F%2Fsenshi.to%2F%22%2C%22Origin%22%3A%22https%3A%2F%2Fsenshi.to%22%7D
```

## MKissa captcha

MKissa sometimes answers an episode request with a Cloudflare Turnstile captcha. Anivexa handles it in this order:

1. The watch request is sent without a captcha. If MKissa asks for one, the server tries to solve it itself in a headless browser (up to 45 seconds).
2. If that fails, or MKissa rejects the token, the route returns the last good result for that episode if it has one in memory. Otherwise it returns `403` with `code: "NEED_CAPTCHA"` and a `captcha` object. After a failed attempt the server does not try to solve again for 2 minutes and returns `403` without trying.
3. Open `captcha.solveUrl` (relative to your API host) in a browser. The page loads MKissa's Turnstile widget, and once it is solved it calls the watch route again with `captchaToken` and `captchaProvider` and prints the JSON result.
4. Alternatively, solve the Turnstile at `captcha.endpoint` yourself and send the token to the watch route in the `captchaToken` query parameter or the `x-captcha-token` header. A request with a token always skips the cached result.

The `403` body has these fields:

| Field | Type | Description |
| --- | --- | --- |
| `error` | string | `MKissa requested captcha` |
| `code` | string | `NEED_CAPTCHA` |
| `captcha.endpoint` | string | MKissa's Turnstile page, `https://api.mkissa.net/captcha/turnstile` |
| `captcha.provider` | string | `turnstile` |
| `captcha.tokenQuery` | string | `captchaToken`, the query parameter that carries the token |
| `captcha.tokenHeader` | string | `x-captcha-token`, the header that carries the token |
| `captcha.solveUrl` | string | `/anime/anivexa/captcha/mkissa?next=...` for this exact episode |

Other MKissa errors carry the same extra fields set to `null`, for example `{"error":"Episode not found","code":null,"captcha":null}`.

### Captcha page

`GET /anime/anivexa/captcha/mkissa`

Returns an HTML page (`text/html`, not cached) titled "MKissa Security Check" that embeds the MKissa Turnstile widget and retries the watch path given in `next` once it is solved.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `next` | Yes | | The MKissa watch path to retry, `/watch/mkissa/:anilistId/sub\|dub/mkissa-:episode`, with or without the `/anime/anivexa` prefix |

Any other `next` value (a different provider, another host) is dropped, and the page shows `{"error":"Missing next watch path"}` after the captcha.

**Example**

```bash
curl "http://localhost:3000/anime/anivexa/captcha/mkissa?next=/anime/anivexa/watch/mkissa/154587/sub/mkissa-1"
```

## Errors

Errors are JSON with an `error` message. Unknown paths, including a watch path whose two provider names differ, a provider name that does not exist, or a fractional episode number, return `404` with the route list.

| Status | When |
| --- | --- |
| `400` | No valid provider in the filtered episodes route; missing or malformed `url` on an HLS helper |
| `403` | HLS helper `url` is not on the allowed hosts; MKissa needs a captcha |
| `404` | Unknown route, AniList id not found, no match on that site, or that site has no such episode or audio |
| `413` | An upstream playlist or segment is larger than the helper allows |
| `500` | Unexpected error inside a provider |
| `502` | The upstream site failed, or a playlist could not be decoded |

```bash
curl "http://localhost:3000/anime/anivexa/watch/anizone/154587/dub/anizone-1"
```

```json
{
  "error": "AniZone dub episode 1 not found"
}
```

```bash
curl "http://localhost:3000/anime/anivexa/watch/kaa/154587/sub/kaa-999"
```

```json
{
  "error": "KAA: episode 999 not found for AniList 154587"
}
```

```bash
curl "http://localhost:3000/anime/anivexa/map/999999999"
```

```json
{
  "error": "No data found for AniList ID 999999999"
}
```

```bash
curl "http://localhost:3000/anime/anivexa/hls/flixcloud/playlist.m3u8?url=https%3A%2F%2Fexample.com%2Fa.m3u8"
```

```json
{
  "error": "Forbidden url"
}
```

## Notes

- Speed: `/episodes/:anilistId` took 4 to 9 seconds uncached in testing and can take 15 to 20 seconds when a site is slow. Watch calls took under 2.5 seconds for most providers; `mkissa` and `2dhive` took about 6 to 7 seconds.
- Caching: successful JSON responses send `Cache-Control: public, max-age=300`. When `REDIS_URL` is set, maps are cached for 12 hours (30 days for finished shows) and watch results for 3 hours, or 1 minute for `anikoto`, `2dhive` and `senshi` because their stream URLs are signed. Full episode responses are kept for 30 days and served from cache; a request also triggers a background check for new episodes and failed providers, at most every 15 minutes (every 5 minutes around a new episode's air time). Without Redis only title matches, AniList lookups and MKissa watch results are kept in memory, so repeat calls are faster but still hit the sites.
- Stream URLs are signed and expire. Fetch them again from `/watch` rather than storing them.
- Some sites list specials with half numbers, such as `watch/kaa/21/sub/kaa-1004.5`. The watch route only accepts whole numbers, so those ids return `404`.
- `2dhive` always returns its two embed URLs, even for an episode that does not exist (episode `999` returned `200` with only the embeds).
- `mkissa` reports `duration` in minutes; the other providers use seconds.
- The same episode can appear with different counts across providers. For One Piece, `kaa` listed 1198 sub entries (including half-number specials) while most others listed 1180.
