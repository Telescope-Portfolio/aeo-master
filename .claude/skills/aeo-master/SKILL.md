# AEO Master Document Generator

Produces a 16-section Answer Engine Optimization master document for the configured company. Pulls Scrunch AI visibility data, scrapes the company website via Firecrawl, cross-references content gaps with zero-visibility prompts, and outputs a prioritized action plan sized to the team.

## Trigger

Invoke with `/aeo-master`. No arguments needed. Reads company config from `config/company.md` in the project root.

---

## Phase 1: Load Config

Read `config/company.md` from the project root. Extract these variables:

| Variable | Source | Required |
|----------|--------|----------|
| `company_name` | Basics > Company name | Yes |
| `website_url` | Basics > Website URL | Yes |
| `industry` | Basics > Industry/vertical | Yes |
| `description` | Basics > One-line description | Yes |
| `competitors` | Competitors section (list of name + description pairs) | Yes (at least 1) |
| `team_size` | Team > Marketing team size | Yes |
| `team_roles` | Team > Key roles | Yes |
| `scrunch_workspace_id` | Scrunch Configuration > Workspace ID | Yes |
| `differentiators` | Key Differentiators (list) | Yes (at least 1) |
| `verticals` | Top Verticals / Segments (list with optional win rates) | No |
| `objections` | Known Buyer Objections (list) | No |

### Validation

Stop and tell the user if any required field is missing or still has placeholder text (contains `[` brackets). Specifically check:
- `scrunch_workspace_id` must be a number, not placeholder text
- `website_url` must start with `http://` or `https://`
- At least 1 competitor must be listed
- At least 1 differentiator must be listed

If validation fails, list exactly which fields need to be filled in and stop.

---

## Phase 2: Scrunch Data Pull

Use the Scrunch API to pull the company's AI visibility baseline. The API base URL is `https://api.scrunchai.com/v1`. Auth is via `Authorization: Bearer $SCRUNCH_API_TOKEN` header.

Run these queries using `curl` via Bash. Use the `scrunch_workspace_id` from config as the `{brand_id}` path parameter.

### Query 1: Overall Scorecard
```
GET /v1/{brand_id}/query?fields=brand_presence_percentage,brand_position_score,brand_sentiment_score,responses
```

Captures: overall brand presence %, position score, sentiment score, total responses tracked.

### Query 2: By Platform
```
GET /v1/{brand_id}/query?fields=ai_platform,brand_presence_percentage,brand_position_score,brand_sentiment_score,responses
```

Captures: presence, position, and sentiment broken out by each AI platform (ChatGPT, Perplexity, Google AI, Claude, Gemini, Meta AI, etc.).

### Query 3: By Buyer Journey Stage
```
GET /v1/{brand_id}/query?fields=stage,brand_presence_percentage,brand_position_score,responses
```

Captures: presence by journey stage (Advice, Awareness, Evaluation, Comparison, Other). **The Advice stage number is the primary KPI** for the entire report.

### Query 4: Competitor Presence
```
GET /v1/{brand_id}/query?fields=competitor_name,competitor_presence_percentage,competitor_position_score,competitor_sentiment_score,responses
```

Captures: each competitor's presence, position, and sentiment.

### Query 5: Per-Prompt Visibility
```
GET /v1/{brand_id}/query?fields=prompt,brand_presence_percentage,responses
```

Captures: visibility for every tracked prompt. **Sort by brand_presence_percentage ascending to find zero-visibility prompts.** These are the content gaps.

### Query 6: Source URLs (What AI Is Citing)
```
GET /v1/{brand_id}/query?fields=source_url,source_type,responses
```

Captures: which URLs AI platforms are citing when they mention the brand. Helps identify which pages are actually driving AI visibility.

### Query 7: Branded vs Non-Branded
```
GET /v1/{brand_id}/query?fields=branded,brand_presence_percentage,responses
```

Captures: visibility on branded queries (someone asks about you by name) vs. unbranded (category queries where you should appear but might not).

### Error Handling

- If `SCRUNCH_API_TOKEN` is not set, tell the user to export it and stop.
- If the API returns 401/403, tell the user their token may be invalid or doesn't have access to this workspace.
- If the API returns empty data (total: 0 rows), warn that the Scrunch workspace may not have prompts configured yet. Continue with the site audit but note in the report that Scrunch data was unavailable.

### Data Assembly

Store all Scrunch data in memory for use in later phases. Key derived metrics to calculate:

