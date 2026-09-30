# 17 — Timeline & Media Coverage (`/timeline`)

## 1. Purpose

`/timeline` is a first-class editorial view of Charles's personal and professional history, including **external media coverage** (for example G1, Folha and others). Charles will provide the specific articles later (OD-22 resolved). **Until then, the timeline ships only the content model and placeholders. Agents MUST NOT invent article titles, dates, quotations or URLs** (CONTENT-15). Humans get the story arc. Agents and recruiters get dated, sourced facts that anyone can check.

## 2. Content model

```ts
// content/media/*.yaml: one file per external article/broadcast
type MediaMention = BaseEntry & {
  publisher: { name: string; url: string; wikidata?: string };   // "G1", "Folha de S.Paulo"
  headline: QuotedText;                // original headline, verbatim (rights.status 'third-party-quotation'; lang usually 'pt-BR')
  headlineTranslation?: QuotedText;    // English gloss by Charles (rights.status 'third-party-derived')
  datePublished: ISODate;
  url: string;                         // canonical external URL
  archivedUrl: string;                 // REQUIRED web.archive.org (or archive.today) snapshot
  format: 'article'|'video'|'radio'|'podcast'|'print';   // selects the JSON-LD press-node type (§5, D6)
  copyrightHolder?: { type: 'Organization'|'Person'; name: string };  // WORK-level: legal copyright holder of the whole press work
                                       //   (article, broadcast, episode). Independent of `publisher` and of the field-level
                                       //   `headline.rights.holder`; never derived from either
  copyrightHolderConfirmed?: true;     // Owner Confirmation of `copyrightHolder`. Schema refinement: both present or both absent.
                                       //   Only then is `copyrightHolder` emitted in JSON-LD (D6). Absent = unknown, not emitted
  author?: string;
  excerpt?: QuotedText;                // short quotation ≤ 300 chars (§4)
  image?: Media;                       // only own media ('LicenseRef-AllRightsReserved', copyrightHolder Charles), openly licensed media (SPDX, e.g. 'CC-BY-4.0'), or 'LicenseRef-UsedWithPermission' + permissionRef (§4)
};

// content/timeline/{id}.yaml: facts, CC0 (20 §1.1)
// content/timeline/{id}.md:   editorial context, "why it mattered" (CC BY-NC). REQUIRED for published entries; long-form entries get /timeline/{slug}
type TimelineEntry = BaseEntry & {
  date: ISODate;                       // precision matters: YYYY-MM allowed
  dateDisplay?: string;                // "Spring 2008" when precision is fuzzy
  category: 'career'|'education'|'publication'|'teaching'|'press'|'award'|'kanji'|'personal'|'community';
  // context: in the sibling .md file, never inline (LIC-08)
  source?: Evidence[];                 // for non-press entries
  mediaMention?: ID;                   // → MediaMention. REQUIRED when category === 'press' (schema refinement): the
                                       //   MediaMention carries publisher, canonical URL and archive, the source provenance
  image?: Media;                       // OPTIONAL for every category, press included. Historical coverage often has no
                                       //   image that may be published, and the entry renders fine without one
  related?: { collection: 'experience'|'project'|'publication'|'achievement'|'education'; id: ID }[];
  featured?: boolean;                  // shown on home "Elsewhere" teaser
  longForm?: boolean;                  // true → the sibling .md gets its own /timeline/{slug} page.
                                       //   Reserved ids (schema rejects them): 'category', 'from-the-start'
};
```

## 3. Experience (human)

- **Newsprint archive skin** (12 §4, Archive identity): year headers as mastheads, entries as clippings in a single-column flow on mobile and a two-column "broadsheet" on wide screens. Press entries render as a typographic **reconstructed clipping**: publisher name, headline and date typeset by the site, *not* a scan or screenshot. A clear "Read at {publisher} ↗" link and an "Archived copy" link follow.
- Category filtering uses static pages (`/timeline/category/press`, …) with links styled as tabs. Year jump links are `#y2008` anchors.
- The ordering is reverse-chronological by default, with a "read from the beginning" link to the ascending static page (`/timeline/from-the-start`).
- Scroll-driven reveal effects are progressive and respect reduced motion.

