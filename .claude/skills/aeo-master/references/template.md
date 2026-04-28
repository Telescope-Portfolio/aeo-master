---
name: AEO Master Document Template
description: 16-section template for comprehensive AEO master documents. Variables use {{name}} syntax.
---

<!-- ============================================================
     AEO MASTER DOCUMENT TEMPLATE
     ============================================================
     This template is used during document assembly. Each {{variable}}
     is replaced with company-specific data at build time.

     Variable naming conventions:
       - Scrunch API fields use their API names (e.g., brand_presence_percentage)
       - Computed/derived fields use descriptive names (e.g., headline_stat)
       - Table rows that repeat per-item use _N suffixes or are generated dynamically
       - HTML comments explain what data fills each variable
     ============================================================ -->

# AEO Master Document: {{company}}

*Prepared by Telescope Partners | {{report_date}}*

---

## 1. Executive Summary

<!-- {{headline_stat}}: The single most compelling number from the audit. Typically the Scrunch
     Advice stage brand_presence_percentage (e.g., "8% visibility on Advice-stage prompts").
     This is the number that opens the conversation and frames the urgency. -->

**{{company}} is {{headline_stat}}.**

<!-- Fill the summary paragraph with the key findings. Variables below come from Scrunch Query API
     and the site audit. -->

Across {{total_prompts_tracked}} tracked prompts on {{platform_count}} AI platforms, {{company}} appears in **{{brand_presence_pct}}%** of responses overall. Position score is **{{position_score}}/100** and sentiment score is **{{sentiment_score}}/100**. On Advice-stage prompts (where buyers ask "what should I use?"), visibility drops to **{{advice_stage_pct}}%**. {{company}} is invisible on **{{zero_visibility_count}}** of {{total_prompts_tracked}} tracked queries.

<!-- {{content_piece_count}}: Number of content pieces recommended in the production plan (Section 7).
     This gives the reader an immediate sense of the work ahead. -->

This document identifies {{content_piece_count}} content pieces to produce across four tiers, a technical remediation plan, and a 16-week roadmap to move from current state to category-defining AI visibility.

| Metric | Current | Target (16 wk) |
|--------|---------|-----------------|
| Brand Presence % | {{brand_presence_pct}}% | {{target_brand_presence_pct}}% |
| Position Score | {{position_score}} | {{target_position_score}} |
| Sentiment Score | {{sentiment_score}} | {{target_sentiment_score}} |
| Advice Stage Visibility | {{advice_stage_pct}}% | {{target_advice_stage_pct}}% |
| Zero-Visibility Queries | {{zero_visibility_count}} | {{target_zero_visibility_count}} |

---

## 2. Situation & Background

<!-- This section provides the context needed to understand the rest of the document.
     All variables here come from the company CONTEXT.md, SKILL knowledge files, or
     the initial briefing conversation with the company. -->

### Company Overview

**{{company}}** is {{product_description}}.

<!-- {{product_description}}: One or two sentences describing what the product does, who it serves,
     and what category it belongs to. Use the company's own language where possible. -->

**Primary GTM motion:** {{primary_gtm_motion}}

<!-- {{primary_gtm_motion}}: e.g., "Inbound-led with sales-assisted close",
     "Outbound-driven enterprise sales", "PLG with self-serve free tier" -->

**Key competitors:** {{competitor_1}}, {{competitor_2}}, {{competitor_3}}

<!-- {{competitor_N}}: The 3 most relevant competitors for AEO purposes. These should be
     the companies that show up in the same AI prompts, not necessarily the closest
     product competitors. Pull from Scrunch competitor configuration or company context. -->

**Marketing team:** {{team_size}} people ({{team_roles}})

<!-- {{team_size}}: Headcount of the marketing team
     {{team_roles}}: e.g., "1 VP Marketing, 1 Content Manager, 1 Demand Gen, 1 Designer" -->

**Current initiative:** {{current_initiative}}

<!-- {{current_initiative}}: Whatever the company is actively working on that intersects with
     this audit. e.g., "Website refresh on Webflow", "Launching new product line",
     "Rebranding from X to Y", "No active web initiative" -->

### Why This Matters Now

<!-- Brief paragraph explaining the AEO opportunity in the context of this company's
     specific situation. Written at assembly time, not templated. -->

{{why_now_paragraph}}

---

## 3. Scrunch Visibility Data

<!-- All data in this section comes from the Scrunch Query API and Responses API.
     Variables use Scrunch API field names. The assembler pulls data for the
     most recent complete reporting period (typically trailing 4 weeks). -->

### Overall Scorecard

| Metric | Score | Interpretation |
|--------|-------|----------------|
| Brand Presence % | {{brand_presence_pct}}% | <!-- % of responses mentioning the brand --> {{brand_presence_interpretation}} |
| Position Score | {{position_score}}/100 | <!-- Weighted score: Top=100, Middle=50, Bottom=0 --> {{position_interpretation}} |
| Sentiment Score | {{sentiment_score}}/100 | <!-- Weighted score: Positive=100, Mixed=50, Negative=0 --> {{sentiment_interpretation}} |
| Total Responses Tracked | {{total_responses}} | <!-- Scrunch `responses` field --> |
| Reporting Period | {{scrunch_start_date}} to {{scrunch_end_date}} | |

### Visibility by Platform

<!-- Query API grouped by ai_platform dimension. Platforms from Scrunch:
     chatgpt, perplexity, google_ai_overviews, google_ai_mode, google_gemini, meta, claude.
     Not all platforms may have data. Only include rows with data. -->

| Platform | Brand Presence % | Position Score | Sentiment Score | Responses |
|----------|-----------------|----------------|-----------------|-----------|
| ChatGPT | {{chatgpt_presence}}% | {{chatgpt_position}} | {{chatgpt_sentiment}} | {{chatgpt_responses}} |
| Perplexity | {{perplexity_presence}}% | {{perplexity_position}} | {{perplexity_sentiment}} | {{perplexity_responses}} |
| Google AI Overviews | {{gaio_presence}}% | {{gaio_position}} | {{gaio_sentiment}} | {{gaio_responses}} |
| Google AI Mode | {{gaim_presence}}% | {{gaim_position}} | {{gaim_sentiment}} | {{gaim_responses}} |
| Google Gemini | {{gemini_presence}}% | {{gemini_position}} | {{gemini_sentiment}} | {{gemini_responses}} |
| Claude | {{claude_presence}}% | {{claude_position}} | {{claude_sentiment}} | {{claude_responses}} |
| Meta AI | {{meta_presence}}% | {{meta_position}} | {{meta_sentiment}} | {{meta_responses}} |

**Platform takeaway:** {{platform_takeaway}}

<!-- {{platform_takeaway}}: 1-2 sentence interpretation. e.g., "Strongest on Perplexity (real-time
     search favors fresh content), weakest on ChatGPT (training data lag, brand not established
     in parametric knowledge)." -->

### Visibility by Buyer Journey Stage

<!-- Query API grouped by stage dimension. Scrunch stages:
     Advice, Awareness, Evaluation, Comparison, Other -->

| Stage | Brand Presence % | Position Score | Sentiment Score | Responses |
|-------|-----------------|----------------|-----------------|-----------|
| Advice | {{advice_presence}}% | {{advice_position}} | {{advice_sentiment}} | {{advice_responses}} |
| Awareness | {{awareness_presence}}% | {{awareness_position}} | {{awareness_sentiment}} | {{awareness_responses}} |
| Evaluation | {{evaluation_presence}}% | {{evaluation_position}} | {{evaluation_sentiment}} | {{evaluation_responses}} |
| Comparison | {{comparison_presence}}% | {{comparison_position}} | {{comparison_sentiment}} | {{comparison_responses}} |
| Other | {{other_presence}}% | {{other_position}} | {{other_sentiment}} | {{other_responses}} |

