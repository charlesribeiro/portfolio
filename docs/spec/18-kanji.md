# 18 — Kanji & Kanken Knowledge Area (`/kanji`)

## 1. Objective

Build, over years, a **credible public resource on advanced kanji study and Kanji Kentei (漢検) preparation**, focused on **準1級 and 1級**. Bret Mayer's kanji resources (bretmayer.com) are a reference for tone and depth. The goal is to establish Charles as a credible advanced kanji/Kanken **educator and researcher**. Charles's personal journey is documented separately, and it **never infers certifications or passed levels**: any statement that Charles passed or holds a Kanken level requires explicit evidence and confirmation from Charles (ADR-0009, CONTENT-13).

For the portfolio, this area shows sustained rigour, information architecture over a large and complex dataset, CJK engineering and teaching ability.

## 2. Three content classes, always visually and semantically distinct

| Class | What | Routes | Visual marker | Schema.org | Authority |
|---|---|---|---|---|---|
| **1. Achievements** (verifiable) | Kanken levels passed, dates, scores if public, certificate evidence. **Empty until Charles provides evidence and confirms** | `/kanji/journey` | 判子 seal stamp, "Verified" label linking to evidence | `EducationalOccupationalCredential` in `Person.hasCredential` | Evidence required (CONTENT-04) |
| **2. Reference** (educational) | Characters, vocabulary, 四字熟語, 故事・ことわざ, 熟字訓/当て字, 国字, readings, radicals, homophone guides | `/kanji/characters/*`, `/words/*`, `/yojijukugo/*`, `/kotowaza/*`, `/topics/*`, `/radicals/*`, `/levels/*` | Dictionary style (washi skin), a "Reference" label, citations block | `DefinedTerm`/`DefinedTermSet`, `LearningResource` | Cites sources (§6) |
| **3. Study notes** (personal) | Methods, logs, mistakes, exam reports, reflections | `/kanji/notes/*` | Notebook/grid-paper style, a "Study note — personal" label, first-person voice | `BlogPosting` | Explicitly non-authoritative |

A reader must always know which class they are looking at. The page label, breadcrumb, JSON-LD type and Markdown front-matter (`contentClass: achievement|reference|note`) all agree. G9 asserts this.

## 3. Entity model

```ts
type Level = 'jun-1' | '1' | '2' | 'jun-2' | '3' | '4' | '5' | '6' | '7' | '8' | '9' | '10'; // 準1級 = 'jun-1'
type Reading = { kana: string; type: 'on'|'kun'|'nanori'|'special'; note?: string; level?: Level };
type Citation = { source: string; locator?: string; url?: string };  // dictionary, page/entry
type KanjiSource = { source: 'authored' };  // CONTENT-16: literal until an import ADR widens it to 'authored' | 'imported' (+ sourceId)

type KanjiCharacter = {
  char: string;                         // single code point; slug
  codepoint: string;                    // "U+9B51"
  variants?: { char: string; kind: 'kyujitai'|'itaiji'|'simplified'; }[];
  radical: ID; components?: ID[]; strokes: number;
  readings: Reading[];                  // 表外読み flagged via level/note
  meanings: string[];                   // English glosses (original wording)
  level?: Level;                        // Kanken level where the character is assigned
  jis?: 1|2|3|4; joyo: boolean; kokuji?: boolean;
  notes?: 'mdx';                        // ORIGINAL commentary: what makes it tricky, mnemonics, confusions
  related?: { words?: ID[]; homophones?: ID[]; lookalikes?: string[] };
  citations: Citation[];
  source: 'authored';                   // = KanjiSource (CONTENT-16). A future import ADR widens this to 'imported' (data only, no page, §6)
};

type KanjiWord = KanjiSource & {        // vocabulary, incl. 熟字訓/当て字 and difficult readings
  id: ID; surface: string; reading: string; alternateReadings?: string[];
  kind: ('jukujikun'|'ateji'|'hyogai-reading'|'difficult'|'standard')[];
  meanings: string[]; level?: Level; examples?: { ja: string; en: string; source?: Citation }[];
  homophoneGroup?: ID; notes?: 'mdx'; citations: Citation[];
};

type Yojijukugo = KanjiWord & { origin?: string; synonyms?: ID[]; antonyms?: ID[]; misreadingTraps?: string[] };
type Kotowaza   = KanjiSource & { id; text; reading; meaning; origin?: string /* 故事 source text */; level?; notes?; citations: Citation[] };
type HomophoneGroup = KanjiSource & { id; reading; members: ID[]; distinctionNotes: 'mdx'; citations: Citation[] };   // 同音異義語 / 異字同訓
                                        // citations support the usage distinctions (e.g. official 異字同訓 guidance); distinctionNotes is the authored explanation
type Radical = KanjiSource & { id; char; name: string /* e.g. さんずい */; position?: string; meaning; characters: computed; citations: Citation[] };
                                        // citations support the radical's name, number and meaning
type KanjiTopic  = BaseEntry & { level?: Level[]; teaches: string[]; body: 'mdx' };   // guides
type StudyNote   = BaseEntry & { date: ISODate; tags: string[]; body: 'mdx' };
type KankenAttempt = { achievementId?: ID; level: Level; date: ISODate; result: 'pass'|'fail'|'upcoming'; score?: number; notes?: ID };
// result 'pass' REQUIRES achievementId → an Achievement with evidence + confirmedByOwner (schema refinement).
// Every KankenAttempt is a personal claim (CONTENT-13). None exist until Charles adds them.
```

