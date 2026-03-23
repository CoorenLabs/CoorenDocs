---
description: Get CoorenLabs on your machine fast. Then inspect the API and route groups.
icon: bolt
---

# Quickstart

{% hint style="success" %}
You should be ready to explore the API in a few minutes.
{% endhint %}

### Before you start

This guide assumes you already know:

* Git and the terminal
* Environment variables
* HTTP, REST, and JSON
* Basic TypeScript workflows

{% hint style="warning" %}
Basic setup for Git, Bun, Node.js, editors, and general web concepts is out of scope here.
{% endhint %}

### Requirements

Make sure you have:

* Git
* [Bun](https://bun.sh/docs/installation)
* A REST client like Postman, Insomnia, or Bruno
* A TypeScript-friendly editor

### Get the code

{% stepper %}
{% step %}
#### Clone the repository

```bash
git clone https://github.com/CoorenLabs/Cooren.git
cd Cooren
```
{% endstep %}

{% step %}
#### Install dependencies

```bash
bun install
```
{% endstep %}

{% step %}
#### Start the API

Run the Bun start command defined by the repository.

Once the server is up, continue with the checks below.
{% endstep %}

{% step %}
#### Open the API docs

Visit:

```
http://localhost:<port>/docs
```

This is the fastest way to inspect available endpoints.
{% endstep %}
{% endstepper %}

### Verify the server

Check the root endpoint first:

```bash
curl http://localhost:<port>/
```

You should get a JSON response with status or metadata.

Then open:

* `/docs` for the OpenAPI UI
* `/` for the base metadata response

### Route groups

Cooren mounts providers under a few top-level prefixes:

* `/anime`
* `/manga`
* `/movie-tv`
* `/music`

If the server is running, those groups should appear in the API docs.

### How the project is organized

The codebase is split into three main areas:

* `src/core`\
  Config, logging, cache, helpers, and route mapping
* `src/providers`\
  Provider implementations grouped by category
* `src/index.ts`\
  App setup, CORS, path normalization, and route registration

### What happens on startup

When Cooren boots, it:

1. Validates configuration
2. Configures CORS and request handling
3. Registers provider routes
4. Exposes `/docs`
5. Exposes `/`

<details>

<summary>Need deeper background?</summary>

Use the official docs for core dependencies:

* [Bun](https://bun.sh/docs)
* [TypeScript](https://www.typescriptlang.org/docs)
* [Node.js](https://nodejs.org/en/docs)
* [Elysia](https://elysiajs.com/docs)
* [Elysia CORS](https://elysiajs.com/plugins/cors)
* [Elysia OpenAPI](https://elysiajs.com/plugins/openapi)
* [Axios](https://axios-http.com/docs/intro)
* [Cheerio](https://cheerio.js.org/)
* [Zod](https://zod.dev/)
* [Upstash Redis](https://upstash.com/docs/redis)

</details>

### Next step

Once the API is running, open `/docs` and test one provider route end to end.
