# 00 — Overview

## 1. Primary objective

Turn visits from technical recruiters and engineering leaders, most of whom arrive from LinkedIn or GitHub, into **interview conversations** for:

- Senior Frontend Engineer (primary)
- Senior Software Engineer
- AI-enabled engineering roles (frontend or product engineering with LLM, agent or RAG work)

The focus is **international remote** roles. Everything else on the site serves this goal, or at least does not work against it.

## 2. Audiences

| Audience | Arrives from | Time budget | Needs |
|---|---|---|---|
| Technical recruiter | LinkedIn profile or InMail, often on mobile | 30–90 s | Role fit, seniority, stack, location/timezone, remote eligibility, English, how to contact, a CV |
| Engineering leader / hiring manager | Recruiter forward, LinkedIn, GitHub | 5–15 min | Depth: architecture decisions, trade-offs, scale, leadership, writing quality |
| Developer / reader | GitHub, articles, Alura, search | Varies | Writing, code, teaching material |
| Kanji learner | Search, Japanese-learning communities | Varies | Accurate, searchable reference material and study methods |
| **AI agents** (search, recruiting, research, LLM tools) | Crawlers, retrieval tools, `llms.txt` | One fetch | Explicit, factual, structured, verifiable data that is readable without JavaScript |

## 3. Positioning hierarchy

This hierarchy is deliberately **not** four equal skill cards:

1. **Senior Frontend Engineering (primary identity).** 9+ years (from the brief; always rendered from a computed value on the site, CONTENT-10). Angular, TypeScript, RxJS, Nx, React. Enterprise frontend architecture and modernization. Large-scale systems, including frontend modernization of United Airlines' Pilot Bidding System (about 17,000 pilots), delivered **as a contractor**, not as a United employee (ADR-0006).
2. **AI Engineering (secondary, growing).** LLMs, MCP, RAG, agents and AI-assisted engineering. Presented as an extension of frontend and product engineering, not as a change of career. See [15-ai-positioning.md](15-ai-positioning.md).
3. **Evidence of depth and a distinctive identity:**
   - *Technical writing and education.* Alura instructor/freelancer. Author or co-author of work on Tailwind CSS + AI and on Small Language Models.
   - *Japanese / advanced kanji.* A public, educational knowledge base aimed at 漢検準1級 and 漢検1級, with the long-term goal of establishing Charles as a credible kanji/Kanken educator and researcher. Any claim that Charles holds a Kanken level requires explicit evidence and confirmation from Charles (ADR-0009).

The headline states (1), the supporting line adds (2), and (3) comes up throughout the site as proof, not as a job title.

## 4. Success metrics

| Metric | Target (first 6 months) | Source |
|---|---|---|
| Contact conversions (email click, LinkedIn message click; booking if added later) per 100 recruiter-segment visits | ≥ 3 | Analytics events (see 11) |
| CV downloads per 100 home visits | ≥ 8 | Analytics events |
| Median scroll depth on flagship case studies | ≥ 60 % | Analytics |
| Lighthouse (mobile) Performance / A11y / Best Practices / SEO | 100 / 100 / 100 / 100 on key routes | CI |
| Agent readability: key facts correctly extracted by a reference LLM prompt | 100 % of fact checklist | CI smoke eval (see 06 §7) |
| Interviews attributed to the site (self-reported by Charles) | Tracked manually | Spreadsheet or Git log |

## 5. Guiding principles

1. **Show, don't tell.** Every claim carries a provenance class (verified, self-reported, derived or external-source; ADR-0005), and evidence is linked where it exists.
2. **Nothing invented.** Agents never fabricate personal facts, article metadata, quotations, URLs or artwork. Placeholders are allowed during development but can never reach production (ADR-0008).
3. **One source, many representations.** Content lives once in Git as typed data. HTML, Markdown, JSON-LD, JSON, feeds, `llms.txt` and the CV are all derived from it.
4. **No JavaScript required.** Information pages work fully without JS (ADR-0016). Interactivity is opt-in progressive enhancement.
5. **Retro look, modern build.** The 2003–2009 enthusiast-web aesthetic is interpreted through semantic HTML, modern CSS and WCAG 2.2 AA.
6. **Honest and specific.** No keyword stuffing, no unverifiable metrics, no confidential client details.
7. **Verifiable by machines.** Every rule an agent must follow is enforced by a CI gate wherever possible.

## 6. Non-goals (at launch)

- No CMS, database, user accounts, comments or guestbook with persistence.
- No server runtime (see 04). An edge function is allowed only for Markdown content negotiation, and is deferred.
- No newsletter.
- No full localization of every page (see OD-05).
- No auto-generated dictionary pages without original content (see 18 §6).
- No blog about general topics at launch. Writing is limited to publications, case studies, kanji notes and a small number of technical articles.

## 7. Constraints (from the brief)

Modern TypeScript stack · low runtime complexity · excellent Core Web Vitals · strong accessibility · mobile-first · static or server-rendered output · content maintainable in Git · minimal dependencies · no confidential client information · supports autonomous coding agents · CI validates agent work objectively.

## 8. Glossary

> Superseded by the root [`CONTEXT.md`](../../CONTEXT.md), which is authoritative. The table below is kept for spec readers.

| Term | Meaning |
|---|---|
| **Entity** | A typed content item with a stable ID (Experience, Project, Publication, TimelineEntry, KanjiEntry…). |
| **Representation** | One rendered form of an entity or page: HTML, Markdown, JSON-LD, JSON, feed item, CV line. |
| **Evidence** | A source that supports a claim: public URL, archived URL, certificate, publisher listing. |
| **Claim** | A factual statement about Charles that could be true or false (dates, roles, achievements, metrics). |
| **Provenance** | The class of a claim: `verified`, `self-reported`, `derived` or `external-source` (ADR-0005). |
| **Placeholder** | Stand-in content or artwork marked as such. It is build-blocked in production (ADR-0008). |
| **Disclosure level** | How much of a client engagement may be published: `public`, `anonymized`, `private`. |
| **Case study** | A structured narrative of one Project, using the template in 14. |
| **Achievement** | A verifiable personal accomplishment, such as passing a Kanken level. Kept separate from reference content and study notes. |
| **Reference content** | Educational material meant to be accurate and citable (kanji, vocabulary, topics). |
| **Study note** | A personal, informal learning log. Not presented as authoritative. |
| **Island** | An interactive client-side component hydrated on an information page that works without it. |
| **Gate** | An automated CI check that must pass before merging. |
