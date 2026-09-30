# Portfolio

Charles's public professional portfolio. It turns recruiters and engineering leaders into interview conversations, is equally legible to AI agents, and hosts a growing kanji/Kanken knowledge base. This file holds stable domain context and vocabulary. Technical decisions live in `docs/adr/`, and requirements live in `docs/spec/`.

## Stable context

**Owner**: Charles, the subject and author of the site, and the only person who can confirm personal facts or approve merges.

**Positioning** (in this order, never flattened into equal "skills"):
1. **Senior Frontend Engineer**: the primary identity (Angular, TypeScript, RxJS, Nx, React; enterprise frontend architecture and modernization).
2. **AI engineering**: a secondary, growing specialization presented as an extension of frontend and product engineering.
3. **Depth signals**: technical writing and teaching (Alura; publications on Tailwind CSS + AI and Small Language Models) and advanced Japanese/kanji.

**Audiences**: technical recruiters (fast, often mobile), engineering leaders (deep reading), developers and readers, kanji learners, and AI agents (search, recruiting, research).

**Invariants**:
- Nothing about Charles is invented. Unknown facts stay unknown.
- Every Claim has a Provenance.
- Client work is described only within its Disclosure level. United Airlines was a **Client**, and Charles worked as a **Contractor**, never as a United employee.
- A Kanken Achievement exists only with Evidence and Owner Confirmation.
- Reference content, Study Notes and Achievements are never mixed.
- Placeholders never reach production.
- Humans merge. Agents never do.
- Being discoverable by agents is not the same as granting permission to train on the site.
- Every repository source file has exactly one licence boundary (generated aggregates such as `llms-full.txt` declare a licence per section): Code, Professional metadata, Editorial content, Personal imagery and artwork, or Third-party material.
- Professional information, evidence, articles, timeline content, kanji educational content and navigation stay understandable without client-side JavaScript. Only Lab demos may require it.
- A failing gate is fixed in the code under test, never by weakening the gate.

## Language

### People and organizations

**Owner**:
Charles, as the authority for personal facts and approvals.
_Avoid_: user, admin

**Employer**:
The organization that employed or contracted Charles for a piece of work. For contract work, this is the contracting company, not the end client.
_Avoid_: company (ambiguous)

**Client**:
The organization that work was delivered *for*, when it differs from the Employer (for example United Airlines).
_Avoid_: customer, employer

**Contractor**:
Charles's relationship to a Client when engaged through an Employer or through Charles's own business rather than employed by the Client.
_Avoid_: consultant (unless literally true), employee

### Claims and trust

**Claim**:
A factual statement about Charles that could be true or false, such as a role, date, achievement, metric or scope of work.
_Avoid_: bullet, highlight (when a fact is meant)

**Provenance**:
The class that says why a Claim is believed: Verified, Self-reported, Derived or External-source.
_Avoid_: confidence, trust level

**Verified**:
Provenance for a Claim backed by inspectable Evidence and confirmed by the Owner.

**Self-reported**:
Provenance for a Claim stated by the Owner with no public Evidence.

**Derived**:
Provenance for a Claim computed from other Claims by a documented rule (for example years of experience).

**External-source**:
Provenance for a Claim asserted by a third party, such as a news article or publisher listing, which the site attributes rather than restates as its own.

**Evidence**:
An inspectable source supporting a Claim: a public URL, an archived copy, a certificate or a publisher listing.
_Avoid_: proof, reference (reserved for Reference content)

**Owner Confirmation**:
An explicit approval by the Owner, given in a reviewed pull request, that a personal Claim may be published.

**Disclosure level**:
How much of a Client engagement may be published: public, anonymized or private.
_Avoid_: confidentiality level, NDA level

**Approved Facts**:
The explicit list of statements the Owner has cleared for a Client. Nothing beyond it is published about that Client.

### Content

**Entity**:
A typed content item with a stable identity, such as an Experience, Project, Publication, Achievement, Timeline Entry, Media Mention or kanji entry.

**Representation**:
One rendered form of the same content: page, Markdown alternate, structured data, dataset, feed or CV.
_Avoid_: version, copy, export

**Case Study**:
A Project told through the fixed narrative template (context, problem, constraints, role, approach, decisions and trade-offs, outcome, reflection).
_Avoid_: portfolio item, showcase

**Timeline Entry**:
A dated moment in Charles's personal or professional history, optionally tied to a Media Mention or other Entities.
_Avoid_: event, post

