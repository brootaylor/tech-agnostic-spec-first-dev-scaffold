# Service Worker

---

## Overview

Add a service worker to the project to provide offline support, a caching
strategy for assets and pages, and a resilient fallback experience for users
on slow or unreliable connections.

---

## Key

A reference for the prefixes used throughout this document.

| Prefix | Stands for | Description |
|--------|------------|-------------|
| `US-##` | User Story | A feature requirement from the user's perspective |
| `AC-##` | Acceptance Criteria | A condition that must be met for the story to be complete |

---

## User stories

Each user story describes a requirement from the user's perspective, followed by
the acceptance criteria that must be met for it to be considered complete.

### US-01 — Offline support

> As a user, I want to be able to access previously visited pages when I am
> offline or on a poor connection.

- [ ] **AC-01** — Previously visited pages are served from cache when the network is unavailable
- [ ] **AC-02** — A fallback offline page is served when the user is offline and the requested page has not been cached
- [ ] **AC-03** — Offline images are replaced with a branded inline SVG placeholder rather than showing a broken image
- [ ] **AC-04** — The service worker is registered in the root layout or main entry file before any other scripts
- [ ] **AC-05** — If service worker installation fails, a message is posted to open clients so the UI can surface it to the user

### US-02 — Asset caching

> As a user, I want the site to load quickly on repeat visits by serving
> cached assets where possible.

- [ ] **AC-01** — Mandatory static assets (CSS, offline page) are cached during the service worker `install` event; installation fails if any of these cannot be cached
- [ ] **AC-02** — Non-mandatory assets (pre-cached pages, supplementary images) are cached opportunistically and do not block installation if they fail
- [ ] **AC-03** — Cached assets are served immediately on repeat visits without a network round-trip
- [ ] **AC-04** — When a new version of the service worker is deployed, outdated caches are deleted during the `activate` event
- [ ] **AC-05** — Page and image caches are bounded by a maximum item count; the oldest entries are evicted when limits are reached

---

## Caching strategy

Choose one strategy and mark it `[active]` in `docs/project-brief.md`. The strategy
selected there is the single source of truth — do not redefine it here.

The table below describes what each strategy does and when to choose it.

| Strategy | When to choose it | How it works |
|----------|-------------------|--------------|
| **Cache first *(assets)* + Network first *(pages)*** | Static documentation sites, marketing sites, component libraries — content changes infrequently | Assets (CSS, JS, images, fonts) are served from cache for speed. Pages are fetched from the network first for freshness, with a timeout fallback to cache on slow connections, and a final fallback to the offline page when fully offline. Recommended default. |
| **Network first *(all)*** | Sites where content changes frequently and stale data is a problem | Every request hits the network first. Cache is only used when the network fails. Slower on repeat visits but always fresh when online. |
| **Stale while revalidate** | Content that changes occasionally and where brief staleness is acceptable | Cache is served immediately (fast), while a network request updates the cache in the background. The updated version is shown on the next visit. |
| **Custom** | Projects with complex routing, authenticated content, or mixed static/dynamic pages | Define the strategy in this file below. |

> *If Custom is selected, describe the strategy here — which routes use which approach,
> what is never cached (e.g. API responses, authenticated pages), and why.*

---

## Cache architecture

For any project using a manual `sw.js` (Eleventy, SvelteKit, or any setup not using
`vite-plugin-pwa`), the recommended architecture uses three separate named caches
rather than a single one. This keeps different content types isolated, makes size
limits and eviction cleaner, and means a full cache invalidation on deploy hits all
three at once.

| Cache | Name | Contents | Size limit |
|-------|------|----------|------------|
| Static | `static@<version>` | Mandatory assets (CSS, offline page) + opportunistically pre-cached pages and non-mandatory assets | No limit — precached at install time |
| Pages | `pages@<version>` | HTML pages cached at runtime as users navigate | 50 entries — oldest evicted first |
| Images | `images@<version>` | Images cached at runtime on first fetch | 100 entries — oldest evicted first |

All three cache names are derived from a single versioned `cacheName` constant, so
they are all invalidated together when a new service worker is deployed.

### Mandatory vs non-mandatory precaching

Not all precached assets need to block service worker installation. Splitting them
prevents a missing non-critical asset (such as a social share image) from preventing
the service worker from installing at all.

**Mandatory** — must succeed or installation fails. Return the promise so it blocks:

```js
return cache.addAll(staticMandatory);
```

**Non-mandatory** — cached opportunistically. Do not return the promise:

