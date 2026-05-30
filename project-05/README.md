# Project 05: Dashboard Builder Agent *(Capstone)*

**Externship:** Wayfair AI Automation via Extern  
**Category Focus:** Area Rugs  
**Workflow:** Dashboard Builder Agent  
**Status:** In Progress
**Role:** Capstone — aggregates outputs from all prior projects into a unified executive dashboard

\---

## Overview

The Dashboard Builder Agent is the capstone of the Wayfair AI Automation Externship. Rather than generating new analysis, it acts as an internal tooling layer — pulling the HTML reports produced by Project 02 (Market Trend Discovery) and Project 03 (Competitor Monitoring) from Google Drive, parsing their structured content using Cheerio, and assembling everything into a single, polished **Category Manager Dashboard** in under a minute.

Where Projects 02–04 were analysts producing detailed reports, Project 05 is the system that converts those reports into an executive briefing a category team can skim in minutes.

> \*\*No AI reasoning is used in this project.\*\* All data already exists in upstream outputs. Cheerio handles extraction; the workflow handles assembly.

\---

## Why a Dashboard?

A dashboard is a visual display that surfaces the most important information from multiple sources in one place — designed for fast comprehension and faster decisions. In a real-world scenario, a **category manager at Wayfair** would use this dashboard every Monday to answer:

* What rug styles are trending right now?
* How does Wayfair's pricing compare to Amazon and Walmart?
* Where do whitespace opportunities exist?
* What should the team prioritize this week?

Without a dashboard, answering those four questions requires manually opening multiple reports, skimming for the relevant sections, and mentally synthesizing them. This agent does all of that in one automated run.

\---

## What Makes This Project Different

|Projects 02–04|Project 05|
|-|-|
|Generate new insights via AI|Aggregates existing insights — no new AI calls|
|Output: individual HTML reports|Output: unified executive dashboard|
|Data sources: Amazon, social APIs, competitor pages|Data sources: P02 + P03 HTML reports on Google Drive|
|Core skill: prompt engineering + n8n workflow logic|Core skill: HTML parsing (Cheerio) + Google Drive OAuth|
|10–20 min runtime|Under 1 minute runtime|

\---

## Workflow Architecture

```
\[STAGE 1] Google Drive Access (OAuth)
  n8n OAuth Setup
    └── Grant Drive permission → search Wayfair Reports folder
          └── Locate latest P02 and P03 HTML files by category + date

\[STAGE 2] File Retrieval
  Fetch P02 Report (Market Trend Discovery HTML)
    └── Fetch P03 Report (Competitor Monitoring HTML)
          └── Both files loaded as raw HTML strings

\[STAGE 3] Cheerio Parsing
  Parse P02 HTML
    ├── Extract: micro-segments, trend names, pricing bands, materials, risk ratings
    └── Extract: key metrics (social signal counts, product counts, category)

  Parse P03 HTML
    ├── Extract: competitor comparison data (Wayfair vs Amazon vs Walmart)
    ├── Extract: pricing gaps, whitespace opportunities, feature gaps
    └── Extract: strategic recommendations

\[STAGE 4] Dashboard Assembly
  Collect All Parsed Sections
    └── Assemble HTML Dashboard
          ├── Header — title, category, report date
          ├── KPI Cards — 4–6 key metrics with icons and color coding
          ├── Trend Breakdown — micro-segments from P02
          ├── Competitor Snapshot — pricing comparison from P03
          ├── Whitespace Opportunities — gaps identified in P03
          └── Priorities Section — ranked next steps

\[STAGE 5] Output
  Validate Structure → Download HTML File → Preview in n8n
    └── (Optional) Upload dashboard back to Google Drive → Wayfair Reports
```

\---

## Stage-by-Stage Breakdown

### Stage 1 — Google Drive OAuth Setup

* **Why OAuth:** Google Drive requires secure, delegated access. OAuth lets n8n search your Drive and download files without sharing passwords.
* **Setup:** Connect Google Drive credentials inside n8n's credential manager → authorize the `drive.readonly` scope
* **Folder structure expected on Drive:**

```
  Wayfair Reports/
    └── Area Rug/
          └── YYYY-MM-DD/
                ├── area-rug-trend-report-YYYY-MM-DD.html      ← P02 output
                └── Competitor\_Analysis\_Report.html             ← P03 output
  ```

* **Without OAuth, the agent cannot function.** This is the single most critical setup step.

\---

### Stage 2 — File Retrieval

* n8n uses the Google Drive node to list files in the target folder
* Selects the most recent P02 and P03 HTML files by matching filename patterns and `modifiedTime`
* Downloads both files as raw HTML strings — no text conversion, no AI summarization
* Passes raw HTML directly into the Cheerio parsing stage

\---

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

|Data Point|HTML Location|
|-|-|
|Micro-segment names|`.segment-name` headings|
|Micro-segment descriptions|`.segment-description` paragraphs|
|Dominant materials|`.attribute-material` cells|
|Price band ranges|`.price-band` cells|
|Risk ratings|`.risk-rating` badges|
|Social signal counts|`.scope-stat` metric cards|

**What gets extracted from P03:**