**Stage takeaway:** {{stage_takeaway}}

<!-- {{stage_takeaway}}: e.g., "Invisible at the Advice stage where buyers ask 'what should I use?'
     This is the highest-leverage gap because Advice prompts drive initial consideration sets." -->

### Competitor Presence

<!-- Query API with competitor_name and competitor_presence_percentage.
     Show the brand alongside its configured competitors for direct comparison. -->

| Company | Brand Presence % | Position Score | Sentiment Score |
|---------|-----------------|----------------|-----------------|
| **{{company}}** | **{{brand_presence_pct}}%** | **{{position_score}}** | **{{sentiment_score}}** |
| {{competitor_1}} | {{comp1_presence}}% | {{comp1_position}} | {{comp1_sentiment}} |
| {{competitor_2}} | {{comp2_presence}}% | {{comp2_position}} | {{comp2_sentiment}} |
| {{competitor_3}} | {{comp3_presence}}% | {{comp3_position}} | {{comp3_sentiment}} |

**Competitive takeaway:** {{competitive_takeaway}}

### Strongest Prompts

<!-- From Responses API: prompts where brand_present=true, sorted by frequency of appearance.
     Show top 5-8 prompts where the brand performs best. -->

| Prompt | Platform | Stage | Presence | Position | Sentiment |
|--------|----------|-------|----------|----------|-----------|
| {{strong_prompt_1}} | {{strong_prompt_1_platform}} | {{strong_prompt_1_stage}} | {{strong_prompt_1_presence}}% | {{strong_prompt_1_position}} | {{strong_prompt_1_sentiment}} |
| {{strong_prompt_2}} | {{strong_prompt_2_platform}} | {{strong_prompt_2_stage}} | {{strong_prompt_2_presence}}% | {{strong_prompt_2_position}} | {{strong_prompt_2_sentiment}} |
| {{strong_prompt_3}} | {{strong_prompt_3_platform}} | {{strong_prompt_3_stage}} | {{strong_prompt_3_presence}}% | {{strong_prompt_3_position}} | {{strong_prompt_3_sentiment}} |
| {{strong_prompt_4}} | {{strong_prompt_4_platform}} | {{strong_prompt_4_stage}} | {{strong_prompt_4_presence}}% | {{strong_prompt_4_position}} | {{strong_prompt_4_sentiment}} |
| {{strong_prompt_5}} | {{strong_prompt_5_platform}} | {{strong_prompt_5_stage}} | {{strong_prompt_5_presence}}% | {{strong_prompt_5_position}} | {{strong_prompt_5_sentiment}} |

### Weakest Prompts (Zero or Near-Zero Visibility)

<!-- From Responses API: prompts where brand_present=false or presence is very low.
     These represent the biggest content gaps. -->

| Prompt | Stage | Current Presence | Competitor Presence | Gap |
|--------|-------|------------------|--------------------|----|
| {{weak_prompt_1}} | {{weak_prompt_1_stage}} | {{weak_prompt_1_presence}}% | {{weak_prompt_1_comp_presence}}% | {{weak_prompt_1_gap}} |
| {{weak_prompt_2}} | {{weak_prompt_2_stage}} | {{weak_prompt_2_presence}}% | {{weak_prompt_2_comp_presence}}% | {{weak_prompt_2_gap}} |
| {{weak_prompt_3}} | {{weak_prompt_3_stage}} | {{weak_prompt_3_presence}}% | {{weak_prompt_3_comp_presence}}% | {{weak_prompt_3_gap}} |
| {{weak_prompt_4}} | {{weak_prompt_4_stage}} | {{weak_prompt_4_presence}}% | {{weak_prompt_4_comp_presence}}% | {{weak_prompt_4_gap}} |
| {{weak_prompt_5}} | {{weak_prompt_5_stage}} | {{weak_prompt_5_presence}}% | {{weak_prompt_5_comp_presence}}% | {{weak_prompt_5_gap}} |

---

## 4. Full Site Audit

<!-- Data in this section comes from running the aeo-audit skill against the company's
     website pages. The assembler should audit the homepage + key product/pricing/about/
     blog pages and consolidate findings here. -->

### Pages Audited

<!-- List every URL that was audited, with page type classification. -->

| # | URL | Page Type |
|---|-----|-----------|
| 1 | {{audit_url_1}} | {{audit_url_1_type}} |
| 2 | {{audit_url_2}} | {{audit_url_2_type}} |
| 3 | {{audit_url_3}} | {{audit_url_3_type}} |
| 4 | {{audit_url_4}} | {{audit_url_4_type}} |
| 5 | {{audit_url_5}} | {{audit_url_5_type}} |
| 6 | {{audit_url_6}} | {{audit_url_6_type}} |
| 7 | {{audit_url_7}} | {{audit_url_7_type}} |
| 8 | {{audit_url_8}} | {{audit_url_8_type}} |

<!-- Add or remove rows as needed. Typical audits cover 6-12 pages. -->

### Questions Answered

<!-- Consolidated from all page audits. Each row is a question the site currently answers,
     with quality assessment per the aeo-audit scoring criteria. -->

| Question | Source Page | Priority | Intent | Quality | Notes |
|----------|------------|----------|--------|---------|-------|
| {{qa_question_1}} | {{qa_source_1}} | {{qa_priority_1}} | {{qa_intent_1}} | {{qa_quality_1}} | {{qa_notes_1}} |
| {{qa_question_2}} | {{qa_source_2}} | {{qa_priority_2}} | {{qa_intent_2}} | {{qa_quality_2}} | {{qa_notes_2}} |
| {{qa_question_3}} | {{qa_source_3}} | {{qa_priority_3}} | {{qa_intent_3}} | {{qa_quality_3}} | {{qa_notes_3}} |
| {{qa_question_4}} | {{qa_source_4}} | {{qa_priority_4}} | {{qa_intent_4}} | {{qa_quality_4}} | {{qa_notes_4}} |
| {{qa_question_5}} | {{qa_source_5}} | {{qa_priority_5}} | {{qa_intent_5}} | {{qa_quality_5}} | {{qa_notes_5}} |

<!-- Continue for all questions identified across audited pages.
     Priority: High / Medium / Low (per AEO 4-criteria qualification)
     Intent: Evaluation / Fear / Outcome / Process
     Quality: Strong / Adequate / Weak -->

### Critical Missing Questions

<!-- Questions a buyer would obviously ask that NO audited page answers. These are the
     highest-priority content gaps. -->

| Question | Priority | Intent | Why It Matters |
|----------|----------|--------|----------------|
| {{missing_q_1}} | {{missing_q_1_priority}} | {{missing_q_1_intent}} | {{missing_q_1_why}} |
| {{missing_q_2}} | {{missing_q_2_priority}} | {{missing_q_2_intent}} | {{missing_q_2_why}} |
| {{missing_q_3}} | {{missing_q_3_priority}} | {{missing_q_3_intent}} | {{missing_q_3_why}} |
| {{missing_q_4}} | {{missing_q_4_priority}} | {{missing_q_4_intent}} | {{missing_q_4_why}} |
| {{missing_q_5}} | {{missing_q_5_priority}} | {{missing_q_5_intent}} | {{missing_q_5_why}} |

<!-- Continue for all critical missing questions. -->

### Intent Coverage Map

<!-- Rate each intent bucket based on the aggregate of all page audits.
     Strong = multiple high-quality answers covering the intent
     Partial = some answers but gaps remain
     Gap = little to no coverage of this intent -->

