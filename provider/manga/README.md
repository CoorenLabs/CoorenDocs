---
icon: book-open
---

# Manga

## Manga

The manga module exposes multiple providers behind one route group.

Use it for manga discovery, filtering, detail pages, reading flows, and proxied images.

{% hint style="info" %}
Base route: `GET /manga/...`
{% endhint %}

### Overview

Current providers:

* `allmanga`
* `mangaball`
* `mangafire`

Each provider covers a different reading and browsing style:

* AllManga for curated discovery and generated chapter flows
* MangaBall for deeper filters, tags, authors, and translated chapter variants
* MangaFire for extensive language support, volume metadata, and high-quality covers

### Providers

#### AllManga

AllManga is a good fit when you need:

* curated home sections
* search with genre and author filters
* generated chapter lists
* a simple end-to-end reading flow

See [All Manga](all-manga.md) for the full route reference.

#### MangaBall

MangaBall is a good fit when you need:

* richer browse categories
* advanced filter search
* person and author exploration
* tag, keyword, and translation-aware chapter data

See [MangaBall](mangaball.md) for the full route reference.

#### MangaFire

MangaFire is a good fit when you need:

* multi-language chapter support
* volume metadata and covers
* curated trending and most-viewed sections
* comprehensive metadata and similar manga recommendations

See [MangaFire](mangafire.md) for the full route reference.

### Typical integration

1. Use AllManga for a simpler discovery-to-reader flow.
2. Use MangaBall when you need stronger filters and author views.
3. Render proxied image URLs from both providers in the reader.
4. Mix both providers if you want broader catalog coverage.
