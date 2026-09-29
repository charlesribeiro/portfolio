---
status: accepted
date: 2026-09-28
---

# Licensing boundaries: MIT code, CC0 professional metadata, CC BY-NC content, all-rights-reserved imagery, third-party as-is

The repository is public and mixes material with different owners and purposes, so licences are assigned **by path and declared machine-readably** (REUSE specification: `LICENSES/` + `REUSE.toml`, verified in CI):

- **MIT**: source code (application code, scripts, tests, configuration, CSS, CI).
- **CC0 1.0**: professional metadata, meaning factual, structured information about Charles's professional profile that is meant for machine reuse (`/data/*.json`, `/resume.json`, every `/llms.txt` section except the kanji one (section licences derive from source paths), the CV, the professional-metadata nodes of embedded JSON-LD, and their source YAML). Editorial and kanji JSON-LD nodes inherit their page's licence. The site's purpose is for recruiting, search and research systems, including commercial ones, to reuse these facts freely. A non-commercial licence would work against that.
- **CC BY-NC 4.0**: original written and editorial content (case-study bodies, articles, about, timeline context, docs), and original kanji educational content, including the representations generated from it.
- **All Rights Reserved**: personal photographs, and original artwork and visual assets unless a file is explicitly licensed otherwise.
- **Third-party material**: keeps its original licence and must carry provenance (source, version, licence, attribution).

Facts and prose live in separate files, so the per-file boundary is exact. Details and the path map are in `docs/spec/20-licensing.md`.

## Considered Options

- **CC BY-NC for everything written, including the data files**: rejected. A strict reading of NC could discourage the commercial recruiting platforms the site is trying to reach.
- **CC BY 4.0 for professional metadata**: workable, but attribution requirements add friction to automated reuse and are often dropped anyway. CC0 removes the doubt, including over EU database rights.

## Consequences

- Because CSS is code, the stylesheet that implements the visual design is MIT. The artwork files it references are not.
- CC0 does not cover quoted third-party text (press headlines and excerpts), trademarks, personality or privacy rights, or linked images. The dedication is irrevocable for published versions, so only approved facts reach CC0 files.
- Licences are not crawler permissions. Training-crawler policy is separate (ADR-0015).
- A future share-alike dataset (for example CC BY-SA) cannot be merged into a CC BY-NC file. Mixed data must ship as separate distributions, each with its own licence (ADR-0009).
