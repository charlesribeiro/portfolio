---
status: accepted
date: 2026-09-28
---

# Crawler policy separates discovery and retrieval from model training

Being agent-first means making the site easy for search engines, AI search and retrieval systems, recruiting, research and citation agents, and user-initiated fetchers to find and read. It does **not** mean automatically granting permission to crawlers that collect content for model training. Crawlers are classified into categories in a versioned policy file, and `robots.txt` is generated from it. Discovery categories are always allowed, which CI enforces. The training category has its own explicit setting, which **defaults to disallowed** until Charles decides otherwise, because that is the reversible choice. The site also publishes Cloudflare's Content Signals (`search`, `ai-input`, `ai-train`) as a supplementary declaration.

## Consequences

- User-agent tokens change over time, so the registry cites each vendor's documentation and is reviewed quarterly.
- Some tokens mix purposes (for example Google-Extended affects Gemini training *and* grounding). The registry records these as `mixed` and gives the explicit decision for each.
- robots.txt is advisory. Enforcement at the edge (Cloudflare bot rules) is optional, must match the policy file, and must never block discovery categories. Cloudflare's managed robots.txt or "block AI bots" toggles must not silently override the repository policy.
