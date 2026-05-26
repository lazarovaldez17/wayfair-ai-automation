# Project 04: AI Insights \& Content Agent

**Externship:** Wayfair AI Automation via Extern  
**Category Focus:** Area Rugs  
**Status:** Pre-Planning Complete · Build In Progress

\---

## Overview

Project 04 shifts the pipeline from *intelligence gathering* to *action*. Where Projects 02 and 03 answered *"What is happening in the market?"*, this project answers the harder business question:

> \*\*"What should Wayfair DO about it?"\*\*

The AI Insights \& Content Agent takes structured trend and competitor data as inputs and transforms them into real, usable marketing content — blog posts, Instagram captions, campaign concepts, and more — written in Wayfair's brand voice, ready for a marketing team to act on.

\---

## Where This Fits in the Pipeline

```
Project 02                  Project 03                  Project 04
Market Trend Discovery  →   Competitor Monitoring   →   AI Insights \& Content
"What's trending?"          "What are rivals doing?"    "What should WE do?"

      ↓                           ↓                           ↓
 Micro-segment              Pricing gaps              Blog posts
 trend report               Whitespace opps           IG captions
 (Area Rugs)                Feature gaps              Campaign ideas
```

Projects 02 and 03 are no longer standalone deliverables — their outputs become the **upstream inputs** that this agent consumes to generate content.

\---

## Project Goals

### Primary Goal

Build and enhance an AI agent that turns market and competitive intelligence into on-brand content artifacts that Wayfair's marketing team could realistically use.

### Secondary Goal (Meta-Skill)

Practice **inheriting and improving an existing system**:

* Reading and understanding someone else's workflow before touching it
* Identifying failure modes without breaking what already works
* Making targeted, justified improvements

\---

## Pre-Planning Phase

Before writing a single node, the following groundwork was completed:

### Step 1 — Import \& Audit the Base Agent

* Import the provided workflow JSON into n8n
* Reconnect API credentials (Gemini, Mistral)
* Read every node and trace the data flow end-to-end before making any changes

### Step 2 — Map the Node Architecture

Understand what each node does and how data transforms across the pipeline:

```
Input (trend + competitor data)
    └── AI Agent (content generation)
          └── Output (blog post / caption / campaign brief)
```

### Step 3 — Run with Real Data

* Feed the agent outputs from Project 02 (Area Rug trend report) and Project 03 (competitor analysis)
* Observe what the agent generates before any enhancements
* Document the baseline output quality

### Step 4 — Study Wayfair's Brand Voice

Analyzed Wayfair's public-facing content across two channels:

|Channel|URL|What to Extract|
|-|-|-|
|Blog|wayfair.com/ideas-and-advice|Tone, article structure, how products are framed|
|Instagram|@wayfair|Caption style, CTA patterns, visual language cues|

**Brand Voice Rubric (built from research):**

|Dimension|Wayfair Standard|Generic AI Default|
|-|-|-|
|Tone|Aspirational but accessible|Formal / corporate|
|Language|Design-forward, room-first|Product-first, spec-heavy|
|Framing|"Imagine your space"|"This product features..."|
|CTA style|Soft invitation|Hard sell|
|Length|Punchy captions, scannable posts|Dense paragraphs|

### Step 5 — Identify Gaps in the Base Agent

Common failure modes audited before enhancement:

* \[ ] Generic tone that doesn't match Wayfair's warm, design-editorial voice
* \[ ] Output reads like a market report, not marketing content
* \[ ] Missing content types beyond basic summaries
* \[ ] No differentiation between content formats (blog vs. caption vs. campaign brief)
* \[ ] Weak use of upstream micro-segment data as creative input

\---

## Enhancement Plan

Three tracks of improvement, applied in order:

### Track 1 — Voice

Rewrite system prompts so the agent writes *as* Wayfair, not *about* Wayfair.

* Add a brand voice reference block to each AI node's system prompt
* Ground outputs in room-first, lifestyle-forward language
* Tune tone separately for each content type (blog ≠ caption ≠ campaign brief)

### Track 2 — New Content Types

Expand the agent's output surface beyond its base capability:

|Content Type|Format|Primary Use|
|-|-|-|
|Blog post|Long-form HTML|SEO + editorial content|
|Instagram caption|Short-form text + hashtags|Social media team|
|Campaign concept|Brief with headline + angle|Marketing strategy|
|Product spotlight|Feature copy block|Merchandising / PDP|

### Track 3 — Expanded Inputs

Connect richer upstream data so outputs are specific, not generic:

* Micro-segment names and descriptions from Project 02
* Pricing gaps and whitespace opportunities from Project 03
* Trend signals (Pinterest, Instagram, blog summaries) from Extern API

\---

## Business Impact

A single well-scoped output from this agent could directly influence:

* **Assortment messaging** — which micro-segments to spotlight in campaigns
* **Social content calendar** — on-brand captions tied to real trend data
* **Editorial strategy** — blog angles backed by competitor gap analysis
* **Speed to market** — content that used to take a copywriter hours generated in minutes

\---

## Models \& Tools

|Purpose|Tool|
|-|-|
|Content generation|Google Gemini API|
|Tone refinement|Mistral Large|
|Workflow automation|n8n|
|Upstream trend data|Project 02 output (Area Rug trend report)|
|Upstream competitor data|Project 03 output (Competitor analysis report)|
|Version control|GitHub|

\---

## Files

|File|Description|
|-|-|
|`README.md`|Project documentation (this file)|

\---

## Key Learnings (Updated as Build Progresses)

* Inheriting an existing system requires a read-first, build-second discipline — the audit phase is not optional
* Brand voice is a system prompt engineering problem, not just a style preference
* Content agents fail when they have no opinion about *format* — blog, caption, and campaign brief each need separate output schemas
* The strongest outputs come from specific inputs: named micro-segments and real pricing gaps produce better content than generic category summaries

\---

*Wayfair AI Automation Externship · Extern · May 2026*

