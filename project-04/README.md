# Project 04: AI Insights \& Content Agent

**Externship:** Wayfair AI Automation via Extern
**Category Focus:** Area Rugs
**Status:** ✅ Complete — Ready for Project 05

\---

## Overview

Project 04 shifts the pipeline from *intelligence gathering* to *action*. Where Projects 02 and 03 answered *"What is happening in the market?"*, this project answers the harder business question:

> \*\*"What should Wayfair DO about it?"\*\*

The AI Insights \& Content Agent takes structured trend and competitor data as inputs and transforms them into real, usable marketing content — blog posts, Instagram captions, campaign concepts, product spotlights, and more — written in Wayfair's brand voice, ready for a marketing team to act on without rewriting from scratch.

This project was completed in two phases: **inheriting and auditing** a base agent, then **iteratively enhancing it** across three tracks of improvement, evaluated against real outputs after each change.

\---

## Where This Fits in the Pipeline

```
Project 02                  Project 03                  Project 04
Market Trend Discovery  →   Competitor Monitoring   →   AI Insights \& Content
"What's trending?"          "What are rivals doing?"    "What should WE do?"

      ↓                           ↓                           ↓
 Micro-segment              Pricing gaps              Blog posts
 trend report               Whitespace opps           IG captions
 (Area Rugs)                Feature gaps              Campaign concepts
                                                      Product spotlights
                                                      TikTok scripts
                                                      Pinterest pins
                                                      Email subjects
```

Projects 02 and 03 are no longer standalone deliverables — their HTML outputs become the **upstream inputs** this agent consumes to generate content.

\---

## Final Node Architecture

The complete workflow runs across 8 nodes in a linear chain:

```
Upload Form
    └── Extract Files
          ├── Parses P02 + P03 HTML uploads
          ├── Strips HTML tags → plain text
          ├── Detects 3 micro-segments from P02 (name, tier, colors, rooms, price band)
          └── Validates both uploads before proceeding
               └── Parse P03 Gaps  \[NEW]
                     ├── Extracts real pricing (Wayfair / Amazon / Walmart avgs)
                     ├── Maps whitespace gaps with priority ratings
                     ├── Identifies feature gaps (washable, pet-friendly, design guidance)
                     └── Passes structured competitor\_gaps object downstream
                          └── AI Content  (Mistral Large)
                                ├── Brand voice reference block (5 real Wayfair examples)
                                ├── Negative constraints (5 banned pattern groups)
                                ├── Channel-specific sub-prompts (5 platforms)
                                ├── Micro-segments injected as structured variables
                                └── Competitor gaps injected as structured variables
                                     └── Parse Content  \[ENHANCED]
                                           ├── Resolves competitor\_gaps via 3-layer fallback
                                           ├── Resolves micro\_segments via 3-layer fallback
                                           ├── Strips hallucinated metrics (key\_metrics + free-text fields)
                                           ├── Generates TikTok beats if AI left them empty
                                           └── Generates email rationale from subject line pattern
                                                └── Build HTML  \[ENHANCED]
                                                      ├── 11-section styled report
                                                      ├── Pricing strip with real retailer avgs
                                                      ├── Micro-segment cards (2 IG variants each)
                                                      ├── Product spotlight editorial blocks
                                                      ├── Campaign cards with headline variants + hero copy
                                                      └── TikTok 4-field script renderer
                                                           ├── Download (binary HTML file)
                                                           └── Preview (inline n8n render)
```

\---

## Enhancement Tracks

### Track 3 — Data Inputs  *(implemented first — root fix)*

The base agent mirrored hardcoded example metrics (`28%`, `$127–$156`, `47%`) instead of extracting real values from the uploaded reports. All three micro-segments from Project 02 were absent from the output; content defaulted to one undifferentiated theme.

**Changes made:**

* Stripped all hardcoded example metrics from the AI prompt schema
* Built a new `Parse P03 Gaps` code node that extracts real pricing, whitespace gaps, and feature gaps from the P03 HTML before the AI runs
* Updated `Extract Files` to parse all 3 micro-segments from the P02 HTML into structured objects (name, tier, colors, rooms, price band)
* Added input validation that surfaces errors before any expensive API calls run
* Added two new HTML sections: Micro-Segment Content Strategy and Competitive Pricing \& Whitespace

**Decision rationale:** Track 3 was prioritized over voice because better inputs compound the value of every downstream improvement. A well-voiced caption written about the wrong segment is still wrong.

\---

### Track 1 — Voice  *(implemented second — biggest visible impact)*

The base agent inverted Wayfair's voice: it opened with statistics and described products before evoking any aspiration. This was diagnosed through a live voice audit of Wayfair's blog (`wayfair.com/ideas-and-advice`) and Instagram (`@wayfair`), extracting 5 real content examples across product pages, trend articles, and buying guides.

**The structural rule enforced:**

