# 08 — Performance

## 1. Core Web Vitals targets

Targets are for mobile devices at p75 in field data (when available), and for Lighthouse mobile emulation in the lab.

| ID | Metric | Target | "Good" threshold |
|---|---|---|---|
| PERF-01 | LCP | ≤ **1.5 s** lab, ≤ 2.0 s field | 2.5 s |
| PERF-02 | INP | ≤ **100 ms** | 200 ms |
| PERF-03 | CLS | ≤ **0.02** | 0.1 |
| PERF-04 | TTFB (CDN) | ≤ 200 ms lab | 800 ms |

## 2. Byte budgets (deterministic, CI gate G15)

Budgets are compressed transfer sizes (Brotli) per route type, for first view.

| ID | Resource | Information pages | `/kanji/search` |
|---|---|---|---|
| PERF-05 | HTML | ≤ 30 KB (≤ 60 KB for long lists such as `/timeline` and level lists) | ≤ 30 KB |
| PERF-06 | JS | Per-bucket budgets in §2.1 (buckets defined in ADR-0016) | §2.1 |
| PERF-07 | CSS | ≤ 25 KB total, critical CSS inlined when it is < 14 KB | same |
| PERF-08 | Fonts | ≤ 90 KB Latin (≤ 3 files). Japanese display font subsets ≤ 60 KB per page (08 §4) | same |
| PERF-09 | Images above the fold | ≤ 120 KB, LCP image preloaded with `fetchpriority="high"` | n/a |
| PERF-10 | Total requests on first view | ≤ 15 | ≤ 20 |

### 2.1 PERF-06: JavaScript budgets (buckets defined in ADR-0016)

This table is **normative**. `config/budgets.json` is its machine-readable twin, and G15 enforces it. A unit test parses this table and fails if `config/budgets.json` is looser. Loosening either one is a gate weakening (ADR-0017). All values are Brotli-compressed bytes.

| Bucket | Scope | Budget |
|---|---|---|
| Required | Information pages (every route except `/lab/**`) | **0 KB** (G11) |
| Initial first-party, per page total (includes the `site-search` and `kanji-search` stubs) | All information pages | **≤ 2 KB**, of which inline executable JS ≤ 0.5 KB (theme bootstrap) |
| On-demand: site search code | **Pagefind bundle**: `pagefind.js` loader, core JS, WebAssembly, `pagefind-ui` JS (the `site-search` stub is in the Initial row) | ≤ 100 KB total. This is a provisional ceiling: the site-search ticket records measured sizes, and may only tighten it |
| On-demand: site search data | Pagefind index chunks fetched per query | ≤ 50 KB per query. Never prefetched |
| On-demand: kanji search code | `kanji-search` island ≤ 12 KB + search Web Worker ≤ 28 KB | **≤ 40 KB total** |
| On-demand: kanji search data | `/kanji/data/search-index*.json`, loaded by the worker | ≤ 150 KB per shard. Sharding is required beyond one shard |
| Idle prefetch | The page's **primary** on-demand feature only, **code only**, after `load`, while idle, never under Save-Data (ADR-0016) | Within that feature's code budget. No other prefetch |
| Third-party | Any page | **0 KB** until an analytics ADR sets a row here (candidate ceiling 4 KB, async after `load`) |
| Service worker | — | Not permitted (ADR-0016). Its ADR would add a row |
| Demo routes (`/lab/**`) | Per demo | Set by each demo's ADR. There are none at launch |

## 3. Techniques (required)

- Static HTML served from a CDN edge. Hashed assets are `immutable` for 1 year, and HTML is `max-age=0, must-revalidate` with stale-while-revalidate at the edge (10 §4).
- Images use AVIF/WebP with a fallback, explicit `width`/`height` (CLS), `loading="lazy"` below the fold, and `decoding="async"`. Pixel art is shipped at 1× in PNG or SVG and scaled with `image-rendering: pixelated` at integer multiples, so it stays crisp on high-DPI screens without large files.
- Background textures (noise, fine stripes, halftone, paper) are CSS gradients or tiny tiled PNGs (≤ 2 KB) that are cached.
- Fonts use `font-display: swap` with metric-matched fallbacks (`size-adjust`, `ascent-override`) to avoid CLS. At most 2 families are preloaded.
- No third-party requests on page load. Analytics, if approved by its own ADR (11), is async, loads after `load`, and has its budget set in PERF-06 §2.1 by that ADR.
- Cross-document View Transitions are progressive enhancement only.
- Speculation Rules (`prefetch` on hover for same-origin nav) come as inline JSON, not a script library.

## 4. Japanese font strategy

Full CJK fonts are 2–10 MB, so they are never shipped whole.

1. **Body Japanese text uses system fonts:** `"Hiragino Mincho ProN", "Yu Mincho", "Noto Serif JP", serif` for the reference serif, and the sans equivalents in UI. There is no download.
2. **Display glyphs** (the big character on `/kanji/characters/{字}`, section headers) use a self-hosted display font such as a Mincho or a Kyōkasho-style font chosen for teaching accuracy (OD-16b). It is **subset at build time per page** to only the glyphs on that page. A script collects the text of each page, subsets it into a hashed WOFF2 of about 5–30 KB, and writes an `@font-face` rule with `unicode-range`.
3. The Kyōkasho (textbook) style matters for teaching because stroke forms differ from Mincho. Glyph choice is an editorial decision (18 §8).

## 5. Measurement

- **Lab:** Lighthouse CI on every PR (G16) and weekly on production (G20).
- **Field:** CrUX for the origin once there is enough traffic. Optionally, INP/LCP/CLS beacons through the analytics tool's custom events, but only if the tool supports this without an extra script (OD-09). Otherwise field data comes from CrUX alone.
- The colophon shows the budgets next to the **byte sizes measured by the same build** (deterministic, generated at build time, so nothing is committed back). Lighthouse results are linked from the colophon to the latest CI run, not embedded.