## 4. Media rights (default policy)

| Asset | Default |
|---|---|
| Headline, publisher, date, link | ✅ Always (facts and links) |
| Short excerpt | ✅ ≤ 300 characters, quoted and attributed, only when it adds something (quotation for criticism or reference) |
| Publisher's photos, video frames, article screenshots | ❌ Not without written permission (`license: 'LicenseRef-UsedWithPermission'` + `permissionRef`) |
| Charles's own photos from the event | ✅ `license: 'LicenseRef-AllRightsReserved'`, copyrightHolder Charles |
| Full article text | ❌ Never |

## 5. Machine-readable representation

- `/timeline` HTML: `<ol reversed>` of `<article class="h-entry">`, each with `<time datetime>`, `<h3>` title, `p-summary`, and `u-url` for the source.
- JSON-LD: a `CollectionPage` whose `mainEntity` is an `ItemList` of entries. Each press entry is a **press node** typed by `format` (D6): `article`/`print` → `NewsArticle`, `video` → `VideoObject`, `radio` → `RadioEpisode`, `podcast` → `PodcastEpisode`. Its properties are `headline` (a quotation, LIC-10), `inLanguage`, `datePublished`, `publisher` Organization, `url`, `archivedAt`, `about` → `#person`, with no `sdLicense` (LIC-10). It has `copyrightHolder` (an `Organization` or `Person` built from the work-level `MediaMention.copyrightHolder`) **only** when `copyrightHolderConfirmed: true`; otherwise it is omitted, never inferred from `publisher` or `headline.rights.holder`, and the `Person` node has `subjectOf` pointing to them.
- `/timeline.md`: a chronological Markdown list with source links.
- `/data/timeline.json`: the structured **facts** (date, title, category, publisher, URLs, related IDs), with a schema, under CC0 (20 §1.1). Each entry links to `/timeline#<id>` (or `/timeline/<slug>` for `longForm` entries) for the editorial context (CC BY-NC). Third-party headlines, their translations and excerpts inside it are quoted, not relicensed. They carry field-level `rights` and are listed in `licenseExclusions` (LIC-10).
- `/feeds/timeline.xml`: Atom feed, with a per-entry `<rights>` naming the holder of any quotation (LIC-10).
- HTML quotations use `<q cite>` / `<blockquote cite>` with a visible `<cite>` attribution. `/timeline.md` lists them as full `QuotedText` objects in front-matter `thirdPartyQuotations` (LIC-10).
- `/llms-full.txt` includes the timeline.

## 6. Placeholders (until Charles provides articles)

- `content/media/_placeholder-*.yaml` and `content/timeline/_placeholder-*.yaml` have `placeholder: true`. Fields with no real value are omitted, not filled with plausible fakes. Obvious sentinels such as `title: "PLACEHOLDER — press item 1"` and `url: "https://example.invalid/"` are allowed, because `.invalid` can never resolve.
- The schema allows placeholders to omit fields that are otherwise required (`archivedUrl`, `datePublished`), and a press placeholder may omit `mediaMention`, but only when `placeholder: true`.
- Preview builds render placeholders with a striped "PLACEHOLDER" treatment so layout and templates can be developed. Production builds exclude them (CONTENT-12). If no real entries exist, `/timeline` shows an honest "archive being assembled" state, and machine outputs emit empty lists instead of fake items.
- Adding real media entries is a `needs-human:claims` PR (CONTENT-13). Charles supplies the URL, and the agent may fetch public metadata to prefill the entry, but Charles confirms every field before merge.

## 7. Editorial rules

- Every press entry has `archivedUrl`, because news links rot (CONTENT-11).
- The context `.md` file is Charles's own voice and says why the entry matters now, not just what happened.
- Personal-category entries are opt-in and minimal. Sensitive personal history is never published (human review in the PR).
