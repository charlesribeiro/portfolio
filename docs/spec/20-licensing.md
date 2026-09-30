# 20 — Licensing & Provenance of Materials

Decision: **ADR-0014**. The goal is that for any file in the repository, and any URL on the site, both a human and a machine can tell who owns it and what reuse is allowed.

## 1. Licences

| Material | Licence | SPDX identifier |
|---|---|---|
| Source code: application code, CSS, scripts, tests, configuration, CI, build tooling | MIT | `MIT` |
| **Professional metadata**: factual, structured information about Charles's professional profile, meant for machine reuse (§1.1) | **CC0 1.0** (public-domain dedication) | `CC0-1.0` |
| Original written/editorial content: about, case studies, articles, timeline context, page copy, project documentation under `docs/` | CC BY-NC 4.0 | `CC-BY-NC-4.0` |
| Original kanji educational content: reference entries, topic guides, study notes, authored kanji datasets | CC BY-NC 4.0 | `CC-BY-NC-4.0` |
| Personal photographs | All Rights Reserved | `LicenseRef-AllRightsReserved` |
| Original artwork and visual assets (masthead art, illustrations, icons, 88×31 buttons, textures, favicon), unless a file is explicitly licensed otherwise | All Rights Reserved | `LicenseRef-AllRightsReserved` |
| Third-party material (fonts, future datasets, quoted excerpts, any vendored code) | Its original licence, with provenance | e.g. `OFL-1.1`, `CC-BY-SA-4.0` |

### 1.1 Professional metadata (CC0)

So recruiting, search and research systems, commercial ones included, can reuse Charles's professional facts without licence friction, the following are dedicated to the public domain under **CC0 1.0**:

| Kind | Published outputs | Source files |
|---|---|---|
| Profile | `/data/profile.json`, `/resume.json`, `/cv.md`, `/cv.pdf`, the `/cv` page | `content/profile.yaml` |
| Agent summary | Every `/llms.txt` section except `## Kanji` (§3) | The respective CC0 YAML (profile, projects, publications, timeline) |
| Experience | `/data/experience.json` | `content/experience/*.yaml` |
| Publications and courses (bibliographic metadata) | `/data/publications.json` | `content/publications/*.yaml` |
| Skills, achievements, education | inside the files above | `content/skills.yaml`, `content/achievements/*.yaml`, `content/education/*.yaml` |
| Project facts (title, period, role, stack, summary, outcome claims) | `/data/projects.json` | `content/projects/*.yaml` |
| Timeline and media facts (date, title, category, publisher, URL, archive URL) | `/data/timeline.json` | `content/timeline/*.yaml`, `content/media/*.yaml` |
| Structured data | The **professional-metadata nodes** of embedded JSON-LD (`Person`, `Occupation`, `Role`, `Organization`, `EducationalOccupationalCredential`, bibliographic `Book`/`Course`/`Article` metadata of publications, with CC0 marked via `sdLicense` on the **CreativeWork** nodes that can carry it, per LIC-10. Press nodes (typed by `format`, 17 §5) carry `copyrightHolder` only when it is confirmed, and their facts are CC0 in `/data/timeline.json`), and `/data/schema/*` | Generated. Editorial and kanji nodes (`Article` bodies and descriptions of case studies, `DefinedTerm`, `LearningResource`, `BlogPosting`) inherit their page's licence (LIC-06) |

To make the boundary work at file level (REUSE is per file), **facts and prose live in separate files**. Case-study bodies (`content/projects/*.mdx`), publication notes (`content/publications/*.mdx`) and timeline context (`content/timeline/*.md`) are editorial prose (CC BY-NC), while their `.yaml` siblings are CC0 facts.

**What CC0 does not cover** (stated in `LICENSING.md` and at `/colophon#licences`): third-party text quoted inside the metadata, such as original press headlines and excerpts, which are quoted, not relicensed (LIC-05) · trademarks and names of organizations (for example United Airlines) · personality, publicity and privacy rights · linked images (photos stay All Rights Reserved even when a CC0 JSON file points to them) · editorial prose. The dedication is **irrevocable** for published versions, so only facts already approved for publication (CONTENT-13) ever reach a CC0 file.

`docs/` counts as editorial content (CC BY-NC), because specs and ADRs are written prose. Code blocks inside `docs/` and `content/` follow the licence of the file they are in. **LIC-OPEN-1** (non-blocking): Charles may later choose to dual-license code snippets in articles under MIT so readers can reuse them.

## 2. Repository layout for licence boundaries

