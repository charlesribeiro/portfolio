---
status: accepted
date: 2026-09-28
---

# Agent discoverability is HTML-first; llms.txt and Markdown are conveniences

AI agents are a first-class audience, but we do not rely on `llms.txt`. The server-rendered semantic HTML, readable without JavaScript and carrying one Schema.org `@graph` with stable `@id`s per page, is the primary contract. Every content page also has a Markdown alternate at `{path}.md`, advertised with `rel="alternate" type="text/markdown"` and linked to `/llms.txt` with `rel="describedby"`. The alternates are served with `X-Robots-Tag: noindex` and a canonical `Link` header, so they help agents without creating duplicates in search indexes. The Markdown and JSON must say exactly what the HTML says, which CI checks for parity. This rules out cloaking and keyword stuffing.

## Consequences

- JSON-LD emits only facts that are visible on the page.
- `llms.txt` and `llms-full.txt` are generated, never hand-written.