| Intent | Coverage | Key Strength | Key Gap |
|--------|----------|-------------|---------|
| **Evaluation** (Is this right for me?) | {{eval_coverage}} | {{eval_strength}} | {{eval_gap}} |
| **Fear** (What could go wrong?) | {{fear_coverage}} | {{fear_strength}} | {{fear_gap}} |
| **Outcome** (What actually changes?) | {{outcome_coverage}} | {{outcome_strength}} | {{outcome_gap}} |
| **Process** (What does this look like?) | {{process_coverage}} | {{process_strength}} | {{process_gap}} |

### Technical Scorecard

<!-- Consolidated technical assessment across all audited pages.
     Pass = implemented correctly
     Fail = missing or broken
     Weak = partially implemented or poorly executed -->

| Check | Status | Notes |
|-------|--------|-------|
| AI Crawler Access (GPTBot, ClaudeBot, PerplexityBot) | {{tech_crawler_status}} | {{tech_crawler_notes}} |
| Schema Markup (Organization, Product, FAQ, Article) | {{tech_schema_status}} | {{tech_schema_notes}} |
| Heading Hierarchy (H1 > H2 > H3, no skips) | {{tech_headings_status}} | {{tech_headings_notes}} |
| Answer Extractability (40-60 word direct answers under H2s) | {{tech_extractability_status}} | {{tech_extractability_notes}} |
| Entity Signals (sameAs, consistent naming, category) | {{tech_entity_status}} | {{tech_entity_notes}} |
| Freshness Signals ("Last Updated" timestamps, recent dates) | {{tech_freshness_status}} | {{tech_freshness_notes}} |
| Citation Triggers (statistics, source citations, expert quotes) | {{tech_citation_status}} | {{tech_citation_notes}} |
| Content Modularity (self-contained sections, passage-ready) | {{tech_modularity_status}} | {{tech_modularity_notes}} |

---

## 5. Content Gaps by Type

<!-- Three gap categories, each representing a different problem and requiring a different fix. -->

### Gap 1: No Educational Content Exists

<!-- Questions where no page on the site addresses the topic at all.
     These require new content creation. -->

| Query / Question | Intent | Why It Matters | Recommended Surface |
|-----------------|--------|----------------|-------------------|
| {{gap1_query_1}} | {{gap1_intent_1}} | {{gap1_why_1}} | {{gap1_surface_1}} |
| {{gap1_query_2}} | {{gap1_intent_2}} | {{gap1_why_2}} | {{gap1_surface_2}} |
| {{gap1_query_3}} | {{gap1_intent_3}} | {{gap1_why_3}} | {{gap1_surface_3}} |
| {{gap1_query_4}} | {{gap1_intent_4}} | {{gap1_why_4}} | {{gap1_surface_4}} |
| {{gap1_query_5}} | {{gap1_intent_5}} | {{gap1_why_5}} | {{gap1_surface_5}} |

### Gap 2: Pages Exist but Are AEO-Invisible

<!-- Pages that address the topic but are structured in ways AI cannot extract or cite.
     These require rewriting/restructuring existing content. -->

| Existing Page | Problem | Query It Should Answer | Fix |
|--------------|---------|----------------------|-----|
| {{gap2_page_1}} | {{gap2_problem_1}} | {{gap2_query_1}} | {{gap2_fix_1}} |
| {{gap2_page_2}} | {{gap2_problem_2}} | {{gap2_query_2}} | {{gap2_fix_2}} |
| {{gap2_page_3}} | {{gap2_problem_3}} | {{gap2_query_3}} | {{gap2_fix_3}} |
| {{gap2_page_4}} | {{gap2_problem_4}} | {{gap2_query_4}} | {{gap2_fix_4}} |
| {{gap2_page_5}} | {{gap2_problem_5}} | {{gap2_query_5}} | {{gap2_fix_5}} |

### Gap 3: Missing Competitor & Comparison Content

<!-- Competitor comparison, alternatives, and "vs." content that buyers search for
     but the company has not created. Highest-leverage BOFU content. -->

| Query Pattern | Competitor(s) Referenced | Current State | Priority |
|--------------|------------------------|---------------|----------|
| {{gap3_query_1}} | {{gap3_competitors_1}} | {{gap3_state_1}} | {{gap3_priority_1}} |
| {{gap3_query_2}} | {{gap3_competitors_2}} | {{gap3_state_2}} | {{gap3_priority_2}} |
| {{gap3_query_3}} | {{gap3_competitors_3}} | {{gap3_state_3}} | {{gap3_priority_3}} |
| {{gap3_query_4}} | {{gap3_competitors_4}} | {{gap3_state_4}} | {{gap3_priority_4}} |
| {{gap3_query_5}} | {{gap3_competitors_5}} | {{gap3_state_5}} | {{gap3_priority_5}} |

---

## 6. Priority Questions to Own

<!-- The filtered, qualified, and prioritized list of questions the company should build
     canonical answers for. Derived from Sections 3-5 (Scrunch gaps + site audit gaps +
     content gaps). Qualified using the 4 AEO criteria from the aeo-audit skill. -->

### High Priority

<!-- High = meets all 4 AEO criteria: decision uncertainty, AI encounter likelihood,
     category-shaping, brand credibility. These are the "must own" questions. -->

| # | Question | Intent | Why Own This |
|---|----------|--------|-------------|
| 1 | {{high_q_1}} | {{high_q_1_intent}} | {{high_q_1_why}} |
| 2 | {{high_q_2}} | {{high_q_2_intent}} | {{high_q_2_why}} |
| 3 | {{high_q_3}} | {{high_q_3_intent}} | {{high_q_3_why}} |
| 4 | {{high_q_4}} | {{high_q_4_intent}} | {{high_q_4_why}} |
| 5 | {{high_q_5}} | {{high_q_5_intent}} | {{high_q_5_why}} |
| 6 | {{high_q_6}} | {{high_q_6_intent}} | {{high_q_6_why}} |
| 7 | {{high_q_7}} | {{high_q_7_intent}} | {{high_q_7_why}} |
| 8 | {{high_q_8}} | {{high_q_8_intent}} | {{high_q_8_why}} |

### Medium Priority

<!-- Medium = meets 2-3 of the 4 AEO criteria. Worth building, but after High priority. -->

| # | Question | Intent | Why Own This |
|---|----------|--------|-------------|
| 1 | {{med_q_1}} | {{med_q_1_intent}} | {{med_q_1_why}} |
| 2 | {{med_q_2}} | {{med_q_2_intent}} | {{med_q_2_why}} |
| 3 | {{med_q_3}} | {{med_q_3_intent}} | {{med_q_3_why}} |
| 4 | {{med_q_4}} | {{med_q_4_intent}} | {{med_q_4_why}} |
| 5 | {{med_q_5}} | {{med_q_5_intent}} | {{med_q_5_why}} |
| 6 | {{med_q_6}} | {{med_q_6_intent}} | {{med_q_6_why}} |

### Top 10 Revenue-Driving Prompts

<!-- The specific AI prompts (from Scrunch tracking or manual testing) that are most likely
     to drive pipeline. Prioritized by: buyer intent stage + purchase proximity + search volume. -->

