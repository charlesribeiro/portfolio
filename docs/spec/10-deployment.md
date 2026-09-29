# 10 — Deployment

## 1. Hosting (approved, ADR-0002)

No concrete blocker for Cloudflare Pages has been found. The known requirements (custom headers, previews, UTF-8 paths, and robots control, SEO-24) are all supported, and IA-04/G17 verify them at implementation.


**Cloudflare Pages** (static assets only, no Functions at launch):

- Free tier, global edge, HTTP/3, Brotli.
- `_redirects` and `config/security-headers.ts` are version-controlled, and `_headers` is generated from the latter at build time (10 §4, needed for 09 §5).
- Per-PR preview URLs for G17 smoke tests.
- A later upgrade path to a single Worker for Markdown content negotiation (OD-17) without migrating.
- Edge traffic and bot/AI-crawler analytics exist on the platform without any client script. Adopting them is decided in the analytics ADR (11 §2, OD-09). The Web Analytics JS beacon stays off unless an analytics ADR approves it.

Alternatives: Netlify (equivalent features), or GitHub Pages (no custom headers, which fails 09 §5, so it is rejected).

## 2. Environments

| Env | Trigger | URL | Indexing |
|---|---|---|---|
| Preview | Every PR | `https://<hash>.<project>.pages.dev` | `X-Robots-Tag: noindex` on all responses. `robots.txt` disallows all |
| Production | Merge to `main` (merge is human-only, ADR-0011) | `SITE_URL` | Indexable |

There are two **build modes** from the same code: `BUILD_MODE=production` (placeholders excluded, robots from `config/crawlers.yaml`) and `BUILD_MODE=preview` (placeholders visible, `robots.txt` disallow-all, `X-Robots-Tag: noindex`). Every PR builds **both**. All `dist/` gates (G6–G16, G22, SEO-22) run on the production-mode build, and only the preview-mode build is deployed to the preview URL.

## 3. Pipeline (GitHub Actions)

```
pull_request:  plan (reads ci/gates.json)
               → build (no secrets): production-mode + preview-mode dist/ → artifacts
               → gate (<id>) matrix, parallel, no secrets: active pre-deploy gates against the production-mode artifact
               → deploy-preview (preview env secret, runs NO repo code): publishes the preview-mode artifact
               → gate (<id>) matrix: active post-deploy gates (G17) against the preview URL
               → verify (aggregate, required): fails unless every active pre-/post-deploy gate succeeded; comments the gate report
pull_request_target (base-branch code only): labeller, gate-integrity (required)
push main:     build → deploy production (production env, runs no repo code) → post-deploy gates against production
               (robots.txt compared with that build's dist/robots.txt, SEO-24)
schedule:      active `scheduled` gates (G0h secret-history scan, G18 links, G19 agent eval, G20 prod Lighthouse) → open/update ISSUES only
```

- Required status checks: **`verify`** and **`gate-integrity`** only. Their names are stable, and the gate set behind them comes from the manifest (ADR-0017).
- **Bootstrap:** the first workflows and gate manifest land before the rulesets require `verify`/`gate-integrity` (06 §1.3).
- **Fork PRs** build without deploying, so they cannot pass `verify` (post-deploy gates need a preview). External PRs are not a request surface.
- **The only secret-bearing gate** is the scheduled G19 eval. It uses an LLM key from the `agent-eval` environment (restricted to `main`) and runs only on `schedule`/`workflow_dispatch`, never on `pull_request` (ADR-0017).
- Scheduled jobs never commit and never open PRs. PRs created with the default `GITHUB_TOKEN` don't trigger workflows, and this limitation is accepted. **No privileged App token is placed in scheduled workflows** to work around it.

- **Build and deploy are separate jobs.** The build job runs repository code (which agents can change) with **no secrets** and uploads `dist/` as an artifact. The deploy job runs **no repository code**: it downloads the artifact and publishes it with a SHA-pinned deploy action, using the Cloudflare API token from the `preview` or `production` environment. A PR therefore cannot exfiltrate the deploy token through changed build scripts. PRs from forks build without deploying.
- The `production` environment is restricted to `main`. Since merging is human-only (ADR-0011), a production deploy always follows a human merge.
- **Actions are pinned by commit SHA.** `permissions:` is least-privilege per job.
- There is a concurrency group per branch.

## 4. Headers (`_headers`)

```
/*
  Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
  Content-Security-Policy: default-src 'self'; img-src 'self' data:; style-src 'self' <build-computed hashes of inlined critical CSS>; script-src 'self' <inline-script hashes> [+ analytics origin only after its ADR]; connect-src 'self' [+ analytics origin only after its ADR]; font-src 'self'; frame-ancestors 'none'; base-uri 'self'; form-action 'self'
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: camera=(), microphone=(), geolocation=(), interest-cohort=()
  Cross-Origin-Opener-Policy: same-origin
  Link: </llms.txt>; rel="describedby"; type="text/markdown"

/_astro/*
  Cache-Control: public, max-age=31536000, immutable

/*.md
  Content-Type: text/markdown; charset=utf-8
  X-Robots-Tag: noindex

/llms.txt
  Content-Type: text/markdown; charset=utf-8
```

**Source of truth:** security headers and CSP are defined in `config/security-headers.ts` (a protected path). `public/_headers` does not exist. The whole `dist/_headers` file is generated at build time from that config, plus per-page `Link` headers.

Per-page `Link: rel=alternate` and `rel=canonical` headers for `.md` are generated into `_headers` at build time. Inline JSON-LD uses `type="application/ld+json"` and is not executed, so it is compatible with CSP. Any inline script (theme bootstrap) uses a build-computed hash in the CSP.

## 5. URL retention

`published-urls.txt` is committed and **maintained inside PRs** (no bot commits to `main`, which the rulesets forbid). G7 compares the build's URL set with it: a removed URL without a `_redirects` rule fails. A new URL that isn't listed fails with the hint "run `pnpm urls:update`". The file is therefore always reviewed alongside the change (IA-06).

## 6. Rollback

Cloudflare Pages keeps immutable deployments, so rollback is a single "promote previous deployment" action or a revert commit. It is documented in `docs/runbook.md`.

## 7. Domain (configurable, ADR-0002)

- The origin is a build-time setting: `SITE_URL`. Everything absolute (canonicals, JSON-LD `@id`s, sitemap, `llms.txt`, feeds, OG URLs) is derived from it, and no origin is hard-coded anywhere (lint rule plus a dist scan for stray origins).
- Until a domain is purchased, production MAY run on the `*.pages.dev` origin. Changing the domain later is a config change plus a redirect from the old origin. `@id` changes are accepted as a one-time migration, and the domain choice does not block implementation.
- A production build without `SITE_URL` fails.
- When the domain exists, DNS goes on Cloudflare with DNSSEC. The contact email is a profile setting and can later move to a domain address that forwards to the personal inbox.
