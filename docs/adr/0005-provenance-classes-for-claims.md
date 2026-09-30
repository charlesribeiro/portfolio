---
status: accepted
date: 2026-09-28
---

# Every claim carries one of four provenance classes

Recruiting and research agents need to know *why* a fact about Charles should be believed, not just what it is. Every Claim, and every claim-bearing field in machine-readable output, is classified as **verified** (inspectable evidence plus Owner Confirmation), **self-reported** (Charles's statement with no public evidence), **derived** (computed by a documented rule from other facts, such as years of experience) or **external-source** (asserted by a third party such as a news article or publisher, and attributed rather than restated). The full classification is published in `/data/*.json` and in Markdown front-matter. JSON-LD expresses evidence relationships (`citation`, `subjectOf`, `recognizedBy`) using standard Schema.org properties only, because Schema.org has no provenance vocabulary and we will not invent properties.

## Considered Options

- **Binary evidence / self-reported flag**: too coarse. Computed and third-party facts are different kinds of trust.
- **Custom JSON-LD vocabulary or `Claim` nodes for everything**: poorly understood by consumers and noisy.

## Consequences

- `verified` can only be set in a PR approved by Charles. Agents may propose it, never finalize it.
- A numeric outcome is published only with evidence, or when its text is listed in the entity's `client.approvedFacts` (Approved Facts). Otherwise it is worded qualitatively. A `self-reported` label alone does not permit a number (amended 2026-09-30, decision D2; the rule is CONTENT-05).