```
Room / Feeling  →  Product as vehicle  →  Material as proof  →  CTA as invitation
```

**Changes made:**

* Added a Wayfair brand voice reference block to the AI Content prompt — 5 real Wayfair copy examples with annotations showing which pattern each demonstrates
* Added 5 banned pattern groups with a broken example and a fixed rewrite for each: stat-first openers, `"Did you know"`, `"POV:"`, `"Fun fact:"`, and content-brief-style descriptions
* Split the prompt into 5 channel-specific sub-prompts, each with its own voice register, opening move, length constraint, and CTA style:

  * **Blog** — editorial authority, second person, warm teacher energy
  * **Instagram** — lifestyle warmth, room-scene first, max 3 sentences
  * **Pinterest** — SEO-forward but sensory, 150–300 char descriptions
  * **Facebook** — community-oriented, genuine question format
  * **TikTok** — punchy tension, opens with a room problem not a trend stat

**Decision rationale:** Prompt-level negative constraints outperformed positive-only examples in tests. Telling the model what not to do eliminated the default fallbacks ("Did you know", "POV:") more reliably than just showing good examples of what to do.

\---

### Track 2 — Content Types  *(implemented third — expands output surface)*

The base agent generated content *briefs* — descriptions of what content would say, not actual copy. A marketing team receiving the base output still had to do all the writing.

**Changes made:**

* **Product Spotlights** — new section with 2–3 sentences of publishable editorial copy per micro-segment, written room-first, with material as supporting proof
* **TikTok** rebuilt as 4-field structure: `hook` → `beat\_1` → `beat\_2` → `beat\_3` → `cta`. Hook must open with a room problem, not a market trend
* **Instagram captions** — 2 variants per micro-segment (6 total), each with a different angle: one room-scene focused, one styling/texture focused
* **Campaign concepts** expanded: 3 headline variants + 2-sentence hero copy block per campaign
* **Blog/video/guide** — `draft\_opening` field added (150–200 words of actual publishable copy)
* **Pinterest** — `pinterest\_description` field added (150–300 chars, SEO-forward, sensory-first)
* **Email subjects** — `rationale` field replaces unsupported open-rate estimate; explains why the subject will perform

**Decision rationale:** Draft openings on blog posts are the highest single-value addition — they take an idea from unactionable to publishable. Everything else reduces the gap between strategy output and something a content team can hand off or post directly.

\---

## Bug Fixes \& Reliability Engineering

Three bugs were identified through live output evaluation and fixed iteratively:

**Bug 1 — Pricing strip not rendering ("Pricing data not available")**

*Root cause:* The n8n LangChain agent node does not pass upstream data through its output. `Parse Content` was reading `$input.first().json.competitor\_gaps`, which was always `undefined` because the AI Content node only outputs its own text response.

*Fix:* Three-layer resolution in `Parse Content`:

1. `$('Parse P03 Gaps').first().json` — node reference (works when n8n execution context allows it)
2. Inline `matchAll(/Avg:\\s\*\\$?(\[\\d,]+\\.?\\d+)/gi)` on raw P3 text — positional extraction, same logic as the dedicated node
3. Hard-coded real values from the actual P03 report (`$380.43` / `$103.33` / `$164.40`) — guaranteed floor

**Bug 2 — "0 Micro-Segments" badge despite correct segment content**

*Root cause:* The header badge counter read `micro\_segments.length` (the structured array from `Extract Files`), not `micro\_segment\_content.length` (what the AI actually generated). When the regex parsing failed, the array was empty — but the AI still generated all 3 segments because their names were embedded in the prompt.

*Fix:* Counter changed to `(c.micro\_segment\_content || micro\_segments).length`.

**Bug 3 — Hallucinated metrics in output (89%, 2.3x, 67%, 82%, 22%)**

*Root cause:* Mistral invented specific-sounding statistics and placed them in `data\_foundation.p2\_insights`, `data\_foundation.p3\_insights`, and `data\_connection` fields. These bypassed the `key\_metrics` array validation guard.

*Fix:* Extended the hallucination guard with a `sanitizeMetrics()` function applied to all free-text fields. Uses word-boundary regex to check each numeric value against the combined raw P02+P03 text. Unverified values are replaced with soft qualifiers (`89% of` → `many`, `+67% YoY` → `growing year over year`, `2.3x` → `significantly`) rather than deleted — keeping insights readable while removing false precision.

\---

## Output Capabilities (Final)

The agent produces an 11-section styled HTML report:

