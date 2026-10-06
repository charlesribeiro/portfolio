# 21 — Agent Identity: the bubbles-server GitHub App

Decision: **ADR-0012**. This document specifies the App. **Do not create it yet.** Charles creates it by hand, as a `ready-for-human` task. How worker runs are isolated, and which trusted components act for them, is in spec 22 (ADR-0018).

## 1. Identities

| Identity | Who | Can | Cannot |
|---|---|---|---|
| **Charles** (personal account, 2FA/passkey) | Owner | Everything, and is the **only merge authority** and sole CODEOWNER | — |
| **`<portfolio-agents>[bot]`** (GitHub App installation) | The bubbles-server supervisor and its publication broker, acting for worker runs (AGENT-01) | Branches, commits, issues, PRs, read CI results | Merge `main`, administer the repo, change security settings, edit workflows, change the App |
| **GitHub Actions `GITHUB_TOKEN`** | CI | Per-job least privilege (10 §3) | Anything beyond the job's declared `permissions:` |

## 2. App registration settings

- Owner: Charles's personal account. Visibility: **private** (only installable by its owner).
- Installation: **only this repository** (`charlesribeiro/portfolio`), with "Only select repositories" chosen.
- Webhooks: **disabled**. Workers poll through the API, so no public endpoint is needed on bubbles-server.
- The user-authorization / OAuth flow is not used: no user-to-server tokens, and "Request user authorization" is off.
- Callback and setup URLs: none.

## 3. Repository permissions (exhaustive)

| Permission | Level | Why |
|---|---|---|
| Metadata | Read | Mandatory |
| Contents | **Read & write** | Clone, create `agent/*` branches, push commits |
| Issues | **Read & write** | Issue claim (label + comment, 19 §5), comment, label, create `needs-triage` follow-ups |
| Pull requests | **Read & write** | Open/update PRs, comment-only reviews, request reviewers |
| Checks | Read | Read gate results |
| Commit statuses | Read | Read gate results |
| Actions | Read | Read workflow logs to diagnose failing gates |
| **Workflows** | **None** | Without it, pushes that modify `.github/workflows/` are rejected, which makes CI infrastructure human-only by construction |
| **Administration** | **None** | No settings, rulesets, collaborators or deletions |
| Secrets, Variables, Environments | None | Cannot read or change deploy credentials |
| Security events, Secret scanning alerts, Dependabot alerts | None | Security posture is human-owned |
| Pages, Deployments | None | Deploys run in CI with their own credentials |
| Repository hooks, Custom properties, Codespaces, Projects | None | Not needed |
| Organization and Account permissions | None | Not needed |

Adding any permission is an ADR-0012 amendment that requires Charles's approval. When a permission change is requested, GitHub makes the owner accept it for the installation, which acts as a second human check.

## 4. Why permissions alone don't prevent merging (and what does)

Merging a PR needs `contents: write` + `pull_requests: write`, and the App has both (it needs them to push and to open PRs). **Merge prevention is therefore enforced by repository rulesets, not by the token.** These are configured by Charles in the governance ticket. **`protect-main` starts without required status checks.** `verify` and `gate-integrity` are added only after the bootstrap PR merges (06 §1.3):

| Ruleset | Target | Rules | Bypass list |
|---|---|---|---|
| `protect-main` | `main` | Require PR · required status checks **`verify`** and **`gate-integrity`** (06 §1.1–1.2, ADR-0017) · **require code-owner review** · dismiss stale approvals on push · require branches to be up to date before merging · block force pushes and deletions · linear history | **Repository admin role only (= Charles).** Charles's approval satisfies the review rule on agent-authored PRs. The admin bypass exists because a single-maintainer repo can't otherwise merge Charles's own PRs (GitHub doesn't allow self-approval). The App is never admin and never on the list |
| `agent-branch-namespace` | all branches except `agent/**`, `dependabot/**`, `renovate/**` | Restrict creations, updates and deletions | Repository admin role (Charles) only |
| `protect-tags` | all tags | Restrict creations and deletions | Repository admin role |

