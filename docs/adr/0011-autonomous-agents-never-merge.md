---
status: accepted
date: 2026-09-28
---

# Autonomous agents implement and propose; only Charles merges

The repository will be developed around the clock by autonomous Claude Code workers on Charles's bubbles-server. Agents may claim issues, implement, test, review (comment-only), push branches and create or update pull requests. **Agents never merge, including their own PRs.** Charles's approval is required for merging, for changes to protected architecture decisions (ADRs, spec, `CONTEXT.md`), for dependencies with significant architectural or security implications, for publishing new personal claims, for publishing confidential or client-sensitive information, and for changes to CI and security infrastructure. This is enforced technically, not by convention: branch protection requires code-owner approval, CI labels PRs by human-approval category, and agents act under a dedicated GitHub App identity with no merge or bypass rights (ADR-0012, `docs/spec/21-agent-github-app.md`).

Every implementation issue is written to be executed independently. It states objective, context, dependencies, requirements, non-goals, acceptance criteria, validation commands, likely files, whether human approval is required, and the spec requirement IDs it covers.

## Consequences

- Throughput is limited by human review. Issues are sized so that reviews stay small.
- Agents stop and ask (`needs-info`) instead of guessing when an issue touches a human-only decision.
