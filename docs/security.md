# Security

> **This is a configuration reference, not a spec.** It uses user stories and
> acceptance criteria to pin down what the configuration has to achieve, but it
> carries no `**Status:**` line and is never promoted to `Ready`. What turns it
> on is its `[active]` mark in `docs/project-brief.md` → Stack.

---

## Overview

Add HTTP security headers and a Content Security Policy (CSP) to the project to
protect users against common web vulnerabilities — clickjacking, Multipurpose
Internet Mail Extensions (MIME) sniffing, cross-site scripting, and unwanted
data leakage.

This doc covers the headers to set, what each one does, and how to apply them
across the supported frameworks and deployment targets. It is the single source
of truth for all security configuration in the project.

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

### US-01 — Protection against common attacks

> As a user, I want the application to protect me against common browser-based
> attacks so that my data and experience are not compromised.

- [ ] **AC-01** — HTTP security headers are present on all responses in production
- [ ] **AC-02** — No sensitive information is leaked via the `Referer` header to third-party origins
- [ ] **AC-03** — The browser cannot be instructed to render the page inside a frame from an untrusted origin

### US-02 — Content Security Policy

> As a developer, I want a Content Security Policy defined and enforced so that
> only trusted sources of scripts, styles, and other resources are permitted.

- [ ] **AC-04** — A CSP header (or meta tag equivalent) is present on all pages
- [ ] **AC-05** — The policy restricts script and style sources to `'self'` by default
- [ ] **AC-06** — Any exception to the default policy (e.g. an external font or analytics script) is explicitly listed in this file and documented with a reason
- [ ] **AC-07** — No `unsafe-eval` is present in the production policy

### US-03 — Consistent enforcement

> As a developer, I want security configuration to be defined in one place so
> that it is applied consistently and does not drift between environments.

- [ ] **AC-08** — All header values are defined in this file and not duplicated elsewhere
- [ ] **AC-09** — If a component spec requires an external resource, it declares it in its Dependencies table — this file is then updated before the component is marked `Complete`

---

## Security headers

The following headers must be set on all production responses. Each entry
describes what the header does, the recommended value for this project, and
any notes on adjusting it.

### `Content-Security-Policy`

Controls which resources the browser is permitted to load. This is the most
important security header and the one most likely to need project-specific tuning.

The baseline policy below is suitable for a project with no external dependencies.
Extend it in the CSP section below if the project loads fonts, scripts, or other
resources from external origins.

```
Content-Security-Policy:
  default-src 'self';
  script-src  'self';
  style-src   'self';
  img-src     'self' data:;
  font-src    'self';
  connect-src 'self';
  frame-ancestors 'none';
  base-uri    'self';
  form-action 'self'
```

> `frame-ancestors 'none'` supersedes `X-Frame-Options` in browsers that support
> CSP Level 2. Both are included in this scaffold's recommended config for
> compatibility with older browsers.

### `X-Frame-Options`

Prevents the page from being embedded in a `<frame>`, `<iframe>`, or `<object>`
on another origin. Protects against clickjacking.

```
X-Frame-Options: DENY
```

### `X-Content-Type-Options`

Prevents the browser from MIME-sniffing a response away from the declared
`Content-Type`. Stops certain content-injection attacks.

```
X-Content-Type-Options: nosniff
```

### `Referrer-Policy`

Controls how much of the current URL is included in the `Referer` header when
navigating to another page. `strict-origin-when-cross-origin` sends the full
path for same-origin requests but only the origin for cross-origin ones.

```
Referrer-Policy: strict-origin-when-cross-origin
```

### `Permissions-Policy`

Restricts access to browser APIs and features. The default below disables
features that most projects do not need. Remove a line if a feature is required.

```
Permissions-Policy:
  camera=(),
  microphone=(),
  geolocation=(),
  payment=()
```

---

## Content Security Policy

### Extending the baseline

The baseline policy defined in the Security headers section is intentionally
restrictive — it only allows resources served from your own domain. If the project
loads anything from an external source (a font service, an analytics script, a
video embed), the policy must be updated to permit it or the browser will block it.

**How to add an exception — a worked example:**

Say the project loads a font from Google Fonts. Without an exception, the browser
blocks the request. To permit it:

1. Identify which directives need updating. Google Fonts requires two:
   - `style-src` — for the CSS it serves
   - `font-src` — for the font files themselves
2. Add only the origins that are actually needed:
   ```
   style-src 'self' https://fonts.googleapis.com;
   font-src  'self' https://fonts.gstatic.com;
   ```