Result: the App can write only to `agent/**` branches (dependency bots keep their own namespaces, DEP-04). It cannot approve its own PRs, because a PR author can't approve and every worker shares the one App identity. It isn't a code owner and isn't on any bypass list, so it cannot merge. Stale-approval dismissal means any agent push after Charles's approval needs a fresh approval. CODEOWNERS is `* @charlesribeiro`, and CODEOWNERS changes themselves need Charles's review.

"Agents update only their own PRs" cannot be expressed as a GitHub permission, because all workers share one identity. It is enforced by the worker protocol (19 §5): each worker has a stable `worker-id`, branches are named `agent/<worker-id>/<issue>-<slug>`, and a worker touches only branches with its own `worker-id`. The publication broker enforces this for every write: it acts only on the requesting run's registry branch, issue and PR (AGENT-02). Most labels are **informational, not security controls**, since the App can edit labels. The one label with enforcement effect, `gate-change-approved`, is honoured by `gate-integrity` only when the issue-events API shows a code owner applied it. The real controls are the required human review and the fact that `needs-human:*` labels are re-applied on every push by a labeller that runs from `main` (19 §3).

## 5. Authentication model

```
agent-token helper (user `agents`)
  └─ reads App private key (PEM) ──▶ signs JWT (RS256, iat-60s, exp ≤ 10 min, iss = App ID)
        └─ POST /app/installations/{installation_id}/access_tokens
               body: { "repositories": ["portfolio"],
                       "permissions": { minimal set for this operation } }
        ◀─ installation access token (expires after 1 hour; revoked after one operation)
  └─ hands the token to the publication broker over a local socket
publication broker (user `agent-worker`, inside the supervisor)
  └─ uses the token for exactly one operation (a push, or an API call), then revokes it (AGENT-04)
worker run (untrusted, its own dynamic uid; spec 22)
  └─ holds no token; asks the broker for each GitHub operation over a Unix socket (AGENT-01, AGENT-02)
```

| ID | Requirement |
|---|---|
| AGENT-01 | **No GitHub credential in a worker run.** A worker run (the coding agent and everything it starts, spec 22) MUST NOT hold any GitHub credential: no installation token, JWT, App key, `gh` login, personal access token, SSH key or agent, or credential helper, and no `GH_TOKEN`, `GITHUB_TOKEN`, `GH_ENTERPRISE_TOKEN`, `GH_CONFIG_DIR`, `SSH_AUTH_SOCK` or `GIT_ASKPASS` in its environment. Git inside the run is configured so it can't authenticate to anything or push. Every GitHub write for the run, including every issue-claim marker, is performed by the publication broker (AGENT-02, AGENT-03). Only the token helper and the broker ever hold App credentials |

- **Private key storage:** on bubbles-server only, in a secrets store or a file owned by the dedicated `agents` system user with mode `0400`, inside a directory that only `agents` can enter. It is never in the repository, never in container images, never in environment files that are committed, and never readable by any process other than the token-minting helper. Worker runs can't reach the helper or receive its tokens (AGENT-01).
- **Token minting helper:** a tiny local service (`agent-token`) running as `agents` that holds the key and returns installation tokens. Its socket accepts only `agent-worker`, whose only client is the publication broker. No worker run sees the PEM or a token, so a compromised worker run holds no GitHub credential at all (AGENT-01).
- **Commit identity:** `<portfolio-agents>[bot] <{bot-user-id}+<portfolio-agents>[bot]@users.noreply.github.com>`. Commits carry `Co-Authored-By` trailers as configured. Signed commits are optional (a later hardening step).
- **Host isolation (ADR-0012).** Charles's attended sessions may run on bubbles-server under his own account. The autonomous path is isolated from it, and these rules are mandatory:
  - **Unix users.** The supervisor and its publication broker run as the dedicated user `agent-worker`, the helper as `agents`, and the model proxy as `agent-llm` (spec 22), never as `charles`. They are system users with no login shell and home directories outside `/home`, which `ProtectHome=yes` hides. None belongs to `sudo`, `docker` or any other privileged group, and `charles` belongs to none of their groups. Worker runs run under a dynamic uid of their own per run, of the class `agent-run` (AGENT-06), never as `agent-worker` or `charles` (amended 2026-10-06, ADR-0018).
  - **systemd hardening.** The supervisor, helper and model proxy services set `ProtectHome=yes` and `NoNewPrivileges=yes`. Worker runs are started by the supervisor as their own hardened service units and never inherit the supervisor's user (AGENT-06, spec 22). (Amended 2026-10-06, ADR-0018: this replaces "worker runs are spawned from the supervisor service and inherit its user, mount namespace and no-new-privileges flag".)
  - **Human credentials stay out.** Charles's GitHub credentials (`gh` login, PATs, SSH keys, SSH agent) are never copied, mounted, forwarded, exported or otherwise passed to the worker path. That includes bind mounts or volumes from `/home/charles`, SSH agent forwarding, and inherited environment such as `GH_TOKEN`, `GITHUB_TOKEN`, `GH_CONFIG_DIR` or `SSH_AUTH_SOCK`. The supervisor builds each run's environment from scratch. Worker runs receive no GitHub credential at all (AGENT-01); only the publication broker uses this repository's App identity.
  - **Agent credentials stay in.** The App key is readable only by `agents`. Installation tokens exist only in the helper and, for one operation, in the publication broker process and the git or HTTP child it starts (AGENT-04). Worker runs, Charles's ordinary processes and unrelated services can read neither. The model-provider key follows the same rule (AGENT-05).
  - **Residual risk.** Root on the host can read both identities. A compromise of `charles` counts as root, because `charles` is in `sudo` and `docker`. This risk is accepted. Damage from a stolen App identity is bounded by §3 and §4, and the response is the Incident row of §6. Damage from stolen human credentials is not bounded by this design.

