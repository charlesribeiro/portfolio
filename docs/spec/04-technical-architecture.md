# 04 — Technical Architecture

## 1. Summary

The site is a **statically generated** site with no server runtime. All HTML, Markdown, JSON-LD, JSON, feeds, `llms.txt` and the CV PDF are generated at build time from typed content in Git and deployed to a CDN. Interactivity is limited to a few progressive-enhancement islands.

Rationale: static output gives the best Core Web Vitals, the smallest attack surface, zero runtime cost, and deterministic builds that CI can verify byte for byte. Nothing in the requirements needs per-request computation. Markdown content negotiation through `Accept` headers is the only possible exception, and it is deferred and optional (09 §5).

## 2. Build pipeline

```
content/*.yaml|mdx ─┐
third-party/data/… ─┼─▶ [1] Content layer (schemas, validation) ─▶ [2] Domain layer (pure TS)
                    │                                                  │  entities, relationships,
                    │                                                  │  derived facts (years, back-links,
                    │                                                  │  timeline ordering, search docs)
                    │                                                  ▼
                    │                      [3] Representation adapters (pure functions of domain objects)
                    │         ┌──────────────┬──────────────┬─────────────┬───────────────┬────────────┐
                    │         ▼              ▼              ▼             ▼               ▼            ▼
                    │   HTML pages     Markdown alts   JSON-LD graph   JSON datasets   llms.txt(+full)  feeds/sitemap
                    │   (.astro)       (*.md routes)   (per page)      /data, /kanji   /resume.json     robots/humans
                    ▼
               [4] Post-build: Pagefind index · CV PDF render · image optimization · dist/ scans (04 §7)
                    ▼
               dist/  ──▶  CDN (see 10)
```

## 3. Layers and boundaries

| Layer | Location | Responsibility | Rules |
|---|---|---|---|
| Content | `content/` (and, only after an import ADR, `third-party/data/`) | Source of truth | Data only, no code (except allow-listed MDX components) |
| Schemas | `src/content.config.ts` | Validation | The only place collection shapes are defined |
| **Domain** | `src/domain/` | Queries and derivations: `getProfile()`, `getCaseStudies({flagship})`, `getSkillEvidence(id)`, `getTimeline({category})`, `getKanjiEntry(slug)`… | Pure, synchronous after load, framework-agnostic, and covered by G5's coverage threshold (`config/thresholds.json`). **Pages MUST NOT read collections directly** (enforced by a lint rule restricting imports of `astro:content` to `src/domain/`). |
| Representations | `src/representations/{html,markdown,jsonld,json,llms,feeds}/` | Turn domain objects into outputs | Pure functions, snapshot-tested. JSON-LD builders are typed with `schema-dts`. |
| Presentation | `src/pages/`, `src/components/`, `src/layouts/`, `src/styles/` | Routes and UI | Thin. Pages call domain, then pass results to components and representations. |
| Islands | `src/islands/` | Client-side enhancement | Vanilla TS custom elements. Each island has a no-JS baseline and a byte budget (PERF-06). |

This design gives agents one small, well-tested surface (the domain layer) for "what is true", and keeps every output format consistent by construction.

## 4. Representations of one page

For a case study `/work/pilot-bidding`, the build emits:

- `/work/pilot-bidding` (HTML) with `<link rel="canonical">`, `<link rel="alternate" type="text/markdown" href="/work/pilot-bidding.md">`, `<link rel="describedby" href="/llms.txt">`, and embedded JSON-LD (`@graph` with WebPage, CreativeWork/Article, Person ref, BreadcrumbList).
- `/work/pilot-bidding.md`: Markdown with a YAML front-matter header (title, canonical, updated, entity type, evidence links) followed by the same prose. It is generated from the same MDX AST, with components mapped to Markdown equivalents.
- An entry in `/sitemap.xml`, `/llms.txt`, `/llms-full.txt` and the relevant feed.

A parity test (TEST-REP-01) checks that headings and paragraph text in the HTML `<main>` equal the Markdown body after normalization. A second test (TEST-REP-02) checks that every fact value emitted in `/data/*.json`, `/resume.json` and the page's JSON-LD for an entity appears in that entity's canonical HTML page as visible text, a link target, or a `datetime`, `lang` or `<data value>` attribute value. Exemptions are closed lists: in `/data/*.json`, keys whose JSON Schema marks them `"x-not-rendered": true` (e.g. `id`, `$schema`, `license`, `licenseExclusions`, `provenance.class`, derivation rules, `rights`). For JSON-LD keywords and properties (`@context`, `@type`, `@id`, `license`, `sdLicense`, …) and for `/resume.json` keys, the exemptions are listed in `config/allowlists/representation-parity.json`, so adding one is a W4 signal. A press node's `copyrightHolder` is never exempt: it is emitted only when confirmed and is then rendered visibly (17 §3).

