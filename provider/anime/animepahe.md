---
description: Search AnimePahe, list episodes and get sub and dub HLS streams with download links.
icon: bolt
---

# AnimePahe

AnimePahe scrapes [animepahe.pw](https://animepahe.pw) for search, the latest releases, anime details, full episode lists and kwik streams. Each episode usually has Japanese audio and an English dub at 360p, 720p and 1080p, each with a direct HLS link, a proxied link and an MP4 download link. The site sits behind Cloudflare, so the API solves the challenge in a browser on the first request.

{% hint style="info" %}
Base route: `/anime/animepahe`
{% endhint %}

## Routes

| Route | Description |
| --- | --- |
| `GET /anime/animepahe` | Lists the provider's routes |
| `GET /anime/animepahe/search/:query` | Search titles |
| `GET /anime/animepahe/latest` | Latest released episodes |
| `GET /anime/animepahe/info/:id` | Anime details |
| `GET /anime/animepahe/episodes/:id` | Every episode of an anime |
| `GET /anime/animepahe/episode/:id/:session` | Streams for one episode, as newline-delimited JSON |

AnimePahe uses two kinds of id:

* `id` identifies an anime. It is the `id` (or `session`) from search results and the `id` from latest results.
* `session` identifies an episode. It is the `session` from episode lists and latest results.

Successful responses are the result object itself. Errors are `{ "error": "..." }`.

## Overview

### Provider index

`GET /anime/animepahe`

Returns the provider name, a description and its routes.

**Example**

```bash
curl "http://localhost:3000/anime/animepahe"
```

```json
{
  "name": "animepahe",
  "description": "Anime provider backed by animepahe — search, info, episodes and kwik streams.",
  "endpoints": [
    "/anime/animepahe/search/:query",
    "/anime/animepahe/latest",
    "/anime/animepahe/info/:id",
    "/anime/animepahe/episodes/:id",
    "/anime/animepahe/episode/:id/:session"
  ]
}
```

## Search

### Search titles

`GET /anime/animepahe/search/:query`

Returns up to 8 matching titles. `id` and `session` hold the same value, the anime id used by the other routes. A search with no matches returns `{ "results": [] }`.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `query` | Yes | Search text, URL-encoded |

**Example**

```bash
curl "http://localhost:3000/anime/animepahe/search/frieren"
```

```json
{
  "results": [
    {
      "id": "513a0085-6e0d-a1df-9c14-aeec15a286f7",
      "title": "Frieren: Beyond Journey's End",
      "type": "TV",
      "episodes": 28,
      "status": "Finished Airing",
      "year": 2023,
      "score": 9.25,
      "poster": "https://i.animepahe.pw/uploads/posters/93a/93a7bacea37d530426ca2a4ff26a6ae40dd1d2d6feb9dfe4752e0bd0e0bca4e1.webp",
      "session": "513a0085-6e0d-a1df-9c14-aeec15a286f7"
    },
    {
      "id": "28890275-f126-0ca5-07f8-69471455d3bb",
      "title": "Frieren: Beyond Journey's End Season 2",
      "type": "TV",
      "episodes": 10,
      "status": "Finished Airing",
      "year": 2026,
      "score": 8.84,
      "poster": "https://i.animepahe.pw/uploads/posters/d78/d78c3999383262d4f3297378f938b5acb9e3ae7e6c0ae6fbf67347ace49f5ee2.webp",
      "session": "28890275-f126-0ca5-07f8-69471455d3bb"
    }
  ]
}
```

`episodes` is `0` for titles that are still airing, for example One Piece.

### Latest episodes

`GET /anime/animepahe/latest`

Returns the first page of AnimePahe's airing feed, newest first. Each item carries both ids the stream route needs: `id` is the anime and `session` is the episode.

**Example**

```bash
curl "http://localhost:3000/anime/animepahe/latest"
```

```json
{
  "results": [
    {
      "id": "e284995c-d4d8-426b-7049-310897f592b9",
      "title": "I Want to Love You Till Your Dying Day",
      "episode": 13,
      "snapshot": "https://i.animepahe.pw/uploads/snapshots/1b1/1b1d24ddcf936c4ce55380a720a473e1c2ccc982aaecb3d7c82ac96048ed8e32.sm.webp",
      "session": "cf2bdd664a5c74a09ed118bfc192b3121f2a06d075346b8721cfd6c95140feaf",
      "fansub": "SubsPlease",
      "created_at": "2026-09-29 12:38:40"
    },
    {
      "id": "69d89177-7ac8-e4eb-4562-0f2b17992f11",
      "title": "One Piece",
      "episode": 1180,
      "snapshot": "https://i.animepahe.pw/uploads/snapshots/ce2/ce2fe6de2b08bbcabe8c17deab306327ff7a9338962aca8d933919fe6c4bb768.sm.webp",
      "session": "e422a3049ab5b8214fe7f80a68810632d8f84346e4b08ed152cdabcd117d7622",
      "fansub": "SubsPlease",
      "created_at": "2026-09-27 17:00:09"
    }
  ]
}
```

## Details

### Anime info

`GET /anime/animepahe/info/:id`

Returns the title, synopsis, artwork, airing dates, genres and links to the same anime on AniList, MyAnimeList, Kitsu and other databases. Line breaks in the synopsis are kept as `\n`.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Anime id from search or latest |

**Example**

```bash
curl "http://localhost:3000/anime/animepahe/info/513a0085-6e0d-a1df-9c14-aeec15a286f7"
```

```json
{
  "id": "513a0085-6e0d-a1df-9c14-aeec15a286f7",
  "name": "Frieren: Beyond Journey's End",
  "description": "During their decade-long quest to defeat the Demon King, the members of the hero's party...\n\nHowever, the time that Frieren spends with her comrades is equivalent to merely a fraction of her life...",
  "poster": "https://i.animepahe.pw/uploads/posters/93a/93a7bacea37d530426ca2a4ff26a6ae40dd1d2d6feb9dfe4752e0bd0e0bca4e1.webp",
  "background": "https://i.animepahe.pw/uploads/defaults/cover_default3.webp",
  "aired": "Sep 29, 2023 to Mar 22, 2024",
  "duration": "24 minutes",
  "genres": ["Adventure", "Award Winning", "Drama", "Fantasy"],
  "externalLinks": [
    "https://anilist.co/anime/154587",
    "https://anidb.net/anime/17617",
    "https://www.animenewsnetwork.com/encyclopedia/anime.php?id=26334",
    "https://kitsu.app/anime/46474",
    "https://myanimelist.net/anime/52991"
  ]
}
```

`poster` and `background` are `null` when the page has no image.

## Episodes

### Episode list

`GET /anime/animepahe/episodes/:id`

Returns every episode, sorted by episode number. The API fetches all of AnimePahe's release pages in parallel, so long series come back in one response (all 1180 One Piece episodes took about 1.4 seconds). `filler` is `true` for filler episodes, and `title` falls back to `Episode <n>` when the site has none.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Anime id from search or latest |

**Example**

```bash
curl "http://localhost:3000/anime/animepahe/episodes/513a0085-6e0d-a1df-9c14-aeec15a286f7"
```

```json
{
  "results": [
    {
      "title": "Episode 1",
      "episode": 1,
      "released": "2023-09-29T09:33:47.000Z",
      "snapshot": "https://i.animepahe.pw/uploads/snapshots/807/80781666d15cb43eb2892191a9b66ad07688aeae40319e02a9f12203223b3d44.sm.webp",
      "duration": "00:26:01",
      "filler": false,
      "session": "8ca7f1c359bc4ca807f72f2bab81a07b44833d85968ac491a253eb580adb6135"
    },
    {
      "title": "Episode 2",
      "episode": 2,
      "released": "2023-09-29T17:27:53.000Z",
      "snapshot": "https://i.animepahe.pw/uploads/snapshots/af6/af6c1923ca5ddad3341ee7c6c3ac9c336f7610604a4a9a672b763986b47a83a7.sm.webp",
      "duration": "00:26:01",
      "filler": false,
      "session": "d2719a0c3b084d4207bbe0aac33d75baa844a417a346e780215840dd42238619"
    }
  ]
}
```

## Streams

### Episode streams

`GET /anime/animepahe/episode/:id/:session`

Returns every audio and quality variant of one episode. The response is newline-delimited JSON (`Content-Type: application/x-ndjson; charset=utf-8`): one stream object per line, written as each variant is resolved. Read it line by line instead of parsing the whole body as one JSON value.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id` | Yes | Anime id |
| `session` | Yes | Episode session from the episode list or the latest feed |

**Example**

```bash
curl "http://localhost:3000/anime/animepahe/episode/513a0085-6e0d-a1df-9c14-aeec15a286f7/8ca7f1c359bc4ca807f72f2bab81a07b44833d85968ac491a253eb580adb6135"
```

The response had six lines (`jpn` and `eng` at 360p, 720p and 1080p). The first two:

```json
{"id":"513a0085-6e0d-a1df-9c14-aeec15a286f7--360--jpn","title":"jpn / 360p","url":"https://kwik.cx/e/aeNSh4eblrse","directUrl":"https://vault-08.uwucdn.top/stream/08/13/63abd0640a098853df01676699553c949b1b3038117d9f59232d56ca53be3fef/uwu.m3u8","proxiedUrl":"http://localhost:3000/proxy/m3u8-proxy?url=https%3A%2F%2Fvault-08.uwucdn.top%2Fstream%2F08%2F13%2F63abd0640a098853df01676699553c949b1b3038117d9f59232d56ca53be3fef%2Fuwu.m3u8&headers=%7B%22Referer%22%3A%22https%3A%2F%2Fkwik.cx%2F%22%7D","quality":"360","audio":"jpn","downloadUrl":"https://vault-08.uwucdn.top/mp4/08/13/63abd0640a098853df01676699553c949b1b3038117d9f59232d56ca53be3fef?file=Frieren_Beyond_Journey_s_End_-_Sub_-_360p_-_Episode_1.mp4","corsHeaders":{"Referer":"https://kwik.cx/"}}
{"id":"513a0085-6e0d-a1df-9c14-aeec15a286f7--720--jpn","title":"jpn / 720p","url":"https://kwik.cx/e/d3ccaeXzK7o4","directUrl":"https://vault-08.uwucdn.top/stream/08/13/71ee7618f3b7b9ad4467c6fdcd0d0bbc4af2345d95d9a793d71db77539a43af7/uwu.m3u8","proxiedUrl":"http://localhost:3000/proxy/m3u8-proxy?url=https%3A%2F%2Fvault-08.uwucdn.top%2Fstream%2F08%2F13%2F71ee7618f3b7b9ad4467c6fdcd0d0bbc4af2345d95d9a793d71db77539a43af7%2Fuwu.m3u8&headers=%7B%22Referer%22%3A%22https%3A%2F%2Fkwik.cx%2F%22%7D","quality":"720","audio":"jpn","downloadUrl":"https://vault-08.uwucdn.top/mp4/08/13/71ee7618f3b7b9ad4467c6fdcd0d0bbc4af2345d95d9a793d71db77539a43af7?file=Frieren_Beyond_Journey_s_End_-_Sub_-_720p_-_Episode_1.mp4","corsHeaders":{"Referer":"https://kwik.cx/"}}
```

**Stream fields**

| Field | Description |
| --- | --- |
| `id` | `<anime id>--<quality>--<audio>` |
| `title` | `<audio> / <quality>p` |
| `url` | The kwik embed page |
| `directUrl` | The HLS playlist. It only plays with `Referer: https://kwik.cx/` |
| `proxiedUrl` | The same playlist through the API's [stream proxy](../../core/proxy.md), with the referer already attached. Use this in a browser player |
| `quality` | `360`, `720` or `1080` |
| `audio` | `jpn` for the original audio, `eng` for the English dub |
| `downloadUrl` | An MP4 download link with a readable file name, or `null` |
| `corsHeaders` | Headers needed to fetch `directUrl` yourself |

Reading the stream in TypeScript:

```ts
const res = await fetch(
  "http://localhost:3000/anime/animepahe/episode/513a0085-6e0d-a1df-9c14-aeec15a286f7/8ca7f1c359bc4ca807f72f2bab81a07b44833d85968ac491a253eb580adb6135",
);
const reader = res.body!.pipeThrough(new TextDecoderStream()).getReader();
let buffer = "";
for (;;) {
  const { value, done } = await reader.read();
  if (done) break;
  buffer += value;
  const lines = buffer.split("\n");
  buffer = lines.pop()!;
  for (const line of lines) if (line) console.log(JSON.parse(line).title);
}
```

A variant that cannot be resolved is skipped; the other lines are still sent.

## Errors

| Status | When |
| --- | --- |
| `404` | `info` or `episodes`: the anime id does not exist (`{ "error": "Anime not found" }`). `episode`: the session is wrong or the episode has no playable streams (`{ "error": "No streams found" }`) |
| `502` | AnimePahe failed, returned something that is not JSON, or blocked the request and the Cloudflare challenge could not be solved, for example `{ "error": "Animepahe responded 503" }` |

Search and latest never return `404`; no matches gives `{ "results": [] }`.

## Notes

* The first request after the server starts is slower because the API solves AnimePahe's Cloudflare challenge in a headless browser (about 7 seconds in testing). Later requests reuse the clearance cookie and take well under a second. If the challenge cannot be solved, the API waits 10 minutes before trying the browser again and returns `502` in the meantime.
* The stream route waits for the first variant before it answers, so a `404` arrives as normal JSON rather than as an empty stream.
* Typical flow: `search` → `episodes/:id` → `episode/:id/:session`. The `latest` feed already gives you `id` and `session`, so you can go straight to the stream route.
* `externalLinks` in `info` contains the AniList id (`https://anilist.co/anime/154587`), which you can use with AniList-based providers such as [Miruro](miruro.md) and [Animelok](animelok.md).