## 4. Routes and navigation

The routes are listed in 01 §1. Additional rules:

- `/kanji` hub: the three doors (Journey · Reference · Notes), search, recently added entries, level progress (from `KankenAttempt`; the block is omitted when there are none), and datasets.
- `/kanji/levels/jun-1` and `/kanji/levels/1`: lists grouped by category, linking only to authored pages. Imported data is shown as compact rows without links.
- Each reference page shows: the large glyph, readings (ruby and plain text, A11Y-11), meanings, level badge, original notes, examples, related entries (computed), and citations.

## 5. Data sources and licensing (OD-23 resolved, ADR-0009)

**At launch, no third-party kanji dataset is imported.** All kanji content is **original educational content authored for this site**. The data architecture is still designed so that licensed datasets can be added later without restructuring.

| Source | Status | Notes |
|---|---|---|
| Charles's authored entries, guides, examples and notes | **Now** | Original content under **CC BY-NC 4.0** (ADR-0014). The Kanji section is **not** CC BY-SA because of a possible future dataset |
| KANJIDIC2 / JMdict (EDRDG), KanjiVG, other open datasets | **Future, not imported** | Each would need its own ADR, licence review and provenance entry. Share-alike obligations apply only to the derived dataset files and fields that contain that source's data, never to authored prose |
| Kanken level assignments | Authored per entry, with citations to public official information | Facts, cited. No proprietary compilations are copied |
| Official Kanken past papers, 漢検 question books | **Never reproduced** | At most a bibliographic citation |
| Commercial dictionaries (大漢和辞典, 新漢語林, 漢検 漢字辞典, …) | Cited for verification only | Content is never copied |

**Future-ready data architecture:**
- Reference entries are **educational content, not Claims about Charles**, so they don't carry Claim provenance. Their trust signal is `citations` (§6). For future imports, each field value records its origin (`origin: 'authored' | { sourceId }`) so merged views keep field-level attribution and licence.
- `third-party/data/{sourceId}/` (future) holds a committed normalized snapshot plus `SOURCE.md` (version, date, checksum, licence, attribution text). An import adapter maps each source into the shared entity types, and merged views keep field-level origin.
- Licences are tracked **per source and per published file**. `/kanji/data/` lists each distribution with its licence. Authored-only distributions use the authored-content licence, and any future distribution containing third-party fields gets that source's licence and attribution.
- The `source` field stays on every reference entity. Its type is the literal `'authored'` (`KanjiSource`, CONTENT-16) until an import ADR widens it to `'authored' | 'imported'`.

## 6. Quality bar (the credibility strategy)

- **No thin pages.** A reference page is generated **only** when there is original authored content: `KanjiCharacter` and `KanjiWord`/`Yojijukugo` need non-empty `notes` or at least one example; `Kotowaza` needs `notes` (its `origin` is quoted source text, not authored content); `HomophoneGroup` always has pages, because `distinctionNotes` is required; `Radical` needs at least 3 linked characters that have pages; topics and notes always have pages. Imported-only data is available through search and datasets, not as thousands of near-duplicate pages. This protects the site's credibility and search quality.
- Every **reference entry** cites at least one authoritative source (`citations.min(1)`). Reference entries are exactly the `KanjiSource` types: `KanjiCharacter`, `KanjiWord` (and so `Yojijukugo`), `Kotowaza`, `HomophoneGroup` and `Radical`. `citations` support the **reference facts** (readings, levels, meanings, distinctions). Authored explanation (`notes`, `distinctionNotes`) needs no citation of its own. Topic guides cite inline where they state facts. Study notes are personal and never require citations. None of these are Claims about Charles, so ADR-0005 provenance does not apply.
- An "Errata" link on each entry opens a pre-filled GitHub issue. Corrections are logged in the page history (the Git log is surfaced as "Revised on …").
- Internal consistency is checked in CI: homophone groups are symmetric, every radical referenced exists, the level on a word does not contradict its characters without a `note`, and there are no duplicate surface+reading pairs. A cross-check against an external dataset becomes possible only after a future import ADR.
- Personal study notes never state facts that the reference entries contradict. A link from note to reference is encouraged.