## 6. Token lifecycle

| ID | Requirement |
|---|---|
| AGENT-04 | **One token per brokered operation.** The publication broker requests a new installation token just in time for each GitHub operation it performs, down-scoped to that operation's permissions. The token exists only in the broker process and in the git or HTTP child it starts for that operation, passed through the environment and never through command-line arguments. Straight after the operation, whether it succeeded or failed, the broker revokes it and confirms the revocation (a later request gets `401`). No token outlives its operation, and none reaches a worker run (AGENT-01) |

| Stage | Rule |
|---|---|
| Issue | One token per brokered operation (AGENT-04), requested just in time and down-scoped to that operation. This replaces "one token per job" (amended 2026-10-06, ADR-0018) |
| Use | In the broker process, or in the git or HTTP child it starts, through the environment only. Never written to disk, logs, PR bodies or commit messages, and never passed to a worker run. Logs pass through a redaction filter for `ghs_` tokens |
| Refresh | Not used: a token never outlives its operation. An operation that could outlast the token's validity mints a new one per step |
| Revoke | `DELETE /installation/token` straight after the operation, whether it succeeded or failed, confirmed by a `401` on the next use |
| Key rotation | Every 90 days, and immediately on suspected compromise. Generate a new key, deploy it to the helper, verify, then delete the old key in App settings |
| Incident | Suspend the App installation (a single switch that stops all agents), rotate the key, review the audit log (`gh api` audit or the repo's security log) |

## 7. Human setup checklist (future `ready-for-human` ticket)

☐ Create the private App with the §3 permissions · ☐ install it on this repo only · ☐ generate the key and store it on bubbles-server (§5) · ☐ configure the §4 rulesets (with `protect-main` initially without required checks) and CODEOWNERS · ☐ after the bootstrap PR merges, add `verify` and `gate-integrity` as required checks · ☐ create the `agents`, `agent-worker` and `agent-llm` users and the `agent-run` group per §5 and spec 22, and confirm that `id` shows none of the users in `sudo`, `docker` or any other privileged group · ☐ negative isolation test: `sudo -u agent-worker test -r /home/charles/.config/gh/hosts.yml` must fail, `sudo -u agent-worker test -r <App key>` must fail, and `test -r <App key>` run as `charles` must fail · ☐ the adversarial suite of spec 22 (AGENT-14) passes for run identities, which `sudo -u` can't address because they are dynamic · ☐ smoke test: an agent can push `agent/test`, open a PR, and is refused on merge, on pushes to `main` and on workflow edits · ☐ record the App ID and installation ID (not secret) in `docs/agents/github-app.md`.
