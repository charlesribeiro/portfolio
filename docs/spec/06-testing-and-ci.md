# 06 — Testing Strategy & CI

The main purpose of this suite is to let a coding agent's pull request be judged **objectively**: if every blocking gate is green, the change meets the spec. Every gate has a requirement ID and a single command. **Blocking** gates (pre-deploy and post-deploy phases) are deterministic, or bounded by explicit tolerances where they measure real systems (G16 Lighthouse: median of 3 runs against thresholds; G17: live deployment with retries). **Scheduled** gates (G0h, G18–G20) are reports that open issues and never block a merge, and only these may be nondeterministic (e.g. the G19 LLM eval).

## 1. Single entry point

```
pnpm verify              # runs every active pre-deploy gate from ci/gates.json locally, serially (agents' pre-push check)
pnpm verify:fast         # typecheck + lint + unit (inner loop)
pnpm gate <id>           # runs one gate exactly as its CI matrix job does
pnpm gates:plan <phase>  # prints the matrix CI will run for a phase (active gates only)
```

### 1.1 Gate manifest and CI matrix (ADR-0017)

`ci/gates.json` is the single list of gates:

```jsonc
{ "id": "G12", "command": "pnpm test:a11y", "phase": "pre-deploy", "status": "active",
  "definitionPaths": ["tests/a11y/axe.config.ts", "config/allowlists/axe.json"],   // REQUIRED when active
  // optional:
  "title": "Accessibility", "requirements": ["A11Y-01", "…"], "needs": ["build:production"],
  "runner": "default", "timeoutMinutes": 20 }
```

- `phase`: `pre-deploy` (on the PR, before preview deploy) · `post-deploy` (against the preview URL on PRs, and against production after merge) · `scheduled` (periodic, opens issues only).
- `status`: `planned` gates are documented and not executed. `active` gates must execute and cannot be silently skipped. The bootstrap PR registers every gate as `planned`. Each implementing ticket flips its gate to `active` and lists its `definitionPaths` (config, threshold, allow-list and runner files; test files are covered by W5 instead). An active entry without `definitionPaths` fails both `plan` and `gate-integrity`.
- **Enumeration, execution and aggregation are inline `jq` in the human-owned workflow YAML.** A matrix job runs the manifest `command` string directly, never via `pnpm gate`, which is a local convenience only.
- **Required set** = ids active in the base manifest ∪ ids active in the PR manifest (a missing base manifest counts as empty). Each id runs its PR-manifest entry when it exists there and is active. Otherwise it runs its base entry. The aggregate **`verify`** job fails unless every id in the required set reported success. With no active gates, the matrix job is skipped by an `if:` guard and `verify` passes vacuously. An approved gate removal keeps `verify` red by design and is merged by Charles with the admin bypass.
- **`verify` and `gate-integrity` are the only required status checks.** Their names never change.
- **Threshold files** (a closed list): `config/budgets.json` (PERF-05…10, including PERF-06 §2.1) and `config/thresholds.json` (Lighthouse, coverage and similar). Every key declares `"direction": "max" | "min"`. **Allow-lists** live only in `config/allowlists/*.json`.

### 1.2 Gate integrity (ADR-0017)

`gate-integrity` runs through `pull_request_target` on `opened`, `synchronize`, `reopened`, `labeled` and `unlabeled`, with **only base-branch code checked out**. It declares minimal `permissions:` (`contents: read`, `pull-requests: read`, `issues: read`, `statuses: write`) and no secrets. It reads the PR's changed files through the API **as data**. It never installs, builds or executes PR content, and never interpolates PR titles or branch names into shell commands. It reports as a commit status named `gate-integrity` on the PR head SHA.

It **fails** on these signals unless a code owner applied the label `gate-change-approved` **after the latest push**. "Base-active" means active in the base manifest.

| # | Weakening signal |
|---|---|
| W1 | A base-active gate removed, set to `planned`, its `phase` or `command` changed, or a path removed from its `definitionPaths` |
| W2 | A value in a threshold file loosened according to its `direction` |
| W3 | Any change to a file in the `definitionPaths` (base ∪ PR) of a base-active gate, other than the threshold files (`config/budgets.json`, `config/thresholds.json`) and allow-lists (`config/allowlists/*.json`) (e.g. `tsconfig.json`, `.lighthouserc.json`, axe or ESLint config) |
| W4 | New entry in an allow-list file already in a base-active gate's `definitionPaths` (allow-lists shipped by an activation PR don't trip it; removing entries is strengthening; `docs/dependencies.md` is not an allow-list) |
| W5 | Added suppressions (`.skip`, `.only`, `test.fixme`, `eslint-disable`, `stylelint-disable`, `@ts-ignore`, `@ts-expect-error`), deleted or renamed test files, or deleted snapshot baselines |
| W6 | A change to a `package.json` script reachable from a base-active gate's `command`, or to `scripts/ci/**` |

