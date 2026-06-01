# Project 05: Dashboard Builder Agent *(Capstone)*

**Externship:** Wayfair AI Automation via Extern
**Category Focus:** Area Rugs
**Workflow:** Dashboard Builder Agent
**Status:** ✅ Complete — Delivered
**Role:** Capstone — aggregates outputs from all prior projects into a unified executive dashboard

---

## Overview

The Dashboard Builder Agent is the capstone of the Wayfair AI Automation Externship. Rather than generating new analysis, it acts as an internal tooling layer — pulling the HTML reports produced by Project 02 (Market Trend Discovery) and Project 03 (Competitor Monitoring) from Google Drive, parsing their structured content using Cheerio, and assembling everything into a single, polished **Category Manager Dashboard** in under a minute.

Where Projects 02–04 were analysts producing detailed reports, Project 05 is the system that converts those reports into an executive briefing a category team can skim in minutes.

> **No AI reasoning is used in this project.** All data already exists in upstream outputs. Cheerio handles extraction; the workflow handles assembly.

---

## Why a Dashboard?

A dashboard is a visual display that surfaces the most important information from multiple sources in one place — designed for fast comprehension and faster decisions. In a real-world scenario, a **category manager at Wayfair** would use this dashboard every Monday to answer:

* What rug styles are trending right now?
* How does Wayfair's pricing compare to Amazon and Walmart?
* Where do whitespace opportunities exist?
* What should the team prioritize this week?

Without a dashboard, answering those four questions requires manually opening multiple reports, skimming for the relevant sections, and mentally synthesizing them. This agent does all of that in one automated run.

---

## Sample Output: What the Agent Produces

📄 `Area_Rug_Dashboard.html` — generated 2026-05-31

A fully interactive, 6-page executive dashboard built from the P02 trend report and P03 competitor report. Renders in any browser, no dependencies required beyond Google Fonts.

### Dashboard Pages

| Page | Title | What It Shows |
|---|---|---|
| 1 — Overview | Executive Overview | One-glance category health: target segment, Wayfair avg price, total products analyzed, key insights from P02 and P03 |
| 2 — Market & Trends | Market & Trends | P02 data: market size metrics, micro-segment breakdown, color palette, moodboard visuals, attribute distribution bars, trend-based recommendations |
| 3 — Competitive | Competitive Intelligence | P03 data: Wayfair vs Amazon vs Walmart comparison table, pricing by retailer, price band distribution, discount tier analysis |
| 4 — Opportunities | Opportunity Radar | P03 whitespace gaps, price gap heatmap, opportunity cards ranked by priority, key supplier callouts |
| 5 — Diagnostics | Risk & Diagnostics | P02 risk cards (expandable), high/medium/low risk counts, trend risk breakdown |
| 6 — Actions | Action Center | Ranked quick wins, strategic initiatives, trend-based recommendations, supplier table, action tracker with owner + due date |

### KPI Snapshot (from actual run)

| Metric | Value | Source |
|---|---|---|
| Products Analyzed | 68 (Wayfair: 48, Amazon: 10, Walmart: 10) | P03 scrape |
| Target Segment | Organic Modern Neutral | P02 micro-segment 1 |
| Wayfair Avg Price | $277.85 | P03 pricing data |
| Market Size (2025) | USD 62.90 Billion | P02 market research |
| Projected Market (2033) | USD 130.62 Billion | P02 market research |
| CAGR (2026–2033) | 9.59% | P02 market research |
| Fastest Growing Segment | Premium | P02 analysis |

### Action Tracker (from actual run)

| Action | Priority | Owner | Timeline |
|---|---|---|---|
| Launch Machine Washable Collection | High | Merchandising | +14 days |
| Spotlight Designer Collaborations | High | Marketing | +7 days |
| Establish Value Line ($50–$100) | High | Sourcing | +30 days |
| AI Style Concierge Tool | Medium | Product | +60 days |

### Suppliers Identified (from P03)

SIXHOME, VUNATE, My Texas House, Drew Barrymore (Beautiful Collection), Chris Loves Julia x Loloi, OLANLY — each with platform badge, price range, and direct search link.

---

## What Makes This Project Different