```js
cache.addAll([...offlinePages, ...staticNonMandatory]);
```

The offline page and the primary CSS file are always mandatory. Pre-cached pages and
supplementary images are non-mandatory.

---

## Offline fallback

### Page fallback

When a user is offline and the requested page is not in the cache, the service
worker must serve a fallback page rather than showing a browser error.

**Requirements:**

- The fallback page must live at `src/offline.html` (or the framework equivalent — see Setup guidance below)
- It must contain no external dependencies — no CDN scripts, no externally hosted fonts, no images referenced by a third-party URL. Local assets that are included in the precache manifest are fine, as they will already be in the cache when this page is served
- It must communicate clearly that the user is offline and suggest they check their connection
- It must be in the mandatory precache list so it is always available

**Minimal example:**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>You're offline</title>
    <style>
      body {
        font-family: sans-serif;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        min-height: 100vh;
        margin: 0;
        padding: 1rem;
        text-align: center;
      }
    </style>
  </head>
  <body>
    <h1>You're offline</h1>
    <p>This page isn't available without a network connection. Check your connection and try again.</p>
  </body>
</html>
```

> The fallback page is intentionally minimal — it should not import tokens or global
> styles because those assets may not be cached yet on a first visit that goes offline.

### Image fallback

When an image cannot be fetched and is not in cache, return an inline SVG placeholder
rather than a broken image. This prevents layout breakage and communicates the offline
state clearly without requiring any additional cached assets.

```js
if (request.url.match(/\.(jpe?g|webp|png|gif|svg)/)) {
  return new Response(
    '<svg role="img" aria-labelledby="offline-title" viewBox="0 0 400 300" ' +
    'xmlns="http://www.w3.org/2000/svg">' +
    '<title id="offline-title">Offline</title>' +
    '<g fill="none" fill-rule="evenodd">' +
    '<path fill="#d1d5db" d="M0 0h400v300H0z"/>' +
    '<text fill="#6b7280" font-family="sans-serif" font-size="48" font-weight="bold">' +
    '<tspan x="120" y="165">Offline</tspan>' +
    '</text></g></svg>',
    { headers: { 'Content-Type': 'image/svg+xml', 'Cache-Control': 'no-store' } }
  );
}
```

Adjust the `fill` values to match your project's colour palette. Keep the
`Cache-Control: no-store` header so the placeholder is never stored in cache and
a real image can replace it as soon as the user is back online.

---

## Cache versioning

### Using `vite-plugin-pwa` (Vanilla + Vite, React + Vite)

The plugin handles versioning automatically using asset content hashes. No manual
cache naming is required. The generated service worker includes a precache manifest
that is updated on every build.

### Using a manual `sw.js` (Eleventy, SvelteKit, or any setup not using the plugin)

Construct the cache name from three values: the site name, the package version, and
a deploy timestamp. This gives you automatic, human-readable versioning without
manual incrementing, and ensures cache busting on every deploy even if the version
number has not changed.

```js
const siteName    = 'my-project';      // replace with your site or package name
const siteVersion = 'v1.0.0';          // from package.json
const deployTime  = '1717000000000';   // Date.now() at build time

const cacheName = siteName + '_' + siteVersion + '.' + deployTime;

