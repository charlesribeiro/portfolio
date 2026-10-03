# 21 — Agent Identity: the bubbles-server GitHub App

Decision: **ADR-0012**. This document specifies the App. **Do not create it yet.** Charles creates it by hand, as a `ready-for-human` task.

## 1. Identities

| Identity | Who | Can | Cannot |
|---|---|---|---|
| **Charles** (personal account, 2FA/passkey) | Owner | Everything, and is the **only merge authority** and sole CODEOWNER | — |
| **`<portfolio-agents>[bot]`** (GitHub App installation) | All bubbles-server workers | Branches, commits, issues, PRs, read CI results | Merge `main`, administer the repo, change security settings, edit workflows, change the App |
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

"Agents update only their own PRs" cannot be expressed as a GitHub permission, because all workers share one identity. It is enforced by the worker protocol (19 §5): each worker has a stable `worker-id`, branches are named `agent/<worker-id>/<issue>-<slug>`, and a worker touches only branches with its own `worker-id`. Most labels are **informational, not security controls**, since the App can edit labels. The one label with enforcement effect, `gate-change-approved`, is honoured by `gate-integrity` only when the issue-events API shows a code owner applied it. The real controls are the required human review and the fact that `needs-human:*` labels are re-applied on every push by a labeller that runs from `main` (19 §3).

## 5. Authentication model

```
agent-token helper (user `agents`)
  └─ reads App private key (PEM) ──▶ signs JWT (RS256, iat-60s, exp ≤ 10 min, iss = App ID)
        └─ POST /app/installations/{installation_id}/access_tokens
               body: { "repositories": ["portfolio"],
                       "permissions": { minimal set for this job } }   ◀── optional per-job down-scoping
        ◀─ installation access token (expires after 1 hour)
  └─ hands the token to the requesting worker over a local socket
bubbles-server worker (user `agent-worker`)
  └─ uses token for git (credential helper over HTTPS) and gh/API (GH_TOKEN in the process env only)
```

- **Private key storage:** on bubbles-server only, in a secrets store or a file owned by the dedicated `agents` system user with mode `0400`, inside a directory that only `agents` can enter. It is never in the repository, never in container images, never in environment files that are committed, and never readable by worker processes except through the token-minting helper.
- **Token minting helper:** a tiny local service (`agent-token`) running as `agents` that holds the key and returns installation tokens. Its socket accepts only the worker user. Workers never see the PEM. This limits the blast radius of a compromised worker to 1-hour tokens.
- **Commit identity:** `<portfolio-agents>[bot] <{bot-user-id}+<portfolio-agents>[bot]@users.noreply.github.com>`. Commits carry `Co-Authored-By` trailers as configured. Signed commits are optional (a later hardening step).
- **Host isolation (ADR-0012).** Charles's attended sessions may run on bubbles-server under his own account. The autonomous path is isolated from it, and these rules are mandatory:
  - **Unix users.** The supervisor and its worker runs (19 §5) run as the dedicated user `agent-worker`, and the helper as `agents`, never as `charles`. Both are system users with no login shell and home directories outside `/home`, which `ProtectHome=yes` hides. Neither belongs to `sudo`, `docker` or any other privileged group, and `charles` belongs to neither user's group.
  - **systemd hardening.** The supervisor and helper services set `ProtectHome=yes` and `NoNewPrivileges=yes`. Worker runs are spawned from the supervisor service and inherit its user, mount namespace and no-new-privileges flag, even when placed in their own transient scope (19 §5). Any other unit that starts a run must set the same options.
  - **Human credentials stay out.** Charles's GitHub credentials (`gh` login, PATs, SSH keys, SSH agent) are never copied, mounted, forwarded, exported or otherwise passed to the worker path. That includes bind mounts or volumes from `/home/charles`, SSH agent forwarding, and inherited environment such as `GH_TOKEN`, `GITHUB_TOKEN`, `GH_CONFIG_DIR` or `SSH_AUTH_SOCK`. The supervisor builds each run's environment from scratch. Workers receive only this repository's App identity.
  - **Agent credentials stay in.** The App key is readable only by `agents`. Installation tokens exist only in the helper and in the worker processes they were issued to. Charles's ordinary processes and unrelated services can read neither.
  - **Residual risk.** Root on the host can read both identities. A compromise of `charles` counts as root, because `charles` is in `sudo` and `docker`. This risk is accepted. Damage from a stolen App identity is bounded by §3 and §4, and the response is the Incident row of §6. Damage from stolen human credentials is not bounded by this design.

## 6. Token lifecycle

| Stage | Rule |
|---|---|
| Issue | One token per job (issue attempt), requested just in time, down-scoped where possible (for example read-only for review-only jobs) |
| Use | In memory or process env only. Never written to disk, logs, PR bodies or commit messages. Worker logs pass through a redaction filter for `ghs_` tokens |
| Refresh | Re-mint when fewer than 10 minutes of validity remain. Jobs longer than an hour re-mint instead of extending |
| Revoke | `DELETE /installation/token` at job end, whether it succeeded or failed |
| Key rotation | Every 90 days, and immediately on suspected compromise. Generate a new key, deploy it to the helper, verify, then delete the old key in App settings |
| Incident | Suspend the App installation (a single switch that stops all agents), rotate the key, review the audit log (`gh api` audit or the repo's security log) |

## 7. Human setup checklist (future `ready-for-human` ticket)

☐ Create the private App with the §3 permissions · ☐ install it on this repo only · ☐ generate the key and store it on bubbles-server (§5) · ☐ configure the §4 rulesets (with `protect-main` initially without required checks) and CODEOWNERS · ☐ after the bootstrap PR merges, add `verify` and `gate-integrity` as required checks · ☐ create the `agents` and `agent-worker` users per §5, and confirm that `id` shows neither in `sudo`, `docker` or any other privileged group · ☐ negative isolation test: `sudo -u agent-worker test -r /home/charles/.config/gh/hosts.yml` must fail, `sudo -u agent-worker test -r <App key>` must fail, and `test -r <App key>` run as `charles` must fail · ☐ smoke test: an agent can push `agent/test`, open a PR, and is refused on merge, on pushes to `main` and on workflow edits · ☐ record the App ID and installation ID (not secret) in `docs/agents/github-app.md`.
