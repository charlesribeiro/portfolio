# 11 — Analytics

## 1. Goals

Measure whether the site produces interview conversations, without cookies, consent banners or personal data collection.

## 2. Tool (OD-09, open: analytics is adopted only through the analytics ADR. That ADR is also required by ADR-0010 whenever a third-party origin is added)

| Option | Cookies | Custom events | Cost | Notes |
|---|---|---|---|---|
| **GoatCounter** (recommended) | No | Yes | Free for personal use (donation) | Small async script (budget per PERF-06 §2.1, set by the analytics ADR). Can also power a real, era-appropriate **public hit counter** on the colophon. |
| Plausible | No | Yes (goals, props) | Paid | Polished dashboards, UTM breakdowns. |
| Cloudflare Web Analytics | No | **No** | Free | Pageviews and Web Vitals through a **client-side JavaScript beacon** served from `static.cloudflareinsights.com` (auto-injected into HTML when enabled for a Pages project). That makes it third-party JS under ADR-0016 |
| Cloudflare edge traffic and bot analytics | No | No | Included | **Server-side**, from edge request logs, with **no client script**: requests, bots and AI crawlers per path. It adds no origin, script or budget, but using it as the project's analytics source is still decided in the analytics ADR (OD-09) |
| Umami (self-host) | No | Yes | Hosting | Adds infrastructure. |

Recommendation (**proposed, pending the analytics ADR, OD-09**): Cloudflare **edge** traffic and bot analytics as the baseline, since it needs no client script, CSP or budget change, plus GoatCounter for events. Nothing is adopted until that ADR is accepted. Cloudflare Web Analytics is **not** script-free and is not enabled by default. The Pages "Web Analytics" auto-injection stays **off** unless an ADR approves the beacon.

**Any analytics ADR must state:** every script and connect origin (for the CSP); how the script loads (async, after `load`, no render blocking); its compressed size and the PERF-06 §2.1 third-party row it sets; measured LCP and INP impact; cookie and storage use (must be none); what it does when blocked or failing (the page must be unaffected); and how the edge injection setting, if any, is controlled.

## 3. Events (if adopted by the analytics ADR, OD-09)

| Event | Trigger | Why |
|---|---|---|
| `cta_contact_email` | `mailto:` click | Primary conversion |
| `cta_linkedin` | Outbound to LinkedIn profile | Secondary conversion |
| `cv_download` | `/cv.pdf` request or click (`source` = page) | Strong intent signal |
| `cv_view` | `/cv` view | Intent |
| `case_study_read` | Scrolled past 60 % of a case study (IntersectionObserver on a sentinel, first-party, counted in the Initial bucket per PERF-06 §2.1) | Engagement depth |
| `outbound_evidence` | Click on an evidence link | Trust-building behaviour |
| `machine_file` | Requests to `llms.txt`, `.md`, `resume.json`, `/data/*` | Agent interest. Measured from edge analytics (once adopted, OD-09), not JS |

If GoatCounter is adopted, events are sent through `data-goatcounter-click` attributes. There is no custom tracking code beyond the scroll sentinel.

## 4. Attribution

- LinkedIn and GitHub profile links use `?ref=li` / `?ref=gh`, and CV PDF links use `?ref=cv`. These are recorded by analytics and stripped from canonicals (IA-07).
- Referrer header data is used as reported by the tool.
- **"Interview obtained"** is recorded by Charles manually in a private spreadsheet (source, date, role), because it is the only metric that really matters.

## 5. Agent and crawler traffic

Once the analytics ADR adopts edge analytics, Cloudflare's bot analytics / AI crawl reports show which AI crawlers fetch which files. They are reviewed monthly to confirm that agents are reaching `llms.txt`, the `.md` alternates and JSON. Findings go into a short `docs/analytics-log.md`.

## 6. Privacy

- There is no consent banner, because nothing is stored on the device and no personal data is collected.
- A privacy note is published at `/colophon#privacy`.
- The analytics origin is allow-listed in the CSP only once the analytics ADR is accepted. If the analytics script fails to load, nothing breaks.
