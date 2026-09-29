---
status: accepted
date: 2026-09-28
---

# Hand-written modern CSS, no CSS framework

Styling is hand-written CSS that uses the platform directly: custom properties as design tokens, cascade layers for ordering, Grid and Flexbox for layout, container queries where components live in different columns, logical properties, modern selectors (`:has()`, `:is()`, `:where()`) and progressive enhancement through `@supports`. There is no Tailwind, CSS-in-JS or component CSS framework. The Photoshop-era chrome (ADR-0007) is bespoke, so a utility framework would add tooling without reducing work. The visual implementation is itself meant to be evidence of advanced CSS engineering. Charles's Tailwind expertise is shown through the Tailwind CSS + AI publication instead.

## Consequences

- Introducing any CSS framework, preprocessor or CSS-in-JS needs a future ADR with a concrete reason.
- Stylelint enforces the conventions (layer order, logical properties, token usage, no `!important` outside the `overrides` layer), so agent-written CSS stays consistent.
