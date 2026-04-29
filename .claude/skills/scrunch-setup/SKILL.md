---
name: scrunch-setup
description: Set up Scrunch AI visibility monitoring for a company. Configures brand, competitors, personas, and 40-70 strategic prompts across 5 buyer journey stages. Triggers on "scrunch setup", "set up scrunch", "AI visibility monitoring", or "configure scrunch".
---

# Scrunch AI Visibility Setup

Automate the full Scrunch configuration: brand creation, competitor tracking, persona configuration, strategic prompt loading, and baseline measurement.

## Prerequisites

- `SCRUNCH_API_TOKEN` must be set as an environment variable
- Company config must exist at `config/company.md` (standalone repo mode) or `<Company>/CONTEXT.md` (portfolio mode)

## Workflow

### Step 1: Load Company Context

Read `config/company.md` (if it exists in the project root) or `<Company>/CONTEXT.md` to extract:
- **Company name** and website
- **What they do** (one-paragraph description)
- **ICP** (who buys, what verticals, company size)
- **Competitors** (names and descriptions)
- **Key differentiators** (used to generate targeted prompts)
- **Top verticals** (used to generate vertical-specific prompts)

### Step 2: Check Existing Scrunch Config

```bash
source ~/.zshrc 2>/dev/null; source ~/.env 2>/dev/null

# List all brands
curl -s "https://api.scrunchai.com/v1/brands?limit=100" \
  -H "Authorization: Bearer $SCRUNCH_API_TOKEN" | python3 -m json.tool

# If brand exists, get its ID and pull current config:
# curl -s "https://api.scrunchai.com/v1/brands/{brand_id}/competitors?limit=100" ...
# curl -s "https://api.scrunchai.com/v1/brands/{brand_id}/personas?limit=100" ...
# curl -s "https://api.scrunchai.com/v1/{brand_id}/prompts?limit=200" ...
```

If the brand already exists, audit the existing config against what the CONTEXT.md says and add/update what's missing. Do NOT create duplicates.

### Step 3: Create or Update Brand

**New brand:**
```bash
curl -s -X POST "https://api.scrunchai.com/v1/brands" \
  -H "Authorization: Bearer $SCRUNCH_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "<Company Name>",
    "website": "<primary website>",
    "description": "<1-2 sentence description focusing on what they do, who they serve, and key differentiator>",
    "alternative_names": ["<short name>", "<domain-style name>"],
    "alternative_websites": ["<app store URL if applicable>"],
    "status": "customer"
  }'
```

**Existing brand update:**
```bash
curl -s -X PATCH "https://api.scrunchai.com/v1/brands/{brand_id}" \
  -H "Authorization: Bearer $SCRUNCH_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"description": "...", "alternative_names": [...]}'
```

### Step 4: Configure Competitors

Only track live, active competitors. Archive dead ones. For each competitor:

```bash
curl -s -X POST "https://api.scrunchai.com/v1/brands/{brand_id}/competitors" \
  -H "Authorization: Bearer $SCRUNCH_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "<Competitor>",
    "alternative_names": ["<alt1>", "<alt2>"],
    "websites": ["<primary URL>", "<app store URL if applicable>"]
  }'
```

**Guidelines:**
- Track 3-7 competitors (enough for coverage, not so many it dilutes data)
- Include the company's app store listing URL where applicable (Shopify, G2, etc.)
- Archive irrelevant auto-generated competitors Scrunch may have added
- Record each competitor_id for reference

### Step 5: Create Personas

Create 4-6 personas based on the company's ICP. Standard persona categories:

| Persona Type | When to Create | Description Pattern |
|---|---|---|
| **Primary buyer** | Always | The person who makes or influences the purchase decision |
| **Hands-on evaluator** | Always | The person who evaluates features day-to-day |
| **Channel partner** | If relevant | Agency, consultant, reseller who recommends the product |
| **Vertical-specific** | If company has strong verticals | Buyer in the company's top 2-3 verticals |

```bash
curl -s -X POST "https://api.scrunchai.com/v1/brands/{brand_id}/personas" \
  -H "Authorization: Bearer $SCRUNCH_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "<Persona Name>", "description": "<2-3 sentence description of role, context, and what they care about>"}'
```

### Step 6: Generate and Load Prompts

This is the core of the setup. Generate 40-70 prompts across 5 journey stages.

#### Start With Buyer Data (Preferred)

If the company has call recording data (Gong, Fathom, Chorus) or a conversational AI agent on sales data (OpenClaw, custom RAG), mine it FIRST. The best Scrunch prompts mirror actual buyer queries, not hypothetical ones. Ask: "What questions do prospects ask before signing?" "What objections come up most?" "What comparisons do buyers make?" Use the exact phrasing from calls as prompt text. This produces dramatically more useful visibility data because you're tracking the queries buyers actually use, not category-generic templates.

**Fallback:** When buyer data isn't available, use the stage templates below to generate prompts. These are good starting points but should be refined once real buyer questions surface.

#### Platform Tiers

- **All 9 platforms** (for 5-7 highest-stakes prompts): `["chatgpt", "perplexity", "google_ai_overviews", "google_ai_mode", "google_gemini", "claude", "copilot", "meta", "grok"]`
- **Tier 1** (for remaining prompts): `["chatgpt", "perplexity", "google_ai_overviews", "google_ai_mode", "google_gemini"]`

#### Standard Tag Taxonomy

Apply consistently across all portfolio companies:

