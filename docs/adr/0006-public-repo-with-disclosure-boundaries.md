---
status: accepted
date: 2026-09-28
---

# Public repository with enforced disclosure boundaries

The repository is public, because the code and CI are part of the evidence of engineering quality. So nothing confidential may ever be committed, not even in history. Client engagements have a disclosure level (public / anonymized / private) and an explicit list of **Approved Facts**. When the public status of a fact is uncertain, the default is to anonymize or leave it out, and to ask for human approval.

For **United Airlines**: Charles worked as a **contractor**, not a United employee. United may be named as the client and project context. The Pilot Bidding System may be described at a high level, including that it served approximately 17,000 pilots and that the work covered frontend modernization, Angular upgrades, architecture and maintainability. The site never publishes proprietary code, internal URLs, credentials, non-public architecture, confidential business rules, private screenshots, names of internal employees (unless approved later), or information whose public status is uncertain. Structured data never states that Charles `worksFor` United Airlines.

## Consequences

- Secret scanning (gitleaks plus GitHub push protection) is a required gate.
- Confidential terms are checked against a **hashed** denylist, so the list isn't readable in plain text. Its salt is public (gate jobs have no secrets), so the hashes hide terms but don't make them secret: anyone can test a guessed name. The scheme (scrypt) and this residual risk are recorded in CONTENT-03 (amended 2026-09-30, decision D5).
- Content changes touching clients or personal claims require Charles's approval (ADR-0011).
