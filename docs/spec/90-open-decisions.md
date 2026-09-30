# 90 — Decisions Register

Legend: ✅ **Resolved** (decided by Charles, 2026-09-28; the second batch was resolved the same day) · 🟡 **Open, safe default**: implementation proceeds on the default unless Charles overrides · 🔴 **Open, blocks decomposition**: needed before the affected tickets can be written · ⏸ **Deferred** · 📝 **Human-supplied content**: not a design decision, but a `ready-for-human` input.

## Resolved

| ID | Decision | Outcome | Record |
|---|---|---|---|
| OD-01 | Framework | ✅ **Astro.** Angular is Charles's strongest framework, but it would add runtime complexity to a content site without enough benefit. Isolated demos MAY later use Angular, React or others where appropriate | ADR-0001 |
| OD-02 | Hosting | ✅ **Cloudflare Pages** (static), unless a concrete blocker is found during implementation. A blocker must be documented in a new ADR | ADR-0002 |
| OD-03 | Domain | ✅ **Configurable** through `SITE_URL`. Implementation does not wait for the final domain | ADR-0002 |
| OD-04 | Visual direction | ✅ **The Photoshop/Illustrator-era enthusiast and personal web, 2003–2009**, with modern engineering. **Not a retro pixel portfolio.** Pixel/8-bit only as accents. Archive (timeline) and Dictionary (kanji) identities within one system | ADR-0007, 12 |
| OD-07 | United Airlines disclosure | ✅ United may be named as **client/project context**. Charles was a **contractor**, not a United employee. High-level description allowed, including ~17,000 pilots and the scope of work (frontend modernization, Angular upgrades, architecture, maintainability). No code, internal URLs, credentials, non-public architecture, business rules, screenshots, employee names or uncertain information. When uncertain, anonymize or ask | ADR-0006, 14 §3.1 |
| OD-10 | Repository visibility | ✅ **Public.** Secret scanning and strict content boundaries are mandatory | ADR-0006, G0 |
| OD-13 | Kanken claims | ✅ **Never inferred.** A claim that Charles passed or holds a level requires explicit evidence plus confirmation from Charles. Educational content and achievements stay separate | ADR-0009, CONTENT-13 |
| OD-16/18 | Artwork | ✅ **No generic AI-generated artwork as identity.** Original CSS/SVG, hand-made assets, historically inspired elements, and assets Charles provides. Placeholders are marked and build-blocked in production. *(The typeface choice itself is settled through style tiles, see OD-16b)* | ADR-0008 |
| OD-20 | Contact | ✅ **Direct contact links** (email, LinkedIn). An optional `bookingUrl` is supported but unused | 13 §2 |
| OD-21 | Salary | ✅ **Never published** | 13 §5 |
| OD-22 | Media timeline | ✅ **Content model and placeholders now.** Charles will provide the articles later. Nothing is invented | 17 §6, ADR-0008 |
| OD-23 | Kanji data licensing | ✅ **No third-party dataset imported yet.** Authored content first. Architecture ready for per-source licensed imports later. **CC BY-SA is not imposed** on the Kanji section | ADR-0009, 18 §5 |
| — | Provenance | ✅ Four classes: verified / self-reported / derived / external-source, exposed in machine-readable data | ADR-0005 |
| — | Agent governance | ✅ Agents never merge. Human approval for merge, architecture, significant dependencies, personal claims, client-sensitive info and CI/security infrastructure. Standard issue format | ADR-0011, 19 |
| OD-24 | Agent GitHub identity | ✅ **Dedicated GitHub App**, installed on this repo only, least privilege (contents/issues/PRs read-write; no admin, workflows, security or bypass). Rulesets enforce "human merges". Specified, not yet created | ADR-0012, 21 |
| OD-11 | Styling | ✅ **Hand-written modern CSS** (custom properties, cascade layers, Grid/Flexbox, container queries, logical properties, modern selectors, progressive enhancement). No CSS framework without a future ADR | ADR-0013, 05 §5 |
| OD-23a | Licences | ✅ **MIT** code · **CC0 1.0** professional metadata (`/data/*.json`, `/resume.json`, `/llms.txt` except its kanji section, the CV, the professional JSON-LD nodes and their source YAML; revised, resolves LIC-OPEN-2) · **CC BY-NC 4.0** written/editorial and kanji content · **All Rights Reserved** photos and original artwork/visual assets (unless explicitly licensed otherwise) · third-party keeps its licence plus provenance | ADR-0014, 20 |
| OD-06 | Crawler policy | ✅ **Discovery and training are separate.** Search, AI search/retrieval and user-initiated agent fetchers are always allowed. Training is an explicit, configurable setting (see OD-06b) | ADR-0015, 09 §4.1 |
| OD-12 | Recruiter route | ✅ **`/hire`** | 01, 13 |
| — | JavaScript accounting | ✅ Four buckets (required / initial / on-demand / third-party). Information pages never require JS, while `/lab/**` demos may. Idle prefetch allowed within budgets. Workers, service workers, inline JS and inline JSON classified. All numbers only in PERF-06 §2.1 | ADR-0016, 08 §2.1 |
| — | CI ownership | ✅ Thin, human-owned workflows running a **dynamic matrix** from `ci/gates.json` (`id`, `command`, `phase`, `status`, plus `definitionPaths` when active). Required checks: `verify` (aggregate) and `gate-integrity`. Weakening, or editing an active gate's non-threshold definition files, needs Charles's `gate-change-approved` label. Activating new gates, adding tests, tightening threshold files and removing allow-list entries don't. Scheduled jobs open issues only, with no privileged token in schedules | ADR-0017, 06 §1.1–1.2, 19 §3.1 |