| # | Prompt | Stage | Current Visibility | Competitor Visibility | Revenue Signal |
|---|--------|-------|-------------------|----------------------|----------------|
| 1 | {{rev_prompt_1}} | {{rev_prompt_1_stage}} | {{rev_prompt_1_visibility}}% | {{rev_prompt_1_comp}}% | {{rev_prompt_1_signal}} |
| 2 | {{rev_prompt_2}} | {{rev_prompt_2_stage}} | {{rev_prompt_2_visibility}}% | {{rev_prompt_2_comp}}% | {{rev_prompt_2_signal}} |
| 3 | {{rev_prompt_3}} | {{rev_prompt_3_stage}} | {{rev_prompt_3_visibility}}% | {{rev_prompt_3_comp}}% | {{rev_prompt_3_signal}} |
| 4 | {{rev_prompt_4}} | {{rev_prompt_4_stage}} | {{rev_prompt_4_visibility}}% | {{rev_prompt_4_comp}}% | {{rev_prompt_4_signal}} |
| 5 | {{rev_prompt_5}} | {{rev_prompt_5_stage}} | {{rev_prompt_5_visibility}}% | {{rev_prompt_5_comp}}% | {{rev_prompt_5_signal}} |
| 6 | {{rev_prompt_6}} | {{rev_prompt_6_stage}} | {{rev_prompt_6_visibility}}% | {{rev_prompt_6_comp}}% | {{rev_prompt_6_signal}} |
| 7 | {{rev_prompt_7}} | {{rev_prompt_7_stage}} | {{rev_prompt_7_visibility}}% | {{rev_prompt_7_comp}}% | {{rev_prompt_7_signal}} |
| 8 | {{rev_prompt_8}} | {{rev_prompt_8_stage}} | {{rev_prompt_8_visibility}}% | {{rev_prompt_8_comp}}% | {{rev_prompt_8_signal}} |
| 9 | {{rev_prompt_9}} | {{rev_prompt_9_stage}} | {{rev_prompt_9_visibility}}% | {{rev_prompt_9_comp}}% | {{rev_prompt_9_signal}} |
| 10 | {{rev_prompt_10}} | {{rev_prompt_10_stage}} | {{rev_prompt_10_visibility}}% | {{rev_prompt_10_comp}}% | {{rev_prompt_10_signal}} |

<!-- {{rev_prompt_N_signal}}: e.g., "Appears in 40% of Comparison-stage prompts",
     "Direct purchase-intent query", "Maps to top sales objection" -->

### Full Tracking Query List

<!-- The complete list of queries to track in Scrunch (or manually) over time.
     Typically 15-20 queries covering all stages and intents. -->

| # | Query | Stage | Intent | Current Visibility | Gap Size |
|---|-------|-------|--------|-------------------|----------|
| 1 | {{track_query_1}} | {{track_stage_1}} | {{track_intent_1}} | {{track_visibility_1}}% | {{track_gap_1}} |
| 2 | {{track_query_2}} | {{track_stage_2}} | {{track_intent_2}} | {{track_visibility_2}}% | {{track_gap_2}} |
| 3 | {{track_query_3}} | {{track_stage_3}} | {{track_intent_3}} | {{track_visibility_3}}% | {{track_gap_3}} |
| 4 | {{track_query_4}} | {{track_stage_4}} | {{track_intent_4}} | {{track_visibility_4}}% | {{track_gap_4}} |
| 5 | {{track_query_5}} | {{track_stage_5}} | {{track_intent_5}} | {{track_visibility_5}}% | {{track_gap_5}} |
| 6 | {{track_query_6}} | {{track_stage_6}} | {{track_intent_6}} | {{track_visibility_6}}% | {{track_gap_6}} |
| 7 | {{track_query_7}} | {{track_stage_7}} | {{track_intent_7}} | {{track_visibility_7}}% | {{track_gap_7}} |
| 8 | {{track_query_8}} | {{track_stage_8}} | {{track_intent_8}} | {{track_visibility_8}}% | {{track_gap_8}} |
| 9 | {{track_query_9}} | {{track_stage_9}} | {{track_intent_9}} | {{track_visibility_9}}% | {{track_gap_9}} |
| 10 | {{track_query_10}} | {{track_stage_10}} | {{track_intent_10}} | {{track_visibility_10}}% | {{track_gap_10}} |
| 11 | {{track_query_11}} | {{track_stage_11}} | {{track_intent_11}} | {{track_visibility_11}}% | {{track_gap_11}} |
| 12 | {{track_query_12}} | {{track_stage_12}} | {{track_intent_12}} | {{track_visibility_12}}% | {{track_gap_12}} |
| 13 | {{track_query_13}} | {{track_stage_13}} | {{track_intent_13}} | {{track_visibility_13}}% | {{track_gap_13}} |
| 14 | {{track_query_14}} | {{track_stage_14}} | {{track_intent_14}} | {{track_visibility_14}}% | {{track_gap_14}} |
| 15 | {{track_query_15}} | {{track_stage_15}} | {{track_intent_15}} | {{track_visibility_15}}% | {{track_gap_15}} |

<!-- {{track_gap_N}}: Qualitative gap size. e.g., "Critical (0% vs. competitors at 45%)",
     "Large (8% vs. leader at 60%)", "Moderate (25% vs. leader at 50%)" -->

---

## 7. Content Production Plan

<!-- Tiered content plan moving from immediate quick wins to sustained production.
     Each content piece is fully specified so a writer can execute without back-and-forth. -->

### Tier 1: This Week

<!-- Highest-urgency content. Typically: rewrites of existing pages to add answer-first
     structure, or net-new BOFU content addressing the biggest visibility gaps. -->

#### {{tier1_title_1}}

- **Why now:** {{tier1_why_1}}
- **Target prompts:**
  - `{{tier1_prompt_1a}}` (baseline: {{tier1_baseline_1a}}% visibility)
  - `{{tier1_prompt_1b}}` (baseline: {{tier1_baseline_1b}}% visibility)
- **Target visibility:** {{tier1_target_visibility_1}}%
- **Content spec:**
  - Word count: {{tier1_wordcount_1}}
  - Opening paragraph: {{tier1_opening_1}}
  - Required honest concessions: {{tier1_concessions_1}}
  - Where {{company}} wins: {{tier1_wins_1}}
  - Surface: {{tier1_surface_1}}

#### {{tier1_title_2}}

- **Why now:** {{tier1_why_2}}
- **Target prompts:**
  - `{{tier1_prompt_2a}}` (baseline: {{tier1_baseline_2a}}% visibility)
  - `{{tier1_prompt_2b}}` (baseline: {{tier1_baseline_2b}}% visibility)
- **Target visibility:** {{tier1_target_visibility_2}}%
- **Content spec:**
  - Word count: {{tier1_wordcount_2}}
  - Opening paragraph: {{tier1_opening_2}}
  - Required honest concessions: {{tier1_concessions_2}}
  - Where {{company}} wins: {{tier1_wins_2}}
  - Surface: {{tier1_surface_2}}

<!-- Repeat for each Tier 1 content piece. Typically 2-3 pieces. -->

### Tier 2: Next 2 Weeks

#### {{tier2_title_1}}

- **Why now:** {{tier2_why_1}}
- **Target prompts:**
  - `{{tier2_prompt_1a}}` (baseline: {{tier2_baseline_1a}}% visibility)
  - `{{tier2_prompt_1b}}` (baseline: {{tier2_baseline_1b}}% visibility)
- **Target visibility:** {{tier2_target_visibility_1}}%
- **Content spec:**
  - Word count: {{tier2_wordcount_1}}
  - Opening paragraph: {{tier2_opening_1}}
  - Required honest concessions: {{tier2_concessions_1}}
  - Where {{company}} wins: {{tier2_wins_1}}
  - Surface: {{tier2_surface_1}}

#### {{tier2_title_2}}

- **Why now:** {{tier2_why_2}}
- **Target prompts:**
  - `{{tier2_prompt_2a}}` (baseline: {{tier2_baseline_2a}}% visibility)
  - `{{tier2_prompt_2b}}` (baseline: {{tier2_baseline_2b}}% visibility)