const staticCacheName = 'static@' + cacheName;
const pagesCacheName  = 'pages@'  + cacheName;
const imagesCacheName = 'images@' + cacheName;
const cacheList = [staticCacheName, pagesCacheName, imagesCacheName];
```

For **Eleventy**, the version and timestamp are injected at build time via Eleventy's
global data — see the Eleventy setup section for the full pattern.

For **SvelteKit**, `version` is injected automatically from `$service-worker` and
changes on every build, so the timestamp is not needed.

> Agents must not hardcode a cache name without a versioning mechanism. Do not use
> a manually incremented suffix (`v1`, `v2`) — it requires a human to update it on
> every deploy and will be forgotten.

---

## Helper functions

The following helpers apply to any project using a manual `sw.js`. Define them once
near the top of the service worker file, before the event listeners. For SvelteKit,
paste them after the imports. For Eleventy, paste them into the `.njk` template.

### `updateStaticCache()`

Opens the static cache and populates it with both mandatory and non-mandatory assets.
Non-mandatory assets are cached fire-and-forget so a failed fetch does not block
installation.

```js
function updateStaticCache() {
  return caches.open(staticCacheName).then(cache => {
    // Non-mandatory — cached opportunistically, does not block installation
    cache.addAll([...offlinePages, ...staticNonMandatory]);
    // Mandatory — must succeed for installation to complete
    return cache.addAll(staticMandatory);
  });
}
```

### `cacheClients()`

Caches the URLs of any pages the user already has open at install time, so those
pages are available offline immediately after the service worker activates without
waiting for the user to navigate to them again.

```js
function cacheClients() {
  const pages = [];
  return clients.matchAll({ includeUncontrolled: true })
    .then(allClients => {
      for (const client of allClients) pages.push(client.url);
    })
    .then(() => {
      return caches.open(pagesCacheName).then(cache => cache.addAll(pages));
    });
}
```

### `clearOldCaches()`

Deletes any caches whose name is not in `cacheList`. Called during the `activate`
event to remove stale caches from previous deployments.

```js
function clearOldCaches() {
  return caches.keys().then(keys =>
    Promise.all(
      keys.filter(key => !cacheList.includes(key)).map(key => caches.delete(key))
    )
  );
}
```

### `trimCache(cacheName, maxItems)`

Recursively deletes the oldest entry from a cache until it is within `maxItems`.
Called after every runtime cache write to keep the pages and images caches bounded.

```js
function trimCache(cacheName, maxItems) {
  caches.open(cacheName).then(cache => {
    cache.keys().then(keys => {
      if (keys.length <= maxItems) return;
      cache.delete(keys[0]).then(() => trimCache(cacheName, maxItems));
    });
  });
}
```

### `addToCache(cacheName, request, response)`

Writes a request/response pair to the named cache and immediately trims the cache
to its maximum size. Use this for all runtime cache writes in the fetch handler.

```js
function addToCache(cacheName, request, response) {
  caches.open(cacheName).then(cache => {
    cache.put(request, response);
    trimCache(cacheName, cacheName === imagesCacheName ? maxImages : maxPages);
  });
}
```

---

## Event handlers

The following event handlers apply to any project using a manual `sw.js`. They
reference the cache names, limits, helper functions, and offline fallbacks defined
in the sections above. Paste them into the service worker file after the helper
functions.

### Install

Populates the static cache, caches any open pages, then calls `self.skipWaiting()`
so the new service worker activates immediately without waiting for existing tabs to
close. On failure, posts a message to open clients so page code can inform the user.

```js
let isInstallFailureMessageShown = false;

self.addEventListener('install', event => {
  event.waitUntil(
    updateStaticCache()
      .then(() => cacheClients())
      .then(() => self.skipWaiting())
      .catch(error => {
        console.error('Service Worker installation failed:', error);
        if (!isInstallFailureMessageShown) {
          self.clients.matchAll().then(clients => {
            clients.forEach(client => client.postMessage({
              type: 'INSTALL_FAILED',
              message: 'Offline functionality could not be set up. Some features may not work offline.'
            }));
          });
          isInstallFailureMessageShown = true;
        }
        throw error; // Re-throw to prevent installation completing
      })
  );
});
```

To handle this message in page code:

```js
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.addEventListener('message', event => {
    if (event.data?.type === 'INSTALL_FAILED') {
      console.warn(event.data.message);
      // Optionally surface a notice to the user here
    }
  });
}
```

### Activate

Clears old caches, enables navigation preload if the browser supports it, then
claims all open clients so the new service worker takes effect immediately.

```js
self.addEventListener('activate', event => {
  event.waitUntil(
    clearOldCaches()
      .then(() => {
        if (self.registration.navigationPreload) {
          return self.registration.navigationPreload.enable();
        }
      })
      .then(() => self.clients.claim())
  );
});
```

> **Navigation preload** allows the browser to start a network request for a
> navigation in parallel with service worker startup rather than waiting for the
> service worker to boot first. Enable it here — it has no effect in browsers that
> do not support it.

### Fetch

Ignores non-GET requests. For HTML, uses a network-first strategy with a timeout
fallback to cache. For all other requests, uses a cache-first strategy. Caches
images at runtime and returns an inline SVG placeholder if an image cannot be
fetched while offline.

```js
const timeout = 3000; // ms before falling back to cache for slow HTML requests