| Projects 02–04 | Project 05 |
|---|---|
| Generate new insights via AI | Aggregates existing insights — no new AI calls |
| Output: individual HTML reports | Output: unified 6-page interactive executive dashboard |
| Data sources: Amazon, social APIs, competitor pages | Data sources: P02 + P03 HTML reports on Google Drive |
| Core skill: prompt engineering + n8n workflow logic | Core skill: HTML parsing (Cheerio) + Google Drive OAuth |
| 10–20 min runtime | Under 1 minute runtime |

---

## Workflow Architecture

```
[STAGE 1] Google Drive Access (OAuth)
  Run Manually Trigger
    └── Find Wayfair Reports Folder (Google Drive search)
          └── List Root Contents → Identify Category & Template

[STAGE 2] File Retrieval
  Download Template (HTML dashboard template from Drive)
    └── Store Template → List Date Folders
          └── Get Latest Date Only (YYYY-MM-DD sort, descending)
                └── List P2/P3 Files → Identify P2 & P3 Files

[STAGE 3] Parallel Download
  Download P2 (Market Trend Discovery HTML)
    └── Rename P2 Binary (key: p2_binary)

  Download P3 (Competitor Monitoring HTML)
    └── Rename P3 Binary (key: p3_binary)

  Merge P2 & P3

[STAGE 4] Cheerio Parsing
  Parse P2 & P3 Reports (Code node — Cheerio)
    ├── P2: key insight, market metrics, micro-segments, color palette,
    │       risk ratings, recommendations, moodboard images (base64),
    │       attribute distribution bars (materials, sizes, colors, prices)
    └── P3: key insight, product counts by retailer, price positioning
            (avg/min/max per retailer), comparison table, price bands,
            price gap heatmap, whitespace opportunities, quick wins,
            strategic initiatives, suppliers, discount tiers

[STAGE 5] Dashboard Assembly
  Build Dashboard HTML (Code node — string interpolation)
    ├── Injects all parsed data into static HTML template via {{PLACEHOLDER}} tokens
    ├── Applies benchmark fallbacks (sourced from Wayfair Area Rug Trend Report PDF)
    │   when Cheerio parsing returns empty values
    └── Assembles 6 pages: Overview, Market, Competitive, Opportunities,
        Diagnostics, Actions

[STAGE 6] Output
  Prepare Download → binary HTML file
    └── Filename: {Category}_Dashboard_{YYYY-MM-DD}.html
```

---

## Stage-by-Stage Breakdown

### Stage 1 — Google Drive OAuth Setup

* **Why OAuth:** Google Drive requires secure, delegated access. OAuth lets n8n search your Drive and download files without sharing passwords.
* **Setup:** Connect Google Drive credentials inside n8n's credential manager → authorize the `drive.readonly` scope
* **Folder structure expected on Drive:**

```
Wayfair Reports/
  └── Area Rug/
        └── YYYY-MM-DD/
              ├── [market-trend-report].html      ← P02 output (name contains "market" or "p2")
              └── [competitor-report].html         ← P03 output (name contains "competitor" or "p3")
```

* **Without OAuth, the agent cannot function.** This is the single most critical setup step.
* The workflow also expects a dashboard HTML template file in the root of the Wayfair Reports folder — identified by filename containing "template" and ending in `.html`.

---

### Stage 2 — File Retrieval

* n8n lists the root of the Wayfair Reports folder to identify the category subfolder and template file
* Selects the most recent date folder by sorting `YYYY-MM-DD` folder names descending
* Downloads the template and both report files as raw HTML strings — no text conversion, no AI summarization
* Passes raw HTML directly into the Cheerio parsing stage

---

### Stage 3 — Cheerio Parsing

**What is Cheerio?**
Cheerio is a JavaScript library that reads and extracts data from HTML files using jQuery-style selectors. Think of it this way:

* A human sees: a styled webpage with headings, tables, and charts
* Cheerio sees: `<h2 class="segment-name">`, `<td class="price-band">`, `<p class="risk-rating">`

Cheerio navigates the HTML *structurally* — it looks for known tags and classes — rather than *conceptually*. That makes it dramatically faster and more reliable than asking an AI to "read" the report again.

**When parsing is the right tool (all three conditions are met here):**

1. The goal is to *extract existing data*, not generate new insights ✅
2. The documents are *structured and predictable* HTML files ✅
3. You already know *where the data lives* inside each report ✅

**What gets extracted from P02:**

