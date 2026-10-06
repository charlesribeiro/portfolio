# Portfolio Specification

**Status:** v1.0 (2026-09-28), ready for ticket decomposition. All blocking decisions are resolved (see [90-open-decisions.md](90-open-decisions.md)). Accepted decisions are recorded as ADRs in [`docs/adr/`](../adr/), and domain vocabulary lives in [`CONTEXT.md`](../../CONTEXT.md).
**Precedence:** ADR > this spec > issue text. If they conflict, the ADR wins, and the conflict must be flagged.
**Scope:** Product and technical specification only. No application code, no dependencies, no implementation tickets.

## What this is

This is the specification for Charles's professional engineering portfolio. The site has three goals:

1. **Convert** technical recruiters and engineering leaders who arrive from LinkedIn or GitHub into interview conversations. Target roles are Senior Frontend Engineer, Senior Software Engineer and AI-enabled engineering roles, with a focus on international remote work.
2. **Be discoverable and legible to AI agents** (search, recruiting and research agents) as well as to humans. This is a first-class requirement, not an add-on.
3. **Be the evidence.** The site itself should show frontend engineering quality instead of describing it.

## Reading order

| # | Document | Answers |
|---|----------|---------|
| 00 | [Overview](00-overview.md) | Goals, audiences, positioning, principles, glossary |
| 01 | [Information architecture](01-information-architecture.md) | Routes, navigation, URL rules, cross-linking |
| 02 | [Page structure](02-page-structure.md) | Anatomy of every page type |
| 03 | [Content model](03-content-model.md) | Entities, fields, relationships, evidence, disclosure |
| 04 | [Technical architecture](04-technical-architecture.md) | Build pipeline, layers, representations, repo layout |
| 05 | [Frontend stack](05-frontend-stack.md) | Tools chosen, alternatives rejected, dependency policy |
| 06 | [Testing & CI](06-testing-and-ci.md) | Quality gates that objectively validate agent work |
| 07 | [Accessibility](07-accessibility.md) | WCAG 2.2 AA requirements, retro-specific rules |
| 08 | [Performance](08-performance.md) | Core Web Vitals targets and budgets |
| 09 | [SEO & agent discoverability](09-seo-and-agent-discoverability.md) | Metadata, JSON-LD, Markdown alternates, llms.txt |
| 10 | [Deployment](10-deployment.md) | Hosting, environments, headers, rollback |
| 11 | [Analytics](11-analytics.md) | Conversion measurement without cookies |
| 12 | [Visual design](12-visual-design.md) | 2003–2009 enthusiast-web direction, built with modern tools |
| 13 | [Recruiter conversion](13-recruiter-conversion.md) | Journeys, calls to action, trust signals |
| 14 | [Case studies](14-case-studies.md) | Template, disclosure levels, candidate list |
| 15 | [AI positioning](15-ai-positioning.md) | Showing AI work without diluting the frontend positioning |
| 16 | [Engineering showcase](16-engineering-showcase.md) | Which parts of the site prove engineering skill |
| 17 | [Timeline](17-timeline.md) | `/timeline` personal and media history |
| 18 | [Kanji](18-kanji.md) | `/kanji` knowledge base and Kanken journey |
| 19 | [Agent-driven development](19-agent-development.md) | How coding agents work on this repo safely |
| 20 | [Licensing](20-licensing.md) | Licence per material, repository boundaries, licence signals |
| 21 | [Agent GitHub App](21-agent-github-app.md) | Agent identity, permissions, rulesets, token lifecycle |
| 22 | [Agent run isolation](22-agent-run-isolation.md) | Untrusted worker runs: per-run identity, private state, network isolation, trusted brokers, capabilities, adversarial tests |
| 90 | [Decisions register](90-open-decisions.md) | Resolved, defaulted, deferred and open decisions |

## Conventions

- **MUST / SHOULD / MAY** follow RFC 2119.
- Requirements have stable IDs (`A11Y-04`, `PERF-02`, `SEO-11`, …). CI checks, tests and future issues MUST refer to these IDs so every gate can be traced back to a requirement.
- `<SITE>` stands for the configured production origin (`SITE_URL`, see ADR-0002). The final domain is intentionally not fixed.
- Personal facts not given in the brief are marked **[confirm]**. Agents MUST NOT fill them in by guessing.
- Accepted decisions are ADRs in `docs/adr/`. Domain terms are defined in the root `CONTEXT.md`, and the glossary in 00 §8 defers to it.