**Not weakening:** activating a new gate (adding its scripts and config), adding tests, tightening a threshold file, and removing allow-list entries. Editing the **non-threshold, non-allow-list** definition files (or reachable scripts) of an already active gate always trips W3/W6, even to strengthen it, and Charles approves it with the label. **Agents must never make CI pass by weakening, disabling or removing a gate** (19 §2). Weakening no machine can see (such as gutted assertions) is covered by Charles's review of every merge.

### 1.3 Bootstrap (ADR-0017)

1. **Governance** (human, 21 §7): create the App, CODEOWNERS, the `agent-branch-namespace` and `protect-tags` rulesets, and `protect-main` with PR + code-owner review but **no required status checks yet**.
2. **Bootstrap PR** (the single promotion point for the initial workflows): the scaffold, workflows drafted in `ci/proposed/` and promoted by Charles, `ci/gates.json` with every gate `planned`, the `pnpm gate` / `gates:plan` runners, `scripts/ci/**`, and `config/budgets.json`, `config/thresholds.json` and `config/allowlists/`. Charles merges it with the admin bypass.
3. Charles adds `verify` and `gate-integrity` as required checks on `protect-main`. From then on, `gate-integrity` and the labeller judge every PR with base-branch code. `ci.yml` runs the PR's copy, which is safe because the App cannot edit workflows.

## 2. Gates