- **Target visibility:** {{tier2_target_visibility_2}}%
- **Content spec:**
  - Word count: {{tier2_wordcount_2}}
  - Opening paragraph: {{tier2_opening_2}}
  - Required honest concessions: {{tier2_concessions_2}}
  - Where {{company}} wins: {{tier2_wins_2}}
  - Surface: {{tier2_surface_2}}

#### {{tier2_title_3}}

- **Why now:** {{tier2_why_3}}
- **Target prompts:**
  - `{{tier2_prompt_3a}}` (baseline: {{tier2_baseline_3a}}% visibility)
  - `{{tier2_prompt_3b}}` (baseline: {{tier2_baseline_3b}}% visibility)
- **Target visibility:** {{tier2_target_visibility_3}}%
- **Content spec:**
  - Word count: {{tier2_wordcount_3}}
  - Opening paragraph: {{tier2_opening_3}}
  - Required honest concessions: {{tier2_concessions_3}}
  - Where {{company}} wins: {{tier2_wins_3}}
  - Surface: {{tier2_surface_3}}

<!-- Repeat for each Tier 2 content piece. Typically 3-5 pieces. -->

### Tier 3: Next 30 Days

#### {{tier3_title_1}}

- **Why now:** {{tier3_why_1}}
- **Target prompts:**
  - `{{tier3_prompt_1a}}` (baseline: {{tier3_baseline_1a}}% visibility)
  - `{{tier3_prompt_1b}}` (baseline: {{tier3_baseline_1b}}% visibility)
- **Target visibility:** {{tier3_target_visibility_1}}%
- **Content spec:**
  - Word count: {{tier3_wordcount_1}}
  - Opening paragraph: {{tier3_opening_1}}
  - Required honest concessions: {{tier3_concessions_1}}
  - Where {{company}} wins: {{tier3_wins_1}}
  - Surface: {{tier3_surface_1}}

#### {{tier3_title_2}}

- **Why now:** {{tier3_why_2}}
- **Target prompts:**
  - `{{tier3_prompt_2a}}` (baseline: {{tier3_baseline_2a}}% visibility)
  - `{{tier3_prompt_2b}}` (baseline: {{tier3_baseline_2b}}% visibility)
- **Target visibility:** {{tier3_target_visibility_2}}%
- **Content spec:**
  - Word count: {{tier3_wordcount_2}}
  - Opening paragraph: {{tier3_opening_2}}
  - Required honest concessions: {{tier3_concessions_2}}
  - Where {{company}} wins: {{tier3_wins_2}}
  - Surface: {{tier3_surface_2}}

#### {{tier3_title_3}}

- **Why now:** {{tier3_why_3}}
- **Target prompts:**
  - `{{tier3_prompt_3a}}` (baseline: {{tier3_baseline_3a}}% visibility)
  - `{{tier3_prompt_3b}}` (baseline: {{tier3_baseline_3b}}% visibility)
- **Target visibility:** {{tier3_target_visibility_3}}%
- **Content spec:**
  - Word count: {{tier3_wordcount_3}}
  - Opening paragraph: {{tier3_opening_3}}
  - Required honest concessions: {{tier3_concessions_3}}
  - Where {{company}} wins: {{tier3_wins_3}}
  - Surface: {{tier3_surface_3}}

<!-- Repeat for each Tier 3 content piece. Typically 3-5 pieces. -->

### Tier 4: Ongoing Production

<!-- Recurring content types that sustain visibility after the initial push.
     These are patterns, not individual pieces. -->

| Content Type | Cadence | Target Prompts | Owner |
|-------------|---------|---------------|-------|
| {{tier4_type_1}} | {{tier4_cadence_1}} | {{tier4_prompts_1}} | {{tier4_owner_1}} |
| {{tier4_type_2}} | {{tier4_cadence_2}} | {{tier4_prompts_2}} | {{tier4_owner_2}} |
| {{tier4_type_3}} | {{tier4_cadence_3}} | {{tier4_prompts_3}} | {{tier4_owner_3}} |
| {{tier4_type_4}} | {{tier4_cadence_4}} | {{tier4_prompts_4}} | {{tier4_owner_4}} |

<!-- Examples of Tier 4 content types:
     - Quarterly competitor comparison refreshes
     - Monthly "state of [category]" blog posts
     - Bi-weekly customer story / proof point articles
     - Weekly FAQ additions from sales call mining -->

---

## 8. Answer Ownership Map

<!-- Maps every high-priority question to a specific surface (page/section) on the site.
     This is the execution blueprint for content placement. -->

| Question | Surface | URL (if exists) | Status | Notes |
|----------|---------|-----------------|--------|-------|
| {{aom_question_1}} | {{aom_surface_1}} | {{aom_url_1}} | {{aom_status_1}} | {{aom_notes_1}} |
| {{aom_question_2}} | {{aom_surface_2}} | {{aom_url_2}} | {{aom_status_2}} | {{aom_notes_2}} |
| {{aom_question_3}} | {{aom_surface_3}} | {{aom_url_3}} | {{aom_status_3}} | {{aom_notes_3}} |
| {{aom_question_4}} | {{aom_surface_4}} | {{aom_url_4}} | {{aom_status_4}} | {{aom_notes_4}} |
| {{aom_question_5}} | {{aom_surface_5}} | {{aom_url_5}} | {{aom_status_5}} | {{aom_notes_5}} |
| {{aom_question_6}} | {{aom_surface_6}} | {{aom_url_6}} | {{aom_status_6}} | {{aom_notes_6}} |
| {{aom_question_7}} | {{aom_surface_7}} | {{aom_url_7}} | {{aom_status_7}} | {{aom_notes_7}} |
| {{aom_question_8}} | {{aom_surface_8}} | {{aom_url_8}} | {{aom_status_8}} | {{aom_notes_8}} |
| {{aom_question_9}} | {{aom_surface_9}} | {{aom_url_9}} | {{aom_status_9}} | {{aom_notes_9}} |
| {{aom_question_10}} | {{aom_surface_10}} | {{aom_url_10}} | {{aom_status_10}} | {{aom_notes_10}} |

<!-- {{aom_surface_N}}: One of:
     - "Dedicated page" (category-shaping questions that appear across sources)
     - "Blog post" (educational, thought leadership, comparison content)
     - "Section on [page name]" (important but secondary)
     - "FAQ on [page name]" (narrow, friction-removing)
     - "Homepage section" (core value prop questions)

     {{aom_status_N}}: One of:
     - "Exists, needs rewrite" (page exists but answer quality is Weak/Adequate)
     - "Exists, adequate" (page exists and answer quality is acceptable)
     - "Does not exist" (net-new content needed)
     - "Wrong surface" (answer exists but on the wrong page type) -->

---

## 9. Writing Rules for Canonical Answers

<!-- This section is largely generic/templatable. It provides the writing playbook
     that the content team follows when producing AEO content. Company-specific
     examples are injected where marked. -->

Every canonical answer produced for {{company}} must follow these five rules:

### Rule 1: Answer-First

State the answer in the first sentence. No preamble, no throat-clearing, no "great question" openers. AI systems extract the first 40-60 words under a heading as standalone snippets. If your answer doesn't appear in that window, it won't get cited.

**Bad:**
> At {{company}}, we believe in providing our customers with the best possible experience. That's why we've designed our platform to be both powerful and easy to use. When it comes to [topic], there are several factors to consider...

**Good:**
> {{company}} {{answer_first_example}}

<!-- {{answer_first_example}}: A real answer-first example using the company's actual product.
     e.g., "charges $99/month for teams up to 10 users, scaling to $299/month for unlimited
     users. All plans include core features." -->

### Rule 2: Tradeoff Honesty

Name the downsides and limitations alongside the strengths. AI models are structurally programmed to present pros and cons. If you don't provide the cons, AI will source them from reviews, Reddit, and competitor content, and you lose control of the framing.