self.addEventListener('fetch', event => {
  const request = event.request;

  if (request.method !== 'GET') return;

  // HTML — network first, timeout fallback to cache, then offline page
  if (request.headers.get('Accept').includes('text/html')) {
    event.respondWith(new Promise(resolve => {
      const cached = caches.match(request);

      const timer = setTimeout(() => {
        // Network too slow — serve from cache if available
        cached.then(response => { if (response) resolve(response); });
      }, timeout);

      fetch(request)
        .then(response => {
          clearTimeout(timer);
          try { addToCache(pagesCacheName, request, response.clone()); } catch (e) { console.error(e); }
          resolve(response);
        })
        .catch(error => {
          clearTimeout(timer);
          console.error(error);
          // Offline — serve cache, fall back to offline page
          caches.match(request)
            .then(response => response || caches.match(offlinePage))
            .then(resolve);
        });
    }));
    return;
  }

  // All other requests — cache first, then network
  event.respondWith(
    caches.match(request).then(cached => {
      return cached || fetch(request)
        .then(response => {
          if (request.url.match(/\.(jpe?g|webp|png|gif|svg)/)) {
            try { addToCache(imagesCacheName, request, response.clone()); } catch (e) { console.error(e); }
          }
          return response;
        })
        .catch(error => {
          console.error(error);
          // Offline image fallback
          if (request.url.match(/\.(jpe?g|webp|png|gif|svg)/)) {
            return new Response(
              '<svg role="img" aria-labelledby="offline-title" viewBox="0 0 400 300" ' +
              'xmlns="http://www.w3.org/2000/svg">' +
              '<title id="offline-title">Offline</title>' +
              '<g fill="none" fill-rule="evenodd">' +
              '<path fill="#d1d5db" d="M0 0h400v300H0z"/>' +
              '<text fill="#6b7280" font-family="sans-serif" font-size="48" font-weight="bold">' +
              '<tspan x="120" y="165">Offline</tspan>' +
              '</text></g></svg>',
              { headers: { 'Content-Type': 'image/svg+xml', 'Cache-Control': 'no-store' } }
            );
          }
        });
    })
  );
});
```

> **Network timeout for HTML:** if the network has not responded within `timeout`
> milliseconds, the service worker serves the cached version of the requested page
> rather than continuing to wait. The network request is still in flight — if it
> completes, the response is written to the pages cache for the next visit. This
> gives fast fallback on slow connections without abandoning freshness on fast ones.

---

## Out of scope

Explicitly listing what this feature does NOT cover prevents scope creep and
helps the agent avoid building more than is required.

- Push notifications
- Background sync
- Full PWA (installable app, manifest, splash screens)
- Server-side caching
- Caching of authenticated or user-specific content
- Caching of API responses unless explicitly specified

---

## Setup guidance

The following covers the most common framework and tooling combinations. For Astro
and Eleventy with a plugin, refer to the relevant documentation links — do not adapt
the Vite examples without checking compatibility first.

---

### Vanilla + Vite

Use `vite-plugin-pwa`. It generates a service worker from your Vite config and
integrates with the existing `vite build` step — no additional npm script is needed.

**Install:**

```bash
npm install --save-dev vite-plugin-pwa
```

**`vite.config.js`:**

```js
import { defineConfig } from 'vite';
import { VitePWA } from 'vite-plugin-pwa';

export default defineConfig({
  plugins: [
    VitePWA({
      registerType: 'autoUpdate',
      includeAssets: ['**/*.{js,css,html,svg,png,ico,woff2}'],
      workbox: {
        navigationFallback: '/offline.html',
        runtimeCaching: [
          {
            urlPattern: ({ request }) => request.mode === 'navigate',
            handler: 'NetworkFirst',
            options: {
              cacheName: 'pages-cache',
              networkTimeoutSeconds: 3,
              expiration: { maxEntries: 50 },
            },
          },
          {
            urlPattern: /\.(jpe?g|webp|png|gif|svg)$/i,
            handler: 'CacheFirst',
            options: {
              cacheName: 'images-cache',
              expiration: { maxEntries: 100 },
            },
          },
        ],
      },
    }),
  ],
});
```

The `networkTimeoutSeconds` and `expiration.maxEntries` values correspond to the
timeout and cache size limits described in this file. Workbox handles precaching,
versioning, and cache invalidation automatically — no manual cache naming is needed.

**Register in `src/scripts/main.js`:**

```js
import { registerSW } from 'virtual:pwa-register';

registerSW({ immediate: true });
```

**Offline fallback:** place `offline.html` in the project root `public/` folder
so Vite copies it to the build output automatically.

---

### React + Vite

Setup is identical to Vanilla + Vite above. Register the service worker in
`src/scripts/main.jsx` (or `main.tsx`) before the app is mounted:

```jsx
import { registerSW } from 'virtual:pwa-register';
import { createRoot } from 'react-dom/client';
import App from './App';

