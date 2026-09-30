# 02 — Page Structure

All pages share the **shell**:

```
<body>
  skip link → #main
  <header> masthead banner (site title as text, not image) · CV · Contact · primary <nav>
  <div class="layout">          ← portal grid: nav column | main | sidebar (collapses on narrow screens)
    <main id="main"> <article> …page content… </article> </main>
    <aside> sidebar boxes (page-specific) </aside>
  </div>
  <footer> sitemap · machine links · licence line (20 §3) · build info
</body>
```

Every page has exactly one `<h1>`, landmarks, a breadcrumb (`<nav aria-label="Breadcrumb">` plus BreadcrumbList JSON-LD) except on the home page, a "last updated" date from content or Git, and a `<link rel="alternate" type="text/markdown">`.

The order of sections below is normative. Visual treatment is covered in 12.

## Home `/`

Sections 4–7 are omitted while the domain layer has no published data for them, and are never filled with stand-ins (D3, ADR-0008). Sections 1–3 and 8 are always present.

1. **Positioning block.** `<h1>` is "{profile.name} — Senior Frontend Engineer". One sentence adds AI engineering. One sentence gives proof points ({years}+ years, computed per CONTENT-10; Angular/TypeScript/RxJS/Nx/React; enterprise modernization; large-scale systems). No emoji greeting.
2. **Quick facts box** (the portal-era "profile card"): role sought · years · core stack · location and timezone · remote/relocation · English level · work authorization [confirm] · availability · links (LinkedIn, GitHub, email). Marked up as `<dl>`, which is also the most agent-friendly block on the page.
3. **Primary calls to action:** "Download CV" and "Contact" (direct links: email, LinkedIn; see 13).
4. **Flagship work:** 3 case-study cards (for example Pilot Bidding System, an enterprise modernization, and one AI engineering project).
5. **Writing and teaching:** publications and Alura courses, with covers where licensing allows.
6. **AI engineering strip:** one paragraph and 2–3 links into `/ai`. Visually secondary.
7. **Elsewhere on the site:** small teaser boxes for Timeline (generated from confirmed `featured` entries, and omitted if there are none) and Kanji (links to the reference area and journey, with no implied level).
8. **Updates log** (the old-web "News" box): the latest 5 dated changes across the site, generated from content dates.

Sidebar: quick facts (on desktop the facts move here), badges, "last updated".

## Case study `/work/{slug}`

See [14-case-studies.md](14-case-studies.md) for the full template: summary box → context → problem → constraints → role → approach → decisions and trade-offs → outcome → what I'd do differently → stack → evidence → related.
Sidebar: facts box (period, role, team size, disclosure level, stack), table of contents.

## Experience `/experience`

A reverse-chronological list of Experience entities. Each shows organization (or its anonymized descriptor), role, period, location/remote, a 2–3 line summary, 3–5 highlights, stack, and links to related case studies. There is also a section for Education, Languages and Certifications. The page links to `/cv` and `/cv.pdf`.

## CV `/cv`

The same data as `/experience`, condensed to two A4/Letter pages. It has a print stylesheet and no site chrome when printed. `/cv.pdf` is rendered from this page at build time (see 04 §6).

## AI hub `/ai`

1. Framing paragraph: AI engineering as part of product and frontend engineering (see 15).
2. Work: AI-related case studies.
3. Writing and teaching: the SLM publication, the Tailwind CSS + AI publication, and relevant Alura material.
4. "How this site is built with agents": links to the colophon and agent workflow docs.
5. Capabilities list (LLMs, MCP, RAG, agents, evals). Each item links to evidence and nothing appears without it.

## Writing `/writing`

Grouped by type: Books/Long-form · Courses (Alura) · Articles · Talks. Each item shows title, role (author/co-author/instructor), publisher, date, language, an outbound canonical link, and a detail page when self-hosted.

## Timeline `/timeline`

See [17-timeline.md](17-timeline.md). There is a year-grouped chronological feed with category filters as links (`?category=` is not used; filtered views are static pages at `/timeline/category/{cat}`). Media entries are shown as clippings.

## Kanji hub `/kanji`

See [18-kanji.md](18-kanji.md). It has three clearly labelled doors: **Journey (achievements)** · **Reference** · **Study notes**, plus search and datasets.

## About `/about`

A long-form biography in the first person, written by Charles. It covers professional arc, teaching and writing, Japanese, and a short personal section. It also includes Languages (with proficiency and evidence) and Education.

## Hire `/hire`

For recruiters and agents: roles sought, role fit, engagement types (full-time/contract), timezone overlap, remote setup, notice period [confirm], the interview process Charles prefers, contact options, and a CV. See 13.

## Colophon `/colophon`

The stack, architecture diagram, performance budgets next to the sizes measured by the same build, accessibility statement, CI badges (real, generated), dependency count, agent workflow, and a link to the public repository (ADR-0006), and the provenance legend (`#provenance`, ADR-0005).

## 404

The retro "page not found" treatment, with search (no-JS: a link to `/site-index`), the top links and a link to report a broken URL (a pre-filled GitHub issue URL).
