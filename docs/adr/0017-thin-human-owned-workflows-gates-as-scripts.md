---
status: accepted
date: 2026-09-28
---

# Thin, human-owned CI workflows run a dynamic gate matrix from `ci/gates.json`

The agents' GitHub App cannot push workflow files (ADR-0012), and CI and security infrastructure need Charles's approval (ADR-0011). Yet most tickets add or change quality gates. So workflow YAML is kept **thin, stable and human-owned**, and gate *checks* live in repository scripts described by a manifest, `ci/gates.json`. The workflow fans out a **dynamic matrix**, one parallel job per active gate for its phase. A single **stable aggregate check, `verify`**, is required, and it passes only if every required gate ran and succeeded. Adding or strengthening a gate needs no YAML change and is ordinary agent work. The mechanics are normative in spec 06 §1.

## Manifest

Each entry has `id`, `command`, `phase` (`pre-deploy` | `post-deploy` | `scheduled`) and `status` (`planned` | `active`). An **active** entry must also list `definitionPaths`: the config, threshold, allow-list and runner files that define how strict the gate is. Test files are not listed, because W5 covers them. The workflow's `plan` step and `gate-integrity` both fail on an active entry without it. A missing base manifest (bootstrap) counts as the empty set.

## Who judges

- **Enumeration, execution and aggregation are inline in the human-owned workflow YAML** (`jq` over the manifests). A matrix job runs the manifest `command` string directly, never through a repository wrapper script. `pnpm gate <id>` is only a local convenience that reads the same manifest.
- **Required set** = ids active in the base manifest ∪ ids active in the PR manifest. Each id runs its **PR-manifest entry** when it exists there and is active. Otherwise it runs its **base entry**. Deactivating or deleting a gate therefore still requires it to pass. A change Charles has approved that removes a gate is merged by Charles **with the admin bypass**, and `verify` stays red on such a PR by design.
- The `ci.yml` copy on a PR branch is trustworthy only because the App cannot edit workflows. **`gate-integrity`** and the labeller go further: they run through `pull_request_target` with **only base-branch code**, and read PR files as data.

## Gate integrity

`gate-integrity` fails on these **weakening signals** unless a code owner applied the label `gate-change-approved` after the PR's latest push. "Base-active" means active in the base manifest.

- **W1:** a base-active gate removed, set to `planned`, its `phase` or `command` changed, or a path removed from its `definitionPaths`.
- **W2:** a value loosened, according to its declared `direction`, in a **threshold file**. The closed list is `config/budgets.json` and `config/thresholds.json`.
- **W3:** any change to a file in the `definitionPaths` (base ∪ PR) of a base-active gate, other than the threshold files (`config/budgets.json`, `config/thresholds.json`) and allow-lists (`config/allowlists/*.json`), for example `tsconfig.json`, `.lighthouserc.json`, `tests/a11y/axe.config.ts` or `eslint.config.js`.
- **W4:** a new entry in an allow-list file that is already in a base-active gate's `definitionPaths`. Allow-lists shipped by a gate's activation PR don't trip it, and Charles reviews them in that PR. Removing an entry is strengthening. `docs/dependencies.md` is not an allow-list (DEP-02/03 govern it).
- **W5:** added suppressions (`.skip`, `.only`, `fixme`, `eslint-disable`, `stylelint-disable`, `@ts-ignore`, `@ts-expect-error`), deleted or renamed test files, or deleted snapshot baselines.
- **W6:** a change to any `package.json` script reachable from a base-active gate's `command`, or to `scripts/ci/**`.

**Not weakening:** activating a new gate (adding its scripts and config files), adding tests, tightening a threshold file, and removing allow-list entries. These do not trip `gate-integrity`. Editing the **non-threshold, non-allow-list** definition files of an *already active* gate always trips W3, even to strengthen it, because strictness can't be judged objectively for arbitrary files. Charles approves those with the label.

Agents must never make CI pass by weakening, disabling or removing a gate. A failing gate is fixed in the code under test, or escalated with `needs-info`. Weakening that no machine can see (for example gutting assertions inside an existing test) is a residual risk, covered by Charles's review of every merge.

## Secrets and scheduled automation

- Build and gate jobs on `pull_request` run repository code with **no secrets**. Deploy jobs run **no repository code**.
- Scheduled jobs (G0h history scan, external links, production Lighthouse, agent-readability eval) **open or update issues**. They never commit to `main` and never open PRs. PRs created with the default `GITHUB_TOKEN` don't trigger other workflows, and that limitation is accepted. **No privileged App token is added to scheduled workflows to bypass it.**
- **The one exception to "no secrets in gate jobs"** is the scheduled agent-readability eval (G19). It needs an LLM API key, which is not a GitHub credential. The key lives in a dedicated `agent-eval` environment restricted to `main`, and the job runs only on `schedule` or `workflow_dispatch` from `main`, never on `pull_request`.

## Bootstrap

1. **Governance** (human): create the App, CODEOWNERS, the `agent-branch-namespace` and `protect-tags` rulesets, and `protect-main` **without required status checks** (PR + code-owner review only).
2. **Bootstrap PR**, the single promotion point for the initial workflows. It contains the scaffold, workflows drafted in `ci/proposed/` and promoted by Charles, `ci/gates.json` with every gate `planned`, the `pnpm gate` / `gates:plan` runners, `scripts/ci/**` and the threshold and allow-list files. Charles merges it with the admin bypass. With zero active gates, the matrix job is skipped by an `if:` guard and `verify` passes vacuously.
3. Charles then adds `verify` and `gate-integrity` as required checks on `protect-main`.

## Considered Options

- **Give the App the `workflows` permission**: rejected. A PR branch's workflow runs with that branch's definition and can reach secrets and write-scoped tokens. This is the main exfiltration path.
- **One hand-written workflow job per gate**: fine-grained status, but every new gate would need a human-only YAML change.
- **A single serial verify job in CI**: no YAML per gate, but slow (two builds, E2E, axe in three colour modes, visual regression and Lighthouse back to back) and expensive with around-the-clock agents.
- **Enumerate or run gates through a repository script** (for example `pnpm gate`): rejected. A PR could rewrite the script to neutralise every gate.
- **Protect secrets instead of workflows** (Environments with required reviewers, and the App allowed to edit workflows): every preview deploy would need a manual approval click, and the default-token scope and cache-poisoning risks remain.
- **Reusable workflows or composite actions in a separate human-only repository**: stronger isolation, but more than a single personal repository needs.

## Consequences

- Rulesets require exactly `verify` and `gate-integrity`, whose names never change.
- PRs from forks cannot pass `verify`, because post-deploy gates need a preview deploy and fork PRs get none. External PRs are not a request surface (`docs/agents/issue-tracker.md`).
- After Charles pushes a promotion commit to an agent branch, agents must not rebase or force-push that branch. Charles rebases if needed.
- Nothing is committed to `main` by CI.
