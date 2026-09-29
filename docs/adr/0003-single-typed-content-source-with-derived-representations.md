---
status: accepted
date: 2026-09-28
---

# One typed content source, every representation derived through a domain layer

All content lives in Git as schema-validated YAML/MDX. A pure TypeScript domain layer is the **only** code allowed to read content collections. Every output (HTML pages, Markdown alternates, JSON-LD, `/data` JSON, `llms.txt`, feeds, JSON Resume, the CV PDF) is a pure function of domain objects. This makes the human and machine versions of a fact consistent by construction, gives agents one small, fully tested place for "what is true", and lets CI prove parity between representations.

## Considered Options

- **Headless CMS**: rejected. Content must be maintainable in Git, and a CMS adds runtime, secrets and a second source of truth.
- **Pages reading collections directly** (the framework default): rejected. Derivations (years of experience, back-links, provenance) would be duplicated across templates and representations would drift.

## Consequences

- A lint rule forbids importing the content API outside the domain layer.
- The CV PDF is printed from the `/cv` page. There is no separate template.
