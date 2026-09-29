---
status: accepted
date: 2026-09-28
---

# Original assets only; placeholders can never reach production

The visual identity uses original CSS/SVG interface design, manually designed assets, era-inspired graphical elements made for this site, and assets Charles explicitly provides. Generic AI-generated artwork is not used as identity. The same rule covers content: agents never invent personal facts, article titles, dates, quotations, URLs or metrics. Development can still proceed with **placeholders**. Each one is explicitly marked (`placeholder: true` for entries, `status: 'placeholder'` for media), rendered with a visible placeholder treatment in previews, excluded from production, and **fails the production build** if published content references it. A placeholder therefore cannot quietly become production artwork or content.

## Consequences

- Every asset records its origin and licence.
- Empty states (for example a timeline with no confirmed articles) are designed as honest states and are never filled with plausible fake data.
