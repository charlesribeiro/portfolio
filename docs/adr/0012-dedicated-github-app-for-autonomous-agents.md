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
- The portfolio App is not configured as an assignable agent app, so it can't be an issue assignee. GitHub supports assigning only specially configured agent apps. And all workers share one identity. So issue claims use App-authored marker comments (claim, release, handoff) carrying a worker slot and run as the source of truth, with a label only as a hint (spec 19 §5). Since the 2026-10-06 amendment, trusted supervisor code posts every such marker after checking the run's registry state (AGENT-03); worker runs only request them.
- **Charles's personal GitHub credentials never reach the autonomous worker path** (amended 2026-10-03, decided by Charles during H-002 planning; this replaces "no personal GitHub credential of Charles's may exist on bubbles-server"). Charles's attended sessions may keep running on bubbles-server under his own account. Autonomous workers and the token helper run as dedicated, unprivileged Unix users, never as `charles` and outside `sudo`, `docker` and every other privileged group, in systemd services hardened with `ProtectHome=yes` and `NoNewPrivileges=yes`. Only the token helper and the publication broker ever hold this repository's App identity. Worker runs hold no GitHub credential at all, and each brokered operation uses its own short-lived, down-scoped installation token that is revoked when the operation ends (amended 2026-10-06, decided by Charles; this replaces "Workers hold only this repository's App identity, as short-lived installation tokens". AGENT-01 and AGENT-04 in spec 21, ADR-0018). The App key and those tokens are readable only on the agent path, never by worker runs, Charles's ordinary processes or unrelated services. A root compromise of the host, which includes a compromise of `charles` (a member of `sudo` and `docker`), exposes both identities. That is an accepted residual risk. The requirements and the negative isolation test are in spec 21 §5 and §7.
- **Worker runs are untrusted** (added 2026-10-06, decided by Charles). Each run is isolated from every other run, holds no GitHub or model-provider credential and has no direct network access. It reaches GitHub only through the publication broker. This adds no App permission. The decision and its requirements are ADR-0018 and spec 22.
