# 19 — Agent-Driven Development

This repository will be developed by **24/7 autonomous Claude Code workers running on Charles's bubbles-server**, all acting through one dedicated, least-privilege GitHub App (ADR-0012, [21-agent-github-app.md](21-agent-github-app.md)). It is designed so that agents can implement, test, review and open PRs on their own, while **CI judges their work objectively** and **humans keep control of merging and of every sensitive decision** (ADR-0011).

## 1. Agent-facing documents

| File | Purpose |
|---|---|
| `CLAUDE.md` (exists now: points to this document and the precedence rule) and `AGENTS.md` (created at scaffold) | Setup and commands (`pnpm verify`, `verify:fast`, `gate <id>`), layer rules (04 §3), forbidden actions, definition of done, where the spec is |
| `CONTEXT.md` | Stable domain context and vocabulary |
| `docs/spec/` | This specification, the source of requirement IDs |
| `docs/adr/` | Accepted decisions. Agents MUST read the relevant ADRs and flag conflicts (per `docs/agents/domain.md`) |
| `docs/dependencies.md` | Approved dependencies (DEP-03) |
| `content/README.md` | Content authoring guide, allowed MDX components, disclosure and provenance rules |

## 2. Permissions

| Agents MAY | Agents MUST NOT |
|---|---|
| Claim unclaimed `ready-for-agent` issues (claim protocol, §5) | **Merge any PR, including their own** |
| Create and push `agent/<worker-id>/**` branches (rulesets block every other branch, 21 §4) | Push to `main`, force-push to branches they didn't create, or bypass branch protection |
| Implement, run tests, run `pnpm verify` | Approve PRs in a way that counts toward merge requirements |
| Open and update PRs, respond to review comments | Push `.github/workflows/` changes (the App lacks the permission; see §3.1) |
| Review other agents' PRs (comment-only reviews) | Invent personal facts, metrics, dates, employers, certifications, article metadata, quotations or URLs (CONTENT-15) |
| Comment on issues, add `needs-info` when blocked | Add artwork that is not marked as a placeholder (ADR-0008) or copy copyrighted text |
| Create follow-up issues labelled `needs-triage` | Close issues (they close only through a merged PR's `Closes #n`), or remove `needs-human:*` labels |
| Add new gates, add tests, tighten threshold files (§3.1). Propose edits to active gates' definitions for approval | **Make CI pass by weakening, disabling, skipping or removing a gate** (06 §1.2). Fix the code under test, or stop with `needs-info` |

## 3. Human approval is required for

**Every merge** needs Charles's approval. In addition, the labeller workflow flags the categories below so the reason for review is visible. **This table is the single path→label map.** Other documents refer to it and do not repeat it.

| Category | Label | Triggered by |
|---|---|---|
| Protected architecture decisions | `needs-human:architecture` | `docs/adr/`, `docs/spec/`, `CONTEXT.md`, `CLAUDE.md`, `AGENTS.md` |
| Significant dependencies | `needs-human:dependency` | Changes to `package.json` `dependencies`/`devDependencies` that add packages not yet in `docs/dependencies.md`, `pnpm.onlyBuiltDependencies` (packages running install scripts or native builds), `src/islands/**` (new islands), third-party origins in `config/security-headers.ts`, `third-party/**` (DEP-02/03, LIC-04) |
| New personal claims | `needs-human:claims` | `content/profile.yaml`, `content/experience/`, `content/achievements/`, `content/media/`, `content/timeline/`, Claims or KankenAttempts anywhere (CONTENT-13, report G21) |
| Confidential or client-sensitive info | `needs-human:claims` | `client.*` fields, case studies with a named client (CONTENT-14, ADR-0006) |
| Protected CI and security infrastructure | `needs-human:security` | `.github/`, `ci/proposed/`, `scripts/ci/`, CODEOWNERS, `config/security-headers.ts`, the denylist, secret-scanning config |
| Gate weakening | `gate-integrity` **fails** until a code owner applies `gate-change-approved` after the latest push | Signals W1–W6 (06 §1.2). Activating a new gate, adding tests and tightening threshold files are *not* in this category. Editing an active gate's non-threshold, non-allow-list definition files is |
| Owner policy | `needs-human:policy` | `config/crawlers.yaml` (`trainingPolicy` is Charles's decision), `LICENSE`, `LICENSES/`, `REUSE.toml`, `LICENSING.md` |
| Visual baselines | `needs-human:visual` | `tests/**/__screenshots__/` |

The labeller runs through `pull_request_target` with **only base-branch code**, re-applies labels on every push, and holds the only write-scoped token used for labelling. Gate jobs stay read-only. Labels explain *why* review is needed. The enforcement is the required human approval (ADR-0011) plus `gate-integrity`.

### 3.1 Who writes CI workflows (ADR-0017)

Workflows are **thin, stable and human-owned**. A `plan` job reads `ci/gates.json` and fans out a **dynamic matrix** of gate jobs, followed by the aggregate `verify` check, deploy, and post-deploy gates. Gates are **scripts registered in the manifest**, so almost all CI work is agent-doable without touching workflow YAML:

- A ticket that implements a gate is an agent ticket. The agent writes the script, sets the manifest entry to `status: active` (or adds a new entry) and proves it with `pnpm gate <id>` and `pnpm verify`. The matrix picks it up with no YAML change. Activating a new gate, adding tests and tightening threshold files don't trip `gate-integrity`. Editing an already active gate's non-threshold, non-allow-list definition files or reachable scripts does (W3/W6), even to strengthen it, so such a PR needs Charles's `gate-change-approved` label.
- A ticket that changes an existing active gate's command or thresholds in a *loosening* direction is a human decision. `gate-integrity` fails until Charles approves (06 §1.2).
- Scheduled automation only opens or updates issues. It never commits and never opens PRs. PRs created with the default `GITHUB_TOKEN` don't trigger workflows, and this is **not** worked around with a privileged App token in scheduled workflows (ADR-0017).
- A ticket that truly needs workflow YAML (a new trigger, job, permission or secret) is split into an agent part and a `ready-for-human` part. The agent writes the proposed YAML to `ci/proposed/<name>.yml`, and `actionlint` validates it inside `pnpm verify`. Charles promotes it to `.github/workflows/` in a commit on the same PR (Charles can push to `agent/**` branches) and merges. After a promotion commit, the agent must not rebase or force-push that branch. Charles rebases if needed. The issue's "Human approval required" field names this step.
- The initial workflows (`ci.yml` with plan / matrix / `verify` / deploy-preview / post-deploy, `deploy-production.yml`, `scheduled.yml`, `labeller.yml`, `gate-integrity.yml`) are drafted by an agent in `ci/proposed/` and promoted by Charles **in the bootstrap PR**, the single promotion point (06 §1.3).

## 4. Issue format (every implementation issue)

Issues are written so that one worker can execute them **independently**, without reading other issues:

```markdown
## Objective
One or two sentences: the outcome, not the activity.

## Context
Why this matters and what exists already. Links to spec sections and ADRs.

## Dependencies
Blocked by: #n, #m (native GitHub dependencies, per docs/agents/issue-tracker.md). "None" if independent.

## Requirements
- [ ] Concrete, checkable requirements, each tagged with spec IDs (e.g. SEO-03, CONTENT-12).

## Non-goals
- Explicitly out of scope for this issue.

## Acceptance criteria
- Observable, objective conditions (for example "G9 passes with new assertions for X", or "`/timeline` renders the empty state in production mode").

## Validation
    pnpm verify
    pnpm test -- src/domain/timeline      # targeted commands where useful

## Likely files / areas
- src/domain/…, src/pages/…, tests/…

## Human approval required
No | Yes: <which category from 19 §3 and why>

## Spec references
docs/spec/17-timeline.md §2, §6 · ADR-0005 · ADR-0008
```

Labels: `ready-for-agent` (fully specified, no human-only decisions inside) or `ready-for-human` (design approval, content confirmation, legal, credentials). Issues that need human input partway through are split so that the human-only part is its own issue.

## 5. Worker protocol

1. **Pick:** use the frontier query from `docs/agents/issue-tracker.md`, adapted: open `ready-for-agent`, no open blockers, **no `agent:claimed` label, and no open PR whose body contains `Closes #<n>`**. The first in map order wins.
2. **Issue claim** (the portfolio App is not configured as an assignable agent app, so it can't be an issue assignee, and all workers share one identity). This must stay correct with any number of workers acting at the same time.
   - **Identities.** A `worker-id` is a logical worker slot (e.g. `bubbles-01`) managed by the bubbles-server **supervisor**, a single process holding a host-wide lock. A `run-id` is a UUIDv4 **minted by the supervisor** each time it starts a worker process. It is written to the supervisor's local run registry **before** the worker process is spawned or any marker naming it is posted (write-ahead: run → slot, issue, state; the PID is still empty). **Startup barrier (D7):** the supervisor then spawns the worker blocked on a barrier, records the operating-system PID and that process's start time in the registry, releases the barrier, and finally marks the row `released`. A worker takes no action before release. The barrier and recovery meet these minimum requirements:
     - **Barrier teardown exits the child.** The barrier is a pipe whose only write end is held by the supervisor (it is close-on-exec, and the child closes its inherited copy before blocking). The child reads one byte to proceed. If it reads end-of-file instead, because the supervisor died before releasing, it exits immediately without acting. So a crash before release never leaves a blocked child behind, whether or not its PID was recorded.
     - **Safe process identity.** Each run is spawned into its own cgroup (a systemd transient scope) named after its run-id, created before the spawn and recorded in the registry row. The supervisor only ever signals a run through that cgroup (`cgroup.kill`) or through a pidfd whose process start time matches the registry. It never signals a bare PID, so PID reuse can't make it kill an unrelated process. A restarted supervisor is no longer the parent of surviving runs, so in this section "reap" means confirming exit: the run's cgroup reports no processes.
     - **Restart.** Before minting any run, the restarted supervisor reconciles every registry row whose run is not yet confirmed exited:
       - **Not marked `released`** (with or without a PID): it kills whatever is left in that run's cgroup and waits until exit is confirmed (a missing cgroup means nothing was spawned). If the run posted no marker, it is treated as never started. If it did post one (the crash fell between release and the `released` mark), its issue claim goes through supervisor recovery below like any exited run.
       - **Marked `released`, cgroup still has processes:** it **adopts** the run only if a pidfd opened on the recorded PID matches the recorded start time and that process is in the run's cgroup. It then monitors the run through that pidfd, and the run keeps its run-id and issue claim. Otherwise it kills the cgroup, confirms exit, and handles the run as exited.
       - **Marked `released`, cgroup empty or missing:** the run has exited, and its issue claim goes through supervisor recovery.

       A replacement run is minted for a slot only after every row is reconciled, and never while that slot has an adopted live run. So a restart never leaves two live runs in one slot.

     A run makes at most one issue claim per issue, so the run-id also identifies the claim. **A worker-id alone authorizes nothing.**
   - **Author filter.** The repository is public, so anyone can comment. A marker is honoured **only** if its comment was created by the portfolio App (`performed_via_github_app.id` equals the App ID, and the author is `<app-slug>[bot]` with `user.type == "Bot"`). **And** it has never been edited by anyone other than the App: the GraphQL `IssueComment.userContentEdits` must list only the App as editor. A marker edited by any other account is ignored, and the supervisor flags it for manual recovery. Readers ignore markers from any other author, including Charles's account. Humans act on claims only through the supervisor's CLI (manual recovery, below).
   - **Source of truth:** the honoured marker comments below. The `agent:claimed` label is only a shared hint that speeds up the frontier query. Winner selection never reads it.
   - **Markers:**

     | Marker | Posted by | Meaning |
     |---|---|---|
     | `<!-- agent-claim worker=W run=R -->` | run R itself | Starts claim R with rank = this comment's ID |
     | `<!-- agent-release worker=W run=R -->` | run R itself | Ends claim R |
     | `<!-- agent-release worker=W run=R reason=stale\|orphaned\|pr-merged\|pr-closed\|manual by=supervisor -->` | supervisor only | Ends claim R |
     | `<!-- agent-handoff worker=W from-run=A to-run=B by=supervisor -->` | supervisor only | Ends claim A **and** starts claim B, which inherits A's rank. Records the A→B relationship |

   - **Ending rule.** For each run, only the **first** honoured ending marker (release or handoff naming it as `from-run`), by lowest comment ID, takes effect. A later release or handoff for an already-ended run is **void and starts nothing**. So a handoff can never revive an ended claim, and never creates two successors.
   - **Active claims and winner.** A claim is active from its start marker (a claim, or a handoff that takes effect) until its effective ending marker. Readers never infer staleness from their own clock. **The winner is the active claim with the lowest rank.** Ranks come from unique comment IDs and pass only through handoffs that take effect, each of which ends its predecessor in the same comment. So, **given consistent reads after the settle period**, at most one run is winner. Re-verification before every push catches a stale read.
   - **Claim:** add `agent:claimed` (idempotent), then post the claim comment. The label is normally present whenever a claim exists; see Releasing for the exception.
   - **Settle, then decide:** after posting, wait at least 30 s, then re-read the comments before creating a branch. This lets concurrent claims with lower IDs become visible.
   - **Losing:** a run that isn't the winner posts a release for **its own** run only, and backs off. **A losing run never removes the `agent:claimed` label, and never posts markers for another run.**
   - **Re-verification:** before creating its branch, before opening the PR and before each push, the run re-reads the comments and continues only if its claim is still the winner. Otherwise it posts the release for its own run (if one isn't there already) and stops without pushing.
   - **Run lifecycle and review.** A run ends its process once its PR is marked ready for review. Its claim stays active and **parked**, so the frontier (step 1) doesn't hand the issue to another worker. When review feedback arrives, the supervisor starts a new run in the same slot and hands the parked claim to it (A→B). The claim ends through a marker in every case: when the `Closes #n` PR is merged (the supervisor posts `reason=pr-merged`), when the PR is closed unmerged (`reason=pr-closed`), when the worker is blocked (step 7), when the worker abandons the issue, or through supervisor recovery.
   - **Heartbeat:** while its process is running, the winner edits **the marker that started its claim** (its claim comment, or the handoff comment for a successor run) at least every 6 h, appending or replacing only the trailing `hb=<ISO timestamp>` field. A heartbeat edit never changes `worker`, `run`, `from-run` or `to-run`. It edits only after re-verification passes, so **an ended claim is never revived by a heartbeat**. Parked claims don't heartbeat.
   - **Releasing:** post the release for its own run. Then remove `agent:claimed` **only if no other active claim remains**, then re-read the comments and re-add the label if an active claim appeared. GitHub has no atomic compare-and-set, so a newcomer's claim can briefly exist without the label. The supervisor reconciles the label every 10 min (present if and only if an active claim exists). Until then, the only effect is that another worker may also claim and then lose.
   - **Restart handoff (same worker slot).** Scenario: run A of `bubbles-01` holds the winning claim, the process restarts, and new run B must continue. **B never releases A, and never becomes winner through its own claim comment.** B asks the supervisor. The supervisor checks four things:
     - A belongs to this slot and issue in its registry;
     - A's claim is **still active on GitHub** (it has no effective ending marker);
     - **A's process has exited** (or the supervisor kills it and reaps it first);
     - B is a run it has just minted and written to the registry for the same slot.

     Only then does it post the single handoff comment A→B. B inherits A's rank, continues on the same `agent/<worker-id>/…` branch, and re-verifies before its first push. Another worker slot can't use this path, because the supervisor only hands off within a slot. If the supervisor crashes mid-handoff, it reads GitHub on restart before acting: if a handoff from A already exists, that handoff is the effective one, and the supervisor adopts its `to-run` from the registry (or marks the claim for recovery) instead of minting another. By the ending rule, a second handoff from A would be void anyway.
   - **Supervisor recovery.** The supervisor acts on its local registry, and **always kills and reaps a live run before posting any release or handoff for it**:
     - An active, non-parked claim whose run has exited without a handoff, or whose heartbeat and branch pushes are both older than 24 h, gets a `reason=stale` release when there is no PR.
     - With a **draft** PR, it gets a handoff to a new run in the same slot (preferred), or a `reason=orphaned` release plus the `agent:orphaned-pr` label and a comment for Charles.
     - Parked claims (PR open and ready for review) are **exempt**, so a PR waiting for Charles is never treated as stalled.

     The frontier skips the issue while its `Closes #n` PR is open, so no duplicate implementation starts. To resume after an orphaned release, a run makes a **fresh claim** with a new rank, not a handoff from the ended claim. An open PR therefore never keeps a dead run's claim alive, and a parked claim is never recovered by mistake.
   - **Manual recovery.** Claims whose run is missing from the registry (lost registry, rebuilt host) are listed by the supervisor's reconciliation. Charles resolves them with the supervisor CLI (`supervisor release --issue <n> --run <R> --reason manual`), which posts through the App identity so the author filter honours it.
   - **Trust boundary (a constraint, not a guarantee).** Outsiders are excluded by the author filter, but every honoured marker comes from the **same App identity**, so GitHub cannot tell a supervisor marker from a worker marker. The protocol protects against **races, crashes, restarts and bugs** among bubbles-server processes that follow it. It does **not** authenticate those processes against each other: a compromised worker process could forge any marker. That risk is bounded by the App's permissions (it can't merge or touch workflows, ADR-0012) and by Charles's review of every merge. Possible future hardening, not adopted: the supervisor signs its markers (Ed25519, public key committed to the repo) and workers ignore unsigned supervisor markers.
   - One issue per run, and one winner per issue.
3. **Branch:** `agent/<worker-id>/<issue-number>-<slug>` from the latest `main`. A worker pushes only to branches carrying its own `worker-id`.
4. **Implement:** keep it small. `pnpm verify:fast` in the inner loop, then `pnpm verify` before pushing.
5. **PR:** open as draft, then mark ready when `pnpm verify` passes locally. The body follows the PR template: `Closes #n`, requirement IDs, a validation log summary, snapshot or visual changes explained, and human-approval categories touched.
6. **CI red:** fix the code under test, **never the gate** (no skips, suppressions, loosened thresholds or deactivation, 06 §1.2). After 3 failed attempts on the same gate, stop, comment with a diagnosis and label `needs-info`. If the gate itself looks wrong, say so in the comment. Changing it is Charles's call.
7. **Blocked by ambiguity or a human-only decision:** stop, comment with the specific question, label `needs-info`, and post a release comment. If the worker has an open PR, it converts the PR to draft and links the question. Don't guess.
8. **Review feedback:** address it on the same branch. Never rewrite history on a PR that is under human review, except to rebase on `main` when asked, and never at all once the branch contains a promotion commit from Charles (then Charles rebases).
9. **Done** means the PR is green and waiting for human review. The issue closes only when Charles merges.

## 6. Definition of done (every PR)

1. `pnpm verify` passes locally, and all required CI gates pass.
2. The PR lists the requirement IDs it implements or affects, and explains any snapshot or visual updates.
3. New behaviour has tests at the lowest effective level (domain unit > contract over `dist/` > Playwright).
4. No new dependencies without DEP-02/03 compliance.
5. Docs are updated if behaviour or rules changed. ADR conflicts are flagged, never silently overridden.
6. No placeholders are referenced by published content in production mode (G22).

## 7. Why the gates make autonomy safe

| Risk from agent work | Caught by |
|---|---|
| Broken types or schema | G3, G6 |
| Layer violations | G2 boundary rules |
| Leaked secrets | G0 + GitHub push protection |
| Confidential or client-sensitive leak | G7 (CONTENT-02/03), G21, human review |
| Unconfirmed personal claims | Labeller (`needs-human:claims`) + G21 report + required human approval |
| Placeholder art or content reaching production | G22 (CONTENT-12) |
| Accessibility regressions | G2, G8, G12 |
| Performance regressions, JS creep | G15, G16 |
| SEO or agent-contract regressions | G9, G10 |
| Visual regressions | G14 (human-approved updates) |
| Broken URLs | G7 (IA-06), G17 |
| Unjustified dependencies | G4 + `needs-human:dependency` |
| Self-merge | Branch protection: code-owner approval required, the GitHub App has no bypass and no `workflows` permission (ADR-0012, 21 §4) |
| Weakening a gate to get green | `gate-integrity` required check, run from base-branch code (ADR-0017, 06 §1.2) |
| Silently dropping a gate | `verify` fails if any active gate didn't run. Deactivation is a weakening signal |

## 8. Suggested decomposition order (not yet created as issues)

Repository governance (human, 21 §7: create the GitHub App, CODEOWNERS, rulesets with `protect-main` **without** required checks) → **pre-bootstrap PRs** (D1, in attended sessions per D8: scaffold, tooling, `ci/gates.json` with every gate `planned`, runners `pnpm gate`/`gates:plan`, `scripts/ci/**`, threshold and allow-list files; each reviewed and merged by Charles) → **bootstrap PR** (agent: draft `ci/proposed/*.yml`; human: promote workflows, merge with admin bypass) → human: add `verify`/`gate-integrity` as required checks (06 §1.3) → content schemas + domain layer (with provenance and placeholders) → shell layout + tokens (after style-tile approval) → home/hire/cv → representations (MD, JSON-LD, llms.txt, /data) → case studies (UA within approved facts) → timeline (model + placeholders) → kanji foundation (authored-only) → search → polish.
