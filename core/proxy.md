---
description: Replay HLS, MP4, subtitle and file requests with the headers the source site expects.
icon: shuffle
---

# Stream Proxy

Video hosts often refuse requests that do not carry their own `Referer` or `Origin`, and browsers cannot set those headers. The stream proxy fetches the resource with the headers you give it and streams it back with CORS enabled.

You rarely build these URLs yourself: streams and subtitles in provider responses already carry a `proxiedUrl` that points here.

{% hint style="info" %}
Base route: `/proxy`
{% endhint %}

## Routes

| Route | Description |
| --- | --- |
| `GET /proxy` | Lists the proxy routes |
| `GET /proxy/m3u8-proxy` | HLS playlists, rewritten so every nested playlist, segment and key also goes through the proxy |
| `GET /proxy/ts-segment` | HLS media segments |
| `GET /proxy/mp4-proxy` | MP4 and other progressive video, with byte-range support |
| `GET /proxy/fetch` | Any other file, such as subtitles or encryption keys |

### Parameters

All four media routes take the same query parameters.

| Name | Required | Description |
| --- | --- | --- |
| `url` | Yes | The upstream URL, URL-encoded |
| `headers` | No | A URL-encoded JSON object of request headers, for example `{"Referer":"https://kwik.cx/"}` |

### HLS playlists

`GET /proxy/m3u8-proxy`

Fetches the playlist and rewrites every URI in it. Variant playlists, audio and subtitle renditions go back through `m3u8-proxy`, media segments through `ts-segment`, and keys or maps through `fetch`. The headers you pass are kept on every rewritten link.

```bash
curl "http://localhost:3000/proxy/m3u8-proxy?url=https%3A%2F%2Ftest-streams.mux.dev%2Fx36xhzz%2Fx36xhzz.m3u8"
```

```
#EXTM3U
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=2149280,CODECS="mp4a.40.2,avc1.64001f",RESOLUTION=1280x720,NAME="720"
http://localhost:3000/proxy/m3u8-proxy?url=https%3A%2F%2Ftest-streams.mux.dev%2Fx36xhzz%2Furl_0%2F193039199_mp4_h264_aac_hd_7.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=246440,CODECS="mp4a.40.5,avc1.42000d",RESOLUTION=320x184,NAME="240"
http://localhost:3000/proxy/m3u8-proxy?url=https%3A%2F%2Ftest-streams.mux.dev%2Fx36xhzz%2Furl_2%2F193039199_mp4_h264_aac_ld_7.m3u8
```

Point any HLS player, such as hls.js, Video.js or a native `<video>` element in Safari, at the proxied playlist URL.

### Segments

`GET /proxy/ts-segment`

Streams a single media segment. Some hosts disguise segments as images or HTML pages; the proxy strips that wrapper and labels the result as `video/mp2t`. Subtitle segments keep their `text/vtt` type. Segments are sent with `Cache-Control: public, max-age=86400`.

### MP4

`GET /proxy/mp4-proxy`

Streams progressive video and passes `Range` requests through, so seeking works.

```bash
curl -r 0-1023 -o /dev/null -D - "http://localhost:3000/proxy/mp4-proxy?url=https%3A%2F%2Ftest-videos.co.uk%2Fvids%2Fbigbuckbunny%2Fmp4%2Fh264%2F360%2FBig_Buck_Bunny_360_10s_1MB.mp4"
```

```
HTTP/1.1 206 Partial Content
Content-Type: video/mp4
Content-Range: bytes 0-1023/991017
Accept-Ranges: bytes
```

### Files

`GET /proxy/fetch`

Streams any other file, for example WebVTT or SRT subtitles and HLS encryption keys.

## Limits and safety

* Only `http` and `https` URLs on public addresses are allowed. Requests to `localhost`, private networks and link-local addresses are refused, and every redirect is checked the same way, up to 5 redirects.
* Maximum sizes: 5 MB for playlists, 50 MB for segments and files, 20 GB for MP4.
* If a host rejects a request with `403`, the proxy retries it with a browser-like TLS fingerprint and remembers which hosts need it.

## Errors

Errors are plain text.

| Status | Body | When |
| --- | --- | --- |
| `400` | `Invalid url` | `url` is not a valid URL |
| `400` | `Invalid headers format` | `headers` is not a JSON object of strings |
| `403` | `Forbidden url` | The URL, or a redirect, points at a private or local address |
| `413` | `Payload too large` | The upstream response is larger than the limit |
| `422` | Elysia validation error | `url` is missing |
| `502` | `Upstream request failed` | The upstream could not be reached |
| Upstream status | Upstream body | The upstream answered with an error, for example `404` |
