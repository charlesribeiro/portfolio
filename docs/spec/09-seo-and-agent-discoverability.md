# 09 — SEO & Agent Discoverability

The site has two readers: people using browsers, and machines (search crawlers, AI agents, recruiting tools, LLM retrieval). Both get **the same facts**, and no reader gets content the others can't see. The HTML is the primary source for agents. `llms.txt` and the Markdown alternates are conveniences, not substitutes.

## 1. Principles

1. **Server-rendered, semantic HTML is the contract.** Every fact is in the HTML without executing JS (G11).
2. **Explicit facts over marketing copy.** Use `<dl>` fact boxes, `<time datetime>`, and clear sentences ("Charles has worked as a frontend engineer since {profile career start} [confirm]"), not slogans.
3. **Every claim is verifiable or labelled.** Evidence links sit next to claims (CONTENT-04).
4. **No keyword stuffing, hidden text or cloaking.** Markdown and JSON mirror the HTML content (TEST-REP-01, TEST-REP-02). Skill aliases are used for search indexing only and never rendered as keyword lists.
5. **One entity graph.** JSON-LD uses stable `@id`s so machines can merge facts across pages.

## 2. Per-page `<head>` requirements

| ID | Requirement |
|---|---|
| SEO-01 | Unique `<title>` (≤ 60 chars, pattern `{Page} — {profile.name}`) and `meta description` from `summary`. |
| SEO-02 | `<link rel="canonical">` with an absolute URL following IA-01. |
| SEO-03 | `<link rel="alternate" type="text/markdown" href="{canonical}.md">` on every content page. |
| SEO-04 | `<link rel="describedby" type="text/markdown" href="/llms.txt">` on every page (site-level description for agents). |
| SEO-05 | `<link rel="alternate" type="application/atom+xml">` for the relevant feeds. `<link rel="alternate" type="application/json" href="/resume.json">` on `/`, `/experience`, `/cv` and `/hire`. |
| SEO-06 | Open Graph and Twitter card tags. The OG image is generated at build time per page from an HTML/SVG template (retro "card" style) rasterized with sharp, 1200×630, ≤ 100 KB. |
| SEO-07 | JSON-LD `<script type="application/ld+json">` containing a single `@graph` per page (§3). |
| SEO-08 | `hreflang` only when translations exist (OD-05). |
| SEO-09 | `<meta name="robots">` is omitted (default index) except on the 404 page and filtered views that duplicate content. |
| SEO-10 | microformats2: `h-card` on the quick-facts box and `h-entry` on articles and timeline entries. They cost nothing, suit the era, and IndieWeb tools read them. |

## 3. JSON-LD entity graph

Stable IDs:

| Entity | `@id` |
|---|---|
| Person | `<SITE>/#person` |
| WebSite | `<SITE>/#website` |
| Organization (each employer/client, when `public`) | `<SITE>/#org-{id}` |
| Skill (DefinedTerm) | `<SITE>/#skill-{id}` |
| Achievement (EducationalOccupationalCredential) | `<SITE>/kanji/journey#cred-{id}` |
| Publication | `<SITE>/writing#{id}` or its page URL |
| Kanji term set | `<SITE>/kanji#termset-{collection}` |

Mapping:

| Content | Schema.org |
|---|---|
| Home, `/about`, `/hire` | `ProfilePage` with `mainEntity` → `Person` |
| Person | `Person`: `name`, `alternateName`, `jobTitle`, `description`, `url`, `image`, `email`, `sameAs` (LinkedIn, GitHub, Alura), `knowsAbout` (DefinedTerm refs with Wikidata `sameAs`), `knowsLanguage`, `hasCredential`, `alumniOf`, `hasOccupation` (`Occupation` with `skills`, `occupationLocation`), `worksFor` (current, if public), `subjectOf` (media mentions) |
| Experience | Past roles as `Role`-wrapped `worksFor`/`alumniOf` entries (`startDate`, `endDate`, `roleName`). `worksFor` always points to the **employer or contracting organization**, never to a client. Client work is expressed as the case study `Article` `about` → client `Organization` with the role described as contractor (ADR-0006). Anonymized organizations are emitted as `Organization` with a `description` only and no `name` |
| Case study | `Article` (or `CreativeWork`), `about` → skills, `author` → `#person` |
| Book / chapter / article | `Book` / `Chapter` / `ScholarlyArticle` / `Article` with `author`/`contributor`, `publisher`, `isbn`, `inLanguage`, `url` |
| Alura course | `Course` with `provider` Alura and `instructor` `#person`, `hasCourseInstance` if applicable |
| Talk | `Event` with `performer` |
| Timeline page | `CollectionPage` + `ItemList` of entries. Press entries → a **press node** whose type follows `MediaMention.format` (D6): `article`/`print` → `NewsArticle`, `video` → `VideoObject`, `radio` → `RadioEpisode`, `podcast` → `PodcastEpisode`. Each press node carries external `url`, `publisher`, `datePublished`, `headline` (a quotation, LIC-10) and `about` `#person`, and never `sdLicense`. It carries `copyrightHolder` only from the work-level `MediaMention.copyrightHolder`, and only when `copyrightHolderConfirmed` is set (17 §2) |
| Kanken (only when an Achievement exists with owner confirmation, ADR-0009) | `EducationalOccupationalCredential` (`credentialCategory` "certificate", `recognizedBy` Organization 日本漢字能力検定協会, `educationalLevel` "準1級", `dateCreated`) |
| Kanji reference entry | `DefinedTerm` in a `DefinedTermSet`, with `inLanguage: "ja"`, `alternateName` for readings |
| Kanji topic guide | `LearningResource` (`educationalLevel`, `teaches`, `inLanguage`) |
| Study note | `BlogPosting` (explicitly personal) |
| Datasets | `Dataset` with `distribution` (`DataDownload`: JSON/CSV), `license`, `creator`, and `isBasedOn` only when a third-party source is actually included |
| Breadcrumbs | `BreadcrumbList` |

