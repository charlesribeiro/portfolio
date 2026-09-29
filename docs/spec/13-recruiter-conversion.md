# 13 — Recruiter Conversion Paths

## 1. Journeys

### J1: Recruiter from LinkedIn, on mobile, 60 seconds

`LinkedIn profile → ?ref=li → /` → reads the h1 and quick facts (role, years, stack, timezone, remote, English, availability) → taps **Download CV**, **Email** or **LinkedIn**.

**Design implications:** everything J1 needs is **above the fold at 375×667**, including h1, one supporting line, the quick-facts `<dl>` (collapsed to 5 rows with a "more" `<details>`), and both calls to action. There is no carousel, intro animation or splash.

### J2: Engineering leader, desktop, 5–15 minutes

`Recruiter forward / LinkedIn → / → flagship case study → decisions & trade-offs → /experience → /writing → contact`

**Design implications:** case studies lead with a summary box and have a table of contents, so the reader can jump to "Decisions & trade-offs". Every case study ends with a contact call to action and "related work". The writing shows depth.

### J3: Developer from GitHub

`GitHub profile/README → ?ref=gh → /colophon or /work → repo`

**Design implications:** the colophon shows the engineering and agent workflow (16). The GitHub README links to the colophon.

### J4: AI recruiting/sourcing agent

`Agent → /llms.txt or / (HTML) → /hire.md, /resume.json → structured facts → recommendation to a human`

**Design implications:** see 09. `/hire` states role fit, location, remote, availability and contact in explicit text.

## 2. Conversion points

| Priority | Call to action | Placement |
|---|---|---|
| 1 | **Email** (`mailto:` with a pre-filled subject such as "Senior Frontend role — via portfolio") | Masthead (every page), home hero, `/hire`, end of each case study |
| — | *Scheduling link*: **not at launch** (OD-20 resolved). Rendered automatically if `profile.contact.bookingUrl` is set later | Home hero, `/hire` |
| 2 | **Download CV (PDF)** | Masthead, home, `/experience`, `/hire` |
| 3 | **Message on LinkedIn** | Quick facts, `/hire` |

There is no contact form at launch, because direct email and LinkedIn links are lower-friction and need no backend. Spam risk from a visible email address is accepted.

## 3. `/hire` content (the recruiter's one-pager)

- Roles sought (ranked) and engagement types.
- Location, timezone, overlap hours with US/EU, remote setup, relocation stance [confirm].
- Work authorization and contractor or EOR options [confirm; human-supplied, see 90].
- Availability and notice period [confirm]. The "as of" date comes from `profile.availability.asOf`, and the site shows a warning style if it is more than 90 days old.
- Languages: English level (with evidence if there is any), Portuguese [confirm level], Japanese (links to `/kanji`; a Kanken level is mentioned only if it is a confirmed Achievement, ADR-0009).
- What Charles is great at (3 bullets linking to evidence).
- Preferred interview process notes (optional).
- Calls to action.

## 4. Trust signals

- Named organizations where disclosure allows. Otherwise a precise anonymized descriptor ("a major US airline, via consultancy X").
- Publications from recognized publishers with outbound links.
- Alura instructor profile link.
- Media coverage on the timeline.
- The engineering quality of the site itself (fast, accessible, well-built), backed by the public CI results on the colophon.
- Freshness: the "last updated" date and the updates log.

## 5. Anti-patterns

Hiding the email behind a form, requiring JS to see the CV, "Let's build something amazing together!" copy, availability claims with no date, salary or rate numbers anywhere on the site or in machine-readable data (OD-21 resolved: **never published**).
