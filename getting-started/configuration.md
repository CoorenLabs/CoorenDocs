---
icon: sliders
---

# Configuration

## Configuration

Cooren reads its configuration from environment variables and core config files.

{% hint style="info" %}
Start with the example below. Then check `config.ts` for the full list and defaults.
{% endhint %}

### Key variables

These are the settings you will most likely change first:

* `PORT`\
  HTTP port for the API
* `NODE_ENV`\
  Runtime mode like `development` or `production`
* `SERVER_ORIGIN`\
  Public server origin used for domain masking
* `CORS_ORIGIN`\
  Allowed origins for CORS
* `CORS_CREDENTIALS`\
  Enables or disables credential support
* `LOG_LEVEL`\
  Logging verbosity
* `ENABLE_CACHE`\
  Enables Redis-backed caching
* `ENABLE_RATE_LIMITING`\
  Enables request rate limiting

### Example `.env`

Use this as your base config:

```dotenv
REPO=https://github.com/CoorenLabs/Cooren

# Server Configuration
PORT=3000
NODE_ENV=development

# Domain Masking Configuration
# Set this to your actual domain in production (e.g. api.yoursite.com)
SERVER_ORIGIN=http://localhost:3000

# Logging Configuration
LOG_LEVEL=info

# Rate Limiting (requests per minute)
RATE_LIMIT_PER_MINUTE=100

# Proxy Configuration
PROXY_TIMEOUT_MS=30000
PROXY_MAX_RETRIES=3
SHOW_PROXIED_URL=true

# CORS Configuration
CORS_ORIGIN=*
CORS_CREDENTIALS=false

# OpenAPI Configuration
OPENAPI_ENABLED=true
OPENAPI_VERSION=3.0.0

# Cache Config
ENABLE_CACHE=false # set `true` to enable redis cache
DEFAULT_CACHE_TTL="-1" # cache TTL in seconds, for storing forever set to "-1"
CACHE_PROVIDER=default # `default` or `uptash`

# default
REDIS_URL=redis://localhost:6379

# uptash
UPSTASH_REDIS_REST_URL=
UPSTASH_REDIS_REST_TOKEN=

# =============================================================================
# PRODUCTION SETTINGS (Uncomment for production)
# =============================================================================

# NODE_ENV=production
# LOG_LEVEL=warn
# CORS_ORIGIN=https://yoursite.com,https://api.yoursite.com
# CORS_CREDENTIALS=true
# RATE_LIMIT_PER_MINUTE=60

# =============================================================================
# DEVELOPMENT SETTINGS
# =============================================================================

# Enable detailed logging in development
# LOG_LEVEL=debug

# =============================================================================
# SECURITY SETTINGS
# =============================================================================

# API Rate Limiting
ENABLE_RATE_LIMITING=false

# Request timeout (milliseconds)
REQUEST_TIMEOUT=60000
```

{% hint style="warning" %}
Adjust these values for local, staging, and production separately.
{% endhint %}

### Configuration groups

#### Server

* `PORT`
* `NODE_ENV`
* `SERVER_ORIGIN`

These control runtime mode and server identity.

#### Logging

* `LOG_LEVEL`

Use `debug` in development. Use `warn` or stricter in production.

#### Rate limiting and timeouts

* `RATE_LIMIT_PER_MINUTE`
* `ENABLE_RATE_LIMITING`
* `REQUEST_TIMEOUT`
* `PROXY_TIMEOUT_MS`
* `PROXY_MAX_RETRIES`

These control request volume, retries, and timeout behavior.

#### CORS

* `CORS_ORIGIN`
* `CORS_CREDENTIALS`

Cooren uses `@elysiajs/cors` with this behavior:

* If `CORS_ORIGIN` is `*`, all origins are allowed
* Otherwise, `CORS_ORIGIN` is split by commas and used as an allowlist
* `CORS_CREDENTIALS` controls whether credentials are allowed

#### OpenAPI

* `OPENAPI_ENABLED`
* `OPENAPI_VERSION`

These control whether the API docs are exposed and which spec version is used.

#### Cache

* `ENABLE_CACHE`
* `DEFAULT_CACHE_TTL`
* `CACHE_PROVIDER`
* `REDIS_URL`
* `UPSTASH_REDIS_REST_URL`
* `UPSTASH_REDIS_REST_TOKEN`

Use `default` for a standard Redis connection.

Use Upstash variables when the cache provider is configured for Upstash.

#### Proxy behavior

* `SHOW_PROXIED_URL`

This controls whether proxied URLs are exposed in responses or logs.

### Reference docs

Use these if you need deeper details:

* [Elysia CORS docs](https://elysiajs.com/plugins/cors)
* [MDN CORS docs](https://developer.mozilla.org/docs/Web/HTTP/CORS)

### Full config reference

For the full list of supported variables and defaults, check the source in `config.ts`.