**Pattern:** "[Strength]. The tradeoff is [honest limitation]. For teams that [use case], this matters less because [reason]."

### Rule 3: Extractable (40-60 Words)

Write a self-contained answer paragraph (40-60 words) that makes complete sense ripped out of context. No pronouns pointing to earlier sections. No "as mentioned above." No acronyms without expansion. This is the passage that AI systems will retrieve and cite.

### Rule 4: Specific Numbers

Use concrete numbers, timelines, and data points. Not "fast implementation" but "14-day average implementation." Not "high customer satisfaction" but "92% retention rate" or "4.6/5.0 on G2 with {{g2_review_count}} reviews." Named statistics with source and year improve AI visibility by 41%.

### Rule 5: Buyer Language

Write in the language buyers use, not the language the marketing team uses. If sales calls reveal that prospects say "{{buyer_term_example}}" rather than "{{marketing_term_example}}", use the buyer's version. AI models match prompts to content based on language overlap.

<!-- {{buyer_term_example}}: A real term buyers use on calls
     {{marketing_term_example}}: The marketing-speak version the company currently uses
     These come from Gong/Fathom transcript mining or sales team interviews. -->

---

## 10. Question Mining Playbook

<!-- Process for continuously finding new questions to answer. Generic framework
     with company-specific action items injected. -->

### Tier 1: Highest Signal (Internal Data)

**Sources:** Sales calls (Gong/Fathom/Chorus), CS tickets, demo recordings, onboarding calls

**Process for {{company}}:**

1. {{mining_tier1_action_1}}
2. {{mining_tier1_action_2}}
3. {{mining_tier1_action_3}}

<!-- {{mining_tier1_action_N}}: Company-specific actions based on their tooling.
     e.g., "Export last 90 days of Gong recordings. Run through Claude with the
     question extraction prompt (see AEO Strategy skill, Step 2)."
     e.g., "Pull Zendesk tickets tagged 'pre-sale' from the last quarter."
     e.g., "Ask [Sales Lead Name] to list the top 10 objections heard this quarter." -->

**What to extract:**
- Every question prospects ask before signing
- Every objection that comes up during pricing conversations
- The language buyers use to describe their problem (not your category name)
- Questions that close deals (these become Outcome content)
- Questions that kill deals (these become Fear content with honest answers)

### Tier 2: High Signal (Public Reviews & Roundups)

**Sources:** G2 reviews, Capterra reviews, app store reviews, "Best [category] tools" roundup articles

**Process for {{company}}:**

1. {{mining_tier2_action_1}}
2. {{mining_tier2_action_2}}
3. {{mining_tier2_action_3}}

<!-- {{mining_tier2_action_N}}: e.g., "Scrape G2 reviews for {{company}} and top 3 competitors.
     Extract every question, complaint, and comparison mentioned."
     e.g., "Find the top 10 '[category] software' roundup articles. Note which questions
     each article answers and which features they compare." -->

**What to extract:**
- Post-purchase regret signals ("I wish I had known...")
- Comparison language ("We switched from X because...")
- Feature expectations that indicate category confusion
- Questions asked in the review Q&A sections

### Tier 3: Supplementary Signal (Community & Social)

**Sources:** Reddit (r/{{subreddit_1}}, r/{{subreddit_2}}), LinkedIn posts/comments, industry forums

**Process for {{company}}:**

1. {{mining_tier3_action_1}}
2. {{mining_tier3_action_2}}
3. {{mining_tier3_action_3}}

<!-- {{mining_tier3_action_N}}: e.g., "Monitor r/saas and r/[industry] for threads mentioning
     [category] or competitor names. Mine comments, not posts."
     e.g., "Search LinkedIn for posts by [buyer persona title] discussing [problem category]." -->

**What to extract:**
- Natural language buyer questions (these mirror AI prompts most closely)
- Emotional uncertainty signals
- Misconceptions about the category
- Questions that indicate where the category explanation is failing

### Qualification Gate

Every mined question goes through the 4 AEO criteria before entering the production backlog:

| Criterion | Test |
|-----------|------|
| Decision uncertainty | Does this question make someone pause before buying? |
| AI encounter likelihood | Would this appear across AI prompts about this category? |
| Category-shaping | Does answering this change how the category is understood? |
| Brand credibility | Can {{company}} credibly own the answer? |

**High priority** = meets all 4. **Medium** = meets 2-3. **Low** = meets 1 or fewer. Discard Low aggressively.

---

## 11. Third-Party Presence Strategy

<!-- AI models trust what others say about you far more than what you say about yourself.
     98.2% of AI citations point to third-party sources (Trakkr Study 006). -->

### Current State by Platform

| Platform | Profile Status | Review Count | Rating | Last Updated | Priority |
|----------|---------------|-------------|--------|-------------|----------|
| G2 | {{g2_status}} | {{g2_reviews}} | {{g2_rating}} | {{g2_updated}} | {{g2_priority}} |
| Capterra | {{capterra_status}} | {{capterra_reviews}} | {{capterra_rating}} | {{capterra_updated}} | {{capterra_priority}} |
| {{industry_review_site}} | {{irs_status}} | {{irs_reviews}} | {{irs_rating}} | {{irs_updated}} | {{irs_priority}} |
| LinkedIn (Company) | {{linkedin_status}} | N/A | N/A | {{linkedin_updated}} | {{linkedin_priority}} |
| Crunchbase | {{crunchbase_status}} | N/A | N/A | {{crunchbase_updated}} | {{crunchbase_priority}} |
| Roundup Articles | {{roundup_status}} | N/A | N/A | N/A | {{roundup_priority}} |

<!-- {{g2_status}}: e.g., "Complete profile", "Incomplete (missing pricing)", "No profile"
     {{industry_review_site}}: The vertical-specific review site (e.g., "TrustRadius",
     "Software Advice", "GetApp", or a niche industry directory) -->

### G2 Review Velocity Plan

<!-- G2 is the #1 most-cited domain in AI search. Review and directory sites have 4.8x
     citation efficiency. Volume matters: 2,000 reviews >> 12 reviews. -->

| Metric | Current | 90-Day Target |
|--------|---------|---------------|
| Total Reviews | {{g2_reviews}} | {{g2_target_reviews}} |
| Average Rating | {{g2_rating}} | {{g2_target_rating}} |
| Reviews in Last 90 Days | {{g2_recent_reviews}} | {{g2_target_recent}} |
| Review Velocity (per month) | {{g2_velocity}} | {{g2_target_velocity}} |

**Tactics:**
1. {{g2_tactic_1}}
2. {{g2_tactic_2}}
3. {{g2_tactic_3}}

<!-- {{g2_tactic_N}}: e.g., "Add G2 review request to post-onboarding email sequence (day 45)",
     "Run a quarterly G2 review campaign timed with G2 reporting periods",
     "Incentivize via G2's native gift card program (compliant with G2 TOS)" -->

### Roundup Influence Tactics

<!-- Getting {{company}} included in "Best [category] tools" roundup articles. -->

1. {{roundup_tactic_1}}
2. {{roundup_tactic_2}}
3. {{roundup_tactic_3}}

<!-- {{roundup_tactic_N}}: e.g., "Identify top 10 roundup articles for '[category] software'.
     Reach out to authors with updated product info and offer a demo/trial.",
     "Create a press page with product screenshots, key stats, and comparison data
     that roundup authors can pull from.",
     "Monitor new roundup articles monthly. Pitch inclusion within 2 weeks of publication." -->

### Original Research Play

<!-- Publishing proprietary data that third parties cite. This is the highest-leverage
     earned media tactic for AEO. -->

