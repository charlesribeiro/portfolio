# Portfolio agent GitHub App

The autonomous agents' only GitHub identity (ADR-0012, [spec 21](../spec/21-agent-github-app.md)). This file records its public identifiers and the H-002 setup evidence (#4). Neither the App ID nor the installation ID is a secret (21 §7). Never record the private key, a JWT or an installation token here (21 §6).

## Identity

| Field | Value |
|---|---|
| App | `bubbles-portfolio-agents`, owned by `charlesribeiro`. Private: an unauthenticated `GET /apps/bubbles-portfolio-agents` returns 404 |
| App ID | `5170277` |
| Installation ID | `167383452` |
| Bot account | `bubbles-portfolio-agents[bot]` (user ID `337172008`) |
| Commit identity | `bubbles-portfolio-agents[bot] <337172008+bubbles-portfolio-agents[bot]@users.noreply.github.com>` |
| Installation scope | `charlesribeiro/portfolio` only, on account `charlesribeiro`: `repository_selection` is `selected`, and `GET /installation/repositories` with an installation token lists only this repository (2026-10-05) |
| Webhooks, user authorization, callback/setup URLs | Off (21 §2): webhook inactive, no callback URL, no setup URL, OAuth user authorization not configured. Checked by Charles on the App settings page, 2026-10-05 |

## Permissions

The App must have exactly the 21 §3 set. Every other repository, organization and account permission is None, including Workflows, Administration, Secrets, Variables, Environments, Pages and Deployments.

| Permission | Level |
|---|---|
| Metadata | Read |
| Contents | Read & write |
| Issues | Read & write |
| Pull requests | Read & write |
| Checks | Read |
| Commit statuses | Read |
| Actions | Read |

Verified on 2026-10-05 from the installation's details as returned by the GitHub API (`GET /app/installations/{installation_id}`, read with the App's own credentials). The installation has exactly these seven permissions at these levels, and nothing else, so it has no `workflows` and no `administration`. It also reports `repository_selection: selected`, a single repository, `charlesribeiro/portfolio`, no suspension (`suspended_at: null`) and no webhook event subscriptions. The smoke test agrees: GitHub issued a token with Contents: write, Pull requests: write and Metadata: read, and refused the workflow push for lack of the `workflows` permission.

## Negative isolation test (21 §5, §7)

Checked on 2026-10-05. The evidence is of three kinds, kept apart below. Only the first table records commands that were run and seen to fail.

### Directly executed

| Check | Run as | Expected | Result |
|---|---|---|---|
| Read the App key | `agent-worker` (the helper's read-only contract run) | fails | Failed |
| Enter the App key directory | `agent-worker` (the helper's read-only contract run) | fails | Failed |
| `test -r <App key>` | `charles` | fails | Failed |
| Enter the App key directory | `charles` | fails | Failed |
| `id agents` | `charles` | no privileged group | Only its own group, `agents` |
| `id agent-worker` | `charles` | no privileged group | Only its own group, `agent-worker` |

### Filesystem and account evidence (not an executed read test)

The literal `sudo -u agent-worker test -r /home/charles/.config/gh/hosts.yml` was not run, and no equivalent was run as `agents`. That neither user can read Charles's GitHub credentials is inferred from:

- `/home/charles` is owned by `charles:charles` with mode `0700` and no ACL. `.config/gh` is `0700`, and `hosts.yml` is `0600`.
- Group `charles` has no members besides `charles`.
- `agents` and `agent-worker` have a `nologin` shell and a home outside `/home`.
- `sudo` and `docker` have `charles` as their only member. Neither agent user is in `adm`, `disk`, `shadow`, `systemd-journal` or `root`.
- The App key directory is owned by `agents` with mode `0700`.

### systemd isolation evidence

Every H-002 helper run (the token minting as `agents`, and the contract and smoke runs as `agent-worker`) ran as a transient unit with `ProtectHome=yes`, `NoNewPrivileges=yes` and `PrivateTmp=yes`, as the host's sudo journal records. `ProtectHome=yes` makes `/home` inaccessible to those processes whatever the file modes. The long-lived supervisor and `agent-token` services are outside H-002 (#4 non-goals), and their hardening is not covered here.

## Smoke test (21 §7)

Run on 2026-10-05 with an installation token scoped to this repository, with Contents: write, Pull requests: write and Metadata: read. `GET /installation/repositories` listed only `charlesribeiro/portfolio`. `main` was `e67dba3` before and after.

| Operation | Expected | Outcome |
|---|---|---|
| Push `agent/test` | allowed | Pushed `482b57e` |
| Open a PR from `agent/test` | allowed | Opened #12 |
| Merge #12 | refused | HTTP 405, repository rule violation: waiting on code owner review. `merged=false` |
| Push to `main` | refused | `GH013` repository rule violation. `main` unchanged |
| Push a commit adding `.github/workflows/h002-smoke.yml` | refused | Rejected: an App can't create or update a workflow without the `workflows` permission. `agent/test` unchanged |
| Cleanup | done | #12 closed unmerged and kept as the audit artifact. `agent/test` deleted (HTTP 404) |
| Token revocation | done | `DELETE /installation/token` returned 204. The next request with it returned 401 |

An earlier attempt the same day minted a token with Metadata: read only, so its push to `agent/test` was refused for lack of permission. It changed nothing and is not part of the evidence above.