| Data Point | HTML Selector |
|---|---|
| Key insight | `.key-insight p`, `.executive-summary p` |
| Market metrics (size, CAGR) | `.profile-card` headings + values |
| Micro-segment names, tiers, descriptions | `.segment-card` blocks |
| Color palette (hex + name) | `.color-swatch`, `.color-card` |
| Risk ratings + mitigations | `.risk-card` + severity class |
| Recommendations | `.rec-item`, `.recommendation-card` |
| Moodboard images | `img[src^="data:image"]` (base64 embeds) |
| Attribute distribution bars | `.attribute-card` → `.data-row` |

**What gets extracted from P03:**

| Data Point | HTML Selector |
|---|---|
| Product counts per retailer | `.scope-card` text regex |
| Price positioning (avg/min/max) | `.comparison-table tbody tr` |
| Competitor comparison table | `.comparison-table tbody tr` |
| Price band distribution | `.band` entries |
| Price gap heatmap | `.gap` entries + severity class |
| Whitespace opportunities | `.opportunity-card` blocks |
| Quick wins | `.quick-wins .rec-card` |
| Strategic initiatives | `.strategic-plays .rec-card` |
| Suppliers | `table tbody tr` (supplier section) |
| Discount tiers | `.discount-tier` entries |

---

### Stage 4 — Dashboard Assembly

The dashboard is assembled by injecting Cheerio-parsed data into a static HTML template via `{{PLACEHOLDER}}` token replacement. No AI writes the HTML — a Code node handles all string interpolation directly.

**Benchmark fallbacks:** When Cheerio returns empty values (e.g., P02 parsing finds no market metrics), the Build Dashboard node falls back to values sourced directly from the Wayfair Area Rug Trend Report PDF, ensuring the dashboard never renders blank sections.

**Dashboard pages and their content:**

| Page | Sections |
|---|---|
| Overview | KPI cards (target segment, avg price, products analyzed), trend key insight box, competitive key insight box, top alerts (high-risk items from P02 + top opportunity from P03), quick action list |
| Market & Trends | Market size KPI cards (2025 size, 2033 projection, CAGR, fastest-growing segment), trend key insight, micro-segment cards (3-column grid), color palette swatches, moodboard images, attribute distribution bars (materials / sizes / colors / prices), trend recommendations |
| Competitive Intelligence | Retailer KPI cards, pricing comparison (Wayfair / Amazon / Walmart avg/min/max), price band bar chart (budget / mid / premium by retailer), full comparison table with winner badges, discount tier breakdown |
| Opportunity Radar | Whitespace opportunity cards (priority-coded), price gap heatmap, key supplier chip cards (clickable, links to retailer search) |
| Risk & Diagnostics | Risk count KPIs (high/medium/low), risk cards (click to expand — shows description + mitigation) |
| Action Center | Strategic initiatives (ranked), quick wins (ranked), trend-based recommendations with impact ratings, full supplier table (name / platform / known for / price range / search link), action tracker (action / priority / status / owner / due date) |

**Design spec:**

* Inter font (Google Fonts CDN)
* CSS custom properties for full theming — no inline styles
* 12-column responsive CSS grid
* Competitor brand colors: Wayfair purple (`#7C3AED`), Amazon orange (`#FF9900`), Walmart blue (`#0071CE`)
* Sidebar navigation — sticky, 280px, gradient background
* Keyboard shortcuts: `Alt+1` through `Alt+6` to jump between pages
* Interactive features: risk card expand/collapse on click, search/filter across all cards, page fade-in animation, hover lift on supplier cards

---

### Stage 5 — Output & Delivery

* **Prepare Download:** Converts assembled HTML string to binary with filename `{Category}_Dashboard_{YYYY-MM-DD}.html`
* **Preview:** Renders the dashboard directly inside n8n for instant review
* Output is a single self-contained `.html` file that renders in any browser

---

## Core Concept: Parsing vs. AI Reasoning

This is the most important engineering decision in Project 05 — and it's intentional.

| | Cheerio Parsing | AI Reasoning |
|---|---|---|
| **Speed** | Milliseconds | 10–30 seconds per call |
| **Cost** | Free | API tokens consumed |
| **Reliability** | 100% deterministic | Output can vary between runs |
| **Best for** | Extracting known, structured data | Generating new analysis |
| **Used in P05?** | ✅ Yes | ❌ No |