**Research concept:** {{research_concept}}

<!-- {{research_concept}}: A specific original research idea based on the company's data.
     e.g., "Annual [Industry] Benchmark Report using anonymized customer data.
     Publish key findings as individual blog posts, with the full report gated." -->

**Why this works for AEO:** Original data is uniquely citable because no other source has it. Journalists, analysts, and AI models all cite primary data over secondary summaries.

### Canonical Entity Description

<!-- A 75-word description aligned across all platforms. Used for entity consistency. -->

> {{canonical_entity_description}}

<!-- {{canonical_entity_description}}: Exactly 75 words. Covers: what the product does,
     who it serves, primary differentiator, founding date, key metric.
     This exact text should appear (or be closely reflected) on: About page,
     G2 profile, LinkedIn company description, Crunchbase, and any directory listings. -->

**Deploy this description to:** Homepage "About" section, G2 profile, LinkedIn company page, Crunchbase, {{other_directories}}.

---

## 12. Technical Foundations

<!-- Specific technical changes needed for AI visibility. Separated into Must-Do
     (blocking AI citation) and Nice-to-Have (incremental improvement). -->

### Must-Do

| # | Item | Current State | Action | Owner | Effort |
|---|------|--------------|--------|-------|--------|
| 1 | {{tech_must_1_item}} | {{tech_must_1_current}} | {{tech_must_1_action}} | {{tech_must_1_owner}} | {{tech_must_1_effort}} |
| 2 | {{tech_must_2_item}} | {{tech_must_2_current}} | {{tech_must_2_action}} | {{tech_must_2_owner}} | {{tech_must_2_effort}} |
| 3 | {{tech_must_3_item}} | {{tech_must_3_current}} | {{tech_must_3_action}} | {{tech_must_3_owner}} | {{tech_must_3_effort}} |
| 4 | {{tech_must_4_item}} | {{tech_must_4_current}} | {{tech_must_4_action}} | {{tech_must_4_owner}} | {{tech_must_4_effort}} |
| 5 | {{tech_must_5_item}} | {{tech_must_5_current}} | {{tech_must_5_action}} | {{tech_must_5_owner}} | {{tech_must_5_effort}} |

<!-- Typical Must-Do items:
     - Unblock AI crawlers in robots.txt
     - Add Organization schema with sameAs
     - Add FAQPage schema to FAQ content
     - Fix heading hierarchy (H1 > H2 > H3)
     - Add "Last Updated" timestamps to cornerstone content
     - Ensure SSR/SSG for JS-heavy pages
     - Add Article schema to blog template -->

### Nice-to-Have

- {{tech_nice_1}}
- {{tech_nice_2}}
- {{tech_nice_3}}
- {{tech_nice_4}}

<!-- Typical Nice-to-Have items:
     - Add llms.txt (low-cost, no proven impact)
     - BreadcrumbList schema for site navigation
     - Product schema with aggregateRating
     - Table of contents on long-form content
     - Author schema on blog posts -->

---

## 13. Platform Awareness

<!-- Prioritized platform focus based on where this company's buyers search
     and where the company has the best shot at visibility. -->

| Platform | Priority | Why | Key Action |
|----------|----------|-----|------------|
| Perplexity | {{perplexity_priority}} | {{perplexity_why}} | {{perplexity_action}} |
| ChatGPT | {{chatgpt_priority}} | {{chatgpt_why}} | {{chatgpt_action}} |
| Google AI Overviews | {{gaio_priority}} | {{gaio_why}} | {{gaio_action}} |
| Google Gemini | {{gemini_priority}} | {{gemini_why}} | {{gemini_action}} |
| Claude | {{claude_priority}} | {{claude_why}} | {{claude_action}} |
| Meta AI | {{meta_priority}} | {{meta_why}} | {{meta_action}} |

<!-- Priority: High / Medium / Low
     Base priority recommendations (adjust per company):
     - Perplexity: High (fastest wins, easiest to get cited, real-time search)
     - ChatGPT: High (largest audience, but training data lag)
     - Google AI Overviews/Mode: High (where most searches start)
     - Gemini: Medium (growing, directory-weighted)
     - Claude: Medium (highest authority bar, training data dependent)
     - Meta AI: Low (pure product crawler, limited B2B relevance)

     {{platform_why}}: One sentence grounded in the company's situation.
     e.g., "Perplexity: High. Real-time search means new content gets cited within days.
     Best platform for quick wins while building longer-term ChatGPT authority."

     {{platform_action}}: One specific action per platform.
     e.g., "Ensure product pages have expert context and use-case framing (GPTBot reads
     product pages 84% of the time but product pages have only 0.5x citation efficiency)." -->

---

## 14. Operating Cadence & Ownership

### Weekly: Question Triage (15 min)

- [ ] Review new questions from sales calls, CS tickets, and social mentions
- [ ] Flag anything new, rising, or showing language changes
- [ ] Add qualified questions to the backlog with source and frequency
- [ ] **Owner:** {{weekly_owner}}

<!-- {{weekly_owner}}: The person responsible for the weekly triage.
     Typically PMM, Content Lead, or Growth Lead. -->

### Monthly: Coverage Review (2 hours)

- [ ] Prioritize backlog against the 4 AEO criteria
- [ ] Assign canonical answers to create or update
- [ ] Check intent coverage map for emerging gaps
- [ ] Review Scrunch visibility trends for tracked prompts
- [ ] Check which answers are getting cited in AI responses (Scrunch Responses API)
- [ ] Refresh 1-2 cornerstone pages with updated stats and timestamps
- [ ] **Owner:** {{monthly_owner}}

### Quarterly: Full Refresh (half-day)

- [ ] Audit top-performing answers for accuracy and freshness
- [ ] Retire or merge answers that have drifted
- [ ] Update comparison content with current competitor positioning
- [ ] Recalibrate question sources (new subreddits, new competitors, new objections)
- [ ] Run entity alignment audit (homepage, G2, LinkedIn, top blog post)
- [ ] Review platform-specific visibility trends and adjust priorities
- [ ] Update this document with new data and revised targets
- [ ] **Owner:** {{quarterly_owner}}

### Role Ownership

| Role | Owns | Person at {{company}} |
|------|------|-----------------------|
| Canonical answers, AEO pages, comparison content, editorial calendar | PMM / Content | {{owner_content}} |
| Question mining automation, coverage tracking, performance measurement | Growth / SEO | {{owner_growth}} |
| Surfacing new objections, validating answer accuracy, using canonical answers in calls | Sales | {{owner_sales}} |
| Flagging expectation gaps, identifying onboarding friction, escalating repeated questions | Support | {{owner_support}} |
| In-product education, feature explanation, reducing support-driven confusion | Product | {{owner_product}} |

<!-- {{owner_content}}, {{owner_growth}}, etc.: Names of the actual people at the company
     who own each area. If no one is assigned, use "TBD (needs hire)" or "TBD (assign from
     existing team)". -->

---

## 15. Measurement Framework

### Metrics to Track

| Metric | Source | Cadence | Baseline | 90-Day Target |
|--------|--------|---------|----------|---------------|
| Brand Presence % (overall) | Scrunch Query API | Weekly | {{baseline_presence}}% | {{target_presence_90d}}% |
| Brand Presence % (Advice stage) | Scrunch Query API | Weekly | {{baseline_advice}}% | {{target_advice_90d}}% |
| Position Score | Scrunch Query API | Weekly | {{baseline_position}} | {{target_position_90d}} |
| Sentiment Score | Scrunch Query API | Weekly | {{baseline_sentiment}} | {{target_sentiment_90d}} |
| Zero-Visibility Queries | Scrunch Responses API | Monthly | {{baseline_zero_vis}} | {{target_zero_vis_90d}} |
| G2 Review Count | G2 Dashboard | Monthly | {{baseline_g2_reviews}} | {{target_g2_reviews_90d}} |
| AI-Referred Traffic | GA4 Referral Sources | Monthly | {{baseline_ai_traffic}} | {{target_ai_traffic_90d}} |
| Brand Search Volume | Google Search Console | Monthly | {{baseline_brand_search}} | {{target_brand_search_90d}} |