| Tag | When to Use |
|---|---|
| `category` | Category-level queries ("best X for Y") |
| `educational` | How-to, strategy, best practices |
| `branded` | Query mentions the brand name |
| `competitor:<name>` | Query mentions a specific competitor |
| `competitor:multi` | Query compares multiple competitors |
| `vertical:<name>` | Query targets a specific vertical |
| `retention` | Churn, retention, loyalty queries |
| `migration` | Platform switching queries |
| `pricing` | Cost, pricing comparison queries |
| `reputation` | Reviews, reliability, trust queries |
| `fear` | Downsides, risks, concerns queries |

#### Prompt Templates by Stage

**AWARENESS (10-15 prompts)** - Educational queries before a buyer has a specific solution in mind:
- "how to [solve core problem] for [ICP context]"
- "how to [achieve key outcome] in [industry/platform]"
- "[core problem] best practices for [vertical]"
- "what [metrics] should [ICP] track"
- "[differentiator topic] strategies for [industry]"

**ADVICE (8-12 prompts)** - Recommendation queries:
- "best [product category] for [platform/industry]"
- "best [product category] for [vertical] brands"
- "which [product category] has the best [differentiator]"
- "what [product category] do [channel partners] recommend"
- "[product category] for [company size] brands"

**EVALUATION (10-15 prompts)** - Active assessment of the brand or alternatives:
- "[Brand] reviews"
- "[Brand] pricing"
- "is [Brand] good for [use case]"
- "what are the downsides of [Brand]"
- "[Competitor] alternatives for [platform]"
- "how to migrate from [Competitor]"

**COMPARISON (10-15 prompts)** - Head-to-head comparisons:
- "[Brand] vs [Competitor]" (one per major competitor)
- "[Brand] vs [Comp1] vs [Comp2]" (three-way)
- "which is better [Brand] or [Competitor] for [use case]"
- "[Brand] vs [Competitor] for [vertical]"
- "[Brand] vs [Competitor] pricing comparison"
- "compare [product category] [year]"

**OTHER (3-5 prompts)** - Miscellaneous:
- "[Brand] [platform] app"
- "who founded [Brand]"
- "[Brand] customer case studies"
- "[product category] market landscape [year]"

#### Creating Prompts

```bash
curl -s -X POST "https://api.scrunchai.com/v1/{brand_id}/prompts" \
  -H "Authorization: Bearer $SCRUNCH_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "<prompt text>",
    "stage": "<Awareness|Advice|Evaluation|Comparison|Other>",
    "persona_id": <persona_id>,
    "tags": ["<tag1>", "<tag2>"],
    "platforms": ["chatgpt", "perplexity", ...]
  }'
```

**Dedup:** Before creating, pull existing prompts and skip any that already exist (case-insensitive match).

**Batch creation:** Write all prompts to a JSON array, then loop through with curl. Add 0.3s delay between calls to avoid rate limiting.

### Step 7: Pull Baseline (if data exists)

Wait 3-5 days after prompt creation for Scrunch to collect responses. Then:

```bash
# Overall scorecard
curl -s "https://api.scrunchai.com/v1/{brand_id}/query?fields=brand_presence_percentage,brand_position_score,brand_sentiment_score,responses" \
  -H "Authorization: Bearer $SCRUNCH_API_TOKEN"

# By platform
curl -s "https://api.scrunchai.com/v1/{brand_id}/query?fields=ai_platform,brand_presence_percentage,brand_position_score" ...

# By stage
curl -s "https://api.scrunchai.com/v1/{brand_id}/query?fields=stage,brand_presence_percentage,brand_position_score" ...

# Competitor comparison
curl -s "https://api.scrunchai.com/v1/{brand_id}/query?fields=competitor_name,competitor_presence_percentage,competitor_position_score" ...

# Per-prompt (find zero-visibility queries)
curl -s "https://api.scrunchai.com/v1/{brand_id}/query?fields=prompt,brand_presence_percentage,brand_position_score" ...
```

### Step 8: Save Configuration

Save two files:

**`outputs/scrunch-config.md`** (standalone) or **`<Company>/knowledge/scrunch-config.md`** (portfolio mode) - Full config with:
- Brand ID (this is the workspace_id the aeo-master skill needs)
- Competitor IDs and names
- Persona IDs and names
- Tag taxonomy
- Prompt IDs organized by stage
- Content-to-prompt impact mapping
- Monitoring cadence

**`outputs/scrunch-baseline-<date>.md`** (standalone) or **`<Company>/deliverables/scrunch-baseline-<date>.md`** (portfolio mode) - Baseline with:
- Overall scorecard (presence, position, sentiment)
- By platform breakdown
- By stage breakdown
- Competitor comparison
- Zero-visibility prompts grouped by content gap
- Key insights and recommended actions

### Step 9: Update Config

In standalone repo mode: update `config/company.md` with the Scrunch Workspace ID (brand_id) so the aeo-master skill can find it.

In portfolio mode: add a line to `<Company>/CONTEXT.md` under Active Projects noting that Scrunch monitoring is configured.

## Monitoring Cadence (for ongoing use)

| Cadence | What | API Call |
|---|---|---|
| Weekly | Overall presence trend | `fields=date_week,brand_presence_percentage` |
| Monthly | Full scorecard | Multiple calls: by platform, stage, competitor, tag |
| Quarterly | Deep dive + prompt refresh | Full baseline re-run + Responses API for top 10 prompts |

## Alert Thresholds

- Brand presence drops >10% week-over-week
- Any competitor gains >15% presence in a month
- Sentiment below 40 on any prompt cluster
- Key competitor exceeds brand on comparison prompts

## Notes

- Scrunch API base URL: `https://api.scrunchai.com/v1`
- Auth: Bearer token in Authorization header
- Full API docs: `_system/scrunch-api-docs.md`
- Prompts cannot be edited (text). To change prompt text, archive and recreate. Tags and platforms can be updated in place.
- Query API data limited to last 90 days
- Response data is immutable (snapshot of what AI said)
