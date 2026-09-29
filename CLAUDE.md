# CLAUDE.md

Before any work, read `docs/spec/19-agent-development.md` (agent permissions, worker protocol, human-approval categories; agents never merge and never weaken CI gates) and the relevant `docs/adr/`.
Precedence: ADR > `docs/spec/` > issue text. `CONTEXT.md` defines the vocabulary. Flag any conflict instead of resolving it silently.

## Agent skills

### Issue tracker

Issues are tracked in GitHub Issues on `charlesribeiro/portfolio` via the `gh` CLI. See `docs/agents/issue-tracker.md`.

### Triage labels

Default vocabulary: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: one `CONTEXT.md` + `docs/adr/` at the repo root. See `docs/agents/domain.md`.
