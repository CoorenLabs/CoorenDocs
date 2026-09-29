---
description: Feeds, rich filters, tags, multi-language chapter lists and page images from MangaBall.
icon: book
---

# MangaBall

MangaBall uses the JSON API behind [mangaball.com](https://mangaball.com), a large catalog of manga, manhwa, manhua and comics (about 187,000 titles on 2026-09-29). It has the richest browsing of the manga providers: feeds, origin and status lists, tag, demographic and sort filters, and chapters in many languages from several scanlation groups. Lists include 18+ titles, flagged with `is18plus`.

{% hint style="info" %}
Base route: `/manga/mangaball`
{% endhint %}

## Routes

| Route | Description |
| --- | --- |
| `GET /manga/mangaball` | Provider status |
| `GET /manga/mangaball/home` | Featured titles |
| `GET /manga/mangaball/recommendation` | Recommended titles |
| `GET /manga/mangaball/latest` | Recently updated titles |
| `GET /manga/mangaball/new-chap` | Recently updated titles (same as `/latest`) |
| `GET /manga/mangaball/added` | Recently added titles |
| `GET /manga/mangaball/popular` | Most viewed titles |
| `GET /manga/mangaball/foryou` | Most read titles in a period |
| `GET /manga/mangaball/recent` | Most read chapters in a period |
| `GET /manga/mangaball/origin` | 12 titles from one origin |
| `GET /manga/mangaball/manga` | Japanese titles |
| `GET /manga/mangaball/manhwa` | Korean titles |
| `GET /manga/mangaball/manhua` | Chinese titles |
| `GET /manga/mangaball/comics` | English titles |
| `GET /manga/mangaball/ongoing` | Ongoing titles |
| `GET /manga/mangaball/completed` | Completed titles |
| `GET /manga/mangaball/on-hold` | On-hold titles |
| `GET /manga/mangaball/cancelled` | Cancelled titles |
| `GET /manga/mangaball/hiatus` | Titles on hiatus |
| `GET /manga/mangaball/search` | Search titles |
| `GET /manga/mangaball/keyword/:keyword` | Search titles, with the text in the path |
| `GET /manga/mangaball/filters` | Search with filters and sorting |
| `GET /manga/mangaball/tags` | All tags, grouped |
| `GET /manga/mangaball/tags-detail` | Tag statistics |
| `GET /manga/mangaball/tags/:id_tags` | Titles with one tag |
| `GET /manga/mangaball/detail/:slug` | Full details and chapter list |
| `GET /manga/mangaball/read/:id_chapter` | Page images of one chapter |
| `GET /manga/mangaball/image/*` | Image proxy |

Every route except the status route and the image proxy returns `{ "status", "success", "data" }`. Errors return the same envelope with a `message` and `data: null`.

### Title lists

List routes return `data.data`, an array of titles, and paged routes add `data.pagination`:

```json
{
  "status": 200,
  "success": true,
  "data": {
    "data": [
      {
        "_id": "685154f0702284f834178433",
        "title": "Sousou no Frieren",
        "alternateTitle": "Frieren: Beyond Journey's End / Frieren at the Funeral / Фрирен, провожающая в последний путь / 葬送のフ...",
        "thumbnail": "http://localhost:3000/manga/mangaball/image/bulbasaur.poke-black-and-white.net/covers/685154f0702284f834178433/cover_1752073746362.webp",
        "tags": [
          { "tag": "Award Winning", "id_tags": "685148fe15e8b86aae68e5a7" }
        ],
        "authors": [
          { "authors": "Yamada Kanehito", "id_authors": "68514d08d2df9377738b7e38" }
        ],
        "status": "Hiatus",
        "slug": "sousou-no-frieren-685154f0702284f834178433",
        "originalLanguage": "ja",
        "updated_at": "2026-05-27T07:00:08.865000",
        "stats_count": {
          "chapters": 33,
          "syncChapter": 1,
          "views": 152219,
          "followers": 103,
          "likes": 4,
          "views_today": 2,
          "views_week": 1059,
          "views_month": 15528
        },
        "is18plus": false
      }
    ],
    "pagination": {
      "total": 39,
      "page": 1,
      "limit": 2,
      "total_pages": 20
    }
  }
}
```

* Fields with no value are left out, so `alternateTitle`, `authors`, `description` and some `stats_count` keys are missing on some titles.
* `alternateTitle` joins all alternative titles with ` / `.
* `slug` is what `/detail/:slug` expects. It ends with the 24-character `_id`.
* `status` is one of `Ongoing`, `Completed`, `On-Hold`, `Cancelled`, `Hiatus`.
* `originalLanguage` is a language code such as `ja`, `ko`, `cn`, `en` or `fr`.
* `limit` is clamped to 1 to 100 on every route that takes it.

## Feeds

### Home

`GET /manga/mangaball/home`

Returns MangaBall's 10 featured titles. There is no `pagination`.

**Example**

```bash
curl "http://localhost:3000/manga/mangaball/home"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "data": [
      {
        "_id": "6a3fda4fdce36b338075d6a3",
        "title": "My Husband Is Definitely a Paladin",
        "alternateTitle": "Holy Night: My Husband is Definitely a Paladin / 남편은 분명 성기사였는데 / My Husband Was Clearly a...",
        "thumbnail": "http://localhost:3000/manga/mangaball/image/bulbasaur.poke-black-and-white.net/covers/6a3fda4fdce36b338075d6a3/cover_1789972998854.webp",
        "tags": [
          { "tag": "Fantasy", "id_tags": "685146c5f3ed681c80f257ea" }
        ],
        "authors": [
          { "authors": "Jagae", "id_authors": "698174836f2a7060594e3af1" }
        ],
        "status": "Ongoing",
        "slug": "my-husband-is-definitely-a-paladin-6a3fda4fdce36b338075d6a3",
        "description": "\"Tonight, too, I want to receive your purification.\"\n\nIt was a miserable life. Despite pos...",
        "originalLanguage": "ko",
        "updated_at": "2026-06-27T14:12:31.655000",
        "stats_count": {
          "chapters": 32,
          "syncChapter": 0,
          "views": 125664,
          "followers": 262,
          "likes": 70,
          "views_today": 733,
          "views_week": 7972,
          "views_month": 24786
        },
        "is18plus": false
      }
    ]
  }
}
```

### Recommended

`GET /manga/mangaball/recommendation`

Returns recommended titles. There is no `pagination`.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `limit` | No | `12` | Number of titles, 1 to 100 |

**Example**

```bash
curl "http://localhost:3000/manga/mangaball/recommendation?limit=5"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "data": [
      {
        "_id": "68516268702284f83417af11",
        "title": "Chainsmoker Cat",
        "alternateTitle": "ヤニねこ / Yani Neko / Tar Cat / Cô mèo nghiện thuốc / Chainsmok...",
        "thumbnail": "http://localhost:3000/manga/mangaball/image/bulbasaur.poke-black-and-white.net/covers/68516268702284f83417af11/cover_1751974204895.webp",
        "tags": [
          { "tag": "Psychological", "id_tags": "685148d715e8b86aae68e507" }
        ],
        "authors": [
          { "authors": "Nyan Nyan Factory", "id_authors": "68516268702284f83417af10" }
        ],
        "status": "Ongoing",
        "slug": "chainsmoker-cat-68516268702284f83417af11",
        "description": "A chain-smoking catgirl struggles with her cigarette addicti...",
        "originalLanguage": "ja",
        "updated_at": "2026-09-29T12:34:33.289000",
        "stats_count": {
          "chapters": 50,
          "views": 70016,
          "followers": 13,
          "likes": 3,
          "views_today": 77,
          "views_week": 719,
          "views_month": 9387
        },
        "is18plus": false
      }
    ]
  }
}
```

### Latest, new chapters, recently added, popular

* `GET /manga/mangaball/latest` returns titles by latest chapter update.
* `GET /manga/mangaball/new-chap` is the same list as `/latest`.
* `GET /manga/mangaball/added` returns titles by the date they were added.
* `GET /manga/mangaball/popular` returns titles by total views.

All four return a paged [title list](#title-lists).

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `page` | No | `1` | Page number |
| `limit` | No | `24` | Titles per page, 1 to 100 |

**Example**

```bash
curl "http://localhost:3000/manga/mangaball/popular?limit=1"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "data": [
      {
        "_id": "685155e6702284f834178607",
        "title": "Tears on a Withered Flower",
        "alternateTitle": "시든 꽃에 눈물을 / Łzy ze zwiędłych kwiatów / Lágrimas Sobre Flores...",
        "thumbnail": "http://localhost:3000/manga/mangaball/image/bulbasaur.poke-black-and-white.net/covers/685155e6702284f834178607/cover_1752065825262.webp",
        "tags": [
          { "tag": "Psychological", "id_tags": "685148d715e8b86aae68e507" }
        ],
        "authors": [
          { "authors": "Gae (개)", "id_authors": "68514ec1d2df9377738b827d" }
        ],
        "status": "Ongoing",
        "slug": "tears-on-a-withered-flower-685155e6702284f834178607",
        "originalLanguage": "ko",
        "updated_at": "2026-09-25T16:37:34.947000",
        "stats_count": {
          "chapters": 136,
          "views": 2125926,
          "followers": 1000,
          "likes": 1452,
          "views_today": 573,
          "views_week": 10840,
          "views_month": 83753
        },
        "is18plus": false
      }
    ],
    "pagination": {
      "total": 187121,
      "page": 1,
      "limit": 1,
      "total_pages": 187121
    }
  }
}
```

### Most read titles

`GET /manga/mangaball/foryou`

Returns the titles whose chapters were read most in a period. Each title appears once, so you can get fewer results than `limit`. Entries only have `_id`, `title`, `thumbnail` and `slug`, and there is no `pagination`.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `time` | No | `day` | `day` (or `today`), `week` or `month`. Any other value returns `400` |
| `limit` | No | `12` | Number of chapters to rank before titles are merged, 1 to 100 |

**Example**

```bash
curl "http://localhost:3000/manga/mangaball/foryou?time=week&limit=5"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "data": [
      {
        "_id": "68515487702284f83417836f",
        "title": "Shangri-La Frontier ~Kusoge Hunter, Kamige ni Idoman to su~",
        "thumbnail": "http://localhost:3000/manga/mangaball/image/bulbasaur.poke-black-and-white.net/covers/68515487702284f83417836f/cover_1752077166895.webp",
        "slug": "68515487702284f83417836f"
      },
      {
        "_id": "688c3da5a943baf927094de1",
        "title": "A Dangerous Deal and the Girl Next Door",
        "thumbnail": "http://localhost:3000/manga/mangaball/image/bulbasaur.poke-black-and-white.net/covers/688c3da5a943baf927094de1/cover_1754022310395.jpg",
        "slug": "688c3da5a943baf927094de1"
      }
    ]
  }
}
```

Here `slug` is the bare `_id`, which `/detail/:slug` also accepts.

### Most read chapters

`GET /manga/mangaball/recent`

Returns the chapters read most in a period, one entry per chapter, so a title can appear more than once. Same parameters as `/foryou`. Each entry adds `chapter_id` (usable with `/read`), `views` for the period, and `chapter_number` or `chapter_name` when MangaBall has them.

**Example**

```bash
curl "http://localhost:3000/manga/mangaball/recent"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "data": [
      {
        "_id": "68516038702284f83417a6b6",
        "title": "Blue Period",
        "thumbnail": "http://localhost:3000/manga/mangaball/image/bulbasaur.poke-black-and-white.net/covers/68516038702284f83417a6b6/cover_1751992207282.webp",
        "slug": "68516038702284f83417a6b6",
        "chapter_id": "6aba8ca30fc93f22ec0d0df5",
        "chapter_number": "90",
        "views": 477
      },
      {
        "_id": "69659d3ad0d7677ebf1b8367",
        "title": "My Girlfriend Was Already Fully Trained",
        "thumbnail": "http://localhost:3000/manga/mangaball/image/bulbasaur.poke-black-and-white.net/covers/69659d3ad0d7677ebf1b8367/cover_1768267265642.jpg",
        "slug": "69659d3ad0d7677ebf1b8367",
        "chapter_id": "6aba7be463c5c72e32019d1f",
        "chapter_name": "Chapter 43",
        "views": 131
      }
    ]
  }
}
```

### By origin

`GET /manga/mangaball/origin`

Returns 12 titles from one origin. `page` and `limit` are ignored and there is no `pagination`; use the [browse routes](#browse) for paged lists.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `origin` | No | `all` | `manga` (or `jp`, `ja`), `manhwa` (or `kr`, `ko`), `manhua` (or `zh`, `cn`), `comics` (or `en`), or `all`. Other values return an empty list |

**Example**

```bash
curl "http://localhost:3000/manga/mangaball/origin?origin=manhwa"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "data": [
      {
        "_id": "685204c868c513c5035d656e",
        "title": "A Compendium of Ghosts",
        "alternateTitle": "신기록 / A Compendium of Ghosts",
        "thumbnail": "http://localhost:3000/manga/mangaball/image/bulbasaur.poke-black-and-white.net/covers/685204c868c513c5035d656e/cover_1754814908516.jpg",
        "tags": [
          { "tag": "Drama", "id_tags": "685148cf15e8b86aae68e4dd" }
        ],
        "status": "Ongoing",
        "slug": "a-compendium-of-ghosts-685204c868c513c5035d656e",
        "description": "Sang, once the little boy too afraid to jump into the lake w...",
        "originalLanguage": "ko",
        "updated_at": "2025-08-10T08:35:11.062000",
        "stats_count": {
          "chapters": 3,
          "syncChapter": 1,
          "views": 266,
          "views_today": 6,
          "views_week": 6,
          "views_month": 92
        },
        "is18plus": false
      }
    ]
  }
}
```

## Browse

### By origin or status

| Route | Lists |
| --- | --- |
| `GET /manga/mangaball/manga` | Japanese manga |
| `GET /manga/mangaball/manhwa` | Korean manhwa |
| `GET /manga/mangaball/manhua` | Chinese manhua |
| `GET /manga/mangaball/comics` | English comics |
| `GET /manga/mangaball/ongoing` | Ongoing titles |
| `GET /manga/mangaball/completed` | Completed titles |
| `GET /manga/mangaball/on-hold` | On-hold titles |
| `GET /manga/mangaball/cancelled` | Cancelled titles |
| `GET /manga/mangaball/hiatus` | Titles on hiatus |

Each returns a paged [title list](#title-lists) sorted by latest update. They are shortcuts for `/filters` with `original_lang` or `status` set.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `page` | No | `1` | Page number |
| `limit` | No | `24` | Titles per page, 1 to 100 |

**Example**

```bash
curl "http://localhost:3000/manga/mangaball/completed?limit=1"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "data": [
      {
        "_id": "68519003c938d678d9652078",
        "title": "Report to the Prince Regent: She, the Big Shot, Specializes in Treating Infertility",
        "alternateTitle": "Baogao Shezheng Wang: Da Lao Ta Zhuan Zhi Bu Yun Bu Yu / 报告摄...",
        "thumbnail": "http://localhost:3000/manga/mangaball/image/bulbasaur.poke-black-and-white.net/covers/68519003c938d678d9652078/cover_1754817358045.jpg",
        "tags": [
          { "tag": "Reincarnation", "id_tags": "685146c5f3ed681c80f257e1" }
        ],
        "status": "Completed",
        "slug": "report-to-the-prince-regent-she-the-big-shot-specializes-in-treating-infertility-68519003c938d678d9652078",
        "originalLanguage": "cn",
        "updated_at": "2026-09-29T12:42:59.050000",
        "stats_count": {
          "chapters": 25,
          "views": 47,
          "views_today": 2,
          "views_week": 2,
          "views_month": 2
        },
        "is18plus": false
      }
    ],
    "pagination": {
      "total": 67664,
      "page": 1,
      "limit": 1,
      "total_pages": 67664
    }
  }
}
```

{% hint style="warning" %}
`/on-hold` returned an empty list (`total: 0`) on 2026-09-29, so MangaBall currently has no titles marked on hold.
{% endhint %}

## Search and filters

### Search

`GET /manga/mangaball/search`

Searches titles by text and returns a paged [title list](#title-lists) in MangaBall's default search order. No match returns an empty list with `total: 0`.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `q` | Yes | | Search text |
| `page` | No | `1` | Page number |
| `limit` | No | `24` | Titles per page, 1 to 100 |

**Example**

```bash
curl "http://localhost:3000/manga/mangaball/search?q=frieren&limit=2"
```

The response is the one shown under [Title lists](#title-lists).

### Keyword

`GET /manga/mangaball/keyword/:keyword`

The same text search as `/search`, with the text in the path. It does not accept the `id_keywords` values from `/detail`; an id returns an empty list.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `keyword` | Yes | Search text, for example `dungeon` |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `page` | No | `1` | Page number |
| `limit` | No | `24` | Titles per page, 1 to 100 |

**Example**

```bash
curl "http://localhost:3000/manga/mangaball/keyword/dungeon?limit=1"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "data": [
      {
        "_id": "68515639702284f83417868d",
        "title": "Dungeon Elf ~Dungeon ni Takarabako ga Aru no wa Atarimae desu ka~",
        "alternateTitle": "ダンジョンエルフ　～ダンジョンに宝箱があるのは当たり前ですか～ / Dungeon Elf: What's a Dung...",
        "thumbnail": "http://localhost:3000/manga/mangaball/image/bulbasaur.poke-black-and-white.net/covers/68515639702284f83417868d/cover_1752063664590.webp",
        "tags": [
          { "tag": "Adventure", "id_tags": "685146c5f3ed681c80f257e6" }
        ],
        "authors": [
          { "authors": "River Slan", "id_authors": "68514f49d2df9377738b8381" }
        ],
        "status": "Ongoing",
        "slug": "dungeon-elf-dungeon-ni-takarabako-ga-aru-no-wa-atarimae-desu-ka-68515639702284f83417868d",
        "originalLanguage": "ja",
        "updated_at": "2026-07-15T00:44:36.647000",
        "stats_count": {
          "chapters": 8,
          "views": 6881,
          "followers": 9,
          "views_today": 0,
          "views_week": 66,
          "views_month": 559
        },
        "is18plus": false
      }
    ],
    "pagination": {
      "total": 398,
      "page": 1,
      "limit": 1,
      "total_pages": 398
    }
  }
}
```

### Filters

`GET /manga/mangaball/filters`

Searches with any combination of text, origin, status, demographic and tag filters, and a sort order. Returns a paged [title list](#title-lists). Every parameter is optional; with none it lists all titles by latest update, 10 per page. Filter values that MangaBall does not know return an empty list rather than an error.

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `q` | No | | Search text |
| `sort` | No | latest update, or MangaBall's search order when `q` is set | Sort field with an optional `_asc` or `_desc` suffix, for example `views_desc`. See the table below |
| `order` | No | `desc` | `asc` or `desc`, used when `sort` has no suffix |
| `original_lang` | No | `any` | `manga` (or `jp`, `ja`), `manhwa` (or `kr`, `ko`), `manhua` (or `zh`, `cn`), `comics` (or `en`), `any` |
| `status` | No | `any` | `ongoing`, `completed`, `on_hold`, `cancelled`, `hiatus`, `any` |
| `demographic` | No | `any` | `shounen`, `shoujo`, `seinen`, `josei`, `any` |
| `tag_included` | No | | Comma-separated tag ids (`id_tags` from `/tags`) a title must have |
| `tag_included_mode` | No | `and` | `and`: a title needs every included tag. `or`: any one of them |
| `tag_excluded` | No | | Comma-separated tag ids a title must not have |
| `page` | No | `1` | Page number |
| `limit` | No | `10` | Titles per page, 1 to 100 |

| `sort` value | Sorts by |
| --- | --- |
| `updated_chapters` (or `updated`, `latest`) | Latest chapter update |
| `created_at` (or `added`) | Date added |
| `views` | Total views |
| `rating_average` | Rating |
| `title` | Title, alphabetically |

Unknown `sort` values do not cause an error; the results come back in MangaBall's default order. Pass several tag ids as one comma-separated value; repeating `tag_included` keeps only the last one.

**Example**

```bash
curl "http://localhost:3000/manga/mangaball/filters?q=dungeon&original_lang=kr&sort=views_desc&limit=1"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "data": [
      {
        "_id": "68515729702284f83417883d",
        "title": "Dungeon Odyssey",
        "alternateTitle": "Dungeon Experience Book / 던전 견문록 / Deonjeon Gyeonmunnok / Mémoires d'un Donjon / Récits de Donjon / Donjon Odyssée / Records of Dungeon Travel / Odisseia da Mas...",
        "thumbnail": "http://localhost:3000/manga/mangaball/image/bulbasaur.poke-black-and-white.net/covers/68515729702284f83417883d/cover_1752056464315.webp",
        "tags": [
          { "tag": "Monsters", "id_tags": "685146c5f3ed681c80f257e2" },
          { "tag": "Action", "id_tags": "685146c5f3ed681c80f257e3" }
        ],
        "authors": [
          { "authors": "Glumph", "id_authors": "68515094d2df9377738b870e" },
          { "authors": "Son Min-woo", "id_authors": "68515094d2df9377738b870f" }
        ],
        "status": "Hiatus",
        "slug": "dungeon-odyssey-68515729702284f83417883d",
        "originalLanguage": "ko",
        "updated_at": "2026-01-11T15:54:50.739000",
        "stats_count": {
          "chapters": 129,
          "followers": 93,
          "views": 125192,
          "likes": 2,
          "views_today": 19,
          "views_week": 888,
          "views_month": 9354
        },
        "is18plus": false
      }
    ],
    "pagination": {
      "total": 36,
      "page": 1,
      "limit": 1,
      "total_pages": 36
    }
  }
}
```

Tag example: `tag_included=685146c5f3ed681c80f257e3,685146c5f3ed681c80f257e9` (Action and Isekai) returned 1,478 titles, and adding `tag_included_mode=or` returned 23,236.

## Tags

### All tags

`GET /manga/mangaball/tags`

Returns every tag grouped by `group`: `genre`, `theme`, `format`, `content` and `origin`. Use `id_tags` with `/tags/:id_tags` or the `tag_included` and `tag_excluded` filters.

**Example**

```bash
curl "http://localhost:3000/manga/mangaball/tags"
```

The response below shows two of the five groups, each trimmed to one tag.

```json
{
  "status": 200,
  "success": true,
  "data": {
    "data": {
      "genre": [
        {
          "name": "Action",
          "slug": "action",
          "description": {},
          "group": "genre",
          "mangadex_id": "391b0423-d847-456f-aff0-8b0cfc03066b",
          "version": 1,
          "stats": {
            "manga": 0,
            "chapter": 0,
            "title": 19403,
            "titlePercent": 11.24
          },
          "created_at": "2025-06-17T10:43:17.403000",
          "updated_at": "2026-03-06T01:20:06.420000",
          "id_tags": "685146c5f3ed681c80f257e3"
        }
      ],
      "theme": [
        {
          "name": "Reincarnation",
          "slug": "reincarnation",
          "description": {},
          "group": "theme",
          "mangadex_id": "0bc90acb-ccc1-44ca-a34a-b9f3a73259d0",
          "version": 1,
          "stats": {
            "manga": 0,
            "chapter": 0,
            "title": 3302,
            "titlePercent": 1.91
          },
          "created_at": "2025-06-17T10:43:17.242000",
          "updated_at": "2025-10-13T10:26:46.848000",
          "id_tags": "685146c5f3ed681c80f257e1"
        }
      ]
    }
  }
}
```

On 2026-09-29 there were 36 genre, 42 theme, 12 format, 2 content and 4 origin tags.

### Tag statistics

`GET /manga/mangaball/tags-detail`

Returns catalog totals, per-group tag counts and a flat list of every tag with its title count. Counts in the `*_info` objects are display strings such as `39.8K`; `all_tags[].count` is a number.

**Example**

```bash
curl "http://localhost:3000/manga/mangaball/tags-detail"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "tags_info": {
      "total_tags": "96",
      "total_title": "187.1K",
      "top_genre": "Romance",
      "avg/tag": "7.2K",
      "manga": "141.0K",
      "manhwa": "12.4K",
      "manhua": "6.2K",
      "comics": "11.0K"
    },
    "genre_info": {
      "total_tags": "36",
      "total_title": "260.0K",
      "total_avg": "37.4%",
      "tags": [
        { "genre": "Romance", "count": "39.8K", "avg": "23.08%" }
      ]
    },
    "theme_info": {
      "total_tags": "42",
      "total_title": "170.6K",
      "total_avg": "24.5%",
      "tags": [
        { "theme": "School Life", "count": "18.4K", "avg": "10.66%" }
      ]
    },
    "format_info": {
      "total_tags": "12",
      "total_title": "88.5K",
      "total_avg": "12.7%",
      "tags": [
        { "format": "Oneshot", "count": "16.6K", "avg": "9.59%" }
      ]
    },
    "content_info": {
      "total_tags": "2",
      "total_title": "5.8K",
      "total_avg": "0.8%",
      "tags": [
        { "content": "Gore", "count": "3.4K", "avg": "1.96%" }
      ]
    },
    "origin_info": {
      "total_tags": "4",
      "total_title": "170.6K",
      "total_avg": "24.5%",
      "tags": [
        { "origin": "Manga", "count": "141.0K", "avg": "81.66%" }
      ]
    },
    "all_tags": [
      {
        "id_tags": "685146c5f3ed681c80f257e3",
        "name": "Action",
        "slug": "action",
        "group": "genre",
        "count": 19403
      }
    ]
  }
}
```

### Titles with a tag

`GET /manga/mangaball/tags/:id_tags`

Returns a paged [title list](#title-lists) of titles that have one tag, sorted by latest update. An unknown id returns an empty list.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id_tags` | Yes | Tag id from `/tags`, `/tags-detail` or a title's `tags`, for example `685146c5f3ed681c80f257e3` (Action) |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `page` | No | `1` | Page number |
| `limit` | No | `24` | Titles per page, 1 to 100 |

**Example**

```bash
curl "http://localhost:3000/manga/mangaball/tags/685146c5f3ed681c80f257e3?limit=1"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "data": [
      {
        "_id": "6852084c68c513c5035d7047",
        "title": "Apocalyptic Forecast",
        "alternateTitle": "天启预报 / Apocalyptic Forecast / Tianqi Yubao (Colored)",
        "thumbnail": "http://localhost:3000/manga/mangaball/image/bulbasaur.poke-black-and-white.net/covers/6852084c68c513c5035d7047/cover_1754805805017.jpg",
        "tags": [
          { "tag": "Monsters", "id_tags": "685146c5f3ed681c80f257e2" }
        ],
        "status": "Ongoing",
        "slug": "apocalyptic-forecast-6852084c68c513c5035d7047",
        "originalLanguage": "cn",
        "updated_at": "2026-08-21T04:42:27.189000",
        "stats_count": {
          "chapters": 50,
          "views": 3316,
          "followers": 1,
          "views_today": 0,
          "views_week": 151,
          "views_month": 809
        },
        "is18plus": false
      }
    ],
    "pagination": {
      "total": 20785,
      "page": 1,
      "limit": 1,
      "total_pages": 20785
    }
  }
}
```

## Details

### Title details

`GET /manga/mangaball/detail/:slug`

Returns full metadata and the chapter list, newest first. Chapters are grouped by number; each chapter has one entry in `translations` per release (language and scanlation group). Pass a translation's `id_chapter` to `/read`.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `slug` | Yes | `slug` from any list, for example `sousou-no-frieren-685154f0702284f834178433`, or the bare 24-character `_id` |

**Query parameters**

| Name | Required | Default | Description |
| --- | --- | --- | --- |
| `lang` | No | `en` | Chapter language code, such as `en`, `fr`, `es`, `pt-br`, `id`, `vi`, `ru`, `it`, `th` or `cn`, or `all` for every language |

**Example**

```bash
curl "http://localhost:3000/manga/mangaball/detail/sousou-no-frieren-685154f0702284f834178433"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "_id": "685154f0702284f834178433",
    "slug": "sousou-no-frieren-685154f0702284f834178433",
    "title": "Sousou no Frieren",
    "title_alter": ["Frieren: Beyond Journey's End", "Frieren at the Funeral"],
    "thumbnail": "http://localhost:3000/manga/mangaball/image/bulbasaur.poke-black-and-white.net/covers/685154f0702284f834178433/cover_1752073746362.webp",
    "description": "The adventure is over but life goes on for an elf mage just beginning to learn what living is all ab...",
    "status": "Hiatus",
    "originalLanguage": "ja",
    "year": 2020,
    "genres": [
      { "name": "Award Winning", "id_tags": "685148fe15e8b86aae68e5a7" }
    ],
    "keywords": [
      { "name": "Character Growth", "id_keywords": "6909ab7627ce41e16a3d0ac2" }
    ],
    "authors": [
      { "authors": "Yamada Kanehito", "id_authors": "68514d08d2df9377738b7e38" }
    ],
    "stars": 10,
    "likes": 4,
    "views": 152219,
    "bookmark": 103,
    "is18plus": false,
    "chapters": {
      "language": "en",
      "total_chapters": 160,
      "all_chapters": [
        {
          "number": 147,
          "title": "Chapter 147",
          "translations": [
            {
              "id_chapter": "68ef0c9444adc98aa222f26f",
              "name": "Chapter 147",
              "language": "en",
              "volume": 0,
              "date": "2025-10-15T02:53:08.029000",
              "views": 244,
              "group": {
                "id_group": "68ca28198e848904a0004d5f",
                "name": "Ursaring",
                "icon": "http://localhost:3000/manga/mangaball/image/mangaball.com/images/groups/icons/ursaring.png"
              }
            }
          ]
        }
      ]
    }
  }
}
```

* `genres` holds every tag of the title; use `id_tags` with `/tags/:id_tags`.
* `total_chapters` counts chapter numbers in the chosen language, not releases. With `lang=fr` the same title returned 142, with `lang=all` 154.
* Fields with no value are left out, as in lists; a translation without a name has no `name` key.

## Read

### Chapter pages

`GET /manga/mangaball/read/:id_chapter`

Returns the page image URLs of one chapter release in reading order.

**Path parameters**

| Name | Required | Description |
| --- | --- | --- |
| `id_chapter` | Yes | `id_chapter` from a translation in `/detail`, or `chapter_id` from `/recent`, for example `68ef0c9444adc98aa222f26f` |

**Example**

```bash
curl "http://localhost:3000/manga/mangaball/read/68ef0c9444adc98aa222f26f"
```

```json
{
  "status": 200,
  "success": true,
  "data": {
    "title_id": "685154f0702284f834178433",
    "chapter_id": "68ef0c9444adc98aa222f26f",
    "chapter_number": "147",
    "chapter_language": "en",
    "images": [
      "http://localhost:3000/manga/mangaball/image/bulbasaur.poke-black-and-white.net/storage/685154f0702284f834178433/0/147/mangabuddy/en/68ef0c9444adc98aa222f26f-001.webp",
      "http://localhost:3000/manga/mangaball/image/bulbasaur.poke-black-and-white.net/storage/685154f0702284f834178433/0/147/mangabuddy/en/68ef0c9444adc98aa222f26f-002.webp"
    ]
  }
}
```

`chapter_volume` is added when the release has a volume. An unknown id returns `404` with `Chapter not found`.

## Images

### Image proxy

`GET /manga/mangaball/image/*`

Serves MangaBall covers, pages and group icons. Every image URL in the responses above already points here, so use them as they are. The part after `/image/` is the upstream URL without `https://`; any query string is passed on.

```bash
curl -o 001.webp "http://localhost:3000/manga/mangaball/image/bulbasaur.poke-black-and-white.net/storage/685154f0702284f834178433/0/147/mangabuddy/en/68ef0c9444adc98aa222f26f-001.webp"
```

The route requests the image with `Referer: https://mangaball.com/`, follows up to 3 redirects and returns the raw bytes with the upstream `Content-Type` (`image/webp`, `image/jpeg`, `image/png`, ...) and `Cache-Control: public, max-age=604800, immutable`. Errors are plain text, not JSON:

| Status | Body | When |
| --- | --- | --- |
| `400` | `Invalid image URL` or `Invalid image host` | The path is not a valid public hostname and path |
| `403` | `Forbidden image host` | The host resolves to a private or local address |
| `404` and other upstream statuses | `Image unavailable` | The upstream returned an error |
| `502` | `Image unavailable` | The upstream could not be reached or did not return an image |

## Errors

| Status | When |
| --- | --- |
| `400` | `q` is missing on `/search`, or `time` is not `day`, `today`, `week` or `month` |
| `404` | Unknown title slug or id (`Not found on Mangaball`) or chapter id (`Chapter not found`) |
| `502` | MangaBall failed or returned an invalid response |
| `503` | MangaBall is rate limiting requests |
| `504` | MangaBall did not answer within 30 seconds |

## Notes

* Flow: any list gives `slug`, `/detail/:slug` gives `chapters.all_chapters[].translations[].id_chapter`, `/read/:id_chapter` gives `images`.
* Most routes answered in 0.5 to 1 second during testing. `/detail` took 2 to 4 seconds because it loads every page of the chapter list, and `/foryou?time=month` once took 20 seconds.
* With Redis enabled, `/detail` results are cached for 10 minutes per title and language.
