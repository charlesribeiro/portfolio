# 07 — Accessibility

**Baseline:** WCAG 2.2 Level AA on every page, with selected AAA criteria noted below. An accessibility statement is published at `/colophon#accessibility`.

## 1. Requirements

| ID | Requirement |
|---|---|
| A11Y-01 | Semantic landmarks (`header`, `nav`, `main`, `aside`, `footer`), one `h1`, no skipped heading levels. |
| A11Y-02 | A skip link is the first focusable element and becomes visible on focus. |
| A11Y-03 | Every interactive element is reachable and operable by keyboard. Focus is visible with a custom focus style (≥ 3:1 contrast, ≥ 2 px, not colour alone). Meets 2.4.11 Focus Not Obscured. |
| A11Y-04 | Text contrast ≥ 4.5:1 (body text target **7:1**, AAA). UI components and graphics ≥ 3:1. Verified for light, dark and every section identity (12 §4–5). |
| A11Y-05 | Body text ≥ 16 px (1 rem) and user-scalable. The layout works at 400 % zoom / 320 CSS px width without horizontal scrolling (1.4.10 Reflow), except for data tables, which scroll inside their own container. |
| A11Y-06 | Text spacing overrides (1.4.12) don't break layouts. No fixed-height text containers. |
| A11Y-07 | `prefers-reduced-motion: reduce` disables all non-essential animation, including view transitions, "blink" and marquee-style effects, and sprite animations. No content flashes more than 3 times per second. |
| A11Y-08 | Forced colors / Windows High Contrast: every piece of chrome (bevels, badges, tabs) stays understandable, using `forced-color-adjust` only where it is justified. G12 runs in forced-colors mode. |
| A11Y-09 | Images: meaningful images have `alt`. Decorative pixel art uses `alt=""` or CSS backgrounds. **No text inside images**: all banner and button text is live text styled with CSS, or has an equivalent accessible name. |
| A11Y-10 | Language: `<html lang="en">`. Every Japanese run has `lang="ja"` and every Portuguese run has `lang="pt-BR"` (3.1.2 Language of Parts), which is enforced by CONTENT-08. |
| A11Y-11 | Ruby: `<ruby>` with `<rp>` fallbacks. Where a reading is essential, it is also available as plain text (for example in a `<dl>` "Reading: かんじ"), because ruby support in screen readers varies. |
| A11Y-12 | Large kanji display glyphs have an accessible name that includes the character and its primary readings. Stroke-order diagrams (if added) have a text alternative (stroke count plus a description) and animation controls. |
| A11Y-13 | 88×31 badges and old-web buttons are real links or images with accessible names that describe the destination, not the image ("WCAG 2.2 AA statement", not "badge"). |
| A11Y-14 | Target size ≥ 24×24 CSS px (2.5.8). Primary calls to action ≥ 44×44. |
| A11Y-15 | Pixel/bitmap fonts are used **only** for short decorative or display chrome (≤ 3 words, ≥ 16 px at an integer scale). They are never used for body text, navigation labels that have no readable fallback, or Japanese text. |
| A11Y-16 | Tables have `<caption>`, `scope` and headers. Timeline and kanji lists use `<ol>`/`<dl>` semantics. |
| A11Y-17 | Search islands: combobox or listbox patterns follow ARIA APG. Result counts are announced through a polite live region. Everything works without JS through links. |
| A11Y-18 | The CV PDF is tagged, has a document title and language, and its reading order is checked manually at each release. |
| A11Y-19 | No information is conveyed by colour alone, which matters for the timeline categories and kanji level colours. |
| A11Y-20 | Link text makes sense out of context. External links are indicated visually and in text ("(external)" visually hidden or shown as an icon with a label). |

## 2. Testing

- **Automated:** axe on every route in 3 colour modes (G12), html-validate (G8), ESLint a11y rules (G2), keyboard E2E tests for the nav, menu and search (G13).
- **Manual (release checklist, `docs/a11y-checklist.md`):** VoiceOver (macOS and iOS Safari) and NVDA (Firefox) on the home page, a case study, the timeline, a kanji entry and search. Also 400 % zoom, a text-spacing bookmarklet, forced colors, and the PDF reading order.
- Automated checks catch roughly a third of issues, so the manual checklist is required before each production content release that adds new templates.