| # | Gate | Tool | Checks | When | Blocking |
|---|---|---|---|---|---|
| G0 | Secrets | gitleaks on the PR diff + GitHub secret scanning with push protection | No credentials, tokens or keys in the repo | PR (`pre-deploy`) | ✅ |
| G0h | Secrets (history) | gitleaks over full history | Same, across all history | Weekly (`scheduled`) | ❌ (opens an issue) |
| G1 | Format | Prettier `--check` | Formatting | PR | ✅ |
| G2 | Lint | ESLint, Stylelint | Code rules, import boundaries (04 §3), a11y lint in templates, MDX import allow-list | PR | ✅ |
| G3 | Types | `astro check` + `tsc --noEmit` | Types, including content schemas | PR | ✅ |
| G4 | Dependency policy | script | DEP-03 | PR | ✅ |
| G5 | Unit | Vitest | Domain layer (≥ 95 % line/branch coverage), representation builders, ja-text utilities, Markdown plugins | PR | ✅ |
| G6 | Build | `astro build` | Schema validation (CONTENT-01/04/06/07), dangling refs | PR | ✅ |
| G7 | Dist scans | `scripts/scan-dist` | CONTENT-02/03/08, internal links and anchors, IA-06 published-URL retention | PR | ✅ |
| G8 | HTML validity | html-validate | Valid, semantic HTML for every page | PR | ✅ |
| G9 | SEO/agent contract | Vitest over `dist/` | SEO-01…SEO-25 (09): canonical, MD alternate exists and parses, `describedby`, JSON-LD parses and matches expected `@type`s and `@id` graph, sitemap completeness, llms.txt links resolve, robots.txt valid | PR | ✅ |
| G10 | Representation parity | Vitest | TEST-REP-01: HTML `<main>` text equals the `.md` body after normalization; JSON datasets equal the domain layer output; TEST-REP-02: every fact in `/data/*.json`, `/resume.json` and JSON-LD is visible in the entity's canonical HTML (04 §4, ADR-0004) | PR | ✅ |
| G11 | No-JS | Playwright (`javaScriptEnabled: false`) | Every **information page** (all routes except `/lab/**`) renders its primary content, navigation works, forms have fallbacks. `/lab/**` routes render their no-JS description (ADR-0016) | PR | ✅ |
| G12 | Accessibility | Playwright + axe (WCAG 2.2 A/AA tags) on **every** route, in light, dark and forced-colors modes | Zero violations. Also keyboard traversal tests for nav, menus and search | PR | ✅ |
| G13 | E2E behaviour | Playwright | Search islands, theme toggle, CV download, contact links, 404 | PR | ✅ |
| G14 | Visual regression | Playwright screenshots in a pinned Docker image (identical fonts and rendering) | Key templates × {375, 768, 1280} × {light, dark} | PR | ✅ (baseline changes get the `needs-human:visual` label and are reviewed in Charles's merge approval) |
| G15 | Budgets | `scripts/check-budgets` | PERF-05…PERF-10 per route type, and PERF-06 §2.1 per JS bucket (ADR-0016), from `config/budgets.json`, which a unit test checks against the spec table | PR | ✅ |
| G16 | Lighthouse | LHCI on the production-mode build served by a local static server (mobile preset, 3 runs, median) | PERF-01…04 assertions, A11y/SEO/Best-Practices = 100 | PR | ✅ |
| G17 | Deploy smoke | Playwright against the deployed URL | Headers (10 §4), Japanese-path URLs (IA-04), `.md` content-type, redirects. On `main`: also production `robots.txt` byte-equal to that build's (SEO-24), and, on preview and production, **the set of `<script>` elements in the deployed HTML (each `src`, plus a hash of each inline script) equals the set in that build's `dist/`**, which catches any edge-injected script (beacons, email obfuscation, Rocket Loader) | `post-deploy` (preview on PR, production after merge) | ✅ |
| G18 | External links | lychee (or similar) | CONTENT-11 | Weekly + manual (`scheduled` phase) | ❌ (opens or updates an issue, never a PR or commit, ADR-0017) |
| G19 | Agent-readability eval | script calling an LLM with `/llms.txt` + home `.md` | Extracts the fact checklist (name, title, years, stack, location, contact) and compares it with `profile.yaml` | Weekly (`scheduled`, from `main` only) | ❌ (opens an issue). Uses the LLM key from the `agent-eval` environment, the only secret-bearing gate (ADR-0017). Stays `planned` until a key exists |
| G21 | Claims & disclosure report | read-only script over the content diff | CONTENT-13/14: lists new or changed Claims, Achievements, KankenAttempts, MediaMentions, client fields and profile as a claim → provenance → evidence table in the job summary. It fails only on malformed provenance. Labelling (`needs-human:claims`) is done by the labeller workflow, not by this gate | PR | ✅ |
| G22 | Placeholders | production-mode build + dist scan | CONTENT-12 | PR (in production mode) | ✅ |
| G23 | Licensing | `reuse lint` + a path-rule script + a contract test over `dist/` | LIC-01…LIC-04, LIC-08, LIC-10, CONTENT-16: every file licensed, binaries only under `assets/` or `third-party/`, every `third-party/*` has `SOURCE.md` | PR | ✅ |
| G24 | CSS conventions | Stylelint (custom config) | CSS-01…CSS-09 (05 §5) | PR | ✅ (its own manifest entry, so it is reported separately from G2) |
| G25 | Proposed workflows | actionlint over `ci/proposed/*.yml` | Syntax and common security mistakes in drafted workflow YAML (19 §3.1) | PR | ✅ |
| G20 | Prod Lighthouse / CrUX | LHCI against production | Detects drift | Weekly | ❌ (opens an issue) |

## 3. Test pyramid and locations

- `src/**/*.test.ts`: unit tests (domain, representations, utilities). These are the fastest and most numerous.
- `tests/seo/`, `tests/contract/`: tests over `dist/` output (G9, G10).
- `tests/e2e/`, `tests/a11y/`, `tests/visual/`: Playwright.
- Fixtures: `content/__fixtures__/` holds a minimal content set used by tests, so tests don't break when real content changes. The real content is still built and scanned by G6–G9.

## 4. Snapshot policy

Representation builders (JSON-LD, Markdown, llms.txt) use inline or file snapshots. **Snapshot updates in agent PRs must be explained in the PR description.** CI comments the snapshot diff on the PR.

## 5. Flakiness policy

- Gates must be deterministic. A flaky test is quarantined within 24 h with an issue and fixed or deleted within 7 days.
- Lighthouse uses the median of 3 runs with generous but meaningful thresholds. Byte budgets (G15) are the strict, deterministic check.

## 6. Branch protection (ADR-0011)

- `main` is protected. The required status checks are **`verify`** (the aggregate of every active pre- and post-deploy gate: G0, G1–G17 and G21–G25 once active; scheduled gates G0h and G18–G20 never block merges) and **`gate-integrity`** (§1.2). The branch must be linear and up to date.
- **Every PR requires approval from Charles (the code owner) before merge. Agents never merge.** Agents act as the dedicated GitHub App, which has no admin role and isn't on any bypass list (ADR-0012, spec 21). Rulesets (21 §4) enforce this.
- Stale approvals are dismissed on new pushes. CODEOWNERS is `* @charlesribeiro`, so every path needs Charles. Paths that additionally get a `needs-human:*` label are defined **only** in the path→label map in 19 §3.
- No force-push to `main`. Only the repository admin role (Charles) is on the bypass list, so Charles can merge Charles's own PRs in a single-maintainer repo (21 §4). The App never is.

## 7. Agent-readability eval (G19) detail

A checked-in `tests/agent-eval/facts.yaml` lists questions and expected answers derived from `profile.yaml` (for example "What is Charles's primary role?" → "Senior Frontend Engineer"). The eval fetches only the public machine-readable surfaces and scores exact or normalized matches. Scores are uploaded as a workflow artifact and summarized in a tracking issue, never committed to `main`. It is a `scheduled` report, not a merge gate, because model output is nondeterministic, but a regression opens an issue.
