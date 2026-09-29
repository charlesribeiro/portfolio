# 03 — Content Model

All content is plain text in Git under `content/`. Schemas are defined once in TypeScript (Zod, through the static-site generator's content layer) and **the build fails on any schema violation**. The schemas below are normative pseudo-TypeScript. The implementation MAY refine names but MUST keep the semantics.

## 1. Shared types

```ts
type ID = string;                 // stable, kebab-case, unique per collection; never reused
type ISODate = `${number}-${number}` | `${number}-${number}-${number}`; // YYYY-MM or YYYY-MM-DD
type Lang = 'en' | 'pt-BR' | 'ja';

type Evidence = {
  kind: 'url' | 'archive' | 'certificate' | 'publisher-listing' | 'repository' | 'press';
  label: string;                  // human-readable ("Alura course page")
  url: string;                    // canonical external URL
  archivedUrl?: string;           // web.archive.org snapshot; REQUIRED for kind 'press' (see 17)
  retrieved?: ISODate;            // when the link was last verified
};

type Disclosure = 'public' | 'anonymized' | 'private';
// public: organization, product and details may be named.
// anonymized: use `publicDescriptor` ("a major US airline"); no internal names, metrics or screenshots.
// private: never rendered; allowed in content only for internal linking/CV-offline use. Build strips it.

// Provenance (ADR-0005). Every Claim, and every fact exposed in /data/*.json, carries exactly one class.
type Provenance =
  | { class: 'verified'; evidence: Evidence[] /* min 1 */; verifiedBy: 'owner'; verifiedOn: ISODate }
      // backed by inspectable evidence AND confirmed by Charles in a human-approved PR
  | { class: 'self-reported' }
      // Charles's own statement; no public evidence. Allowed, but labelled.
  | { class: 'derived'; from: string[] /* JSON pointers or entity IDs */; rule: string }
      // computed by the build from other facts (e.g. years of experience from experience dates)
  | { class: 'external-source'; source: Evidence /* kind press|publisher-listing|url */ };
      // asserted by a third party (article, publisher page); the site attributes it and does not restate beyond it

type Claim = {
  text: string;
  provenance: Provenance;          // REQUIRED (CONTENT-04)
};

// Third-party quoted text (press headlines, their translations, excerpts).
// One rights contract for source YAML and every output (LIC-10). Never a bare string.
type QuotedText = {
  text: string;                    // ≤ 300 chars (LIC-05)
  lang: Lang;
  rights: {
    status: 'third-party-quotation' | 'third-party-derived';  // 'derived' = Charles's translation of a quotation
    holder: string;                // publisher / rights holder
    source: string;                // canonical URL of the quoted work
  };
};

type Media = {
  status: 'final' | 'placeholder'; // placeholders are build-blocked in production (CONTENT-12, ADR-0008)
  origin: 'owner-provided' | 'hand-made' | 'css-svg' | 'commissioned' | 'third-party-licensed'; // never 'ai-generated' for identity artwork
  src: string;                    // repo-relative image path (processed at build)
  alt: string;                    // REQUIRED, non-empty unless decorative: true
  decorative?: true;
  credit?: string;
  license: string;                // SPDX ID. Default 'LicenseRef-AllRightsReserved' for own photos/art (ADR-0014); third-party media keep their licence
  copyrightHolder: string;        // 'Charles …' or the third-party owner
  permissionRef?: string;         // REQUIRED when license === 'LicenseRef-UsedWithPermission': where the written permission is recorded
};

type BaseEntry = {
  id: ID;
  title: string;
  summary: string;                // ≤ 280 chars; used for meta description, llms.txt, cards
  lang?: Lang;                    // default 'en'
  updated?: ISODate;              // default: last Git commit date of the file
  draft?: boolean;                // drafts are excluded from production builds
  placeholder?: boolean;          // structural stand-in with no real facts; excluded from production, and the build FAILS if a published entry references one (CONTENT-12)
};
```

## 2. Collections

### `profile` (singleton, YAML)

```ts
{
  name: string; alternateNames?: string[];      // e.g. Japanese name rendering [confirm]
  headline: string;                             // "Senior Frontend Engineer"
  secondaryFocus: string;                       // "AI engineering: LLMs, MCP, RAG, agents"
  yearsExperienceSince: ISODate;                // years are computed from this, never hard-coded
  location: { city?: string; country: string; timezone: string /* IANA */ };
  remote: { openTo: ('remote'|'hybrid'|'relocation')[]; timezonesOverlap: string };
  workAuthorization?: string[];                 // [confirm]; human-supplied (90)
  availability: { status: 'open'|'selective'|'not-looking'; noticePeriod?: string; asOf: ISODate };
  rolesSought: string[];
  engagement: ('full-time'|'contract')[];
  contact: { email: string; linkedin: string; bookingUrl?: string };   // bookingUrl: optional, unused at launch (OD-20 resolved: direct links)
  sameAs: { linkedin: string; github: string; alura?: string; [k: string]: string | undefined };
  languages: { lang: string; level: 'native'|'C2'|'C1'|'B2'|'B1'|'A2'|'A1'; evidence?: Evidence[] }[];
}
```

### `experience`

```ts
BaseEntry & {
  // Who employed or contracted Charles. For contract work, this is the contracting company or Charles's own business.
  organization: { name?: string; publicDescriptor: string; url?: string; disclosure: Disclosure };
  // The organization the work was delivered FOR, when different from the employer.
  // Rendered as "Contractor for {client}", never "at {client}" or as an employee of the client (ADR-0006).
  client?: { name?: string; publicDescriptor: string; disclosure: Disclosure; approvedFacts?: string[] };
  relationship: 'employee' | 'contractor' | 'freelance' | 'consultancy-placement';
  role: string;
  employmentType: 'full-time'|'contract'|'freelance'|'part-time';
  start: ISODate; end?: ISODate;                // end absent = current
  location: string; remote: boolean;
  highlights: Claim[];                          // 3–5
  skills: ID[];                                 // → skill
  projects?: ID[];                              // → project
}
```

### `project` (case studies are projects with `caseStudy` set)

```ts
BaseEntry & {
  period: { start: ISODate; end?: ISODate };
  experience?: ID;                              // → experience
  disclosure: Disclosure;
  themes: ('frontend-architecture'|'modernization'|'monorepo'|'performance'|'accessibility'
          |'design-systems'|'ai-engineering'|'teaching'|'open-source')[];
  role: string; teamSize?: string;
  skills: ID[];
  caseStudy?: { flagship: boolean; body: 'mdx' };  // body authored in the entry's MDX file (see 14)
  outcomes: Claim[];                            // metrics MUST have evidence or be qualitative (CONTENT-05)
  links?: Evidence[];
  cover?: Media;
}
```

### `publication`

```ts
BaseEntry & {
  type: 'book'|'book-chapter'|'article'|'course'|'talk'|'video'|'podcast';
  role: 'author'|'co-author'|'instructor'|'contributor'|'speaker';
  coAuthors?: string[];
  publisher: { name: string; url?: string };    // e.g. Alura; the actual publisher per item is human-supplied [confirm]
  date: ISODate;
  language: Lang;
  url: string;                                  // canonical external URL
  isbn?: string;
  skills?: ID[];
  cover?: Media;
  body?: 'mdx';                                 // self-hosted excerpt/notes if permitted
}
```

### `skill`

```ts
{ id: ID; name: string; category: 'frontend'|'language'|'tooling'|'ai'|'practice';
  aliases?: string[];                           // "TS" → TypeScript; used by search, never displayed as stuffing
  wikidata?: string; }                          // e.g. Q978185 for TypeScript; used in JSON-LD `sameAs`
```

There are **no self-rated proficiency levels or skill bars.** Seniority is shown through the evidence graph: which experiences and projects used the skill, and for how long (computed).

### `achievement`

```ts
BaseEntry & {
  kind: 'certification'|'exam'|'award'|'recognition';
  issuer: { name: string; url: string };        // e.g. 日本漢字能力検定協会
  date: ISODate;
  level?: string;                               // "準1級"
  credentialId?: string;                        // published only if Charles explicitly approves
  evidence: Evidence[];                         // REQUIRED, min 1 (e.g. redacted certificate scan hosted here)
  domain: 'kanji'|'professional'|'education'|'other';
  confirmedByOwner: true;                       // REQUIRED literal. Only valid in a PR approved by Charles (CONTENT-13, ADR-0009)
}
// Achievements are always provenance 'verified'. Without evidence plus owner confirmation, the fact cannot be published
// as an achievement. At most it can appear as a self-reported study-note statement, and only if Charles writes it.
```

### `education`

`BaseEntry & { institution; credential; field; start; end?; evidence? }`

### Provenance exposure

- `/data/*.json` emit a `provenance` object next to every claim-bearing field (schema published under `/data/schema/`).
- The `.md` alternates render provenance as a compact suffix, e.g. `— *verified* ([certificate](…))`, `— *self-reported*`.
- HTML shows provenance unobtrusively: evidence links sit next to claims, self-reported claims are unmarked in prose but labelled in fact boxes, and the legend is at `/colophon#provenance`.
- JSON-LD exposes evidence relationships as described in 09 §3.1.

### `mediaMention` and `timelineEntry`

These are specified in [17-timeline.md](17-timeline.md).

### Kanji collections

`kanjiCharacter`, `kanjiWord`, `yojijukugo`, `kotowaza`, `homophoneGroup`, `kanjiTopic`, `radical` and `studyNote` are specified in [18-kanji.md](18-kanji.md).

## 3. Relationships and integrity

- References use typed `reference()` fields. The build fails on dangling references (CONTENT-01).
- Back-references (for example Skill → Projects) are computed by the domain layer (04 §3) and are never authored.
- IDs are immutable once published (IA-06).

## 4. Content rules (enforced)

| ID | Rule | Enforcement |
|---|---|---|
| CONTENT-01 | No dangling references. | Schema/build |
| CONTENT-02 | `disclosure: private` content never appears in `dist/`. | Post-build scan |
| CONTENT-03 | Confidential terms denylist (client-internal names, codenames, colleagues' names) never appears in `dist/`. The denylist is stored as salted hashes of normalized tokens in the repo, so the list itself does not leak. | Post-build scan |
| CONTENT-04 | Every `Claim` has a `provenance`. `verified` requires ≥ 1 evidence item. `derived` requires `from` + `rule`. | Schema |
| CONTENT-05 | Numeric outcomes (%, ×, ms, $) require evidence or an approved-by-client note. Otherwise, use qualitative wording. | Lint on `Claim.text` |
| CONTENT-06 | Every `Media` has `alt` (or `decorative`) and a `license`. | Schema |
| CONTENT-07 | `summary` ≤ 280 characters. Titles are unique within a collection. | Schema |
| CONTENT-08 | Japanese strings inside English content use the ruby/lang markup helpers (18 §8). A raw run of CJK characters outside `lang="ja"` in the output fails the check. | Post-build HTML scan |
| CONTENT-09 | Skills without at least one supporting entity are not rendered. | Domain layer + test |
| CONTENT-10 | `yearsExperience` is computed, never written in prose. Prose uses the `<Years/>` component or `{{years}}` in Markdown. | Lint for `\d+\+? years` in content |
| CONTENT-11 | Every external `url` passes the weekly link check. Press links also have `archivedUrl`. | Scheduled job |
| CONTENT-12 | No `placeholder: true` entry and no `Media.status: 'placeholder'` asset appears in a production build. The production build fails if one is referenced by published content. Preview builds render placeholders with a visible "PLACEHOLDER" treatment. | Build + dist scan |
| CONTENT-13 | PRs that add or change Claims, Achievements, KankenAttempts, MediaMentions, `profile.yaml` or client disclosure fields get the `needs-human:claims` label from the labeller, and G21 reports each new or changed claim with its provenance in the job summary. Merge requires Charles's approval (ADR-0011). | Labeller + G21 |
| CONTENT-14 | Client-sensitive fields (`client.*`, case studies with a named client) may contain only facts listed in `client.approvedFacts` or already public. New facts need human approval. Uncertain status defaults to anonymization. | Review + G21 |
| CONTENT-15 | Agents MUST NOT invent article titles, dates, quotations, URLs, certifications, metrics or employers. Unknown values stay absent, or the entry stays a placeholder. | Review + CONTENT-12 |
| CONTENT-16 | Every kanji **reference** entity (`KanjiCharacter`, `KanjiWord` and therefore `Yojijukugo`, `Kotowaza`, `HomophoneGroup`, `Radical`) carries `source` and accepts only `'authored'` until an import ADR widens the schema. `KanjiTopic` and `StudyNote` are authored prose by definition. No third-party dataset files exist under `third-party/data/` without an ADR. | Schema + G23 path rule |

## 5. Authoring formats

| Kind | Format |
|---|---|
| Structured entities (profile, experience, skill, achievement, publication metadata, timeline, mediaMention) | YAML |
| Long-form (case studies, about, articles, topics, notes) | MDX with front-matter. MDX is used only for a small allow-list of components (Figure, Callout, Ruby, KanjiRef, Evidence). |
| Kanji reference entries | YAML, one file per authored entry. Imported data only after a future import ADR (18 §5) |

Components allowed in MDX are listed in `content/README.md`. An MDX file that imports anything else fails lint, so that agents cannot introduce arbitrary code into content.
