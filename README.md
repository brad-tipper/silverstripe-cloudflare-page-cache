# silverstripe-cloudflare-page-cache

Full-page cache for the public pages of a Silverstripe site, with Cloudflare edge caching and purge.

An anonymous GET of a page is rendered once, stored in a PSR-16 cache and served from it until content changes. Any CMS write, publish or delete (pages, site config, Elemental blocks) or a flush bumps a generation counter that makes every stored page unreachable at once, and purges the Cloudflare cache for the zone.

Only safe responses are stored: status 200, HTML/XML/text, no cookies set, no CSRF token in the body, and a request with no login cookie, no `Authorization` header and no query string (tracking parameters excepted). `admin`, `Security`, `dev`, `api` and similar prefixes are never cached. Only the site's own hostnames are cached.

The cache is on in `live` only.

## Install

```sh
composer require bradtipper/silverstripe-cloudflare-page-cache
```

## Configure

```yaml
BradTipper\AppBackend\Support\CloudflarePurge:
  zone_id: 'your-zone-id'
  api_token: 'token with Cache Purge permission'   # keep out of the repo, live-only config

BradTipper\AppBackend\Cache\PageCacheMiddleware:
  public_url: 'https://example.com'                # its host (and www.) is cached
  extra_hosts: []
  ttl: 2592000                                     # stored for 30 days
  browser_max_age: 3600
  shared_max_age: 2592000                          # Cloudflare holds pages for this long
```

To try it outside live: `BradTipper\AppBackend\Cache\AppCache: enabled: true`.

Add the cache bust to your own unversioned models that show on the public site:

```yaml
App\Model\Testimonial:
  extensions:
    - BradTipper\AppBackend\Cache\BustsSiteCache
```

Cloudflare needs a cache rule that honours the origin's `Cache-Control` for HTML. The purge sends a single "purge everything" request per PHP request, after the response is out.

If `bradtipper/silverstripe-restful-framework` is installed, `public_url` and the purge token default to its `AppConfig` values.

## Licence

BSD-3-Clause