Licences are assigned **by path** and declared with the [REUSE](https://reuse.software) specification:

```
/
├─ LICENSE                      MIT (the default for code; GitHub shows this)
├─ LICENSES/                    Full texts: MIT.txt, CC-BY-NC-4.0.txt, LicenseRef-AllRightsReserved.txt, OFL-1.1.txt, …
├─ REUSE.toml                   Path → licence and copyright annotations (the source of truth)
├─ LICENSING.md                 Human-readable summary of this document
├─ src/  scripts/  tests/  .github/  config files        → MIT
├─ src/styles/                  → MIT (CSS is code; see §4)
├─ content/                     → CC-BY-NC-4.0 by default (text only; binary media are not allowed here)
│  └─ profile.yaml, experience/, publications/*.yaml, projects/*.yaml, skills.yaml,
│     achievements/, education/, timeline/*.yaml, media/   → CC0-1.0 (professional metadata, §1.1)
├─ docs/                        → CC-BY-NC-4.0
├─ assets/                      → LicenseRef-AllRightsReserved (default for everything below)
│  ├─ photos/                   personal photographs
│  ├─ art/                      original artwork and visual assets
│  └─ placeholder/              placeholders (ARR, never in production, ADR-0008)
└─ third-party/                 → per-directory licence + SOURCE.md provenance
   ├─ fonts/{family}/           font files + OFL.txt + SOURCE.md
   └─ data/{sourceId}/          (future, ADR-0009) dataset snapshots + licence + SOURCE.md
```

Rules:

| ID | Rule |
|---|---|
| LIC-01 | Every file is covered by `REUSE.toml` or an SPDX header. `reuse lint` passes (CI gate G23). |
| LIC-02 | Binary media (images, video, audio, fonts) may live **only** under `assets/` or `third-party/`, never under `content/` or `src/`. Content references media by path. **Exception:** generated test baselines under `tests/**/__screenshots__/` are allowed and declared `LicenseRef-AllRightsReserved` in `REUSE.toml`, because they render the artwork. |
| LIC-03 | An exception to the default licence of a path (for example one piece of artwork released under CC BY) is declared per file in `REUSE.toml`. It is never implied. |
| LIC-04 | Every `third-party/` directory has a `SOURCE.md` with origin URL, version or date, checksum, licence, required attribution text and the reason for inclusion. Adding one is `needs-human:dependency`. |
| LIC-05 | Quoted third-party excerpts in content (for example a ≤ 300-character press quotation, 17 §4) are attributed inline and recorded in the entry's `excerpt` field. They are not relicensed under CC BY-NC. |
| LIC-06 | Files with different licences are never merged into one distribution, **except generated aggregates that declare a licence per section** (`llms.txt`, `llms-full.txt`). Generated outputs inherit the licence of their source path. A generated artefact that combines sources (for example `llms-full.txt`) states each component's licence in a header section. |
| LIC-07 | Placeholder assets are ARR as well. The placeholder rules still apply (CONTENT-12). |
| LIC-08 | CC0 files contain only professional metadata (§1.1). In a CC0-licensed YAML file, any string value longer than 280 characters or containing a blank line counts as prose and fails G23. **Exempt:** `QuotedText` values (`headline`, `headlineTranslation`, `excerpt` in `content/media/*.yaml`, ≤ 300 characters, LIC-05). Each carries its own `rights` object, so the exclusion from CC0 is machine-readable in the source file itself (LIC-10). In `REUSE.toml`, `content/media/*.yaml` is annotated `SPDX-License-Identifier = "CC0-1.0 AND LicenseRef-ThirdPartyQuotation"`. `LICENSES/LicenseRef-ThirdPartyQuotation.txt` explains that `QuotedText` values belong to their stated holders. Editorial prose goes in the sibling `.md`/`.mdx`. |
| LIC-09 | Each output declares the licence it inherits: `/data/*.json` and `/resume.json` carry `"license": "https://creativecommons.org/publicdomain/zero/1.0/"`. The `/cv` page and `/hire` fact box use `rel="license"` → CC0. Pages that mix sources use the stricter licence for the page. In JSON-LD, the licence of the structured data is marked with `sdLicense` on CreativeWork nodes (LIC-10). The colophon explains this. |
| LIC-10 | **One rights contract for third-party quotations**: `QuotedText` (03 §1), `{ text, lang, rights: { status, holder, source } }`, applied the same way in every representation. Factual metadata (URL, publisher, date, language) stays CC0. **Source YAML:** quotation fields are `QuotedText` objects. **`/data/*.json`:** emitted unchanged as `QuotedText` objects, with the top level listing their paths in `"licenseExclusions"` (JSONPath) next to `"license"`. **JSON-LD:** schema.org defines `sdLicense` (the licence of the structured data about a work) only on `CreativeWork`. So it is set on CreativeWork nodes: CC0 on professional ones (`ProfilePage`/`WebPage` of pure-metadata pages such as `/cv` and `/hire`, publication `Book`/`Course`/`Article` metadata, `EducationalOccupationalCredential`), and CC BY-NC on editorial and kanji ones (case-study `Article`, `LearningResource`, `DefinedTermSet`, `BlogPosting`). Non-CreativeWork nodes (`Person`, `Organization`, `Occupation`, `Role`) can't carry it, and their CC0 status is stated by the pure-metadata page node, `/data/*.json` and the colophon. **Press nodes omit `sdLicense`.** These are the JSON-LD nodes for media mentions, typed by `format` as `NewsArticle`, `VideoObject`, `RadioEpisode` or `PodcastEpisode` (17 §5, D6). Their `headline` (schema.org Text) is a third-party quotation that a CC0 marking would wrongly cover. They carry `inLanguage`, and `copyrightHolder` only from the confirmed work-level `MediaMention.copyrightHolder` (17 §2), never from `publisher` or a quotation's field-level `rights.holder`, which attributes the quoted text only. Their facts are CC0 in `/data/timeline.json`. **HTML:** `<q cite>` / `<blockquote cite>` with a visible `<cite>` attribution. **Markdown alternates:** front-matter `thirdPartyQuotations:` holding full `QuotedText` objects (`text`, `lang`, `rights`). **`llms.txt`, `llms-full.txt`:** quotations in quotation marks with publisher attribution, and the section licence line states the exclusion. **`/feeds/timeline.xml`:** a feed-level `<rights>` stating the feed licence and the quotation exclusion, plus a per-entry `<rights>` naming the holder. **`/resume.json`** never contains `QuotedText` (G23 asserts this). G23's contract test fails on a quotation field that is a bare string or lacks `rights` (source and `/data`), on a press node with an `sdLicense`, on a press node whose `copyrightHolder` does not match a confirmed work-level `MediaMention.copyrightHolder` (unconfirmed, absent, or taken from `publisher`/`rights.holder`), on `sdLicense` on a non-CreativeWork node, on a missing `licenseExclusions` entry, or on `QuotedText` in `/resume.json`. |

## 3. Licence signals on the published site

| Surface | Signal |
|---|---|
| Footer (every page) | "Code: MIT · Professional data: CC0 · Writing & kanji content: CC BY-NC 4.0 · Photos & artwork: © All rights reserved · Third-party: see credits", linking to `/colophon#licences` |
| Content pages | `<link rel="license">` per LIC-09 (CC BY-NC for editorial and kanji pages, CC0 for `/cv` and pure-metadata pages), JSON-LD `license` on the `CreativeWork` / `Article` / `LearningResource` / `DefinedTermSet` |
| Images | JSON-LD `ImageObject` with `license`, `copyrightHolder` and `creditText` where images are described. Visible credit for third-party images |
| `.md` alternates | Front-matter `license:` (SPDX ID + URL) |
| `/data/*.json`, `/resume.json` | Top-level `license`: CC0 1.0 (LIC-09), plus `licenseExclusions` for third-party quotation fields (LIC-10) |
| `/kanji/data/*` | Top-level `license` (CC BY-NC 4.0 for authored data) and `attribution`. The `Dataset` JSON-LD has `license` per `DataDownload` |
| `llms.txt` / `llms-full.txt` | **Per-section licence lines derived from source paths** (LIC-06), computed by the generator, never hand-assigned. `llms.txt`: Profile, Work, Writing, Timeline and Data sections come from CC0 YAML (summaries, bibliographic data, facts) → CC0. The `## Kanji` section comes from kanji content → CC BY-NC. `## Optional` lists only CC0-sourced items. `llms-full.txt`: profile and fact sections are CC0, and case-study bodies, about, publication notes, timeline context and kanji are CC BY-NC |
| `/colophon#licences` | The full human explanation, the third-party credits list generated from `third-party/*/SOURCE.md`, and a link to `LICENSING.md` |

## 4. Consequences worth knowing

- **The visual design is partly reusable.** CSS is MIT, so someone may copy the stylesheet that implements the look. The artwork it references (`assets/art/`) is ARR and may not be copied.
- **Professional data is CC0 on purpose** (LIC-OPEN-2 resolved). Recruiting platforms, commercial ones included, may reuse Charles's professional facts freely. Prose, kanji material and imagery keep their stricter licences.
- **NC and future share-alike data.** CC BY-NC and CC BY-SA are incompatible in a single work, so future imported data ships as separate files (ADR-0009, LIC-06).
- **Licences are not crawler permissions.** Whether training crawlers may fetch the site is the robots policy (09 §4.1, ADR-0015). A licence says what people may do with copies they obtain.