registerSW({ immediate: true });

createRoot(document.getElementById('root')).render(<App />);
```

---

### Svelte + SvelteKit

SvelteKit has native service worker support. No plugin is required.

**Create `src/service-worker.js`:**

SvelteKit automatically registers any file at `src/service-worker.js`. SvelteKit
injects `build`, `files`, and `version` automatically — `version` changes on every
build and is used as the cache version string, so no deploy timestamp is needed.

```js
import { build, files, version } from '$service-worker';

// Cache name configuration
const siteName = 'my-project'; // replace with your site name

const staticCacheName = 'static@' + siteName + '_' + version;
const pagesCacheName  = 'pages@'  + siteName + '_' + version;
const imagesCacheName = 'images@' + siteName + '_' + version;
const cacheList = [staticCacheName, pagesCacheName, imagesCacheName];

// Cache limits and timeout
const maxPages  = 50;
const maxImages = 100;
const timeout   = 3000;

// Assets
const offlinePage        = '/offline';
const staticMandatory    = [...build, ...files, offlinePage];
const staticNonMandatory = []; // add paths to any non-critical supplementary assets
const offlinePages       = []; // SvelteKit: open pages are handled by cacheClients()
```

After this configuration block, paste in:

1. All five helper functions from the Helper functions section above
2. All three event handlers from the Event handlers section above

> Do not import `build`, `files`, or `version` from anywhere other than `$service-worker`.

**Offline fallback:** place `offline.html` in the `static/` folder so SvelteKit
includes it in the build output.

---

### Eleventy

Eleventy has no built-in service worker support. The recommended approach is to
generate the service worker from a Nunjucks template so the pre-cached page list
is built dynamically from your site's collections and navigation data at build time,
rather than maintained by hand.

#### Template approach

**Create `src/sw.njk`:**

The template below outputs to `/sw.js` at the path defined in its frontmatter. The
cache name is constructed from `pkg.name`, `pkg.version`, and `site.timeCurrent` — a
deploy timestamp that changes on every build and busts the cache automatically.

```njk
---
permalink: '/sw.js'
eleventyExcludeFromCollections: true
---

// Cache name configuration
const siteName    = '{{ pkg.name }}';
const siteVersion = 'v{{ pkg.version }}';
const deployTime  = '{{ site.timeCurrent }}';
const cacheName   = siteName + '_' + siteVersion + '.' + deployTime;

const staticCacheName = 'static@' + cacheName;
const pagesCacheName  = 'pages@'  + cacheName;
const imagesCacheName = 'images@' + cacheName;
const cacheList = [staticCacheName, pagesCacheName, imagesCacheName];

// Cache limits and timeout
const maxPages  = 50;
const maxImages = 100;
const timeout   = 3000;

// Offline page URL
const offlinePage = '/offline';

// Pages to pre-cache — built at compile time from site collections and navigation.
// The last 5 posts, all main and footer nav pages, and all pages tagged `page`
// are included. External links and documents are excluded.
const offlinePages = [
  {%- set recentPosts = collections.post | reverse %}
  {%- for item in recentPosts.slice(0, 5) %}
  '{{ item.url | pretty }}',
  {%- endfor %}
  {%- for item in navigation.mainnav %}
  {%- if (item.document !== true) and (item.external !== true) %}
  '{{ item.url }}',
  {%- endif %}
  {%- endfor %}
  {%- for item in navigation.footernav %}
  {%- if (item.document !== true) and (item.external !== true) %}
  '{{ item.url }}',
  {%- endif %}
  {%- endfor %}
  {%- for item in collections.page | reverse %}
  '{{ item.url | pretty }}',
  {%- endfor %}
  '{{ site.start_url }}'
];

// Non-mandatory assets — cached opportunistically, do not block installation
const staticNonMandatory = [
  '/assets/images/common/logo-social.png'
];

// Mandatory assets — must cache or installation fails
const staticMandatory = [
  '/styles/main.css?version={{ pkg.version }}.{{ site.timeCurrent }}',
  offlinePage
];