### 3.1 Provenance and evidence in JSON-LD (ADR-0005)

Schema.org has no single provenance vocabulary, so the mapping is pragmatic. It uses only well-understood properties and never invents properties:

| Provenance | JSON-LD expression |
|---|---|
| `verified` | The fact's node carries `subjectOf` / `citation` → the evidence `CreativeWork` (`url`, `publisher`, `datePublished`). Credentials use `EducationalOccupationalCredential.recognizedBy` plus `url` of the evidence |
| `external-source` | The source is a press node (typed by `format`, §3) or a `WebPage` node, with `publisher` and `url`. The Person links to it through `subjectOf`, and the site does not restate the source's claims as its own |
| `derived` | Emitted as plain values only. The derivation rule is documented in `/data/schema/` and exposed in `/data/*.json` |
| `self-reported` | Emitted as plain values with no citation. The WebPage `author`/`publisher` is `#person`, which makes it clear that Charles is the source |

The **complete** four-class provenance is machine-readable in `/data/*.json` (`provenance` objects) and in `.md` front-matter. JSON-LD carries the evidence *relationships*. SEO-21: every JSON-LD citation URL equals an evidence URL in content (contract test).

JSON-LD rules: `sdLicense` appears only on CreativeWork nodes (CC0 for professional ones, CC BY-NC for editorial and kanji ones). Press nodes (any type chosen by `format`, 17 §5) omit it, and carry `copyrightHolder` only when it is confirmed (LIC-10, D6). Builders are typed with `schema-dts` and snapshot-tested. They emit only facts that appear visibly on the page (Google's guideline, and it prevents cloaking). `@graph` is validated in CI by parsing plus structural assertions per page type (G9). A manual Rich Results Test is run at release.

## 4. Site-level machine files

| File | Content |
|---|---|
| `/robots.txt` | Generated from `config/crawlers.yaml` (§4.1). Discovery categories are allowed, and training follows its explicit setting. Lists `Sitemap:` and Content Signals. |
| `/sitemap.xml` | Every indexable HTML URL with `lastmod` from Git. `.md` and JSON files are excluded. |
| `/llms.txt` | Follows the llmstxt.org format: `# {profile.name}`, a blockquote summary (role, years, stack, AI focus, location/remote, contact), then `## Profile`, `## Work`, `## Writing`, `## Timeline`, `## Kanji`, `## Data` sections that list `.md` URLs with one-line descriptions, and `## Optional` for secondary material (CC0-sourced items only). Each section carries a licence line derived from its source paths (20 §3). Generated, ≤ 10 KB. |
| `/llms-full.txt` | The concatenated Markdown of profile, experience, case studies, about, publications (with notes), the timeline (with context) and the Kanji journey, each section with its licence line (20 §3) (not the whole kanji reference, which is linked through datasets instead). ≤ 500 KB. Generated. |
| `/{page}.md` | Markdown alternate (IA-02). YAML front-matter includes `canonical`, `title`, `type`, `updated`, `evidence[]`. |
| `/resume.json` | JSON Resume schema, generated from the domain layer. |
| `/data/*.json` | Profile, experience, publications, projects and timeline facts as JSON (CC0, 20 §1.1), with a `$schema` URL pointing to published JSON Schemas under `/data/schema/` (generated from Zod). |
| `/humans.txt` | Credits, stack, the "made by hand" note. Era-appropriate. |
| `/.well-known/security.txt` | Contact. |

### 4.1 Crawler policy (ADR-0015)

Agent discoverability and permission to train models are **separate requirements**. The policy file `config/crawlers.yaml` is the single source of truth. `robots.txt`, the colophon's "Crawler policy" section and a test fixture are all generated from it.

| Category | Purpose | Examples (tokens per vendor docs, re-verified at implementation) | Policy |
|---|---|---|---|
| `search` | Classic web search indexing | Googlebot, Bingbot, DuckDuckBot, Applebot, YandexBot | **Always allow** (enforced) |
| `ai-search` | AI search and retrieval indexes that cite sources | OAI-SearchBot, Claude-SearchBot, PerplexityBot | **Always allow** (enforced) |
| `user-agent-fetch` | A human asked an assistant or agent to fetch a page (recruiting, research, citation) | ChatGPT-User, Claude-User, Perplexity-User, meta-externalfetcher | **Always allow** (enforced). These fetchers may not consult robots.txt at all |
| `training` | Collection for model training | GPTBot, ClaudeBot, Applebot-Extended, meta-externalagent, Bytespider | **`trainingPolicy` setting. Default `disallow`** until Charles decides (OD-06b) |
| `dataset` | Open corpora used for training *and* research | CCBot | Follows `trainingPolicy` unless overridden per entry |
| `mixed` | Tokens that control more than one purpose | Google-Extended (Gemini training and grounding) | Explicit per-entry decision with a written rationale. Default follows `trainingPolicy` (Google Search indexing is unaffected) |
| `unknown` | Everything else | `*` | Allow (the default robots behaviour). Edge rate-limiting is allowed |

```yaml
# config/crawlers.yaml (shape)
trainingPolicy: disallow          # allow | disallow — OD-06b
contentSignals: { search: yes, ai-input: yes, ai-train: no }   # ai-train mirrors trainingPolicy
reviewedOn: 2026-09-28            # CI warns when older than 100 days
crawlers:
  - token: GPTBot
    vendor: OpenAI
    category: training
    docs: https://…                # vendor documentation URL, REQUIRED
  - token: Google-Extended
    category: mixed
    decision: follow-training     # REQUIRED for mixed
    rationale: "…"
```

| ID | Requirement |
|---|---|
| SEO-22 | CI (G9) asserts that the generated `robots.txt` never disallows any `search`, `ai-search` or `user-agent-fetch` token, or `*`, for indexable paths. The training entries match `trainingPolicy`. Every registry entry has a `docs` URL, and every `mixed` entry has a `decision` and a `rationale`. |
| SEO-23 | Content Signals line: `Content-Signal: search=yes, ai-input=yes, ai-train=<value>`, where `<value>` is `yes` or `no` derived from `trainingPolicy`. This is an emerging convention and supplementary to robots rules. |
| SEO-24 | Cloudflare edge settings (managed robots.txt, "Block AI bots", AI Crawl Control) must be off or configured to match `config/crawlers.yaml`. The post-deploy smoke test (G17) fetches `/robots.txt` from production and compares it with the build output byte for byte. |
| SEO-25 | Licences (20) and crawler policy are documented separately. Neither implies the other. |

## 5. HTTP-level discovery (headers, see 10 §4)

- HTML: `Link: </llms.txt>; rel="describedby"; type="text/markdown"` and `Link: <{path}.md>; rel="alternate"; type="text/markdown"`.
- `.md` files: `Content-Type: text/markdown; charset=utf-8`, `Link: <{canonical}>; rel="canonical"`, `X-Robots-Tag: noindex`. This keeps duplicates out of search indexes while leaving them fetchable by agents.
- **Deferred (OD-17):** `Accept: text/markdown` content negotiation on canonical URLs through an edge function. Nice to have, and it breaks the "no server runtime" rule, so it needs approval.

## 6. Classic SEO

- Target queries are ones people actually search, not stuffing: "{profile.name} frontend", "Senior Angular engineer remote {profile.location.country}", publication titles, and for kanji "漢検準1級 四字熟語", "漢検1級 対策", specific terms.
- Internal linking comes from the entity graph (01 §5).
- The kanji reference area is where organic reach will come from over the long term. Its quality bar is set in 18 §6.
- Search Console and Bing Webmaster Tools are verified through DNS, not meta tags.

## 7. SEO/agent CI contract (G9) — assertions

SEO-11 every HTML page in `dist/` has SEO-01…07 · SEO-12 every canonical is in the sitemap, and every sitemap URL exists · SEO-13 every `.md` alternate exists, is non-empty, and its front-matter canonical matches · SEO-14 every link in `llms.txt` resolves in `dist/` · SEO-15 JSON-LD parses and contains the required `@type`s per template · SEO-16 `#person` is defined identically everywhere (hash compared) · SEO-17 `/resume.json` validates against the JSON Resume schema · SEO-18 `/data/*.json` validate against their published schemas · SEO-19 no `noindex` on indexable templates · SEO-20 titles and descriptions are unique across the site.