## Implementation decisions D1–D11 ✅ (decided by Charles, 2026-09-30, during ticket decomposition)

| ID | Decision | Outcome | Record |
|---|---|---|---|
| D1 | Bootstrap split | ✅ The scaffold, tooling, gate manifest, runners, `scripts/ci/**` and threshold files may land in **pre-bootstrap PRs** that Charles reviews and merges. The bootstrap PR remains the single workflow promotion point | ADR-0017, 06 §1.3, 19 §8 |
| D2 | Numeric claims | ✅ A number needs evidence, or its text must be in `client.approvedFacts`. Otherwise use qualitative wording. A self-reported label alone never permits a number | ADR-0005, CONTENT-05 |
| D3 | First release and navigation | ✅ Release slices (below). The nav shows only sections with at least one published page. Home sections 4–7 are omitted while empty | 01 §2, 02 Home |
| D4 | Japanese-path smoke test before kanji | ✅ The preview-only `/__smoke/日本語` route, removed once a real Japanese-path page exists | IA-04, 06 G17, 10 §2 |
| D5 | Denylist hashing | ✅ scrypt with a committed public salt. The residual risk that names can be guessed is documented | CONTENT-03, ADR-0006 |
| D6 | Press JSON-LD | ✅ The type follows `format`. `copyrightHolder` only with `copyrightHolderConfirmed`. Never `sdLicense` | 09 §3, 17 §2, 17 §5, 20 §1.1, LIC-10 |
| D7 | Remaining PR #1 review threads | ✅ (a) G17 checks preview no-index and the disallow-all `robots.txt` · (b) App-assignee wording qualified · (c) startup barrier for PID registration · (d) "press requires media" already satisfied by the required `mediaMention`; no change | 06 G17 · ADR-0012, 19 §5, `docs/agents/issue-tracker.md`, `CONTEXT.md` · 19 §5 · — |
| D8 | Who runs the pre-CI tickets | ✅ Pre-bootstrap and bootstrap work runs in attended sessions. PRs may come from Charles's identity and are merged with the admin bypass until required checks exist | ADR-0017, 06 §1.3, 19 §8, `CONTEXT.md` (Gate) |
| D9 | Agent-made visual material | ✅ Agents write CSS chrome and geometric SVG. Pictorial artwork and the favicon come from Charles. Interim `<link rel="icon" href="data:,">` | 12 §7 |
| D10 | Production deploy switch | ✅ `deploy-production.yml` runs only when `PRODUCTION_DEPLOY_ENABLED == 'true'`, which Charles sets at the first production release | 10 §3 |
| D11 | Fixture-backed gate builds | ✅ Explicit non-production fixture content. `dist/` and browser gates build `GATE_CONTENT_SET` (fixtures → real, switched by Charles). G6 always builds real content. Production always builds real content and refuses fixture output | 06 §3, 10 §2, 10 §3, `CONTEXT.md` (Fixture content) |

