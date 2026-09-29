---
description: Convert an anime id between AniList, MAL, TMDB, IMDb, Kitsu, AniDB and more.
icon: arrows-rotate
---

# ID Mappings

Different providers identify the same anime by different ids: Miruro and Anivexa use AniList ids, movie and TV providers use TMDB ids. `/mappings` looks up one id and returns every id it knows for that title, using [ani.zip](https://api.ani.zip).

{% hint style="info" %}
Route: `GET /mappings`
{% endhint %}

### Lookup

`GET /mappings`

Pass one of the ids below. Results are cached for 24 hours when Redis is enabled.

**Query parameters**

| Name | Required | Description |
| --- | --- | --- |
| `anilist_id` | One of these | AniList id |
| `mal_id` | One of these | MyAnimeList id |
| `kitsu_id` | One of these | Kitsu id |
| `anidb_id` | One of these | AniDB id |
| `themoviedb_id` | One of these | TMDB id |
| `imdb_id` | One of these | IMDb id, for example `tt0388629` |

**Example**

```bash
curl "http://localhost:3000/mappings?anilist_id=21"
```

```json
{
  "animeplanet_id": "one-piece",
  "kitsu_id": 12,
  "mal_id": 21,
  "type": "TV",
  "anilist_id": 21,
  "anisearch_id": 2227,
  "anidb_id": 69,
  "notifymoe_id": null,
  "livechart_id": 321,
  "thetvdb_id": 81797,
  "imdb_id": "tt0388629",
  "themoviedb_id": "37854"
}
```

Fields the source does not know are `null`.

## Errors

| Status | Body | When |
| --- | --- | --- |
| `400` | `{ "error": "Provide at least one ID parameter (mal_id, anilist_id, kitsu_id, anidb_id, themoviedb_id, imdb_id)" }` | No id was given |
| `404` | `{ "error": "No mappings found for the given ID" }` | The id is unknown |