## 7. Search

- **Site-wide:** Pagefind, which supports CJK segmentation, indexes all pages.
- **Kanji search island** (`/kanji/search`):
  - `/kanji/search` is an **information page** (ADR-0016): every entry is reachable without JS through the static browse indexes below. The island is an enhancement.
  - Index: `/kanji/data/search-index*.json` (sharded when needed) built at build time (authored entries at launch) and lazy-loaded by the worker. HTTP caching only: no service worker (ADR-0016).
  - Runs in a **Web Worker** to protect INP. The island and worker are On-demand code, and the index shards are On-demand data. The island and worker (code) MAY be idle-prefetched on `/kanji/search`, and shards never are. All numbers are in PERF-06 §2.1.
  - Queries: by character or word (exact/substring), **by reading** with kana normalization (katakana↔hiragana, long vowels, and optional romaji → kana), by meaning (English), by radical/component, and by stroke count. Filters: level, category (四字熟語, 熟字訓, 国字…), content class.
  - Results link to authored pages. (If imported data is added later, imported-only results expand inline.)
  - **Without JS**, `/kanji/search` shows browse indexes (by level, radical, reading row あ〜わ, category) as static pages. Every entry is reachable in ≤ 3 clicks without JS.
- A query-string state (`?q=`) makes results shareable. The server always returns the same static page. The Initial `kanji-search` stub reads the parameter and loads the on-demand code immediately, and results render once it has loaded. Without JS, the page shows the browse indexes.

## 8. Japanese text handling

- UTF-8 throughout. NFC normalization of all content at build (a CI check). Variant selectors (IVS) are preserved where they matter and documented per entry.
- Authoring syntax for ruby in Markdown: `{漢字|かん|じ}` (per-character) or `{漢字|かんじ}` (group). A project-owned remark plugin emits `<ruby lang="ja">漢字<rp>(</rp><rt>かんじ</rt><rp>)</rp></ruby>`, and the Markdown representation emits `漢字（かんじ）`.
- A `<Ja>` component (and a Markdown `:ja[…]` directive) wraps Japanese runs with `lang="ja"` (CONTENT-08).
- Glyph forms: display fonts are chosen for 教科書体 or 明朝体 accuracy (OD-16b). Kanken grading follows 常用漢字表 / 漢検 guidance on 字体, so the notes explain the accepted variants.
- URLs use Japanese slugs (IA-04, IA-05).

## 9. Machine-readable educational resources

| Resource | Format | Contents |
|---|---|---|
| `/kanji/data/` | HTML landing page + `Dataset` JSON-LD | Description, schema, licence, provenance, changelog |
| `/kanji/data/characters.json`, `words.json`, `yojijukugo.json`, `kotowaza.json`, `homophones.json`, `radicals.json` | JSON (+ JSON Schema) | Authored entries with origin (`source`) and citations (imported fields only after a future import ADR, with per-file licence) |
| `…/*.csv` | CSV (UTF-8 with BOM for Excel compatibility) | Flat views for Anki/spreadsheets |
| `/kanji/data/anki/*.tsv` (future) | TSV importable into Anki | Study decks by level/category |
| `.md` alternates | Markdown | Every reference page |
| `DefinedTermSet` JSON-LD | Per collection | Term sets with `hasDefinedTerm` |

Licensing is per distribution (§5). At launch, every distribution is authored-only and uses **CC BY-NC 4.0** (ADR-0014). A future CC BY-SA source cannot be merged into those files and ships as separate distributions (LIC-06).

## 10. Growth plan

1. **Launch:** journey page (narrative and study approach; the achievements block renders only confirmed Achievements and is otherwise absent, not faked), 3–5 topic guides (for example "準1級 → 1級: what changes", "熟字訓 you must know", "国字 overview", "異字同訓 distinctions"), ~50 authored entries, search, and datasets.
2. **Year 1:** a steady cadence (for example 5 authored entries/week), homophone groups, 四字熟語 by theme, exam reports as notes.
3. **Later:** licensed dataset imports (each with its own ADR), stroke-order diagrams (KanjiVG, CC BY-SA, confined to its own files), Anki exports, a quiz demo under `/lab/**` (possibly Angular/RxJS per ADR-0001; it may require JS, but every entry it uses stays readable without JS on its reference page, ADR-0016), and JA-language versions of guides (OD-05).