// Paste all five helper functions from the Helper functions section here
// Paste all three event handlers from the Event handlers section here
```

Adjust `offlinePages`, `staticNonMandatory`, and `staticMandatory` to match your
project's actual file paths and naming conventions.

#### Global data requirements

The template relies on the following being available in Eleventy's data cascade:

| Variable | Source | Description |
|----------|--------|-------------|
| `pkg.name` | `package.json` via a data file | Used in the cache key |
| `pkg.version` | `package.json` via a data file | Used in the cache key |
| `site.timeCurrent` | A global data file | `Date.now()` at build time — ensures cache busting on every deploy |
| `site.start_url` | A global data file | The root URL (`/`) |
| `navigation.mainnav` | A data file | Array of main navigation items |
| `navigation.footernav` | A data file | Array of footer navigation items |
| `collections.post` | Eleventy collections | Pages tagged `post` |
| `collections.page` | Eleventy collections | Pages tagged `page` |

A minimal `src/_data/pkg.js` to expose `package.json` values:

```js
// src/_data/pkg.js
const pkg = require('../../package.json');
module.exports = pkg;
```

A minimal `src/_data/site.js` to expose the deploy timestamp:

```js
// src/_data/site.js
module.exports = {
  timeCurrent: Date.now(),
  start_url: '/'
};
```

Navigation data depends on your site's structure. A minimal example:

```js
// src/_data/navigation.js
module.exports = {
  mainnav: [
    { url: '/about',             label: 'About' },
    { url: '/writing',           label: 'Writing' },
    { url: 'https://github.com', label: 'GitHub', external: true }
  ],
  footernav: [
    { url: '/privacy',      label: 'Privacy' },
    { url: '/resume.pdf',   label: 'Résumé', document: true }
  ]
};
```

Navigation items with `external: true` or `document: true` are excluded from the
offline pages list by the template conditionals.

**Register in your base layout** (e.g. `src/_includes/base.njk`):

```html
<script>
  if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('/sw.js');
  }
</script>
```

**Eleventy config** — pass through the offline page to the build output. The `sw.njk`
template is processed by Eleventy and output to `/sw.js` via its frontmatter
`permalink` — no passthrough copy is needed for it:

```js
// .eleventy.js
eleventyConfig.addPassthroughCopy('src/offline.html');
```

> **Cache versioning** — the cache name is versioned automatically using `pkg.version`
> and `site.timeCurrent`. The timestamp changes on every build, so the cache is busted
> on every deploy without any manual step.

---

### Astro

Use `@vite-pwa/astro` (the official Astro PWA integration). Do not adapt the Vite
examples above — the Astro integration wraps the plugin differently.

```bash
npm install --save-dev @vite-pwa/astro
```

Refer to the [Astro PWA integration documentation](https://vite-pwa-org.netlify.app/frameworks/astro)
for full configuration. The offline fallback, cache size limits, and network timeout
guidance in this file still applies — configure them via the Workbox `runtimeCaching`
options as shown in the Vanilla + Vite section above. Only the plugin wiring differs.

---

## Notes for AI agents

- Read the active caching strategy selection in `docs/project-brief.md` before implementing — do not default to a strategy if none is marked active
- For any manual `sw.js`, implement the three-bucket cache architecture (static, pages, images) and all five helper functions from the Helper functions section — do not use a single cache or omit the helpers
- The cache name must include a deploy timestamp or build-time version — do not use a manually incremented suffix (`v1`, `v2`) that requires a human to update it on every deploy
- The default `timeout` is `3000` ms and the default size limits are `maxPages: 50` and `maxImages: 100` — do not change these without an explicit instruction
- The service worker file must be served from the root of the build output so it controls the full site scope — verify the output path is `/sw.js`, not `/assets/sw.js` or similar
- Register the service worker in the root layout or main entry file, wrapped in a `'serviceWorker' in navigator` guard; also add the `message` event listener to handle `INSTALL_FAILED` messages
- Always create `offline.html` and always include the inline SVG image fallback in the fetch handler — both are required, not optional
- Do not cache API responses, authenticated routes, or user-specific content unless explicitly specified in this file
- For **Eleventy**: generate the service worker from `src/sw.njk` using the template approach documented above — do not create a static `sw.js` with a hardcoded page list. Ensure `src/_data/pkg.js` and `src/_data/site.js` exist and provide `pkg.name`, `pkg.version`, and `site.timeCurrent` before writing the template
- For **`vite-plugin-pwa`** setups: configure `runtimeCaching` with separate named caches for pages and images using `expiration.maxEntries` and `networkTimeoutSeconds` as shown in the Vanilla + Vite section — do not leave these unconfigured
- Add the service worker build output to `.gitignore` if the plugin generates it at a known path (e.g. `public/sw.js` for some plugin configurations) — confirm with the plugin's documentation. Do not add `src/sw.js` or `src/sw.njk` to `.gitignore`