Using AI to re-read the P02 and P03 reports would be slower, more expensive, and less reliable than reading the HTML directly. The upstream agents already did the reasoning work. Cheerio harvests the results.

---

## Tools & Technologies

| Tool | Purpose |
|---|---|
| **n8n** | Workflow automation platform |
| **Cheerio (JavaScript)** | HTML parsing and structured data extraction |
| **Google Drive API (OAuth)** | Secure file retrieval — P02 and P03 HTML reports + dashboard template |
| **HTML / CSS** | 6-page dashboard template and styling |
| **GitHub** | Version control (`lazarovaldez17/wayfair-ai-automation`) |

---

## Inputs Required

| Input | Description | Where it comes from |
|---|---|---|
| `P02 HTML Report` | Area Rug Trend Report | P02 workflow output → Google Drive |
| `P03 HTML Report` | Competitor Analysis Report | P03 workflow output → Google Drive |
| `Dashboard Template` | Base HTML template with `{{PLACEHOLDER}}` tokens | Stored in root of Wayfair Reports folder on Drive |
| `Category` | Rug category to filter by | Detected from subfolder name on Drive |

> **Dependency:** Both P02 and P03 must have been run at least once and their reports uploaded to Google Drive before this agent can execute. The HTML dashboard template must also exist in the Wayfair Reports root folder.

---

## Output

A single self-contained `.html` file — `{Category}_Dashboard_{YYYY-MM-DD}.html` — that renders as a complete, 6-page interactive executive dashboard in any browser.

The dashboard answers the four questions a category manager needs every Monday:

1. What's trending in Area Rugs?
2. How does Wayfair stack up on price vs. competitors?
3. Where are the gaps we can capitalize on?
4. What should the team prioritize?

---

## Key Learnings

* **Parsing beats AI for extraction tasks:** When data already exists in a structured document, Cheerio is faster, cheaper, and more reliable than prompting an LLM to re-read it
* **Benchmark fallbacks prevent blank dashboards:** When Cheerio selectors return empty values (due to upstream report formatting changes), hard-coded fallback values from the Wayfair Area Rug Trend Report PDF ensure the dashboard always renders with meaningful data
* **OAuth is a hard dependency:** Google Drive integration requires delegated access — no workaround exists; this must be configured before any workflow logic is built
* **Dashboard ≠ report:** A report explains. A dashboard decides. Stripping out narrative and surfacing only the key numbers is a design discipline, not just a technical one
* **Capstone = integration, not invention:** The value of Project 05 is not in novel AI logic — it's in demonstrating that four separate agents can be connected into a coherent, decision-ready internal tool

---

## Capstone Context

This project is the final deliverable of the Wayfair AI Automation Externship. It synthesizes the output of all prior work:

| Project | Agent | Output fed into P05 |
|---|---|---|
| **P02** | Market Trend Discovery Agent | Area Rug trend report HTML (micro-segments, risks, attributes, moodboards) |
| **P03** | Competitor Monitoring Agent | Competitor analysis HTML (pricing gaps, whitespace, recommendations) |
| **P04** | AI Insights & Content Agent | Optional: content strategy context |
| **P05** | **Dashboard Builder Agent** | **Unified Category Manager Dashboard** ← *this project* |

The externship traces a complete product intelligence workflow:
**Discover trends → Monitor competitors → Generate content → Surface insights in one dashboard.**

---

## Planned Improvements

- [ ] Add P04 content strategy output as a third data source — surfacing ready-to-publish content assets alongside trend and competitor data in the same dashboard view
- [ ] Build a scheduled trigger to run the full pipeline (P02 → P03 → P05) automatically every Monday and deliver the dashboard to Slack or email
- [ ] Add week-over-week change indicators to KPI cards — flagging when a metric has shifted significantly from the prior run
- [ ] Extend beyond Area Rugs: the parameterized architecture means any category with upstream P02/P03 reports can be plugged in without rebuilding the workflow

---

## Files

| File | Description |
|---|---|
| `README.md` | Project documentation (this file) |
| `Dashboard_Builder_Agent.json` | n8n workflow export — all credentials removed (Google Drive credential ID and n8n instance ID replaced with placeholder values) |
| `Area_Rug_Dashboard.html` | Sample output — fully interactive 6-page Category Manager Dashboard (Area Rugs, generated 2026-05-31) |

---

*Wayfair AI Automation Externship · Extern · May 2026*
