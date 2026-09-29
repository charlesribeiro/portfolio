---
status: accepted
date: 2026-09-28
---

# Zero JavaScript by default; islands and runtime dependencies need an ADR

Information pages (every route except `/lab/**`, ADR-0016) **require** no JavaScript: they are fully functional with JS disabled. What little JS ships is accounted for exactly as defined in ADR-0016. Interactivity is progressive enhancement through a small, listed set of vanilla-TypeScript custom-element islands (site search, kanji search, theme toggle, copy button), each with a no-JS baseline and a byte budget enforced by CI. Adding an island, a runtime dependency or a third-party origin requires an ADR. Every dependency, including dev dependencies, must be justified in `docs/dependencies.md`, and CI rejects unlisted ones. This keeps Core Web Vitals and security posture deterministic when many autonomous agents contribute, because agents naturally reach for libraries.

## Consequences

- No UI framework runtime (React, Angular, Preact) on information pages. Isolated demos follow ADR-0001 and live under `/lab/**` (ADR-0016).
- Platform features (`<details>`, `<dialog>`, popover, View Transitions, scroll-driven animations) are preferred over libraries.
