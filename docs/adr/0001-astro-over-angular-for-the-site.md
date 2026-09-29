---
status: accepted
date: 2026-09-28
---

# Build the site with Astro, not Angular

Angular is Charles's strongest professional framework, so a reader would expect the portfolio to be built with it. We chose Astro anyway. The site is mostly static content, and its goals are no required JS on information pages (ADR-0016), excellent Core Web Vitals, typed Git-based content, and non-HTML outputs (Markdown, JSON, `llms.txt`) as first-class routes. Angular (including Analog) would add framework runtime and hydration to every page without enough benefit. Choosing the right tool over the familiar one is the architectural judgement the portfolio is meant to show, and the colophon explains this.

## Considered Options

- **Analog (Angular SSG)**: shows the primary stack directly, but ships Angular runtime to information pages and has a less mature static-content story.
- **Next.js / Remix**: React runtime on every page, server-oriented.
- **Eleventy**: minimal, but weaker TypeScript and typed-content ergonomics for agent-written code.

## Consequences

- Individual interactive demos or case-study artefacts MAY use Angular, React or other technologies where appropriate. Each one lives under `/lab/**` with its own performance budget and its own ADR (ADR-0016), and never adds runtime to information pages.
- Angular expertise is shown through case studies (for example the United Airlines modernization), writing and teaching, not through the site's own framework.
