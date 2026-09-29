---
status: accepted
date: 2026-09-28
---

# JavaScript is classified into four buckets; information pages never require it

**Invariant:** professional information, evidence, articles, timeline content, kanji educational content and navigation stay understandable and accessible **without client-side JavaScript**.

"Zero JS" was ambiguous: a theme toggle, copy buttons, on-demand search, possibly analytics, and future interactive demos all need some JS. To make budgets, gates and public claims objective, every byte of client-executed code (JavaScript and WebAssembly) belongs to exactly one bucket:

1. **Required**: code without which a page's content or navigation is unavailable. **None is allowed on information pages**, which are every route except `/lab/**`. G11 renders them with JS disabled.
2. **Initial first-party**: first-party code fetched or executed during page load.
3. **On-demand**: first-party code fetched only after user interaction, **or** prefetched while the browser is idle after `load`. Idle prefetch is allowed only for the page's **primary** on-demand feature (for example kanji search on `/kanji/search`), only for **code, never data**, only within that feature's budget, and never when the user has signalled data saving (`Save-Data`, or `navigator.connection.saveData`).
4. **Third-party**: code served from another origin. None is allowed unless its own ADR approves it (analytics is the only foreseen case, ADR-0010).

This ADR defines the model. **All numeric thresholds live in PERF-06** (spec 08 §2.1) and its machine-readable twin, `config/budgets.json`.

## Classification

| Kind | Bucket | Notes |
|---|---|---|
| Page interaction JS (islands, event handlers) | Initial if loaded with the page, otherwise On-demand | Must enhance a working no-JS baseline on information pages |
| Inline executable `<script>` | Initial | Counted byte for byte and allowed by a CSP hash. Only for things that must run before first paint (theme bootstrap) |
| Web Worker scripts | On-demand | Counted in the owning feature's code budget |
| WebAssembly modules (for example Pagefind's search core) | On-demand | Counted in the owning feature's code budget, like the JS that loads them |
| Data fetched by a feature (search indexes, shards) | On-demand data | Budgeted separately from code in PERF-06. Never idle-prefetched |
| Service Worker | Not permitted without its own ADR | If introduced, the registration snippet is Initial and the worker script gets its own budget. It must never be required for content and must never change content (no cloaking, ADR-0004) |
| Inline JSON and other non-executable data (`application/ld+json`, `speculationrules`) | Not code | Counted in the HTML budget (PERF-05) |
| Third-party scripts (analytics) | Third-party | Async, after `load`, CSP-allow-listed only after its ADR |

## Interactive demonstrations (`/lab/**`)

A demo route MAY require JS when the interaction itself is the product being demonstrated (for example an Angular/RxJS kanji quiz, per ADR-0001). Each demo:
- has its own ADR and its own PERF-06 budget row;
- includes a no-JS description (purpose, how it works, a text alternative or screenshots, a source link);
- is never the only place where a professional fact or educational content appears;
- is excluded from the "No JS required" claim, which is scoped to information pages.

## Considered Options

- **Strictly 0 KB everywhere** (no theme toggle, no copy buttons): the purest option, but it gives up small, harmless UX gains, and demos would be impossible.
- **Per-island budgets without buckets**: simpler, but it can't tell page-load cost from on-demand cost, and that difference drives LCP/INP and the public claim.
- **One total for all JS**: search and demos would consume the whole budget and punish features that cost nothing until used.
- **Lighthouse scores only**: not deterministic, and they don't separate first-party from third-party or required from optional JS.

## Consequences

- Public copy says "No JS required" and scopes it to information pages. Nobody claims "0 KB JS".
- Changing a threshold means editing PERF-06 and `config/budgets.json`, not this ADR. Loosening one is a gate weakening (ADR-0017).
- A new kind of client-executed code that fits no row above needs an amendment to this ADR before it ships.