**Media Mention**:
External coverage of Charles by a publisher (for example G1 or Folha), recorded with its original headline, publisher, date, canonical URL and archived copy.
_Avoid_: press item, article (ambiguous with Charles's own articles)

**Placeholder**:
Stand-in content or artwork that is explicitly marked, visible as such in previews, and excluded from production.
_Avoid_: dummy, mock, sample (when it could be mistaken for real content)

**Fixture content**:
A separate, clearly fictional content set (`content/__fixtures__/`) used by tests and by gate builds, so neither waits for real facts. It is selected as a whole at build time, never mixed into real content, and never published. Placeholders, by contrast, live inside the real content.
_Avoid_: sample data, demo content, placeholder (a different concept)

### Kanji area

**Achievement**:
A verifiable personal accomplishment, such as passing a Kanken level, published only with Evidence and Owner Confirmation.
_Avoid_: certification (unless it literally is one), badge

**Reference content**:
Educational material meant to be accurate and citable: characters, vocabulary, 四字熟語, 故事・ことわざ, 熟字訓/当て字, 国字, readings, radicals and distinction guides.
_Avoid_: dictionary (the site is not a full dictionary), database

**Study Note**:
A personal, first-person learning record that is explicitly non-authoritative.
_Avoid_: article, lesson

**Content class**:
Which of Achievement, Reference content or Study Note a kanji-area page belongs to. Every page has exactly one.

**Authored content**:
Original material written for this site by the Owner, as opposed to Imported data.

**Imported data**:
Content taken from a third-party dataset under its own licence. None exists yet, and adding any requires a decision record.

**Kanken**:
The Japan Kanji Aptitude Test (日本漢字能力検定, 漢検). The focus levels are 準1級 and 1級.
_Avoid_: JLPT (a different exam)

### Machine audiences

**Discovery crawler**:
Any automated reader that finds, indexes, retrieves or cites the site for search, AI search, recruiting, research or a user's direct request. Always welcome.
_Avoid_: AI crawler (ambiguous)

**Training crawler**:
An automated reader that collects content for model training. Its access is a separate, explicit policy choice.
_Avoid_: AI crawler (ambiguous)

**Crawler policy**:
The declared rules for each crawler category, from which `robots.txt` is produced.

### Licensing

**Licence boundary**:
The category that decides a file's licence: Code (MIT), Professional metadata (CC0), Editorial content (CC BY-NC 4.0, including kanji educational content), Personal imagery and artwork (All Rights Reserved), or Third-party material (its original licence).

**Professional metadata**:
Factual, structured information about Charles's professional profile, meant for machine reuse, such as profile, experience, publications, project facts, timeline facts and the professional nodes of structured data. It is dedicated to the public domain and never contains editorial prose.
_Avoid_: profile data (ambiguous), CV text

**Editorial content**:
Prose written by the Owner, such as case-study bodies, articles, the about page, timeline context and kanji educational material.

**Third-party material**:
Anything not created by the Owner, such as fonts, future datasets or quoted excerpts. It keeps its own licence and always carries provenance.
_Avoid_: external assets, vendor files

### Collaboration

**Agent**:
An autonomous coding worker on bubbles-server. It implements issues and opens pull requests through the Agent App, and never merges.
_Avoid_: bot (except for the GitHub account name), assistant

**Agent App**:
The single GitHub App identity that all Agents act through. It is scoped to this repository and has no merge, admin or bypass rights.
_Avoid_: service account, machine user

**Merge authority**:
The Owner's human GitHub identity, the only one that can approve and merge into `main`.

**Gate**:
An automated check registered in the gate manifest as planned or active. Active pre-deploy and post-deploy gates must pass before a pull request can be merged (except PRs merged before the required checks exist, namely the pre-bootstrap and bootstrap PRs, and approved gate removals, all of which the Owner merges with the admin bypass), and scheduled gates report through issues.

**Gate weakening**:
Any change that makes an active Gate less strict, or that can't be objectively shown not to: exactly the signals W1–W6 in `docs/spec/06-testing-and-ci.md` §1.2. It always needs the Owner's approval.

**Information page**:
Any page whose purpose is to convey professional information, evidence, articles, timeline content, kanji educational content or navigation, i.e. every route outside `/lab/**`. It never requires JavaScript.

**Lab demo**:
An interactive demonstration where the interaction itself is the product. It may require JavaScript and lives apart from Information pages.
_Avoid_: island (an island enhances an Information page)

**Issue claim**:
A worker run's declared ownership of an issue, started by a claim or handoff marker comment (posted by the portfolio App) carrying its worker slot and a supervisor-minted run ID. The App-authored markers are authoritative, and the `agent:claimed` label is only a hint. It lasts until a release for that run, or a supervisor handoff to a successor run that inherits its rank, and it is kept while the run's PR awaits review. Always write "issue claim", never a bare "claim", which means a Claim about Charles.
_Avoid_: assignment (the Agent App is not configured as an assignable agent app, so Agents can't be assignees)

**Human-approval category**:
One of the reasons a change is flagged for the Owner's review: architecture decisions, significant dependencies, personal Claims, client-sensitive information, CI/security infrastructure, Gate weakening, Owner policy (crawler and licence files), or visual baselines. Every merge needs the Owner regardless.