3. Record the change in the Exceptions table below.
4. Update your deployment config (the `_headers`, `vercel.json`, or middleware
   file for your framework — see Setup guidance) to use the updated policy.

Do not update `default-src` to add an exception — `default-src` is the fallback
for every directive not explicitly set, so widening it grants more permission than
intended. Always update the specific directive instead.

### Exceptions

A log of every deviation from the baseline policy. Update this table before
updating the policy in your deployment config. Replace the example row with
real entries as exceptions are added.

| Directive | Value added | Reason | Added by |
|-----------|-------------|--------|----------|
| `style-src`, `font-src` | `fonts.googleapis.com`, `fonts.gstatic.com` | Google Fonts | Example — replace |

### Inline scripts and styles

Avoid inline `<script>` and `<style>` blocks wherever possible. They require
`'unsafe-inline'` in the policy, which significantly weakens it.

If an inline script is unavoidable (e.g. a theme initialisation snippet that
must run before first paint to avoid a flash), use a `nonce` approach:

- Generate a unique `nonce` value per response on the server
- Add it to the `script-src` directive: `script-src 'self' 'nonce-<value>'`
- Add the matching `nonce="<value>"` attribute to the inline `<script>` tag

For static sites with no server component, a `sha256` hash of the script
content is the alternative — generate the hash at build time and add it to
`script-src`. See the Setup guidance section for per-framework notes.

> Never use `'unsafe-inline'` in production unless there is no viable alternative
> and it has been explicitly approved. Document any approved use in the Exceptions
> table above with the reason.

---

## Secure coding

HTTP headers and CSP policies control what the browser permits at the network
level. Secure coding conventions control what the codebase does in the first
place. Both are needed — a strong CSP cannot compensate for `eval()` or
unsanitised `innerHTML` in application code.

The secure coding rules for this project live in the Coding conventions section
of `docs/project-brief.md` under the "Secure coding" heading. They are the
source of truth — do not redefine them here.

### Linting

When the ESLint option is active in `docs/project-brief.md`, the following
plugins must be installed and configured. They enforce the secure coding rules
automatically on every file save and in continuous integration.

| Plugin | Catches |
|--------|---------|
| `eslint-plugin-security` | Dangerous patterns: `eval`, unsafe regex, `innerHTML` misuse, non-literal `require` |
| `eslint-plugin-no-secrets` | Accidentally committed credentials and API keys |

**Install:**

```bash
npm install --save-dev eslint eslint-plugin-security eslint-plugin-no-secrets
```

**`eslint.config.mjs` — minimum config:**

```js
import pluginSecurity from 'eslint-plugin-security';
import noSecrets from 'eslint-plugin-no-secrets';

export default [
  pluginSecurity.configs.recommended,
  {
    files: ['**/*.js'],
    plugins: { 'no-secrets': noSecrets },
    rules: {
      'no-secrets/no-secrets': 'error',
      'no-eval': 'error',
    },
  },
];
```