- **Zero-visibility prompts**: all prompts where `brand_presence_percentage` = 0 or rounds to 0
- **Strongest prompts**: top 5 prompts by `brand_presence_percentage`
- **Advice stage gap**: the `brand_presence_percentage` for stage = "Advice"
- **Competitor advantage**: any competitor whose `competitor_presence_percentage` exceeds the brand's `brand_presence_percentage`

---

## Phase 3: Site Audit

Use Firecrawl to scrape and audit the company's website. Use the `firecrawl_scrape` MCP tool if available, or fall back to `firecrawl_search` / `firecrawl_map` tools, or use the Firecrawl CLI (`npx firecrawl`), or use `WebFetch` as a last resort.

### Step 3a: Discover Pages

First, scrape the sitemap or use `firecrawl_map` to discover all pages on the site:

```
firecrawl_map(url: "{website_url}")
```

From the sitemap/map results, identify key pages to audit. Prioritize these page types (in order):

1. **Homepage** (always)
2. **Pricing page** (look for /pricing, /plans)
3. **Comparison/vs pages** (look for /vs-, /compare, competitor names in URL)
4. **Product/features pages** (look for /product, /features, /platform)
5. **About page** (look for /about, /company, /team)
6. **Vertical/industry pages** (look for industry names, /industries, /solutions)
7. **Blog index** (look for /blog, /resources)
8. **2-3 top blog posts** (most recent or most relevant to the company's category)

Target: audit 8-15 pages total. Don't scrape the entire site.

### Step 3b: Scrape Each Page

For each page, scrape the full content:

```
firecrawl_scrape(url: "{page_url}")
```

### Step 3c: Check robots.txt

Fetch `{website_url}/robots.txt` to check for AI crawler blocks:

Look for blocks on: `GPTBot`, `ClaudeBot`, `PerplexityBot`, `Applebot-Extended`, `Google-Extended`, `CCBot`, `anthropic-ai`. If any are blocked, flag as a critical finding.

### Step 3d: Score Each Page

For each scraped page, evaluate against this rubric:

#### Question Coverage (per page)

For each page, identify what buyer questions it answers. Rate each answer:

| Rating | Criteria |
|--------|----------|
| **Strong** | Direct answer in first 1-2 sentences. Specific numbers or evidence. Could be extracted by an LLM and make sense standalone. 40-60 words of quotable, factual content. |
| **Adequate** | Answer exists but buried in marketing copy. Numbers present but without context. Requires reading surrounding paragraphs to understand. |
| **Weak** | Topic mentioned but not directly answered. One-sided (company wins every comparison). Marketing language instead of factual statements. |
| **Missing** | Question not addressed anywhere on the page. |

#### Technical Scorecard (per page)

| Check | Pass | Fail |
|-------|------|------|
| AI crawler access | robots.txt allows GPTBot, ClaudeBot, PerplexityBot | Any major AI crawler blocked |
| Schema markup | Any structured data present (JSON-LD, microdata) | No structured data |
| Heading hierarchy | Clean H1 > H2 > H3 nesting | Broken hierarchy or missing H1 |
| Answer extractability | At least one 40-60 word factual paragraph under an H2 that an LLM could quote verbatim | All content is marketing copy, design-element stats, or requires context |
| Entity signals | Company name, founding info, category, team size in plain text | Thin or missing entity information |
| Freshness signals | "Last updated" or publication date visible | No dates on cornerstone pages |
| Citation triggers | Specific metrics with sample sizes, named frameworks, external citations | Vague claims without evidence or methodology |
| Content modularity | Sections stand alone (each can be extracted independently) | Sections require surrounding context to make sense |

#### Intent Coverage Map

After auditing all pages, assess coverage across these four buyer intent types:

| Intent | Description | Assessment Criteria |
|--------|-------------|-------------------|
| **Evaluation** | "Is this right for me?" Comparison, feature, fit questions | Look for: honest comparisons with tradeoffs, vertical-specific positioning, "who this is for / not for" content |
| **Fear** | "What could go wrong?" Migration risk, reliability, downsides | Look for: honest tradeoff content, migration details, "what to watch out for," self-aware limitations |
| **Outcome** | "What results will I get?" ROI, metrics, case studies | Look for: specific numbers with sample sizes, named customer results, methodology |
| **Process** | "How does this work?" Implementation, setup, day-to-day | Look for: step-by-step guides, timelines, "what your first 30 days look like" |

Rate each intent as: **Strong**, **Partial**, or **Gap**.

---

## Phase 4: Gap Analysis

Cross-reference Scrunch data with the site audit to produce three gap types.

### Gap 1: Educational Content Gaps

These are topics where:
- Scrunch shows zero or near-zero visibility on related prompts
- The company has clear expertise (based on differentiators, industry, product)
- No educational/how-to content exists on the site

For each gap, note:
- The specific queries at 0% visibility
- Why the company is credible to answer (which differentiator or product feature supports it)
- Whether any page partially addresses it (and why it's not working)

### Gap 2: Existing Pages with Zero AI Visibility

These are pages that exist on the site but have 0% presence in AI responses. Common causes:
- Content is marketing copy, not answer-formatted
- Comparison pages are one-sided (company wins every row)
- Vertical/industry pages are generic and lack specificity
- Pages lack freshness signals or extractable answer paragraphs

For each, note the page URL, the related zero-visibility prompts, and what specifically needs to change.

### Gap 3: Missing Competitive Content

These are competitor-related queries where the brand is absent from the AI conversation. Look for:
- "[Competitor A] vs [Competitor B]" queries where the brand isn't part of the answer
- "Alternative to [Competitor]" queries
- "Best [category] for [use case]" queries where only competitors are mentioned

### Full Zero-Visibility Prompt List

Compile all prompts from Query 5 where `brand_presence_percentage` = 0 or effectively 0 (< 0.1%). This becomes the targeting list for the content production plan.

---

## Phase 5: Strategic Layer

Without CRM or Gong data, use these generic buyer question frameworks to build the priority questions list. Map company differentiators and objections from the config to specific question types.

### Buyer Question Templates

Generate priority questions by applying the company's context to these templates:

**Evaluation questions:**
- "Best [category] for [vertical/use case]" (one per vertical from config)
- "[Company] vs [Competitor]" (one per competitor from config)
- "What should I look for in a [category]?"
- "Is [Company] worth it?"
- "[Company] alternatives"

**Fear questions:**
- "What happens when I switch from [Competitor] to [Company]?"
- "What are the downsides of [Company]?"
- "Is [Company] reliable?"
- "[Company] reviews"
- "How long does [Company] implementation take?"

**Outcome questions:**
- "How to [achieve outcome the product delivers]"
- "[Industry] [key metric] best practices"
- "How does [differentiator feature] work?"
- "What [metrics] should [target buyer] track?"

**Process questions:**
- "How to migrate from [Competitor]"
- "How to set up [category] for [use case]"
- "What does onboarding look like with [Company]?"

### Ranking Criteria

Rank questions by:
1. **Purchase intent** (Fear and Process questions close deals; Evaluation questions build pipeline; Outcome questions build authority)
2. **AI encounter likelihood** (Is this a query someone would type into ChatGPT/Perplexity? Not all questions are AI queries.)
3. **Current visibility gap** (Cross-reference with Scrunch zero-visibility prompts)
4. **Competitive urgency** (Is a competitor already winning this query?)
5. **Brand credibility** (Does the company have evidence, data, or product features that make them the right answer?)

Produce:
- **Top 8 high-priority questions** with intent type and rationale
- **Top 6 medium-priority questions**
- **Top 10 revenue-driving prompts** ranked by purchase intent

---

## Phase 6: Document Assembly

Assemble the full 16-section document. Use the template structure below. Fill every section with real data from Phases 2-5. Do not use placeholder text or generic advice. Every recommendation must reference specific Scrunch data, specific pages from the site audit, or specific differentiators from the config.

### Section Template

---

#### FRONTMATTER

```
---
company: {company_name}
type: master
domain: AEO/GEO
date: {YYYY-MM-DD}
status: final
---
```

#### Section 1: Executive Summary

- Open with the single most important finding (usually: brand presence is X%, meaning the company appears in Y out of Z AI responses)
- State position and sentiment scores (when the brand IS mentioned, how does it look?)
- Name the core problem in one sentence
- State the fix in one sentence (X content pieces targeting Y zero-visibility prompts)
- Name the primary KPI and target (usually Advice stage brand presence)

Keep to 4-5 paragraphs. No fluff.

#### Section 2: Situation and Background

- What the company does (from config description + industry)
- Why AEO matters specifically for this company right now (timing, competitive threats, team context)
- Key competitive context (who is the incumbent, who is rising, who has better AI sentiment)

#### Section 3: Scrunch Visibility Data

- **Overall Scorecard** table: brand_presence_percentage, position_score, sentiment_score, total responses
- **By Platform** table: each AI platform with presence, position, sentiment, response count
- **By Buyer Journey Stage** table: each stage with presence, position, response count. Call out the Advice stage gap.
- **Competitor Presence** table: each competitor with presence, position, sentiment
- **Strongest Prompts** table: top 5 prompts by visibility (these are usually branded queries)
- Key insight paragraph: Is the problem uniform across platforms (content problem) or platform-specific (indexing problem)?

#### Section 4: Full Site Audit

- **Pages Audited** list with URLs
- **Questions Answered (Quality Ratings)** table: question, priority, intent, rating, notes
- **Critical Missing Questions** table: question, priority, intent, why it matters
- **Intent Coverage Map** table: intent type, coverage rating, assessment
- **Technical Scorecard** table: each check with pass/fail and notes
- **What's Working** bullet list (acknowledge strengths)

#### Section 5: Content Gaps by Type

- **Gap 1: Educational Content** with specific queries and rationale
- **Gap 2: Existing Pages with Zero Visibility** with URLs and what's wrong
- **Gap 3: Missing Competitive Content** with specific queries
- **Full Zero-Visibility Prompt List** (every prompt at 0%)

#### Section 6: Priority Questions to Own

- **High Priority (Top 8)** table: question, intent, why it's critical
- **Medium Priority (Top 6)** table: question, intent, notes
- **Top 10 Revenue-Driving Prompts** table: prompt, why it drives revenue
- **Full Tracking List** table: query, priority, current visibility, gap size

#### Section 7: Content Production Plan

Organize into tiers:

- **Tier 1: This Week** (2-3 pieces, highest impact)
- **Tier 2: Next Two Weeks** (2-3 pieces)
- **Tier 3: Next 30 Days** (2-3 pieces)
- **Tier 4: Ongoing** (review velocity, entity signals, quick wins)

For each content piece, provide:
- **Why now** (one paragraph linking to Scrunch data and competitive context)
- **Target prompts and current baseline** (specific prompts with current visibility %)
- **Target** (what visibility % to aim for within 60-90 days)
- **Content spec:**
  - Word count target (1,500-3,000 depending on type)
  - Opening paragraph template (50 words, direct answer, the exact text to start with)
  - Section structure (what H2s should this page have)
  - Required honest concessions (for comparison content: minimum 2-3 things the competitor does better)
  - Where the company wins (specific differentiators with evidence)
  - Named customer examples if applicable

#### Section 8: Answer Ownership Map

Table mapping each priority question to the right content surface:
- Dedicated page (for high-intent, high-volume questions)
- Homepage section (for brand-defining positioning)
- Blog post (for educational content and original research)
- FAQ section (for quick-answer questions)

Include a "Notes" column with word count targets and any special requirements.

#### Section 9: Writing Rules for Canonical Answers

Five rules that apply to every piece of content the team publishes:

1. **State the answer in the first sentence.** Not setup, not context. The answer.
2. **Name the tradeoffs honestly.** One-sided content doesn't get cited.
3. **Make it extractable.** 40-60 words that an AI could pull verbatim and it would make sense standalone.
4. **Use specific numbers.** Metric + sample size + context. No "significant improvement."
5. **Use the buyer's language.** How prospects actually phrase the question, not marketing copy.

Include 1-2 examples of a well-written extractable answer paragraph using the company's actual data.

#### Section 10: Question Mining Playbook

Sized to the team. Three tiers of question sources:

**Tier 1: Highest Signal (company already has this data)**
- Sales call recordings (if available): pre-price questions, post-price objections, objections disguised as curiosity
- Customer support tickets: first-30-day questions, expectation gaps, confidence questions

**Tier 2: High Signal (public, accessible)**
- G2 and review platform reviews: mine 3-4 star reviews for "great product, but..." language
- Third-party roundup articles: how is the company described vs. competitors?

**Tier 3: Medium Signal (monitor, don't chase)**
- Reddit (relevant subreddits for the industry)
- LinkedIn (industry operators and influencers)

Include specific subreddit and search term suggestions based on the company's industry.

#### Section 11: Third-Party Presence Strategy

- Current state assessment for: G2, Capterra, industry-specific review sites, roundup articles, LinkedIn, Crunchbase
- G2 review velocity plan with targets (25+ reviews in 90 days as a starting goal)
- Strategy for influencing roundup article descriptions
- Original research opportunity (what proprietary data does the company have that could be published?)
- Entity consistency: provide a canonical company description (50-60 words) to use across all platforms

#### Section 12: Technical Foundations

Two categories:

**Must-Do (implement with next site update):**
- Answer extractability fixes
- Freshness signals (Last updated timestamps on cornerstone pages)
- Organization schema with `sameAs` links to review profiles
- Content modularity (sections that stand alone)

**Nice-to-Have (post-launch):**
- Article schema on blog posts
- BreadcrumbList schema across all pages
- FAQ schema on pages with Q&A content
- `llms.txt` file
- Citation-friendly formatting (key stats in blockquote or structured paragraph format)

#### Section 13: Platform Awareness

Table with each AI platform, priority level for this company, and rationale:

| Platform | Priority | Rationale |
|----------|----------|-----------|
| Perplexity | Usually High | Easiest to influence with fresh content. Increasingly used for vendor research. |
| ChatGPT | Usually High | Largest audience. Requires strong review and citation presence. |
| Google AI Overviews | Medium | Traditional SEO signals still matter. Schema and structured content help here. |
| Claude | Low-Medium | Highest authority bar. Will improve as earned media grows. |
| Gemini | Medium | Google ecosystem. Responds to content + schema improvements. |
| Meta AI | Low | Smaller audience for B2B queries. |

Adjust priorities based on where the company's Scrunch data shows the biggest gaps or opportunities.

#### Section 14: Operating Cadence and Ownership

Scale the cadence to the team size from config:

**Weekly (15 minutes):**
- Scan sales calls or support tickets for recurring questions
- Add new questions to shared backlog

**Monthly (2 hours):**
- Prioritize top 3-5 questions from backlog
- Assign one canonical answer to create or update
- Check one roundup article for accuracy
- Review G2 review count
- Pull Scrunch scorecard

**Quarterly (half day):**
- Audit top 10 pages for freshness
- Rewrite or retire comparison content
- Run full query baseline check
- Review which content is getting cited (Scrunch per-prompt data)

**Ownership table:** Map each cadence task to a role from the team config. If the team is smaller than 3 people, consolidate ownership accordingly.

#### Section 15: Measurement Framework

- **Metrics to Track** table: metric name, how to measure, frequency
- **Content-to-Prompt Map** table: each planned content piece mapped to specific Scrunch prompts with current baseline and 60-day target
- **Measurement Cadence:** after each publish (wait 2-3 weeks, then pull Scrunch), weekly dashboard check, monthly full scorecard, alert threshold for drops
- **Scrunch API Queries:** provide the exact curl commands using the company's workspace ID:

```
# Overall scorecard
GET /v1/{scrunch_workspace_id}/query?fields=brand_presence_percentage,brand_position_score,brand_sentiment_score

# By stage (watch the Advice number)
GET /v1/{scrunch_workspace_id}/query?fields=stage,brand_presence_percentage

# Per-prompt (find zero-visibility prompts)
GET /v1/{scrunch_workspace_id}/query?fields=prompt,brand_presence_percentage

# Citation sources
GET /v1/{scrunch_workspace_id}/query?fields=source_url,source_type,responses
```

#### Section 16: Phase Roadmap

4-phase timeline:

| Phase | Timeline | Focus |
|-------|----------|-------|
| Phase 1: Site Refresh | Now to 8 weeks | Fix Fear + Process gaps, technical foundations, comparison rewrites |
| Phase 2: Answer Engine | Weeks 8-16 | Publish canonical answers to top 8 questions |
| Phase 3: Earned Media | Ongoing from week 4 | G2 reviews, original research, roundup outreach |
| Phase 4: Sustain | Ongoing | Operating cadence, monthly Scrunch reviews, quarterly audits |

Include 3-5 key deliverables per phase, grounded in the content production plan from Section 7.

---

## Phase 7: Output

### Save the Document

Save the assembled document to:

```
outputs/aeo-master-{YYYY-MM}.md
```

Where `{YYYY-MM}` is the current year and month.

### Report to User

After saving, tell the user:
1. Where the file was saved
2. The three most important findings (usually: overall presence %, Advice stage gap, number of zero-visibility prompts)
3. The single highest-priority content piece to start with
4. How to track progress (the Scrunch API queries from Section 15)

---

## Quality Checks

Before finalizing the document, verify:

- [ ] Every table has real data from Scrunch or the site audit, not placeholder values
- [ ] Every content spec in Section 7 has a specific word count, opening paragraph, and target prompts with baseline %
- [ ] Competitor names match what's in the config (not generic "Competitor A")
- [ ] The Scrunch workspace ID in the API query examples matches the config
- [ ] The operating cadence in Section 14 references roles from the team config
- [ ] The phase roadmap in Section 16 references content pieces from Section 7
- [ ] Comparison content specs include at least 2-3 honest concessions (things the competitor does better)
- [ ] The executive summary can be read in 60 seconds and conveys the core finding, core problem, and primary KPI
- [ ] No section contains generic advice that could apply to any company. Everything should reference this company's specific data, competitors, differentiators, or industry.