|Data Point|HTML Location|
|-|-|
|Competitor pricing comparison|Comparison table rows|
|Wayfair vs Amazon vs Walmart feature gaps|`.gap-row` entries|
|Whitespace opportunities|`.whitespace-item` list items|
|Strategic recommendations|`.recommendation-card` sections|
|Supplier/brand callouts|`.supplier-row` entries|

\---

### Stage 4 — Dashboard Assembly

The dashboard is assembled from a **static HTML template** with placeholder tokens replaced by parsed data. No AI writes the HTML — the Code node handles string interpolation directly.

**Dashboard sections:**

|Section|Source|Description|
|-|-|-|
|Header|Workflow metadata|Category name, auto-generated date, agent version|
|KPI Cards (×4–6)|P02 + P03|Key numbers: product count, competitors tracked, micro-segments identified, top price band|
|Trend Breakdown|P02|Named micro-segments with materials, colors, and risk ratings|
|Competitor Snapshot|P03|Side-by-side pricing for Wayfair, Amazon, and Walmart|
|Whitespace Map|P03|Feature and price gaps Wayfair can exploit|
|Priority Recommendations|P03|Ranked quick wins and strategic initiatives|

**Design spec (CSS only, no external libraries):**

* Gradient backgrounds with 2026-trending color palette
* Card-based layout with drop shadows
* Responsive grid — works on desktop and mobile
* Emoji/Unicode icons for KPI cards
* Smooth hover transitions
* Rounded corners, consistent spacing

\---

### Stage 5 — Output \& Delivery

* **Section Validator:** Confirms all expected sections are present; logs warnings for missing data
* **Create Download File:** Converts HTML string to binary → filename: `category-manager-dashboard-YYYY-MM-DD.html`
* **HTML Preview:** Renders the dashboard directly inside n8n for instant review
* **Google Drive Upload (optional):** Saves the final dashboard back to `Wayfair Reports/Area Rug/YYYY-MM-DD/`

\---

## Core Concept: Parsing vs. AI Reasoning

This is the most important engineering decision in Project 05 — and it's intentional.

||Cheerio Parsing|AI Reasoning|
|-|-|-|
|**Speed**|Milliseconds|10–30 seconds per call|
|**Cost**|Free|API tokens consumed|
|**Reliability**|100% deterministic|Output can vary between runs|
|**Best for**|Extracting known, structured data|Generating new analysis|
|**Used in P05?**|✅ Yes|❌ No|

Using AI to re-read the P02 and P03 reports would be slower, more expensive, and less reliable than just reading the HTML directly. The upstream agents already did the reasoning work. Cheerio harvests the results.

\---

## Tools \& Technologies

|Tool|Purpose|
|-|-|
|**n8n**|Workflow automation platform|
|**Cheerio (JavaScript)**|HTML parsing and structured data extraction|
|**Google Drive API (OAuth)**|Secure file retrieval — P02 and P03 HTML reports|
|**HTML / CSS**|Dashboard template and styling|
|**GitHub**|Version control (`lazarovaldez17/wayfair-ai-automation`)|

\---

## Inputs Required

|Input|Description|Where it comes from|
|-|-|-|
|`P02 HTML Report`|Area Rug Trend Report|P02 workflow output → Google Drive|
|`P03 HTML Report`|Competitor Analysis Report|P03 workflow output → Google Drive|
|`Category`|Rug category to filter by|Detected from filenames on Drive|

> \*\*Dependency:\*\* Both P02 and P03 must have been run at least once and their reports uploaded to Google Drive before this agent can execute.

\---

## Output

A single self-contained `.html` file — `category-manager-dashboard-YYYY-MM-DD.html` — that renders as a complete, styled executive dashboard in any browser.

The dashboard answers the four questions a category manager needs every Monday:

1. What's trending in Area Rugs?
2. How does Wayfair stack up on price vs. competitors?
3. Where are the gaps we can capitalize on?
4. What should the team prioritize?

\---

## Key Learnings

* **Parsing beats AI for extraction tasks:** When data already exists in a structured document, Cheerio is faster, cheaper, and more reliable than prompting an LLM to re-read it
* **OAuth is a hard dependency:** Google Drive integration requires delegated access — no workaround exists; this must be configured before any workflow logic is built
* **Dashboard ≠ report:** A report explains. A dashboard decides. Stripping out narrative and surfacing only the key numbers is a design discipline, not just a technical one
* **Capstone = integration, not invention:** The value of Project 05 is not in novel AI logic — it's in demonstrating that four separate agents can be connected into a coherent, decision-ready internal tool

\---

## Capstone Context

This project is the final deliverable of the Wayfair AI Automation Externship. It synthesizes the output of all prior work:

|Project|Agent|Output fed into P05|
|-|-|-|
|**P02**|Market Trend Discovery Agent|Area Rug trend report HTML (micro-segments, risks, attributes)|
|**P03**|Competitor Monitoring Agent|Competitor analysis HTML (pricing gaps, whitespace, recommendations)|
|**P04**|AI Insights \& Content Agent|Optional: content strategy context|
|**P05**|**Dashboard Builder Agent**|**Unified Category Manager Dashboard** ← *this project*|

The externship traces a complete product intelligence workflow:  
**Discover trends → Monitor competitors → Generate content → Surface insights in one dashboard.**

\---

## Files

|File|Description|
|-|-|
|`README.md`|Project documentation (this file)|

\---

*Wayfair AI Automation Externship · Extern · May 2026*