## 5. Islands (the complete list at launch)

| Island | Page(s) | No-JS baseline | JS buckets (ADR-0016). Numbers live in PERF-06 §2.1 |
|---|---|---|---|
| `site-search` | global | The HTML site index `/site-index` (every page, grouped by section) plus section indexes. No external search form (OD-14) | **`site-search` stub** (the custom element that waits for focus or click): Initial. **Pagefind bundle** (`pagefind.js` loader, core JS, WASM, `pagefind-ui` JS): On-demand code (idle prefetch only where site search is the page's primary feature). Index chunks: On-demand data, never prefetched |
| `kanji-search` | `/kanji/search` | Static browse indexes (level, radical, reading row, category) | **`kanji-search` stub** (reads `?q=`, waits for input): Initial. Island + Web Worker: On-demand code, idle-prefetched on `/kanji/search` (its primary feature), and loaded immediately when `?q=` is present (a shared query counts as user intent). Index shards: On-demand data, never prefetched |
| `theme-toggle` | global | `prefers-color-scheme` | Inline bootstrap + toggle: Initial |
| `copy-button` | code blocks, citations | Text is selectable | Initial |

Budgets are defined **only** in PERF-06 §2.1. Adding an island requires an ADR and a PERF-06 row. Interactive demonstrations are not islands. They live under `/lab/**` (ADR-0016).

## 6. CV PDF

The `/cv` route has a print stylesheet. A post-build step uses Playwright (already a dev dependency for tests) to print `dist/cv.html` to `dist/cv.pdf` with embedded fonts and PDF metadata (title, author, subject). The PDF is tagged for accessibility where Chromium supports it. There is no separate PDF template, so the CV never drifts from the site.

## 7. Post-build scans (run as separate gates G7, G9 and G15, not inside `astro build`)

- CONTENT-02/03: private and confidential content scan.
- CONTENT-08: untagged CJK scan.
- SEO-*: every HTML page has a canonical, a Markdown alternate that exists, valid JSON-LD, and an entry in the sitemap.
- Links: all internal links and anchors resolve.
- Budgets: HTML/CSS/JS/font/image byte budgets (08).

## 8. Repository layout (target)

```
/
├─ AGENTS.md  CLAUDE.md  CONTEXT.md
├─ docs/{spec,adr,agents}/
├─ content/
│  ├─ profile.yaml
│  ├─ experience/*.yaml   projects/*.{yaml,mdx}   publications/*.yaml
│  ├─ skills.yaml         achievements/*.yaml     education/*.yaml
│  ├─ timeline/*.yaml     media/*.yaml            pages/{about,hire,colophon}.mdx
│  └─ kanji/{characters,words,yojijukugo,kotowaza,homophones,topics,radicals,notes}/…
├─ public/                static text passthrough only (e.g. humans.txt source). _headers is generated (10 §4), and binaries live under assets/ or third-party/ (LIC-02)
├─ assets/{photos,art,placeholder}/   All Rights Reserved (20 §2)
├─ third-party/{fonts,data}/          original licences + SOURCE.md
├─ LICENSE  LICENSES/  REUSE.toml  LICENSING.md  (20)
├─ config/crawlers.yaml               crawler policy (09 §4.1), owner-decided
├─ config/security-headers.ts         CSP and security headers → generated _headers (10 §4)
├─ ci/gates.json                      gate manifest (ADR-0017)
├─ ci/proposed/                       agent-drafted workflow YAML awaiting promotion
├─ config/budgets.json  config/thresholds.json   threshold files (06 §1.1)
├─ config/allowlists/*.json           allow-lists (06 §1.2 W4)
├─ scripts/ci/                        gate-integrity helpers (run from base branch)
├─ src/
│  ├─ content.config.ts
│  ├─ domain/             (+ *.test.ts)
│  ├─ representations/    (+ *.test.ts, __snapshots__)
│  ├─ components/ layouts/ pages/ islands/ styles/
│  └─ lib/                small shared utils (dates, ja-text normalization)
├─ scripts/               build-cv-pdf, scan-dist, check-budgets, claims-diff (import adapters only after an import ADR)
├─ tests/{e2e,a11y,visual,seo}/
└─ .github/workflows/
```

## 9. Determinism

- Builds are reproducible: pinned lockfile, pinned Node version, no network access during build except fetching fonts from the repo (fonts are self-hosted).
- `updated` dates come from Git (`git log -1 --format=%cs -- <file>`, a `YYYY-MM-DD` value matching `ISODate`). CI checks out full history (`fetch-depth: 0`).
- Any future imported dataset is a committed snapshot with provenance and licence (ADR-0009), and is never fetched at build time.