> [!IMPORTANT]
> **This must be flat config, and the file must not be named `.eslintrc`.**
> *[ESLint v10 removed eslintrc support outright](https://eslint.org/docs/latest/use/migrate-to-10.0.0) —
> "the old configuration format is no longer supported", and `ESLINT_USE_FLAT_CONFIG`
> is no longer honoured. `npm install --save-dev eslint` installs v10, so an
> `.eslintrc` file is simply not read: ESLint reports no config rather than
> failing loudly, and the security rules never run. The eslintrc config name also
> changed — `plugin:security/recommended` is now
> `plugin:security/recommended-legacy` — so an old config that **is** read still
> errors on the extend. The `.mjs` extension avoids depending on whether
> `package.json` sets `"type": "module"`. ESLint v10 requires Node.js v20.19+;
> pin Node in `.nvmrc` accordingly.*

> Add framework-specific configs (e.g. `eslint-plugin-svelte`,
> `eslint-plugin-react`) as further array entries after the security config. The
> security rules apply regardless of framework and must always be present.

Add a lint script to `package.json`:

```json
"scripts": {
  "lint": "eslint src"
}
```

---

## Out of scope

Explicitly listing what this configuration does NOT cover prevents scope creep
and helps the agent avoid building more than is required.

- Authentication and session management
- Server-side input validation and sanitisation
- CORS configuration (defined separately where a backend exists)
- Dependency vulnerability scanning (use `npm audit` or a dedicated tool)
- `Strict-Transport-Security` — automatic on Netlify and Vercel, but **not on Cloudflare, and not documented either way on Render**; see the note under Notes for "Ai" agents before deciding whether to set it
- Rate limiting and DDoS protection — handled at the infrastructure layer

---

## Setup guidance

Security headers can be set in two ways depending on the framework and deployment
target:

- **Deployment platform config** — a headers file read by the host at the edge.
  Works for any framework producing static output. No code change required.
- **Framework middleware / server hooks** — headers are set programmatically
  in server-side code. Required for frameworks with a server component (e.g.
  SvelteKit in server-side rendering (SSR) mode, Astro SSR, Next.js in any
  mode other than static export).

For purely static projects the deployment platform approach is preferred — it
keeps security config out of application code and applies to all responses
including build artefacts.

---

### Netlify (all frameworks — static output)

Create a `_headers` file in the directory that Netlify serves as the site root.
For most frameworks this is the build output directory; check your framework's
documentation for the exact path.

Place this file in the project root under `public/` (for frameworks that copy
a `public/` directory to the output) or directly in the output directory root.

```
/*
  Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self'; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=()
```

> The `_headers` file uses a specific syntax — one header per line, indented with
> two spaces under the path pattern. Refer to the
> [Netlify headers documentation](https://docs.netlify.com/routing/headers/) for
> the full syntax and how to set headers for specific paths only.

---

### Vercel (all frameworks — static output)

Add a `headers` block to `vercel.json` at the project root. Create the file if
it does not already exist.

```json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "Content-Security-Policy",
          "value": "default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self'; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'"
        },
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
        { "key": "Permissions-Policy", "value": "camera=(), microphone=(), geolocation=(), payment=()" }
      ]
    }
  ]
}
```

---

### Cloudflare Pages (all frameworks — static output)

Cloudflare Pages supports the same `_headers` file syntax as Netlify. Place it
in the build output root. The content is identical to the Netlify example above.

---

### Render (all frameworks — static output)

A Render static site takes headers from a `headers` list on the service. It can
be set in the dashboard, but the version worth committing lives in `render.yaml`
at the project root — the same file `/release` writes. Each entry is a `path`,
a `name` and a `value`; `/*` matches every request path.

```yaml
services:
  - name: my-site
    type: web
    runtime: static
    buildCommand: npm run build
    staticPublishPath: ./dist
    headers:
      - path: /*
        name: Content-Security-Policy
        value: "default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self'; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'"
      - path: /*
        name: X-Frame-Options
        value: DENY
      - path: /*
        name: X-Content-Type-Options
        value: nosniff
      - path: /*
        name: Referrer-Policy
        value: strict-origin-when-cross-origin
      - path: /*
        name: Permissions-Policy
        value: camera=(), microphone=(), geolocation=(), payment=()
```

> `staticPublishPath` must match the build output directory your framework
> writes to. And note that
> [Render preserves dashboard header rules the blueprint does not mention](https://render.com/docs/blueprint-spec)
> — a rule added by hand earlier survives a deploy and can shadow what you think
> you are setting, so check the dashboard list against `render.yaml` after the
> first deploy.
>
> This covers Render **static sites**. A Render web service running a Node
> process has no `headers` list; it sets headers in application code, so use the
> framework section below that matches your stack.

---

### SvelteKit

For SvelteKit projects, headers can be set in a server hook so they apply
regardless of deployment target.

**Create or update `src/hooks.server.js`:**

```js
export function handle({ event, resolve }) {
  return resolve(event, {
    transformPageChunk({ html }) {
      return html;
    },
  }).then((response) => {
    response.headers.set(
      'Content-Security-Policy',
      "default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self'; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'"
    );
    response.headers.set('X-Frame-Options', 'DENY');
    response.headers.set('X-Content-Type-Options', 'nosniff');
    response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
    response.headers.set(
      'Permissions-Policy',
      'camera=(), microphone=(), geolocation=(), payment=()'
    );
    return response;
  });
}
```

> If deploying SvelteKit as a static site via `@sveltejs/adapter-static`, the
> deployment platform approach (Netlify, Vercel, Cloudflare Pages, or Render
> above) is simpler and equally effective. The server hook approach is most useful when
> deploying with a Node or edge adapter.

---

### Astro

For Astro projects in static output mode (`output: 'static'`), use the deployment
platform approach above.

For Astro projects in SSR mode (`output: 'server'`), set headers in middleware:

> `output: 'hybrid'` was [removed in Astro 5](https://docs.astro.build/en/guides/upgrade-to/v5/)
> and merged into `'static'`, which now covers the old hybrid behaviour: routes
> opt out of prerendering individually with `export const prerender = false`. If a
> static-output project has any such route, it is server-rendered and needs this
> middleware too.

**Create `src/middleware.js`:**

```js
import { defineMiddleware } from 'astro:middleware';

export const onRequest = defineMiddleware(({ locals }, next) => {
  return next().then((response) => {
    response.headers.set(
      'Content-Security-Policy',
      "default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self'; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'"
    );
    response.headers.set('X-Frame-Options', 'DENY');
    response.headers.set('X-Content-Type-Options', 'nosniff');
    response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
    response.headers.set(
      'Permissions-Policy',
      'camera=(), microphone=(), geolocation=(), payment=()'
    );
    return response;
  });
});
```

---

### Next.js

Next.js sets headers with a `headers()` function in `next.config.js`. It applies
on every deployment target — Vercel, Render, or a self-hosted Node server — which
makes it the portable choice, and the right one for a Next.js project even when
the host could set the same headers at the edge.

**Create or update `next.config.js`:**

```js
const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value:
      "default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self'; connect-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'",
  },
  { key: 'X-Frame-Options', value: 'DENY' },
  { key: 'X-Content-Type-Options', value: 'nosniff' },
  { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
  {
    key: 'Permissions-Policy',
    value: 'camera=(), microphone=(), geolocation=(), payment=()',
  },
];

module.exports = {
  headers() {
    return [{ source: '/:path*', headers: securityHeaders }];
  },
};
```

> `source: '/:path*'` is what matches every route. Reach for `'/(.*)'` instead
> and a project with `i18n` configured will silently lose coverage: Next.js
> rewrites that pattern to `/(en|fr|de)/(.*)`, which then misses the top-level
> `/` route.
>
> **Static export is the exception.** Next.js lists
> [Headers among the features `output: 'export'` does not support](https://nextjs.org/docs/app/guides/static-exports)
> — the export is plain files, with no server left to run `headers()`. An
> exported Next.js app takes its headers from the deployment platform section
> above instead.

---

### Eleventy

Eleventy produces static output only. Use the deployment platform approach
(Netlify, Vercel, Cloudflare Pages, or Render above).

If you are running a local Express or similar server in front of your Eleventy
output (e.g. for testing), set headers in that server's middleware instead —
do not add a `_headers` file and expect it to work with a custom server.

---

## Notes for "Ai" agents

- Read the active security selection in `docs/project-brief.md` before implementing — do not add any security config if the selection is `None`
- The header values defined in the Security headers section of this file are the single source of truth — do not invent or modify values without updating this file first
- When adding security headers, apply them to all responses (`/*` or equivalent) unless a specific path restriction is documented in the Exceptions table
- Never add `'unsafe-eval'` or `'unsafe-inline'` to a CSP directive without first checking the Exceptions table and confirming it is documented there
- If a component spec lists an external resource in its Dependencies table, update the Exceptions table in this file before marking the component `Complete`
- **`Strict-Transport-Security` depends on the platform — do not apply one blanket rule.**
  [Netlify](https://docs.netlify.com/manage/domains/secure-domains-with-https/https-ssl/) sends
  `max-age=31536000` by default, and [Vercel](https://vercel.com/docs/cdn-security/encryption)
  sends `max-age=63072000; includeSubDomains; preload` on `.vercel.app` domains
  and HTTP Strict Transport Security (HSTS) on custom domains, so on those two
  do not add it to project config — you would only duplicate the header. **Cloudflare does not**:
  [HSTS is off until you explicitly enable it](https://developers.cloudflare.com/ssl/edge-certificates/additional-options/http-strict-transport-security/)
  under SSL/TLS → Edge Certificates. A Cloudflare Pages site following a blanket
  "never set it" rule ships with no HSTS at all, and nothing reports that. Enable
  it in the dashboard there rather than in `_headers`, so the header is not sent
  twice. **Render documents neither behaviour** — it redirects HTTP to HTTPS but
  says nothing about sending HSTS — so do not assume either way: check a deployed
  response with `curl -I` and, if the header is absent, add it to the `headers`
  list in `render.yaml` alongside the others. Adding `includeSubDomains` or
  `preload` on a custom domain needs configuration on every platform
- For static output frameworks (Vanilla, React, Eleventy, Astro static mode, Next.js `output: 'export'`), prefer the deployment platform approach over a meta tag — response headers cannot be overridden by the page, whereas a `<meta http-equiv>` CSP tag can be stripped by an injected script
- If using a `<meta http-equiv="Content-Security-Policy">` tag as a fallback (e.g. no deployment platform config is possible), note that `frame-ancestors` is not supported in meta tags — it must be set as a response header to be effective
