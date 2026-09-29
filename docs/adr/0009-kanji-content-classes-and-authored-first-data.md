---
status: accepted
date: 2026-09-28
---

# Kanji area: three separate content classes, authored-first data, per-source licensing

The kanji area aims to establish Charles as a credible advanced kanji/Kanken educator and researcher (focus 漢検準1級 and 1級). To protect that credibility, every page belongs to exactly one content class: **Achievement** (verifiable), **Reference content** (educational, cited) or **Study Note** (personal, non-authoritative). The class is visible in the UI, the structured data type and the Markdown front-matter. **No passed level or certification is ever inferred.** A Kanken Achievement exists only with Evidence and Owner Confirmation.

No third-party dataset (EDRDG KANJIDIC/JMdict, KanjiVG or others) is imported yet. All content starts as original authored material. The data model still carries per-field origin (source) and per-source licensing, so a licensed dataset can be added later through its own ADR. Its licence (for example CC BY-SA) would apply only to the distributions and fields containing that data, never to the whole Kanji section.

## Considered Options

- **Import KANJIDIC/JMdict now for coverage**: rejected for now. It would produce thin auto-generated pages, raise share-alike obligations early, and dilute the authored-expertise signal.

## Consequences

- Reference pages are generated only for entries with original authored content.
- The schema accepts only `source: 'authored'` until an import ADR widens it.
