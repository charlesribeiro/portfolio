---
status: accepted
date: 2026-10-06
---

# Untrusted agent runs hold no credentials and reach the world only through trusted brokers

An autonomous worker run executes text that outsiders can influence (issue, PR and review comments reach its prompts) and repository code it didn't write (dependencies, install scripts, tests). So the run is treated as **untrusted**, towards the outside world and towards other runs. It holds **no GitHub credential and no reusable model-provider credential**, it has **no direct network access**, and each run is isolated from every other run. Everything it needs from outside comes from trusted components that run under their own Unix users and check every request:

- the **publication broker**, inside the supervisor, performs every GitHub write and every issue-claim marker for the run, and supplies git data as bundles;
- the **model proxy** holds the provider key and serves the run through a per-run capability with limits;
- the **package proxy** serves read-only package retrieval from a fixed upstream.

These boundaries are enforced by Unix users, systemd units and kernel namespaces, not by prompts or tool settings. The requirements (AGENT-02, AGENT-05 to AGENT-08, AGENT-10 to AGENT-14) are in `docs/spec/22-agent-run-isolation.md`. AGENT-01 and AGENT-04 are in spec 21, and AGENT-03 and AGENT-09 in spec 19 §5. This ADR adds no GitHub App permission (ADR-0012). The rules incorporate isolation and crash-consistency findings from the security review of the bubbles-agent implementation design.

## Considered Options

- **Give the coding process an installation token (spec 21 as first written)**: rejected. Any repository code it runs inherits the token and can push, comment or forge issue-claim markers.
- **Give the run a raw provider key, hidden from tool subprocesses by a sandbox scrub**: rejected. The key would still be in the untrusted process, and the enforcement would live inside it.
- **One Unix uid shared by all runs**: rejected. The host allows same-uid tracing and has no hidden `/proc`, so one run could read another's capability, inject into it and act as it.
- **One static uid per worker slot**: workable, but successive runs in a slot inherit leftovers unless a privileged cleaner wipes them, and it doesn't isolate the network.
- **Per-run containers**: workable, but heavier to maintain than systemd-native units for the same boundaries.
- **A tunnelling proxy with a hostname allowlist (GitHub, npm)**: rejected. Untrusted code can bring its own GitHub or npm token and write data out through an allowed host.

## Consequences

- A fourth system user, `agent-llm`, holds the provider key. Runs use per-run dynamic uids of the class `agent-run`. Spec 21 §5 and §7 list the users and the negative test.
- Issue-claim markers keep their GitHub-visible form, but trusted code posts them after checking the run's registry state. A compromised run can no longer forge another run's marker (19 §5).
- Each run needs its repository through broker bundles and its packages through the package proxy. Validation that needs any other network path fails closed and becomes a question for Charles.
- Crash recovery only ever moves forward. After a crash, local state may temporarily be more restrictive than GitHub state, but never more permissive (AGENT-09).
- Accepted residual risks, recorded in spec 22 §4: a limited covert channel through the choice and timing of public package requests (accepted on condition that the listed mitigations stay in force), prompts reaching the model provider, kernel exploits and side channels, and root on the host.
