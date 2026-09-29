---
status: accepted
date: 2026-09-28
---

# Static output on Cloudflare Pages, with the origin as configuration

The site is fully pre-rendered at build time with **no server runtime**, and is deployed to **Cloudflare Pages**. We chose Pages over GitHub Pages because the site needs version-controlled custom headers (`Link` discovery headers, `.md` content types, `X-Robots-Tag`, CSP), per-PR preview URLs for smoke tests, and a path to a single edge Worker later without migrating. The production origin is a build setting (`SITE_URL`) and is never hard-coded, so buying or changing the domain does not block implementation. A production build without it fails.

## Consequences

- Features that need per-request logic (for example `Accept: text/markdown` negotiation) are deferred and need a new ADR, because they break the no-runtime rule.
- If a concrete Cloudflare blocker appears during implementation, it is documented in a superseding ADR. Netlify is the fallback, since it offers equivalent headers and previews.
- Changing the domain later changes JSON-LD `@id`s, canonicals and feed IDs. This is accepted as a one-time migration, handled with redirects from the old origin.
