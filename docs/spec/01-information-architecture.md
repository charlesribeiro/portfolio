# 01 — Information Architecture

## 1. Site map

```
/                         Home: positioning, quick facts, flagship work, calls to action
/work                     Case studies index (filterable by theme using links, not JS)
/work/{slug}              Case study
/experience               Full professional history (the detailed web CV)
/cv                       Print-optimized CV (same data) → /cv.pdf generated at build
/ai                       AI engineering hub (projects, writing, courses, how Charles uses AI to build)
/writing                  Publications, courses (Alura), talks, articles
/writing/{slug}           Self-hosted article or publication detail page
/timeline                 Chronological personal/professional history plus media coverage
/timeline/{slug}          Timeline entry detail (only for entries with long-form context)
/timeline/category/{cat}  Static category-filtered view (17 §3)
/timeline/from-the-start  Ascending chronological view (17 §3)
/kanji                    Kanji & Kanken knowledge area hub (see 18)
  /kanji/journey          Verifiable Kanken achievements plus journey narrative
  /kanji/characters/{字}  Reference: single character
  /kanji/words/{語}        Reference: vocabulary (includes 熟字訓/当て字, difficult readings)
  /kanji/yojijukugo/{語}   Reference: 四字熟語
  /kanji/kotowaza/{slug}  Reference: 故事・ことわざ
  /kanji/topics/{slug}    Reference guides (homophones, 国字, radicals, …)
  /kanji/levels/{level}   Level lists (jun-1, 1)
  /kanji/radicals/{slug}  Radical/component pages
  /kanji/notes/{slug}     Personal study notes
  /kanji/search           Search (works without JS through static browse indexes; no form fallback, OD-14)
  /kanji/data             Dataset landing page (downloads, schema, licence)
/about                    Long-form biography, languages, education, interests
/hire                     Recruiter page: availability, role fit, logistics, contact (route approved, OD-12)
/colophon                 How this site is built: stack, budgets, CI badges, agent workflow
/404                      Themed not-found page with search and links

Machine-oriented (see 09):
/llms.txt  /llms-full.txt  /sitemap.xml  /robots.txt  /humans.txt
/{any-page}.md            Markdown alternate of each content page
/resume.json              JSON Resume (jsonresume.org schema)
/data/profile.json  /data/experience.json  /data/publications.json  /data/projects.json  /data/timeline.json   (CC0, 20 §1.1)
/site-index               HTML index of every page (the site-wide no-JS search baseline)
/kanji/data/*.json|csv    Kanji datasets
/feeds/writing.xml  /feeds/timeline.xml  /feeds/kanji.xml   Atom feeds
```

## 2. Primary navigation

Order reflects the positioning hierarchy and the recruiter path:

**Work · Experience · AI · Writing · Timeline · Kanji · About · Hire**

- "Hire" is visually distinct: the old-web "enter" button treatment (see 12).
- On mobile, primary nav collapses into a `<details>`-based menu that works without JS.
- A persistent "CV" and "Contact" pair lives in the masthead on every page.

## 3. Secondary structures

- **Sidebar boxes** (the portal-era right column) hold context-specific modules: quick facts, "last updated", related entities, badges. On mobile they flow below the content.
- **Footer:** sitemap links, machine-readable links (`llms.txt`, JSON Resume, feeds, datasets), licences, colophon, last build SHA and date.

## 4. URL rules

| ID | Rule |
|---|---|
| IA-01 | HTML canonical URLs have **no trailing slash** and no `.html` extension: `/work/pilot-bidding`. |
| IA-02 | The Markdown alternate is the canonical path plus `.md`: `/work/pilot-bidding.md`. Home: `/index.md`. |
| IA-03 | Latin slugs are lowercase kebab-case ASCII. |
| IA-04 | Kanji reference slugs MAY be Japanese (`/kanji/words/魑魅魍魎`). Canonicals, sitemap and `llms.txt` emit them percent-encoded. A CI deploy smoke test MUST fetch at least one Japanese-path URL from the preview deployment. |
| IA-05 | Homographs are disambiguated as `{word}-{reading-in-hiragana}`, e.g. `/kanji/words/上手-うわて`. |
| IA-06 | URLs are permanent. Renames require an entry in the redirects file. CI fails if a previously published URL (recorded in the committed `published-urls.txt`, see 10 §5) disappears without a redirect. |
| IA-07 | Tracking parameters (`ref`, `utm_*`) never change content and are stripped from canonicals. |

## 5. Cross-linking graph

Entities link in both directions. Back-links are computed at build time, never written by hand:

```
Experience ──uses──▶ Skill ◀──uses── Project/CaseStudy
     │                                    │
     └──produced──▶ Project ◀──about── Publication/Course
TimelineEntry ──references──▶ any entity (Experience, Publication, Achievement, MediaMention)
Achievement (Kanken) ◀── TimelineEntry, /kanji/journey, /about, JSON-LD hasCredential
KanjiEntry ◀──mentions── StudyNote, Topic
```

Every Skill that appears on the site links to the Experiences and Projects that support it. A Skill with no supporting entity MUST NOT be displayed (CONTENT-09).

## 6. Language strategy

- Site language: **English** (`<html lang="en">`), since the target is international remote roles.
- Japanese text is inline with `lang="ja"` on the smallest enclosing element (A11Y-10).
- Portuguese source titles (for example G1 and Folha headlines) are shown in the original with `lang="pt-BR"` and an English gloss.
- Full PT-BR or JA page translations are deferred (OD-05). If added, they go under the `/pt/` or `/ja/` prefix with `hreflang` pairs.
