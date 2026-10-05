# Portfolio agent GitHub App

The autonomous agents' only GitHub identity (ADR-0012, [spec 21](../spec/21-agent-github-app.md)). This file records its public identifiers and the H-002 setup evidence (#4). Neither the App ID nor the installation ID is a secret (21 §7). Never record the private key, a JWT or an installation token here (21 §6).

## Identity

| Field | Value |
|---|---|
| App | `bubbles-portfolio-agents` (private, owned by `charlesribeiro`) |
| Bot account | `bubbles-portfolio-agents[bot]` (user ID `337172008`) |
| Commit identity | `bubbles-portfolio-agents[bot] <337172008+bubbles-portfolio-agents[bot]@users.noreply.github.com>` |
| App ID | `TODO(Charles)` |
| Installation ID | `TODO(Charles)` |
| Installation scope | `charlesribeiro/portfolio` only ("Only select repositories") |
| Webhooks, user authorization, callback/setup URLs | Off (21 §2) |

## Permissions

Exactly the 21 §3 set. Every other repository, organization and account permission is None, including Workflows, Administration, Secrets, Variables, Environments, Pages and Deployments.

| Permission | Level |
|---|---|
| Metadata | Read |
| Contents | Read & write |
| Issues | Read & write |
| Pull requests | Read & write |
| Checks | Read |
| Commit statuses | Read |
| Actions | Read |

Confirmed against the App settings on `TODO(Charles): date`.

## Negative isolation test (21 §5, §7)

Run on `TODO(Charles): date`. Each read must fail.

| Check | Expected | Result |
|---|---|---|
| `sudo -u agent-worker test -r /home/charles/.config/gh/hosts.yml` | fails | `TODO(Charles)` |
| `sudo -u agent-worker test -r <App key>` | fails | `TODO(Charles)` |
| `test -r <App key>` as `charles` | fails | `TODO(Charles)` |
| `id agents` | no privileged group | `TODO(Charles)` |
| `id agent-worker` | no privileged group | `TODO(Charles)` |

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
