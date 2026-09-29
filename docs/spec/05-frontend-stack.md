# 05 — Frontend Stack

## 1. Choices

| Concern | Choice | Why |
|---|---|---|
| Language | **TypeScript**, `strict` + `noUncheckedIndexedAccess` + `exactOptionalPropertyTypes` | Required by the brief. Maximum type safety for agent-written code. |
| Site framework | **Astro** (current stable major at scaffold time), **approved in ADR-0001** | Static-first, zero JS by default, typed content collections with Zod, file-based routing, non-HTML endpoints (`.md`, `.json`, `.txt`) as first-class routes, islands for progressive enhancement. Minimal runtime. |
| Styling | **Hand-written modern CSS**: cascade layers, custom properties (design tokens), container queries, `:has()`, logical properties, native nesting | **Approved in ADR-0013.** The Photoshop-era chrome is bespoke, and the CSS itself is evidence of skill (16). Conventions are in §5. |
| Interactivity | **Vanilla TS custom elements** (islands) | No UI framework runtime needed for four small islands. |
| Content | Astro content layer + MDX (allow-listed components) + YAML | Git-native and schema-validated. |
| Search | **Pagefind** (site-wide full text, static, CJK-capable) + a custom kanji index (18 §7) | No server, no third-party search service. |
| Images | Astro assets pipeline (sharp) → AVIF/WebP, responsive `srcset`; pixel art uses integer-scaled PNG/SVG | Build-time only. |
| Fonts | Self-hosted WOFF2, subset (08 §4) | No third-party requests. |
| Structured data types | `schema-dts` (types only, zero runtime) | JSON-LD checked at compile time. |
| Package manager / runtime | **pnpm**, **Node active LTS** pinned in `.nvmrc` and `engines` | Fast, strict dependency resolution. |
| Format / lint | Prettier (+ Astro plugin), ESLint (typescript-eslint, astro, jsx-a11y rules for `.astro`), Stylelint (standard config + order) | Objective and automatable. |
| Unit tests | Vitest | TS-native and fast. |
| Browser tests | Playwright + `@axe-core/playwright` | E2E, no-JS mode, a11y, visual, PDF generation, one tool. |
| HTML validation | `html-validate` | Semantic correctness of the output. |
| Lab performance | Lighthouse CI | Budget assertions. |

## 2. Rejected alternatives

| Option | Reason rejected |
|---|---|
| **Next.js / Remix** | A React runtime on every page, server-oriented, heavier than needed for static content. |
| **Analog (Angular meta-framework)** | Would show Charles's primary stack directly. However, Angular hydration ships a framework runtime to information pages, and the static content story is less mature. Rejected in **ADR-0001**: Angular would add runtime complexity to information pages without enough benefit. Angular, React or others MAY be used for isolated demos where appropriate. |
| **Eleventy** | Excellent and minimal, but weaker TypeScript and typed-content ergonomics. Agents benefit from typed schemas. |
| **SvelteKit** | Good, but adds a framework that has nothing to do with the positioning. |
| **Tailwind / CSS-in-JS / CSS frameworks** | Rejected (ADR-0013). Reconsidering requires a future ADR with a concrete reason. |
| **Headless CMS** | Content must be maintainable in Git. A CMS adds runtime dependencies and secrets. |

## 3. Dependency policy

| ID | Rule |
|---|---|
| DEP-01 | **Production runtime JS dependencies: zero** at launch, apart from what Astro compiles away. Pagefind's UI is loaded lazily from the build output. |
| DEP-02 | Every new dependency (dev or prod) requires a one-line justification in `docs/dependencies.md`. A new *runtime* dependency or island requires an ADR. |
| DEP-03 | CI fails if `package.json` gains a dependency that is not listed in `docs/dependencies.md`. |
| DEP-04 | Renovate (or Dependabot) groups updates weekly. Majors are manual. |
| DEP-05 | Prefer platform features (`<details>`, `<dialog>`, View Transitions, `popover`, CSS scroll-driven animations) over libraries. |
| DEP-06 | Remark/rehype plugins that the project writes itself (ruby syntax, heading anchors, Markdown serialization) live in `src/lib/markdown/` with tests. Third-party plugins must be justified under DEP-02. |

## 4. Expected dependency list (for approval, not installed)

- **Runtime (build-time only):** `astro`, `@astrojs/mdx`, `@astrojs/sitemap` (or custom, since it is simple), `sharp`, `pagefind`
- **Types:** `typescript`, `schema-dts`
- **Tooling:** `prettier`, `prettier-plugin-astro`, `eslint` + `typescript-eslint` + `eslint-plugin-astro` + `eslint-plugin-jsx-a11y`, `stylelint` + `stylelint-config-standard`
- **Testing:** `vitest`, `@playwright/test`, `@axe-core/playwright`, `html-validate`, `@lhci/cli`
- **Fonts:** `subfont` or `glyphhanger`-style subsetting through a script (OD-15)

## 5. CSS architecture (ADR-0013)

```css
@layer reset, tokens, base, layout, components, identities, utilities, overrides;
```

| ID | Convention |
|---|---|
| CSS-01 | A single layer-order declaration, loaded first. Every rule lives in a named layer. |
| CSS-02 | Colours, spacing, type sizes, radii, shadows and textures come from custom-property tokens (`src/styles/tokens.css`). Raw colour literals outside the `tokens` layer fail lint. |
| CSS-03 | Logical properties only (`margin-inline`, `inset-block-start`, …). Physical properties fail lint unless there is a comment explaining a physical-only need. |
| CSS-04 | Layout uses Grid (page and portal structure) and Flexbox (one-dimensional). Components that appear in more than one column use **container queries**, and the page shell uses media queries. |
| CSS-05 | Modern selectors (`:has()`, `:is()`, `:where()`, `:focus-visible`) are used deliberately. `:where()` keeps base specificity at zero. ID selectors are not allowed. |
| CSS-06 | Progressive enhancement: newer features (View Transitions, scroll-driven animations, `text-spacing-trim`, anchor positioning, `@starting-style`) are wrapped in `@supports` or degrade harmlessly. The baseline must be correct without them. |
| CSS-07 | `!important` only in the `overrides` layer. No vendor-prefixed properties except where autoprefixing isn't used and the feature needs one (documented). |
| CSS-08 | Section identities (`data-identity`) may change only tokens and components in the `identities` layer (ADR-0007). |
| CSS-09 | Motion rules sit behind `@media (prefers-reduced-motion: no-preference)` (A11Y-07). Forced-colors adjustments sit in `@media (forced-colors: active)` (A11Y-08). |

Native CSS nesting is allowed. No preprocessor. Lightning CSS or Vite's default pipeline may minify output. Browser support target: the last 2 versions of evergreen browsers, plus Safari iOS, with baseline "widely available" features required for correctness.
