---
name: AEO Scoring Rubric
description: Question quality ratings, intent coverage criteria, technical scorecard, and gap categorization for AEO master document production
---

# AEO Scoring Rubric

## 1. Question Quality Ratings

How to rate whether a page answers a question well:

- **Strong** -- States the answer in the first sentence, includes specific numbers, could be extracted by AI verbatim, names tradeoffs honestly
- **Adequate** -- Answer exists but is buried in marketing language, requires inference, or lacks specificity
- **Weak** -- Claims to answer but uses vague language, is one-sided, or requires full page context to understand

### Quality Dimensions

| Dimension | Strong | Adequate | Weak |
|-----------|--------|----------|------|
| Directness | Answer in first sentence | Answer present but buried | No clear answer |
| Tradeoff honesty | Names downsides | Acknowledges complexity | Only lists benefits |
| AI reusability | 40-60 word extractable block | Partially extractable | Requires full page context |
| Specificity | Numbers, timelines, examples | Some specifics | Vague claims ("fast", "easy") |

## 2. AEO Question Qualification (4 Criteria)

For each question, score against:

| Criterion | Test |
|-----------|------|
| Decision uncertainty | Does this make someone pause before buying? |
| AI encounter likelihood | Would this appear in AI prompts about this category? |
| Category-shaping | Does answering this change how the category is understood? |
| Brand credibility | Can this brand credibly own the answer? |

**Priority assignment:** High (meets all 4), Medium (meets 2-3), Low (meets 1 or fewer)

## 3. Intent Coverage Map (4 Intents)

| Intent | Core question | What to look for | Coverage levels |
|--------|--------------|-------------------|-----------------|
| Evaluation | Is this right for me? | Fit language, use cases, "who this is for" | Strong/Partial/Gap |
| Fear | What could go wrong? | Risk acknowledgment, honest tradeoffs | Strong/Partial/Gap |
| Outcome | What actually changes? | Results, ROI, before/after | Strong/Partial/Gap |
| Process | What does this look like? | Implementation, timeline, setup | Strong/Partial/Gap |

## 4. Technical Scorecard (8 Checks)

For each check, define Pass/Fail/Weak criteria:

### 1. AI Crawler Access

- **Pass:** robots.txt allows GPTBot, ClaudeBot, PerplexityBot.
- **Fail:** Any blocked.

### 2. Schema Markup

- **Pass:** Organization + at least 2 of (FAQPage, Product, Article, BreadcrumbList).
- **Weak:** Only basic WebPage.
- **Fail:** No schema markup present.

### 3. Heading Hierarchy

- **Pass:** Clean H1 > H2 > H3, no skipped levels.
- **Fail:** Inconsistent or missing.

### 4. Answer Extractability

- **Pass:** 40-60 word direct answers under H2s.
- **Fail:** Marketing copy only, no extractable blocks.

### 5. Entity Signals

- **Pass:** sameAs links, founding date, HQ, team size on About page.
- **Weak:** Some present.
- **Fail:** Missing.

### 6. Freshness Signals

- **Pass:** "Last updated" timestamps on cornerstone pages.
- **Fail:** No dates anywhere.

### 7. Citation Triggers

- **Pass:** Statistics with sources, expert quotes, data citations.
- **Weak:** Stats without sources.
- **Fail:** No data.

### 8. Content Modularity

- **Pass:** Each section stands alone as extractable passage.
- **Fail:** Sections require surrounding context.

## 5. Content Gap Categories

- **Gap 1: No educational content** -- Company has expertise but no published content that teaches buyers. The site describes features rather than answering questions.
- **Gap 2: Pages exist but AEO-invisible** -- Pages load and have content, but 0% AI visibility. Content isn't structured for citation (no extractable answers, no freshness, generic rather than specific).
- **Gap 3: Missing competitor content** -- Company is absent from conversations where buyers compare its competitors to each other.

## 6. Citation Efficiency by Page Type (Quick Reference)

| Page Type | Efficiency | Implication |
|-----------|------------|-------------|
| Blog/Editorial | 1.4x | Best workhorse, 20.2% of all citations |
| Service/Use Case | 1.9x | Frame around what you solve, not what you sell |
| Product Pages | 0.5x | Read 2x more than recommended |
| Help/FAQ | 0.2x | Lowest, AI reads but never cites |
| Review/Directory | 4.8x | Third-party authority dominates |
| About/Contact | 4.6x | Key AI trust signal |
| Comparison | 0.6x | AI may prefer third-party comparisons |

Source: Trakkr Study 006, 337K citations, 882 brands, March 2026.
