# 14 — Project / Case-Study Structure

## 1. Template (normative section order)

| # | Section | Content | Length |
|---|---|---|---|
| 0 | **Summary box** (sidebar on desktop, top on mobile) | Period · role · team size · org (or descriptor) · disclosure level · stack · 1-sentence outcome | `<dl>` |
| 1 | **TL;DR** | 3 sentences: context, what Charles did, result | ~60 words |
| 2 | **Context** | Business domain, users, scale (qualitative unless public) | 100–200 |
| 3 | **Problem** | What was wrong or needed, and why it mattered | 100–200 |
| 4 | **Constraints** | Legacy, compliance, team, timeline, browser support | bullets |
| 5 | **My role** | What Charles owned vs. what the team owned, stated honestly | 50–150 |
| 6 | **Approach** | Architecture and process, with an optional diagram (SVG, accessible, with text alternative) | 200–500 |
| 7 | **Decisions & trade-offs** | 2–4 decisions, each with options considered → choice → why → consequences (ADR-lite) | 200–600 |
| 8 | **Outcome** | Claims with evidence, or qualitative wording (CONTENT-05) | 100–200 |
| 9 | **What I'd do differently** | Signals seniority | 50–150 |
| 10 | **Stack & skills** | Links to skill entities | list |
| 11 | **Evidence & links** | Public links, talks, articles, repos | list |
| 12 | **Related** | Computed: related projects, publications, timeline entries | computed |
| 13 | **Call to action** | "Hiring for similar work? Email / LinkedIn / CV" | fixed |

Sections 7 and 9 are what separate a senior case study from a project list, so they are required for `flagship: true`.

## 2. Disclosure rules

| Level | Can include | Must not include |
|---|---|---|
| `public` | Client or employer name, product name, public URLs, public facts | Internal metrics, internal code, internal architecture diagrams, colleague names without consent |
| `anonymized` | Descriptor ("major US airline", "global logistics company"), generic architecture, qualitative outcomes | Names, logos, screenshots, codenames, internal metrics, anything that lets the org be identified together with the dates |
| `private` | Nothing is rendered | — |

**Publishing checklist (human, in the PR template for case studies):** ☐ disclosure level confirmed with the employer or contract terms · ☐ no internal names or codenames (CONTENT-03 scan passes) · ☐ metrics are public or approved · ☐ diagrams are redrawn generically · ☐ screenshots approved.

## 3. Candidate case studies

| Candidate | Theme | Flagship? | Disclosure (proposed) |
|---|---|---|---|
| United Airlines Pilot Bidding System: frontend modernization as a contractor | modernization, frontend-architecture | ✅ | `public` within the approved facts (§3.1) |
| Enterprise Angular modernization (AngularJS→Angular, or Angular major upgrades, standalone/signals migration) | modernization | ✅ | [confirm] |
| Nx monorepo and shared design system / library strategy | monorepo, design-systems | ◻ | [confirm] |
| AI engineering project (MCP server, RAG system, or agent workflow) | ai-engineering | ✅ | Public (personal/OSS) [confirm which] |
| This portfolio: agent-driven development with objective CI gates | ai-engineering, frontend-architecture | ◻ (meta, but strong) | public |
| Teaching at Alura: turning senior experience into curricula | teaching | ◻ | public |

### 3.1 United Airlines: approved disclosure (OD-07 resolved, ADR-0006)

**Relationship:** Charles worked as a **contractor** and was never a United Airlines employee. The experience entry names the contracting organization as `organization` [confirm name and whether it may be named] and United Airlines as `client`. Copy says "contractor for United Airlines" / "client: United Airlines". JSON-LD never emits `worksFor: United Airlines`.

**Approved facts** (recorded in `client.approvedFacts`):
- United Airlines may be named as the client and project context.
- The Pilot Bidding System may be described at a high level.
- It served approximately 17,000 pilots (provenance: `self-reported` unless a public source is added, then `external-source`).
- Charles's work included frontend modernization, Angular upgrades, architecture, maintainability and related engineering work.

**Never published:** proprietary source code · internal URLs, hostnames or environment names · credentials · internal architecture that is not public · confidential business rules (for example bidding logic details) · private screenshots · names of internal employees (unless approved later) · any information whose public status is uncertain.

**When uncertain:** anonymize or leave it out, and open the PR with `needs-human:claims` (CONTENT-14). Architecture diagrams in this case study are generic, show patterns only, and are labelled "illustrative".

## 4. Non-case-study projects

Smaller projects (OSS, experiments) are `project` entries without `caseStudy`. They are listed on `/work` under "Other work" as compact rows: title · one line · stack · links.