### Content-to-Prompt Map

<!-- Maps each content piece to the prompts it targets, so measurement can attribute
     visibility changes to specific content actions. -->

| Content Piece | Target Prompt(s) | Baseline Visibility | Post-Publish Visibility | Delta |
|--------------|-------------------|--------------------|-----------------------|-------|
| {{c2p_content_1}} | {{c2p_prompts_1}} | {{c2p_baseline_1}}% | {{c2p_post_1}}% | {{c2p_delta_1}} |
| {{c2p_content_2}} | {{c2p_prompts_2}} | {{c2p_baseline_2}}% | {{c2p_post_2}}% | {{c2p_delta_2}} |
| {{c2p_content_3}} | {{c2p_prompts_3}} | {{c2p_baseline_3}}% | {{c2p_post_3}}% | {{c2p_delta_3}} |
| {{c2p_content_4}} | {{c2p_prompts_4}} | {{c2p_baseline_4}}% | {{c2p_post_4}}% | {{c2p_delta_4}} |
| {{c2p_content_5}} | {{c2p_prompts_5}} | {{c2p_baseline_5}}% | {{c2p_post_5}}% | {{c2p_delta_5}} |

<!-- Post-publish and delta columns are filled during the monthly review cycle.
     Leave as "Pending" at initial document creation. -->

### Measurement Cadence

- **Baseline (Week 0):** Record all metrics above before any AEO work ships. This is the control.
- **Weekly:** Pull Scrunch visibility data. Flag any 15%+ drops for immediate investigation (possible competitive displacement).
- **Monthly:** Full metrics review. Update Content-to-Prompt Map. Pair visibility changes with downstream signals (brand search, direct traffic, AI-referred traffic).
- **Quarterly:** Comprehensive audit. Which content earned citations? Which platforms matter most? Do downstream business signals correlate with visibility changes?

### Scrunch API Queries

<!-- Pre-built API queries for the measurement cadence. Replace {{workspace_id}} with
     the company's Scrunch brand ID. -->

**Overall visibility (weekly pull):**
```bash
curl "https://api.scrunchai.com/v1/{{workspace_id}}/query?fields=date_week,brand_presence_percentage,brand_position_score,brand_sentiment_score&start_date={{measurement_start_date}}&end_date={{measurement_end_date}}" \
  -H "Authorization: Bearer $SCRUNCH_API_KEY"
```

**Visibility by platform:**
```bash
curl "https://api.scrunchai.com/v1/{{workspace_id}}/query?fields=ai_platform,brand_presence_percentage,brand_position_score,brand_sentiment_score&start_date={{measurement_start_date}}&end_date={{measurement_end_date}}" \
  -H "Authorization: Bearer $SCRUNCH_API_KEY"
```

**Visibility by stage:**
```bash
curl "https://api.scrunchai.com/v1/{{workspace_id}}/query?fields=stage,brand_presence_percentage,brand_position_score,brand_sentiment_score&start_date={{measurement_start_date}}&end_date={{measurement_end_date}}" \
  -H "Authorization: Bearer $SCRUNCH_API_KEY"
```

**Competitor comparison:**
```bash
curl "https://api.scrunchai.com/v1/{{workspace_id}}/query?fields=competitor_name,competitor_presence_percentage,competitor_position_score,competitor_sentiment_score&start_date={{measurement_start_date}}&end_date={{measurement_end_date}}" \
  -H "Authorization: Bearer $SCRUNCH_API_KEY"
```

**Response-level data for prompt analysis:**
```bash
curl "https://api.scrunchai.com/v1/{{workspace_id}}/responses?start_date={{measurement_start_date}}&end_date={{measurement_end_date}}&limit=100" \
  -H "Authorization: Bearer $SCRUNCH_API_KEY"
```

---

## 16. Phase Roadmap

| Phase | Name | Timeline | Key Deliverables | Success Criteria |
|-------|------|----------|-----------------|------------------|
| **1** | Site Refresh | Weeks 0-8 | {{phase1_deliverables}} | {{phase1_criteria}} |
| **2** | Answer Engine | Weeks 8-16 | {{phase2_deliverables}} | {{phase2_criteria}} |
| **3** | Earned Media | Ongoing (starts Week 8) | {{phase3_deliverables}} | {{phase3_criteria}} |
| **4** | Sustain | Ongoing (starts Week 16) | {{phase4_deliverables}} | {{phase4_criteria}} |

### Phase 1: Site Refresh (Weeks 0-8)

**Goal:** Fix the technical foundation and rewrite existing content for AI visibility.

<!-- This phase runs concurrently with any website refresh the company is doing.
     If there's no active refresh, this IS the refresh. -->

- [ ] {{phase1_task_1}}
- [ ] {{phase1_task_2}}
- [ ] {{phase1_task_3}}
- [ ] {{phase1_task_4}}
- [ ] {{phase1_task_5}}
- [ ] {{phase1_task_6}}

<!-- Typical Phase 1 tasks:
     - Implement all Must-Do technical items (Section 12)
     - Rewrite homepage for answer-first structure
     - Produce Tier 1 content (Section 7)
     - Align canonical entity description across platforms (Section 11)
     - Set up Scrunch monitoring with full prompt list
     - Record baseline measurements -->

### Phase 2: Answer Engine (Weeks 8-16)

**Goal:** Build the canonical answer library and fill the biggest content gaps.

- [ ] {{phase2_task_1}}
- [ ] {{phase2_task_2}}
- [ ] {{phase2_task_3}}
- [ ] {{phase2_task_4}}
- [ ] {{phase2_task_5}}
- [ ] {{phase2_task_6}}

<!-- Typical Phase 2 tasks:
     - Produce Tier 2 and Tier 3 content (Section 7)
     - Build comparison/alternatives pages for top 3 competitors
     - Launch G2 review velocity campaign
     - Mine sales calls for new questions (first full mining cycle)
     - Publish first piece of original research
     - Measure first visibility deltas from Phase 1 content -->

### Phase 3: Earned Media (Ongoing, starts Week 8)

**Goal:** Build third-party presence and get cited by sources AI trusts.

- [ ] {{phase3_task_1}}
- [ ] {{phase3_task_2}}
- [ ] {{phase3_task_3}}
- [ ] {{phase3_task_4}}

<!-- Typical Phase 3 tasks:
     - Pitch inclusion in top 10 roundup articles
     - Publish original research and secure press coverage
     - Get executives quoted in industry publications
     - Maintain G2 review velocity at target rate -->

### Phase 4: Sustain (Ongoing, starts Week 16)

**Goal:** Operate the AEO program at steady state with the operating cadence.

- [ ] {{phase4_task_1}}
- [ ] {{phase4_task_2}}
- [ ] {{phase4_task_3}}
- [ ] {{phase4_task_4}}

<!-- Typical Phase 4 tasks:
     - Execute weekly/monthly/quarterly cadence (Section 14)
     - Refresh cornerstone content quarterly
     - Continue Tier 4 ongoing content production
     - Quarterly entity alignment audit
     - Respond to competitive displacement alerts within 48 hours -->

---

*This document should be treated as a living artifact. Update Section 15 (Measurement) monthly and Section 16 (Roadmap) quarterly. The operating cadence in Section 14 keeps the program running between updates.*