**Release slices (D3).** Every production release needs verified real content. Fixture content is never published (D11).

| Release | Adds |
|---|---|
| v0.1 | Home, Hire, Experience, CV (+ `/cv.pdf`), Colophon, 404, `/site-index`, and every machine-readable surface for those pages (Markdown alternates, JSON-LD, `/data`, `/resume.json`, `llms.txt`, robots, sitemap, `humans.txt`, `security.txt`) |
| v0.2 | Work and case studies, Writing, AI hub, About, feeds, site search |
| v0.3 | Timeline |
| v0.4 | Kanji |

## Open: blocks ticket decomposition 🔴

**None.** Every decision that blocked ticket decomposition is resolved.

## Open: safe default, implementation proceeds 🟡

| ID | Decision | Default in effect |
|---|---|---|
| OD-06b | Training-crawler permission (`trainingPolicy`) | **`disallow`** (the reversible choice). Discovery categories are always allowed regardless. One-line config change (09 §4.1) |
| OD-05 | Languages | English site with inline `lang="ja"` / `lang="pt-BR"`. Translations later under `/ja/`, `/pt/` (additive) |
| OD-09 | Analytics tool | **Default: no analytics** until the analytics ADR is accepted. Proposed: Cloudflare **edge** analytics (no script, no CSP or budget change) + GoatCounter for events. The Cloudflare Web Analytics JS beacon is off by default (11 §2). **Nothing is adopted until the analytics ADR is accepted**, and that ADR is also required by ADR-0010/0016 for any third-party origin, so the analytics ticket is `ready-for-human` at the decision step. It comes last in order |
| OD-14 | No-JS site-search fallback | Static browse indexes only. No external search form |
| OD-15 | Font subsetting tool | Delegated to implementation. Requires DEP-02 justification (`needs-human:dependency` if it runs native code) |
| OD-16b | Typefaces | Chosen through the style-tile deliverable (12 §9), a `ready-for-human` approval ticket |
| OD-19 | Profile photo | No photo until Charles provides one. The layout works without it |
| LIC-OPEN-1 | Dual-license article code snippets under MIT? | No. Snippets follow their file's licence (20 §1) |
| — | G19 agent-readability eval: LLM provider/key | Disabled until a key is provided. Not a required gate |

## Deferred ⏸

| ID | Decision |
|---|---|
| OD-17 | `Accept: text/markdown` content negotiation through a Worker (would break "no server runtime", needs a new ADR) |
| — | Scheduling link, `/lab/**` demos (Angular/React, each with its own ADR under ADR-0016), service worker, licensed kanji dataset imports, stroke-order diagrams, translations |

## Human-supplied content 📝 (`ready-for-human` tickets, not blockers)

- Full name form for display and JSON-LD, location, timezone, and the start date of the frontend career.
- Contracting organization for the United Airlines engagement, whether it may be named, and the dates.
- The other experience entries (disclosure level per client).
- Work authorization, remote/relocation stance and notice period (content only: no design decision is pending).
- Publications: exact titles, publishers, dates, co-authors, URLs (Tailwind CSS + AI, Small Language Models), and the Alura course list.
- Education, and language levels with any evidence.
- Media articles for the timeline (G1, Folha, others): URLs only. Metadata is confirmed by Charles.
- Kanken history: only if and when Charles chooses to publish it, with evidence.
- Candidate case studies beyond United (14 §3) and their disclosure levels.
- The Cloudflare account and API token, stored as a GitHub environment secret by Charles.
- Creating the GitHub App, installing it, storing its key on bubbles-server, and configuring rulesets and CODEOWNERS (21 §7).
