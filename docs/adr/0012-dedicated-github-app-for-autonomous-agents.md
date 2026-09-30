---
status: accepted
date: 2026-09-28
---

# Autonomous agents act through a dedicated, least-privilege GitHub App

The bubbles-server workers authenticate as a **dedicated GitHub App** installed only on this repository, never with Charles's personal credentials. A separate identity is the only way to enforce "agents never merge". GitHub does not let a PR's author approve their own PR, so a shared identity would force Charles to bypass review. The App has read/write access to contents, issues and pull requests, plus read-only access to metadata, checks, statuses and Actions. It has **no** administration, workflows, secrets, environments, security or Pages permissions and is not on any bypass list. Charles's human account remains the only merge authority. Permissions, the authentication model and the token lifecycle are specified in `docs/spec/21-agent-github-app.md`.

## Considered Options

- **Charles's personal access token**: rejected. It makes self-merge possible and conflicts with required reviews.
- **Machine user with a fine-grained PAT**: workable, but uses a seat and a long-lived credential. App installation tokens are short-lived (1 h) and scoped per repository.

## Consequences

- Token permissions do not stop a merge on their own, because merging needs `contents: write`, which pushing also needs. **Rulesets are the enforcement:** `main` requires a code-owner (Charles) approval, stale approvals are dismissed on push, and only the repository admin role (Charles) can bypass. That bypass is needed because a single-maintainer repo can't self-approve. The App is never admin and never on a bypass list.
- Without the `workflows` permission, the App cannot push changes to `.github/workflows/`. Those changes are human-only by construction (see ADR-0017 for how CI work is split).
- The portfolio App is not configured as an assignable agent app, so it can't be an issue assignee. GitHub supports assigning only specially configured agent apps. And all workers share one identity. So issue claims use App-authored marker comments (claim, release, handoff) carrying a worker slot and run as the source of truth, with a label only as a hint (spec 19 §5).
- No personal GitHub credential of Charles's may exist on bubbles-server.