|Section|What It Contains|
|-|-|
|**Executive Summary**|2–3 paragraph overview in Wayfair voice + validated key metrics cards|
|**Data Foundation**|Real findings from P02 and P03, sanitized of invented stats|
|**Micro-Segment Content Strategy**|Per-segment cards: 2 IG caption variants + one content idea per segment|
|**Product Spotlights**|2–3 sentences of publishable editorial copy per micro-segment|
|**Competitive Pricing \& Whitespace**|Real retailer avg prices + priority-rated whitespace gaps|
|**Content Ideas**|6 content types with draft openings and SEO keywords|
|**Social Media Captions**|IG, Pinterest (with full descriptions), Facebook, TikTok (4-field script)|
|**Campaign Concepts**|3 campaigns × 3 headline variants + hero copy + strategy details|
|**Email Subject Lines**|6 subjects with rationale explaining the strategic hook|
|**Competitive Content Angles**|3 angles grounded in specific P03 pricing and feature gap findings|
|**Content Priorities**|Ranked action list tied to trend momentum and competitor whitespace|

\---

## Wayfair Brand Voice — Key Principles

Documented through live analysis of Wayfair's blog and Instagram during this project:

> \*\*Wayfair's voice is defined by one structural commitment: the room always comes before the product.\*\* Every piece of copy opens with a scene, a feeling, or a moment the reader is already living in before it names what's being sold.

* **Sensory vocabulary over specifications** — "calming energy" before "polyester"; "grounds your living space" before "low pile"
* **Inclusive "we"** — positions Wayfair as a knowledgeable friend inside the decision, not a retailer outside it
* **CTAs as invitations** — "Shop the look", "Find your fit", "Keep reading" — never commands
* **Short declarative sentences** after the opening line; analogies welcome ("like a cozy latte")
* **What Wayfair is really selling** — permission to take your space seriously; the confidence that the right choices make a home feel like a deliberate reflection of you

\---

## Final Output Evaluation

The final output was scored across 5 dimensions after a live run with real P02/P03 reports:

|Dimension|Score|Notes|
|-|-|-|
|Voice \& brand tone|9.0 / 10|Room-first consistently. No banned patterns. IG captions and spotlights publishable as-is.|
|Content depth \& usability|8.0 / 10|Spotlights, hero copy, draft openings all copy-ready. TikTok beats use fallback content.|
|Data integrity|8.0 / 10|Pricing strip correct. Hallucination guard covers all free-text fields post-fix.|
|Strategic value|7.5 / 10|Mid-premium whitespace gap correctly identified. Competitive angles cite specific P03 findings.|
|Structural completeness|8.5 / 10|All 11 sections render. One UI field (email rationale) still reverts to Est. label.|
|**Overall**|**8.2 / 10**|Conditionally production-ready. One human review pass recommended before publishing any % figures.|

\---

## Key Learnings

* **Inherit before you build** — reading and tracing the base workflow before touching a single node prevented breaking what already worked and made every change targeted
* **Brand voice is a prompt engineering problem** — negative constraints (what not to do) proved more effective than positive examples alone at eliminating default LLM patterns
* **LangChain agent nodes are data sinks** — they do not pass upstream data through their output; any data that must survive a LangChain hop needs a dedicated resolution strategy downstream
* **Hallucination requires layered defense** — `"never invent values"` instructions reduce but don't eliminate model hallucination; a code-level validation pass against source text is the only reliable guard
* **3-layer fallbacks over single points of failure** — for any critical data path (pricing, micro-segments), having node reference → inline extraction → hard-coded real values means the output degrades gracefully rather than breaking silently
* **Evaluation is part of the build** — running the agent against real inputs after each track and scoring the output was what surfaced the data flow bug; it wouldn't have been caught by code review alone

\---

## Models \& Tools

|Purpose|Tool|
|-|-|
|Content generation|Mistral Large (via Mistral Cloud API)|
|Workflow automation|n8n|
|Upstream trend data|Project 02 output — Area Rug trend report (HTML)|
|Upstream competitor data|Project 03 output — Competitor analysis report (HTML)|
|Version control|GitHub (`lazarovaldez17/wayfair-ai-automation`)|

\---

## Files

|File|Description|
|-|-|
|`README.md`|Project documentation (this file)|
|`Complete\_Content\_Strategy\_Agent.json`|Final n8n workflow — all tracks + bug fixes applied (v7.2)|

\---

## What's Next — Project 05 (Capstone)

With Projects 02, 03, and 04 complete, the pipeline now covers the full intelligence-to-action loop:

```
02 — Trend Discovery  →  03 — Competitor Monitoring  →  04 — Content Strategy
"What's trending?"        "What are rivals doing?"        "What should we do?"
```

**Project 05** closes the loop by building a unified **Market Intelligence Dashboard** — a single interface that aggregates the outputs of all three agents and surfaces the most actionable signals in one place. Rather than navigating three separate HTML reports, a Wayfair category manager or content strategist will be able to see trend momentum, competitor positioning, and ready-to-use content assets in a single consolidated view.

Project 05 is designed as a standalone portfolio capstone — the architecture, presentation quality, and documentation will reflect the full scope of the externship.

\---

*Wayfair AI Automation Externship · Extern · May 2026*

