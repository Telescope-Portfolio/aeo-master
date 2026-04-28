---
name: strategic-analysis
description: >
  Deep strategic analysis for B2B SaaS portfolio companies. Pulls CRM data (Salesforce or HubSpot),
  call transcripts (Gong, Fathom, or manual), interview evidence, and optional Revelio Labs enrichment.
  Answers: why we win/lose, ICP (descriptive and prescriptive), competitive positioning, feature gaps
  with ARR impact, pricing sensitivity, and sales cycle dynamics. Layered output (exec summary / medium /
  deep dive) plus Google Doc export. Use when the user says "strategic analysis", "win/loss", "why are we
  losing", "deal analysis", "ICP analysis", "competitive analysis", or wants a comprehensive view of a
  portfolio company's go-to-market performance.
user_invocable: true
---

# Strategic Analysis

Pull CRM deals, call transcripts, and interview evidence for a B2B SaaS company. Link them together, analyze by segment, and produce a layered report answering: why you win, why you lose, who your ICP is (and should be), what feature gaps cost you, how competitors beat you, and what data you're missing.

This skill is **CRM-agnostic** (Salesforce or HubSpot) and **call-recorder-agnostic** (Gong, Fathom, or manual paste).

**Output:** A layered .md report (executive summary, medium dive, deep dive) plus a standalone ICP file and a Google Doc.

## Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `--company Name` | *(required)* | Portfolio company name from companies.json |
| `--days N` | 180 | Lookback window in days |
| `--depth shallow\|medium\|deep\|all` | all | Which output layers to generate |
| `--segment acv\|vertical\|lifecycle\|source\|competitor\|owner\|all` | all | Segmentation dimensions to include |
| `--enrich` | false | Enable Revelio Labs enrichment via Snowflake MCP (adds headcount, growth, industry data) |
| `--skip-calls` | false | Skip live call recorder API calls; use only exported files |
| `--skip-ml` | false | Skip XGBoost predictive modeling |
| `--skip-gdoc` | true | Skip Google Doc creation (requires gws CLI, not included in this repo) |

**Announce:** "Running `/strategic-analysis` for {company} -- pulling CRM data, call transcripts, and interview evidence."

---

## Analytical Principles

These rules apply to EVERY analysis module and EVERY finding in the report. Violating these rules produces misleading analysis. Follow them strictly.

### Repeatability Over Outliers

- **Minimum sample size:** Do not draw conclusions from segments with fewer than 5 deals. Show the data, but label it "Insufficient sample (N={count}), not actionable."
- **Outlier flagging:** For any segment, identify deals that are >3x the median deal size. Report the metric WITH and WITHOUT the outlier. Example: "Partner channel: 55.6% win rate (20W/16L). Removing the $76K Smartrr migration outlier: 51.5% (17W/16L). The pattern holds."
- **Pattern threshold:** A finding is a "pattern" only if it appears in 5+ deals AND persists across at least 2 quarters. Otherwise label it "Emerging signal" not a conclusion.
- **Revenue concentration check:** For any revenue-based finding, report what % of the total comes from the top 1-2 deals. If >50% of a segment's revenue comes from 1 deal, lead with the deal count story, not the revenue story.
- **Median over mean:** Always use median for deal size and velocity. Report mean only as a secondary reference. Mean is easily distorted by a single large deal in a field of small ones.
- **Sub-segment before recommending:** Before recommending action on any segment (e.g., "invest in Partner channel"), always break it into sub-segments (e.g., agency vs. tech partners, by tier within the channel). The actionable insight is usually one level deeper than the top-line number.
- **One-time events:** Flag any pattern that could be driven by a non-repeatable event (e.g., a competitor shutting down, a platform migration wave, a one-time referral batch). Ask: "Would this pattern hold next quarter if that event didn't recur?"

### Segmented by Default

**Never present a company-wide metric without immediate segmentation.** Every core metric must be shown by at least 2 segment dimensions. Aggregates hide the real story.

- **Win rate:** Always show by tier AND source AND deal type (new vs. existing). A 57% overall win rate that is really 33% new business and 100% renewals is misleading.
- **Deal velocity:** Always show by tier AND outcome (won vs. lost). A 21-day median that is really 6 days for SMB and 31 days for enterprise tells a different story.
- **Pricing:** Always show win rate by price band AND by tier. A $10K ceiling might only apply to SMB; enterprise might have a different ceiling.
- **Loss reasons:** Always show by tier AND by competitor. "Bad Timing" might mean something different for a $5K SMB deal vs. a $75K enterprise deal.
- **Competitor analysis:** Always show by tier AND by source channel. Losing to Recharge at Tier 2 is a different problem than losing to Recharge at Tier 3.
- **Revenue metrics:** Always separate new business from existing business. Renewals inflate every aggregate.

---

## Step 0: Load Configuration and Check Prerequisites

### 0a: Read Company Config

Read `.claude/skills/strategic-analysis/companies.json` (relative to project root) and look up the `--company` value (case-insensitive match).

### 0b: Company Not Found

If the company is not in companies.json, ask the user:

> "{Company}" isn't configured yet. I need to set up your CRM connection.
>
> 1. Which CRM do you use? (Salesforce or HubSpot)
> 2. If Salesforce: have you already run `sf org login web`? (See docs/setup-guide.md Step 5)
> 3. If HubSpot: is the HubSpot MCP server configured? (See docs/setup-guide.md Step 5)

Once the user confirms their CRM is connected, create a new entry in `.claude/skills/strategic-analysis/companies.json` with:
- `crm.type` set to `"salesforce"` or `"hubspot"`
- `crm.sf_org` set to the Salesforce org alias (if Salesforce)
- `crm.field_mapping` set to `"auto"` (the skill will discover fields automatically)
- `call_recorder.type` set to `"gong"`, `"fathom"`, or `"none"`
- `portfolio_path` set to `"./"`

Then continue to Step 0c.

### 0c: Check CRM Connection

**Salesforce:** Run `sf org list --json` and verify the company's `crm.sf_org` value appears. If not, stop with auth instructions.

**HubSpot:** Verify HubSpot MCP tools are available by attempting a test query. If MCP is unavailable, stop with setup instructions.

### 0d: Check Call Recorder

**Gong:** Run `source ~/.zshrc 2>/dev/null; source ~/.env 2>/dev/null` and check `GONG_ACCESS_KEY` and `GONG_ACCESS_SECRET` exist. If missing and `--skip-calls` is not set, warn and auto-enable `--skip-calls`.

**Fathom:** Run `source ~/.zshrc 2>/dev/null; source ~/.env 2>/dev/null` and check `FATHOM_API_KEY` exists. If missing and `--skip-calls` is not set, warn and auto-enable `--skip-calls`.

**Manual:** No check needed. User will paste transcripts during Agent 2.

### 0e: Check Portfolio Path

Verify `portfolio_path` from config exists. If `portfolio_path` is `"./"` (standalone repo mode), use `outputs/` as the output directory. Otherwise check for `knowledge/` and `deliverables/` subdirectories and create `deliverables/` if missing.

### 0f: Check Python Dependencies (if --skip-ml not set)

```bash
python3 -c "import xgboost, sklearn, pandas, numpy; print('ML dependencies OK')" 2>/dev/null
```

If this fails, attempt install:
```bash
pip3 install --user xgboost scikit-learn pandas numpy 2>/dev/null || pip3 install --break-system-packages xgboost scikit-learn pandas numpy 2>/dev/null
```

If install fails, auto-set `--skip-ml` and note: "Skipping predictive modeling (XGBoost not available). The rest of the analysis proceeds without it."

### 0g: Check Snowflake (if --enrich set)

Verify Snowflake MCP tools are available by checking for `mcp__snowflake__run_snowflake_query`. If unavailable, warn: "Snowflake MCP not connected. Skipping Revelio Labs enrichment." and disable enrichment.

---

## Step 0.5: Feature Intel (Skipped in standalone mode)

The `/feature-intel` skill is not included in this repo. The analysis will rely on CRM fields, call transcripts, and interview evidence for feature gap and competitive claims. Proceed to Step 1.

---

## Step 1: Field Discovery

**Only runs if `crm.field_mapping` is `"auto"` or empty.** If a mapping already exists in companies.json, skip to Step 2.

### Salesforce Field Discovery

```bash
sf sobject describe Opportunity --target-org {config.crm.sf_org} --json
```

Parse the response `fields` array. For each analysis concept, look for matching field names using these heuristics:

| Concept | Search patterns |
|---------|----------------|
| amount | `Amount`, `*ARR*`, `*ACV*`, `*Revenue*` |
| loss_reason | `*Lost*Reason*`, `*Closed*Lost*`, `*Loss*Reason*` |
| competitor | `*Competitor*`, `*Lost*To*` |
| deal_source | `*Source*`, `*Lead*Source*`, `*Channel*` |
| vertical_or_industry | `*Industry*`, `*Vertical*`, `*Segment*` |
| deal_tier | `*Tier*`, `*Segment*` (if not matched to vertical) |
| employee_count | `*Employee*`, `*Company*Size*` |
| product_gap | `*Gap*`, `*Feature*`, `*Product*Reason*` |
| stage_date_fields | `Date_Entered_*` pattern |

Present the discovered mapping to the user:

> I found these field mappings for {company}'s Salesforce:
>
> | Concept | Field | Sample Values |
> |---------|-------|---------------|
> | Loss reason | Closed_Lost_Reason__c | "Competitor", "Price", "No Decision" |
> | ...
>
> Does this look right? Any corrections?

After confirmation, save the mapping to `companies.json`.

### HubSpot Field Discovery

Use HubSpot MCP to list all deal properties. Apply the same heuristic matching. Additionally:
- Query pipeline stage definitions to map stage IDs to human-readable names
- Note that HubSpot deal stages use internal IDs, not text values

### Post-Discovery Checks

1. **Stage date fields:** If no `Date_Entered_*` or equivalent fields found, note: "No stage date fields found. Deal velocity analysis will be skipped."
2. **Loss reason type:** Check if loss_reason values are from a picklist (structured) or free text. If >20 unique values and average length >20 characters, flag as free text: "Loss reasons appear to be free text. I'll cluster them into categories during analysis."
3. **Sparse fields:** For each mapped field, check population rate. Flag any field with <50% fill rate.

---

## Step 2: Parallel Data Collection

Launch up to 4 agents simultaneously. Each returns structured data.

### Agent 1: CRM Deals

Build a query dynamically from `crm.field_mapping`. Include only fields that have non-empty mappings.

**Salesforce:**

Construct the SELECT clause from all mapped field names. Build the WHERE clause from `won_values`, `lost_values`, `dq_values`, `nurture_values`:

```bash
sf data query --query "
SELECT {all_mapped_fields_comma_separated}
FROM Opportunity
WHERE StageName IN ({all_stage_values_quoted})
AND (CloseDate >= LAST_N_DAYS:{DAYS} OR CreatedDate >= LAST_N_DAYS:{DAYS})
ORDER BY CloseDate DESC NULLS LAST
" --target-org {config.crm.sf_org} --json
```

If result set >2000 records, add `--bulk` flag.

**HubSpot:**

Use HubSpot MCP to search deals with filters:
- `dealstage` IN (mapped stage values)
- `closedate` >= lookback date
- Request all mapped properties
- Paginate (max 100 per request)

**Important:** For HubSpot, first query pipeline stage definitions to build a stage ID-to-name map. Use this map to translate stage IDs throughout the analysis.

**Data Validation (both paths):**

After pulling deals, report:

> Found **{N}** closed deals ({won} won, {lost} lost, {dq} disqualified, {nurture} nurture) from {date_range}.
>
> Quick sanity check:
> - Total deal value: ${total}
> - Average deal size (won): ${avg_won}
> - Win rate: {win_rate}% (won / (won + lost))
>
> Does this look roughly right?

If <30 deals, warn: "Small sample size ({N} deals). Analysis will have limited statistical power. Consider expanding the lookback window with `--days 365`."

### Agent 2: Call Transcripts

**If `--skip-calls` is set:** Skip this agent. Read fallback files from `config.interview_paths.gong_exports` glob pattern instead. Note in methodology.

#### Gong Path (if call_recorder.type is "gong")

**Authentication:**
```bash
source ~/.zshrc 2>/dev/null; source ~/.env 2>/dev/null
AUTH=$(echo -n "${GONG_ACCESS_KEY}:${GONG_ACCESS_SECRET}" | base64)
```

**2a: Get calls in date range:**
```bash
curl -s -X POST "${GONG_BASE_URL:-https://us-55616.api.gong.io}/v2/calls/extensive" \
  -H "Authorization: Basic ${AUTH}" \
  -H "Content-Type: application/json" \
  -d '{
    "filter": {
      "fromDateTime": "{START_ISO}",
      "toDateTime": "{END_ISO}"
    },
    "contentSelector": {
      "exposedFields": {
        "content": { "topics": true, "trackers": true, "pointsOfInterest": true },
        "parties": true
      }
    }
  }'
```

Handle pagination: if `records.cursor` is present, make additional requests with that cursor.

**2b: Get CRM associations:**
```bash
curl -s "${GONG_BASE_URL:-https://us-55616.api.gong.io}/v2/calls/manual-crm-associations" \
  -H "Authorization: Basic ${AUTH}"
```

Build mapping: `{ sfOpportunityId: [gongCallId, ...] }`.

**2c: Get transcripts for linked calls (batches of up to 100):**
```bash
curl -s -X POST "${GONG_BASE_URL:-https://us-55616.api.gong.io}/v2/calls/transcript" \
  -H "Authorization: Basic ${AUTH}" \
  -H "Content-Type: application/json" \
  -d '{ "filter": { "callIds": ["{CALL_ID_1}", "{CALL_ID_2}"] } }'
```

**2d: Get AI summaries:**
```bash
curl -s -X POST "${GONG_BASE_URL:-https://us-55616.api.gong.io}/v2/calls/ai-content" \
  -H "Authorization: Basic ${AUTH}" \
  -H "Content-Type: application/json" \
  -d '{ "filter": { "callIds": ["{CALL_ID_1}", "{CALL_ID_2}"] } }'
```

**Rate limits:** 3 requests/second, 10,000/day. Add 350ms delay between requests. Handle 429 with `Retry-After` header.

#### Fathom Path (if call_recorder.type is "fathom")

**List recent meetings:**
```bash
source ~/.zshrc 2>/dev/null; source ~/.env 2>/dev/null && curl -s "https://api.fathom.ai/external/v1/meetings?created_after=$(date -v-{DAYS}d +%Y-%m-%dT00:00:00Z)" \
  -H "X-Api-Key: $FATHOM_API_KEY"
```

Response is in `.items[]`. Paginate with `next_cursor`.

**Select representative sample:** Pick 10-15 calls that appear to be sales/prospect calls (filter by attendee domains, exclude internal). Prioritize calls associated with closed deals if association data exists.

**Get transcripts:** For each selected meeting, fetch `/recordings/{id}/transcript`.

**Get summaries:** For broader pattern detection, fetch `/recordings/{id}/summary` for all calls.

**Linking to deals:** Fathom has no native CRM association. Match calls to deals by:
1. Attendee email domain matching to account domain
2. Attendee company name matching to deal account name
3. Note lower linking confidence in methodology

#### Manual Path (if call_recorder.type is "manual")

> No call recorder connected. You can still provide transcripts:
>
> **Option A:** Paste transcript text directly into chat
> **Option B:** Provide file paths to transcript files (.txt, .md, .pdf)
>
> Ideally provide 3-5 transcripts from a mix of won and lost deals. For each, tell me whether the deal was won or lost.

**All paths return:** Call ID/title, date, duration, participant names and roles, transcript text, AI summary (if available), linked deal ID (if known).

### Agent 3: Knowledge Files

Read all qualitative data from the company's portfolio folder at `{config.portfolio_path}`.

**Files to read (in order of priority):**

1. Paths from `config.interview_paths.knowledge_files` (synthesized knowledge files)
2. `config.interview_paths.churn_interviews` directory
3. `config.interview_paths.market_research` directory
4. Files matching `config.interview_paths.gong_exports` glob
5. Standard locations: `knowledge/product.md`, `knowledge/competitive-landscape.md`, `knowledge/icp.md`

**Extract from each file:**
- Company names mentioned
- Churn/loss reasons cited
- Competitor mentions (positive, negative, neutral)
- Product gaps or feature requests
- Direct quotes explaining why a customer left or chose a competitor
- Market perception themes

**Return:** Structured index organized by company evidence and thematic evidence. If no interview files exist, note: "No interview data available. Analysis based on CRM and call data only."

### Agent 4: Revelio Labs Enrichment (if --enrich set)

**Only runs if `--enrich` flag is set and Snowflake MCP is available.**

After Agent 1 returns the deal list, take the unique account/company names and match them to Revelio Labs data.

**Step 4a: Match companies to Revelio RCIDs**

For each unique account name from CRM deals, query Snowflake:

```sql
SELECT RCID, PRIMARY_NAME, COMPANY_DISPLAY_NAME, URL,
  RAW_COUNT, YEAR_FOUNDED, NAICS_DESC, NAICS_DESC_2D,
  SAAS_COMPANY, HQ_COUNTRY, HQ_STATE
FROM REVELIO_LABS.TELESCOPE.COMPANY_REF_LATEST
WHERE LOWER(PRIMARY_NAME) LIKE LOWER('%{account_name}%')
   OR LOWER(COMPANY_DISPLAY_NAME) LIKE LOWER('%{account_name}%')
LIMIT 5
```

If multiple matches, pick the best match by name similarity. If no match, skip that account.

**Step 4b: Pull growth metrics for matched companies**

For each matched RCID:

```sql
SELECT RCID, COMPANY, CATEGORY, CATEGORY_TYPE,
  COUNT, RAW_COUNT, PERCENT_OF_COMPANY,
  GROWTH_1M, GROWTH_3M, GROWTH_6M, GROWTH_1Y
FROM REVELIO_LABS.TELESCOPE.GROWTH_METRICS_LATEST
WHERE RCID = {rcid}
AND CATEGORY_TYPE = 'all'
```

**Step 4c: Build enrichment dataset**

For each deal, attach (where available):
- `revelio_headcount`: actual employee count from Revelio (fills CRM gaps)
- `revelio_growth_1y`: 1-year headcount growth rate (growing vs. shrinking company)
- `revelio_industry`: NAICS industry classification (more standardized than CRM)
- `revelio_year_founded`: company age
- `revelio_is_saas`: SaaS flag

These enriched fields become additional columns in the XGBoost feature matrix (Step 4H) and provide backup data when CRM fields are empty.

**Report:** "Enriched {X} of {Y} accounts with Revelio Labs data (headcount, growth, industry)."

### Agent 5: Account-Level Retention Data (MANDATORY for Salesforce)

**This agent is REQUIRED, not optional.** Retention by segment is necessary for any segment recommendation (see Step 5.5). Without it, recommendations are labeled "Conditional."

**Purpose:** Calculate quarterly retention (logo and GDR) by tier/segment from Account-level data. The Opportunity object only has new business and competitive losses. Churn data lives on the Account object.

**Salesforce — discover retention fields:**

First, discover which Account fields exist for retention analysis:

```bash
sf sobject describe --sobject Account --target-org {config.crm.sf_org} --json
```

Search for fields matching these patterns:
- Status: `*Status*`, `*Stage*`, `*Active*`, `*Live*`, `*Customer*`
- Churn: `*Churn*`, `*Cancel*`, `*Churned*`
- Revenue: `*ARR*`, `*MRR*`, `*Revenue*`, `*Projected*`
- Dates: `*Live_Date*`, `*Go_Live*`, `*Churn_Date*`
- Tier: `*Tier*`, `*Segment*`
- Churn detail: `*Churned_Reason*`, `*New_Subscription_Vendor*`, `*Lost_To*`

**Salesforce — pull Account data:**

```bash
sf data query --query "
SELECT {tier_field}, {status_field}, {arr_field}, 
  {live_date_field}, {churn_date_field}, {churn_arr_field},
  {churn_reason_field}, {new_vendor_field}, Name
FROM Account 
WHERE {live_date_field} != null AND {tier_field} != null
ORDER BY {live_date_field} ASC
" --target-org {config.crm.sf_org} --json
```

**Calculate quarterly retention cohorts:**

For each quarter in the analysis period and each tier:
1. **Starting cohort**: accounts live before quarter-start AND not yet churned
2. **Churned in quarter**: accounts from starting cohort that churned during the quarter
3. **Logo retention**: (starting - churned) / starting
4. **GDR**: (starting ARR - churned ARR) / starting ARR

**Output required:**

```
Quarter | Tier   | Start | Churned | Logo Ret | Start ARR | Churned ARR | GDR
Q1'25   | Tier 1 |    66 |       5 |   92.4%  | $2.82M    | $799K       | 71.7%
Q1'25   | Tier 2 |    67 |       3 |   95.5%  | $531K     | $49K        | 90.9%
...
```

Plus annualized retention by tier, churn reasons by tier, and "churned to" by tier.

**REQUIRED output tables (all must be present or logged as gaps):**

1. **Logo retention by tier (quarterly):**

| Quarter | Tier | Starting Accounts | Churned | Logo Retention % |
|---------|------|-------------------|---------|-----------------|

2. **Dollar retention by tier (quarterly):**

| Quarter | Tier | Starting ARR | Churned ARR | GDR % |
|---------|------|-------------|------------|-------|

3. **Churn concentration (full period):**

| Tier | Total Churned Logos | Total Churned $ | % of All Churned $ | % of All Churned Logos | Avg $ per Churn |
|------|--------------------|-----------------|--------------------|----------------------|----------------|

4. **Churn reasons by tier:**

| Tier | Top Reason | Count | ARR Impact |
|------|------------|-------|-----------|

If any table cannot be populated, log as CRITICAL data gap. The churn concentration table is the most important: it reveals whether churn dollars are distributed evenly across tiers or concentrated in one segment. This directly impacts segment recommendations.

**If Account-level retention fields don't exist:** Log as CRITICAL data gap: "Cannot calculate retention by segment. Account object lacks churn date, status, or ARR fields. This prevents validating any segment-level recommendation." Ask the user if this data exists elsewhere (e.g., a separate churn tracking system, a BI tool, or a finance template).

**HubSpot:** Use HubSpot company object with lifecycle stage and deal associations. Calculate retention from company stage transitions (Customer → Former Customer). Revenue data may require summing associated deal values.

**Report:** "Retention by tier: T1 GDR {X}%, T2 GDR {Y}%, T3 GDR {Z}%. {N} accounts churned in period, ${ARR} total ARR lost. Churn concentration: T1 = {X}% of churned $, T2 = {Y}%, T3 = {Z}%."

---

## Step 3: Data Linking

After all agents return, link the data together.

### 3a: Deal-to-Call Linking

**Gong:** Match Gong CRM association `crmObjectId` values to Salesforce Opportunity `Id` field.

**Fathom:** Match by attendee email domain or company name (fuzzy). Flag matches as lower confidence.

**Manual:** User specified which deal each transcript relates to.

### 3b: Deal-to-Interview Linking

Match interview evidence to deals using case-insensitive substring matching on company/account names. If no match, the evidence still contributes to thematic analysis but isn't linked to a specific deal.

### 3c: Deal-to-Enrichment Linking (if --enrich)

Match Revelio data to deals via the account name-to-RCID mapping from Agent 4.

### 3d: Build Unified Records

For each deal, create a unified record containing:
- All CRM fields
- Array of linked call transcripts + summaries
- Array of linked interview evidence
- Revelio enrichment data (if available)
- Evidence coverage flag: `crm_only`, `crm+calls`, `crm+interviews`, `crm+calls+interviews`

**Report:** "Linked {X} deals to {Y} call transcripts, {Z} to interview evidence. {W} deals have evidence from all sources."

---

## Step 4: Analysis Modules

Run these modules sequentially. Each builds on prior outputs.

**Throughout all modules:** Track data gaps. Every time a module encounters a missing field, sparse data, or insufficient sample, log it as a gap entry for Step 6.

### Module 4A: Temporal Foundation

**Purpose:** Establish the time dimension before any segmentation.

1. Group all deals by quarter (or month if <6 months of data)
2. Calculate overall win rate per period (Won / (Won + Lost), exclude DQ and Nurture)
3. Identify inflection points: any quarter where win rate changed >10 percentage points from prior quarter
4. Calculate additional per-period metrics: total revenue won, total revenue lost, average deal size, DQ rate
5. **For the most recent quarter**: show monthly breakdown (deals, win rate, bookings) to surface within-quarter momentum

**Report:**

> **Timeline Overview (BOTH count and dollar views required):**
> | Quarter | Deals | Won | Lost | DQ | Win Rate | Revenue Won | Revenue Lost | Net |
> |---------|-------|-----|------|----|----------|-------------|--------------|-----|
> | {quarter} | {N} | {won} | {lost} | {dq} | {rate}% | ${X} | ${Y} | ${Won-Lost} |
>
> **Most Recent Quarter Monthly Breakdown:**
> | Month | Deals | Won | Lost | Win Rate | Bookings | Trend vs Prior Month |
> |-------|-------|-----|------|----------|----------|---------------------|
>
> {If inflection point:} **Something changed in {quarter}** -- win rate shifted from {X}% to {Y}%. We'll investigate what drove this in the segment analysis.

### Module 4B: Segmented Analysis

For each requested segment dimension (`--segment` parameter), calculate per-segment metrics.

**ACV Tier segmentation:**
- Use `deal_tier` field first if it exists and is populated
- If not, auto-detect boundaries from the deal amount/ARR distribution:
  - Calculate 25th, 50th, 75th percentiles
  - Tier 1 (Enterprise): top quartile
  - Tier 2 (Mid-Market): middle 50%
  - Tier 3 (SMB): bottom quartile
  - Report the boundaries found

**Other segments:** Use the mapped field directly:
- Vertical/Industry: group by `vertical_or_industry` values
- Source: group by `deal_source`
- Competitor: group by `competitor` (which competitor was the deal lost to)
- Owner: group by `owner`
- Lifecycle: group by `deal_type` combined with stage (New Business Won, New Business Lost, Existing Business Won, Existing Business Lost/Churn, DQ, Nurture)

**Per-segment metrics:**

| Metric | Calculation |
|--------|-------------|
| Deal count | Total, Won, Lost |
| Win rate (overall) | Won / (Won + Lost) for entire lookback |
| Win rate (last 2 quarters) | Same calculation, recent data only |
| Win rate trend | Improving (recent > overall by >5pp), Declining (recent < overall by >5pp), or Stable |
| Revenue won | Sum of ARR/amount for won deals |
| Revenue lost | Sum of ARR/amount for lost deals |
| **Net revenue impact** | Revenue won minus revenue lost (a segment can have a high WR but negative net if it loses big deals) |
| Avg deal size (won vs. lost) | Mean ARR/amount by outcome |
| Top loss reasons | Distribution of loss_reason values within this segment |
| Recency flag | If last 2 quarters diverge from trailing average by >10pp, flag it |

**MANDATORY dual-axis views:** For pipeline and bookings analysis, ALWAYS produce BOTH:
1. **Opps count view** (deals by segment, source, etc.)
2. **Dollar value view** ($ by segment, source, etc.)

These often tell dramatically different stories. Example: Enterprise may represent 5% of opps but 40% of pipeline dollars. Present both views side by side or as companion tables. If you only show one, you are hiding the other story.

**Flag any segment where:**
- Win rate is >10pp above or below the overall average
- Revenue lost exceeds revenue won
- The trend direction has reversed recently

**Cross-segment analysis (required):**
For each primary segment dimension, cross-tabulate with at least one other dimension:
- Tier x Source: win rate for each tier within each source (e.g., "Partner channel: Tier 3 wins at 66.7%, Tier 1-2 wins at 27.3%")
- Tier x Competitor: which competitors you face at each tier
- Source x Competitor: which competitors show up in which channels
- Source secondary breakdown: if a source has sub-types (e.g., Partner has Agency vs. Tech partners in Deal_Source_Secondary__c), always break it down. The sub-segment is often more actionable than the top-level segment.

**Outlier handling (required):**
For each segment with <10 deals, check if a single deal represents >30% of the segment's revenue. If so, report both the with-outlier and without-outlier metrics. Lead with the deal count story, not the revenue story.

### Module 4C: Loss Reason Analysis

**If loss_reason field exists and is structured (picklist):**
1. Group lost deals by loss reason
2. Rank by frequency AND by revenue impact
3. For each top-10 reason:
   - Count of deals lost
   - Total ARR/revenue lost
   - Which segments are most affected
   - Trend: compare reason's share in the last 2 quarters vs. trailing average
   - When it emerged: flag any reason that was <10% of losses 6+ months ago but >25% recently (emerging threat)

**If loss_reason is free text:**
1. Read all unique loss reason values
2. Cluster similar reasons into 8-12 categories (use judgment to group synonyms: "too expensive" / "pricing" / "budget constraints" all map to "Price")
3. Present proposed categories for confirmation
4. Then run the same ranked analysis as structured data

**If loss_reason field doesn't exist:**
Log a Critical data gap: "No loss reason field. Cannot determine why deals are lost from CRM data. Relying on call transcripts and interviews only. Recommend adding a 'Closed Lost Reason' picklist to your CRM."

### Module 4D: Competitive Intelligence

**If feature intel was run in Step 0.5:** Read `{portfolio_path}/knowledge/feature-comparison-*.md` before writing any competitive analysis. Use the radar scores, unique advantages, and critical gaps to ground claims. Specifically:
- Where call/interview evidence says "competitor X is stronger on analytics" — cross-check against the feature intel radar score. Is the gap real and quantified, or is it perception?
- Where CRM identifies a competitor as a deal-loss driver — check whether the feature intel identifies a specific differentiator (e.g., BigQuery integration, native loyalty engine) vs. broad category weakness. The specific gap is more actionable.
- Flag any feature gap claimed from CRM/calls that the feature intel shows the portfolio company already has (false gap — exists but isn't marketed or discovered in demos).

**From CRM data:**
- Group lost deals by `competitor` field
- Rank competitors by: (1) frequency, (2) ARR lost to them, (3) segments where they appear
- Calculate your win rate when each competitor is present vs. absent
- Identify incumbent providers you successfully displace vs. fail to displace (from `current_provider` field)

**From call transcripts:**
For each competitor identified in CRM data, search call transcripts for mentions. Extract:
- Direct quotes about the competitor (positive and negative)
- How your reps position against them
- What prospects say they like about the competitor
- What prospects say your product does better

**From interview evidence:**
Cross-reference competitor mentions from knowledge files and interview transcripts.

**Competitor trend analysis:**
- Compare each competitor's share of losses in recent 2 quarters vs. trailing average
- Flag emerging threats: competitors appearing in >25% of recent losses but <10% historically
- Flag fading competitors: declining by >50% across the lookback period

**Repeatability check per competitor (required):**
For each competitor, before making claims about displacement difficulty:
- Show displacement rate by tier (don't aggregate across tiers)
- Show displacement rate by source channel
- Flag if the sample is driven by 1-2 large deals (revenue concentration check)
- Separate "deals where the prospect was actively leaving" from "deals where we tried to displace them" (if distinguishable from CRM fields)
- If total deals against a competitor is <5, label as "Insufficient sample, not actionable"

**Output per competitor:**
- Deals lost: {N} (${ARR} lost)
- Segments most affected: {tier, vertical}
- Displacement rate by tier: Tier 1: X%, Tier 2: Y%, Tier 3: Z%
- Trend: growing / stable / fading
- Your win rate against them: {X}% (with outlier check if N<10)
- Their advantage (from calls/interviews): "{quote}"
- Your advantage (from calls/interviews): "{quote}"
- Repeatability: Is this a persistent pattern (5+ deals, 2+ quarters) or an emerging signal?

### Module 4E: Deal Velocity Analysis

**Skip if no `stage_date_fields` exist in field_mapping.** Log as a High-priority data gap.

Calculate days between consecutive stage date timestamps for each deal. Use median (not mean) to reduce outlier impact.

**Stage transitions to measure (adapt to whatever stage_date_fields exist):**
1. CreatedDate to first stage date
2. Each consecutive stage pair
3. Last stage date to close (Closed Won or Closed Lost)
4. Total: CreatedDate to final outcome

**Velocity output:**

| Stage Transition | Won (median days) | Lost (median days) | DQ (median days) | Delta (Won vs Lost) |
|-----------------|-------------------|--------------------|--------------------|---------------------|
| {transition} | {X} | {Y} | {Z} | {+/- N days} |
| Total Cycle | {X} | {Y} | {Z} | {+/- N days} |

**Stall point identification:** For lost deals, find the stage with the longest median dwell time. This is where deals die. Cross-reference with Gong transcripts from calls during that stage.

**Velocity cuts (required, not optional):**
All velocity metrics must be segmented. Never present a single company-wide median without the breakdown.
- Won vs. Lost vs. DQ
- By ACV tier (required)
- By competitor (if data exists)
- By deal source (if data exists)
- By deal size band (if pricing ceiling exists, show velocity above and below it)

### Module 4F: Pricing Analysis

**From CRM deal amounts:**
1. Calculate win rate by price band (split deals into quintiles or deciles by amount/ARR)
2. Identify pricing sweet spot: the band with the highest win rate weighted by volume
3. Identify pricing ceiling: the price point above which win rate drops sharply (>15pp decline)
4. Compare average deal size for won vs. lost deals (are we winning smaller deals and losing bigger ones?)

**From contract type (if field exists):**
- Win rate by contract structure (annual vs. monthly, etc.)
- Average deal size by contract type

**From call transcript evidence:**
Search transcripts for pricing-related terms: "price", "cost", "budget", "expensive", "cheaper", "discount", "ROI", "value". Extract relevant quotes and categorize sentiment (price seen as fair, high, or low).

**Pricing by segment (required):**
- Show win rate by price band within each tier (the ceiling may differ by tier)
- Show pricing sweet spot separately for new business vs. existing business
- If a price band has <5 deals, merge with adjacent band rather than drawing conclusions from thin data

**Output:**
- Pricing sweet spot by tier: Tier 3: ${X}-${Y} ({Z}% win rate, {N} deals). Tier 2: ${X}-${Y}. Tier 1: ${X}-${Y}.
- Pricing ceiling by tier (the point where win rate drops >15pp)
- Contract preference: {type} wins at {X}% vs. {Y}% for other types
- Prospect sentiment on pricing: {summary with quotes}
- Outlier check: does a single large deal distort any price band?

### Module 4G: Feature Gap Analysis

**From CRM (product_gap field, if exists):**
- Group lost deals by product gap reason
- Rank by: (1) count of deals affected, (2) total ARR lost
- Show which segments and competitors are associated with each gap

**From call transcripts:**
- Extract feature requests, "I wish it could...", "we need...", "does it support..." mentions
- Categorize into product gap themes
- Match to CRM product_gap categories where possible

**From interview evidence:**
- Pull product gap mentions from knowledge files and churn interviews
- Add interview quotes to each gap

**Revenue quantification:**
For each top-5 gap:
- Deals lost due to this gap: {N}
- ARR lost: ${X}
- Evidence strength: Strong / Moderate / Emerging
- Evidence block: SF data + Gong quote + Interview quote

**Roadmap cross-reference:**
If `{portfolio_path}/knowledge/product.md` exists, read it and note which gaps are on the roadmap vs. not. Flag gaps with high revenue impact that are NOT on the roadmap.

**Feature intel cross-reference (if Step 0.5 ran):**
For each gap identified from CRM/calls, check the feature intel taxonomy before reporting it:
- **Confirmed real gap:** Feature is `missing` or `implied` in the scraped taxonomy → report with full evidence
- **False gap (exists but isn't discovered):** Feature shows `explicit` in help center crawl but wasn't mentioned in demos or marketing pages → report as "exists, not marketed" rather than a product gap. This is a sales/positioning fix, not a product fix.
- **Depth gap:** Feature is `explicit` but with only 1-2 thin help articles vs. competitor's 10+ → report as "feature exists but is shallow vs. competitor's mature implementation"

This prevents the analytical mistake of recommending product investment in features that already ship.

### Module 4H: Predictive Analysis (XGBoost)

**Skip if `--skip-ml` is set.** Note in data gaps.

**Step A: Prepare the feature matrix**

Convert each deal into a feature row. Use all available numeric and categorical fields:

| Feature | Source | Encoding |
|---------|--------|----------|
| estimated_arr | CRM | Numeric (log-transform if skewed) |
| acv_tier | CRM | One-hot encode |
| industry | CRM | One-hot top 5, collapse rest to "Other" |
| deal_source | CRM | One-hot encode |
| contract_type | CRM | One-hot encode |
| current_provider | CRM | One-hot encode |
| employee_count | CRM or Revelio | Numeric |
| deal_type | CRM | One-hot encode |
| total_cycle_days | Calculated | Numeric |
| gong_call_count | CRM | Numeric |
| has_competitor | CRM | Binary |
| has_gong_calls | Linked data | Binary |
| revelio_headcount | Enrichment | Numeric (if --enrich) |
| revelio_growth_1y | Enrichment | Numeric (if --enrich) |
| revelio_is_saas | Enrichment | Binary (if --enrich) |
| company_age_years | Enrichment | Numeric (if --enrich) |

Target variable: `is_won` (1 = Won, 0 = Lost). Exclude DQ and Nurture.

**Step B: Correlations**

```python
import pandas as pd
import numpy as np

numeric_cols = [all numeric features from matrix]
correlations = {}
for col in numeric_cols:
    valid = df[[col, 'is_won']].dropna()
    if len(valid) > 10:
        corr = valid[col].corr(valid['is_won'])
        correlations[col] = round(corr, 3)

sorted_corr = sorted(correlations.items(), key=lambda x: abs(x[1]), reverse=True)
```

Interpret: |r| > 0.3 = meaningful, |r| > 0.5 = strong, |r| < 0.1 = negligible.

**Step C: XGBoost**

```python
import xgboost as xgb
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import LabelEncoder
from sklearn.metrics import accuracy_score, roc_auc_score

X = df[feature_cols].copy()
for col in X.select_dtypes(include='object').columns:
    X[col] = LabelEncoder().fit_transform(X[col].astype(str))
X = X.fillna(-1)
y = df['is_won'].astype(int)

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42, stratify=y)

model = xgb.XGBClassifier(
    n_estimators=100, max_depth=4, learning_rate=0.1,
    random_state=42, verbosity=0, eval_metric='logloss'
)
model.fit(X_train, y_train)

y_pred = model.predict(X_test)
y_proba = model.predict_proba(X_test)[:, 1]
accuracy = accuracy_score(y_test, y_pred)
auc = roc_auc_score(y_test, y_proba)

importance_df = pd.DataFrame({
    'feature': feature_cols,
    'importance': model.feature_importances_
}).sort_values('importance', ascending=False)
```

**Step D: Interpret**

Report top 10 features. For each:
- Feature name and importance score
- Direction (higher = more likely to win or lose?)
- Does it align with segment analysis findings?
- If surprising (high importance but not in segment findings), flag as "hidden driver"

**Model quality:**
- AUC < 0.6: "Weak predictive power. Structured fields don't strongly predict outcomes. Qualitative factors likely dominate."
- AUC 0.6-0.75: "Moderate predictive power. Feature rankings are directional."
- AUC > 0.75: "Strong predictive power. Feature importance rankings are reliable."

**Interaction effects (if >100 deals):**
Check if deal source effectiveness varies by tier, or if competitor impact varies by vertical. Flag any interaction where a variable's win rate impact differs by >15pp across segments.

### Module 4I: ICP Synthesis

Synthesize all prior modules into an Ideal Customer Profile.

**Descriptive ICP ("who your best customer IS today"):**

*Firmographic profile* (from segment win rates + XGBoost + enrichment):
- Optimal ACV tier: highest win rate, weighted by volume (80% win rate on 5 deals is less reliable than 60% on 50 deals)
- Best-fit verticals: top 2-3 by recency-weighted win rate
- Company size sweet spot: employee/subscriber count range where win rate peaks
- Revenue range: P25, median, P75 of won deal amounts
- Include "last 2 quarters" vs. "overall" for each, flag divergence

*Behavioral signals* (from deal patterns):
- Best deal sources (highest recent win rates)
- Contract preferences (which type wins most)
- Provider displacement targets (which incumbent providers you successfully displace)
- Engagement level (Gong call count correlation with winning, if available)

*Competitive positioning*:
- Who you beat consistently
- Who beats you consistently
- Top 3 differentiators (from call transcripts)
- Top 3 weaknesses (from call transcripts + loss reasons)

**Prescriptive ICP ("who your best customer SHOULD BE"):**

*Under-invested winners:*
Segments with high win rate (>average + 10pp) but low pipeline volume (<20% of total deals). Calculate: "If you doubled pipeline in this segment, at the current {X}% win rate, that's ${Y} additional revenue per quarter."

*Emerging segments:*
Small sample (<15 deals) but strong recent performance (win rate >60% in last 2 quarters). Flag as "worth monitoring, too early to bet on."

*Fixable losers:*
Segments with addressable loss reasons (specific product feature, pricing, sales process). Quantify: "${X} in lost revenue from {N} deals, addressable by {specific action}."

*Abandon candidates:*
Segments with consistently low win rates (<25% recent) and structural disadvantages. Calculate: "You spent resources on {N} deals in this segment and won {M}. Reallocating that effort to {winning segment} would yield ${X} more."

**Anti-ICP** (who to avoid):
- Segments with <25% recent win rate
- Deal characteristics that predict losing (from XGBoost)
- Customer profiles from churn interviews

**Velocity benchmarks:**
- Median cycle time for ICP deals vs. non-ICP
- Expected time per stage for ICP
- Red flag: if a deal exceeds 75th percentile dwell at any stage, it's at risk

**Repeatability filter for all recommendations:**
Before recommending investment in any segment:
1. Confirm the pattern holds across sub-segments (e.g., Partner channel works because of agency partners specifically, not all partners)
2. Confirm the pattern is not driven by a one-time event (e.g., a platform shutting down, a migration wave that won't recur)
3. Report how many unique deals support the recommendation (minimum 10 for a strong recommendation, 5-9 for a directional recommendation, <5 is "insufficient data")
4. If a recommendation depends on a specific sub-segment, say so explicitly: "Invest in agency partner referrals for Tier 3 deals" not "Invest in the Partner channel"
5. For every recommendation, ask: "Would this pattern hold next quarter if the conditions that created it didn't recur?"

**Comparison with existing ICP:**
If `{portfolio_path}/knowledge/icp.md` exists, compare:
- What the data confirms
- What the data contradicts
- What's new (patterns not in the current ICP)

---

## Step 5: Evidence Cross-Referencing

For every significant finding (appears in 3+ deals OR represents >15% of a segment):

1. **CRM evidence:** Cite specific deals and their relevant field values
2. **Call evidence:** Find relevant transcript quotes. Cite as: `[Call: "{title}", {date}, "{quote}"]`
3. **Interview evidence:** Find relevant quotes from knowledge files. Cite as: `[Interview: {source_file}, "{quote}"]`

**Evidence strength rating:**
- **Strong**: All 3 sources confirm (CRM + Call + Interview)
- **Moderate**: 2 of 3 sources confirm
- **Emerging**: 1 source only, flag as "needs further investigation"

Tag every finding with its evidence strength.

---

## Step 5.5: Recommendation Validation

**This step is MANDATORY before writing any strategic recommendation.** Every finding that leads to an "invest in X" or "stop doing Y" recommendation must pass this checklist. If any item fails, either get the data or label the recommendation as **"Conditional on [missing data]."**

### Full-Funnel Check (REQUIRED for any segment recommendation)

Before recommending focus on ANY segment (tier, vertical, channel, customer profile):

- [ ] **Acquisition data present**: Win rate, deal count, and bookings $ for the recommended segment
- [ ] **Retention data present**: GDR, NDR, and logo retention for the recommended segment, broken out by tier. If not available by segment, flag as **Critical data gap** and label the recommendation as conditional. Do NOT present a segment recommendation without retention data and call it definitive.
- [ ] **Expansion data present**: Upsell/cross-sell rate for the recommended segment. If not available, note it.
- [ ] **LTV math checked**: Does the segment's retention rate support the acquisition cost? A 55% win rate on $6K ACV with 70% annual retention has a fundamentally different LTV profile than a 30% win rate on $30K ACV with 82% retention. Run the math explicitly. Show it.
- [ ] **Churn concentration checked**: Show what % of churned DOLLARS comes from the recommended segment vs. others. A segment with high win rates but disproportionate churn dollars may not be the right focus.

### Enforcement (MANDATORY)

Before writing the final report, verify EACH segment recommendation against this matrix. Copy this table into the report:

| Recommendation | Acquisition? | Retention? | Churn Concentration? | LTV Math? | Status |
|---|---|---|---|---|---|
| [each segment rec] | Yes/No | Yes/No | Yes/No | Yes/No | Validated / Conditional |

If ANY segment recommendation has "No" for Retention or Churn Concentration, it MUST be labeled:

> **Conditional Recommendation:** This recommendation is based on acquisition data only. Retention by segment has not been validated. The recommendation may change if segment-level retention reveals different patterns (e.g., if SMB churns at 50% logo retention, the "focus on SMB" argument weakens despite high win rates).

Do NOT present conditional recommendations as definitive. This enforcement exists because it was missed in the Stay AI analysis (April 2026), where a "focus on SMB" recommendation was made without retention data. Nicole caught it: "I don't think we can credibly make a recommendation without looking at retention both logo and NRR by segment."

### Dual-Axis Views (REQUIRED for any chart-based finding)

- [ ] Every pipeline chart shown in BOTH opps count AND dollar value. The stories often differ materially (e.g., enterprise may be small in opps but dominant in $).
- [ ] Every bookings view shown by segment in dollars, not just win rates
- [ ] Every source/channel view shown in both opps and dollars
- [ ] If only one view is shown, explicitly state why and what the other view would show

### Denominator Check (REQUIRED for any "this is working" claim)

- [ ] For any segment claimed as "working," show both wins AND losses (count + dollars)
- [ ] Calculate **net impact**: won $ minus lost $. A channel with 67% WR but negative net $ impact tells a different story.
- [ ] For onboarding/displacement claims (e.g., "Recharge displacement is working"), show total attempts (won + lost), not just wins. State the overall displacement win rate.
- [ ] For any "X accounts did Y" claim, state what fraction of the total that represents

### Trend Validation (REQUIRED for any "improving" or "declining" claim)

- [ ] Show **monthly breakdown within the most recent quarter** (not just quarterly snapshots)
- [ ] Compare Q-on-Q AND versus the same quarter prior year (if available)
- [ ] State explicitly: **"This is a trend (3+ quarters of consistent direction)"** vs. **"This is a single-quarter signal"** vs. **"This is within-quarter momentum (monthly trajectory)"**
- [ ] For the most recent quarter, state whether the monthly trajectory is accelerating, decelerating, or flat

### Current-State Verification (REQUIRED for any operational recommendation)

- [ ] For "hire X" recommendations: verify if the role is already filled (check team knowledge file, recent meeting notes, or ask the user)
- [ ] For "stop doing X" recommendations: verify the current resource allocation
- [ ] For "invest in X" recommendations: verify current spend level
- [ ] For "fix X" recommendations: verify whether the fix is already in progress

### Counter-Argument Test (REQUIRED for every recommendation)

- [ ] For each recommendation, explicitly state the strongest argument against it
- [ ] Example: For "focus on SMB" — "The counter-argument is that SMB retention is 70% (per Cohorts v3), meaning 30% of acquired SMB customers churn within 12 months. The LTV of a $6K SMB deal at 70% retention is $X, versus a $30K MM deal at 82% retention which has LTV of $Y."
- [ ] If the counter-argument is strong enough to change the recommendation, change it or add conditions

---

## Step 6: Data Gaps Analysis

Collect all gaps logged throughout the analysis. Deduplicate and prioritize.

**Priority logic:**
- **Critical**: Prevents answering a core strategic question (e.g., no loss reasons = can't explain losses)
- **High**: Significantly weakens a major analysis (e.g., no competitor field = competitive analysis relies only on call mentions)
- **Medium**: Limits a supplementary analysis (e.g., no stage dates = no velocity analysis)
- **Low**: Nice-to-have (e.g., XGBoost skipped = no predictive modeling)

**Data quality issues to surface:**
- Fields with >40% blank/"Other" values
- Stale data (e.g., Gong exports >3 months old)
- Sample size warnings per segment (<10 deals)
- Free-text fields that should be picklists
- Missing CRM fields that would significantly improve the analysis

**Output table:**

| Data Gap | What You Can't Know | How to Fix | Priority |
|----------|-------------------|------------|----------|
| {gap} | {what it prevents} | {specific action} | {Critical/High/Medium/Low} |

---

## Step 7: Generate Layered Report

Save to: `outputs/strategic-analysis-{YYYY-MM-DD}.md` (standalone repo mode) or `{portfolio_path}/deliverables/strategic-analysis-{YYYY-MM-DD}.md` (portfolio mode)

### Layer 1: Executive Summary (Shallow Dive)

A CEO reads this in 3 minutes and has the full picture.

```markdown
# Strategic Analysis: {Company Name}
**Period:** {start_date} to {end_date} | **Generated:** {today}
**Data:** {N} closed deals, {M} call transcripts, {P} interview sources

---

## Executive Summary

### Headline Metrics

| Metric | Value | By Segment | Note |
|--------|-------|------------|------|
| Win Rate | {X}% | New Biz: {Y}%, Existing: {Z}% | New biz is the real number |
| Revenue Won | ${X} | New Biz: ${Y}, Existing: ${Z} | |
| Revenue Lost | ${X} | Tier 3: ${Y}, Tier 1-2: ${Z} | |
| Avg Deal Size (Won) | ${X} median | Tier 3: ${Y}, Tier 1: ${Z} | Use median, not mean |
| Avg Cycle Time (Won) | {X} days median | Tier 3: {Y}d, Tier 1: {Z}d | |
| Biggest Competitor | {name} (${X} lost) | Tier breakdown: {details} | |

### Top 5 Findings

Each finding must specify the segment, sample size, and repeatability.

1. **{Finding}** {one sentence with segment, sample size, and a number}. *{Repeatable Pattern / Directional Signal / Insufficient Data}. Evidence: {Strong/Moderate/Emerging}*
2. **{Finding}** ...
3. **{Finding}** ...
4. **{Finding}** ...
5. **{Finding}** ...

### Your ICP Today
{2-3 sentence data-driven description: tier, vertical, size, source, win rate}

### Where You Should Be Investing
{2-3 sentences on the prescriptive ICP: under-invested segments, revenue upside, specific recommendation}

### What You Can't Know (Data Gaps)
1. {Gap}: {what it prevents, one line}
2. {Gap}: ...
3. {Gap}: ...

### Recommended Actions
1. **This week:** {highest-impact, lowest-effort action}
2. **This month:** {medium-effort change}
3. **This quarter:** {strategic shift}
```

### Layer 2: Medium Dive

A VP of Sales or Marketing reads this in 15 minutes and knows what to do.

```markdown
---

## Analysis Detail

### Win/Loss by Segment

#### By ACV Tier
| Tier | Deals | Win Rate (Overall) | Win Rate (Last 2Q) | Trend | Revenue Won | Revenue Lost | Avg Size |
|------|-------|--------------------|--------------------|----|-------------|--------------|----------|

#### By Source
| Source | Deals | Win Rate (Overall) | Win Rate (Last 2Q) | Trend | Revenue Won | Avg Size |
|--------|-------|--------------------|--------------------|----|-------------|----------|

#### By Vertical
| Vertical | Deals | Win Rate (Overall) | Win Rate (Last 2Q) | Trend | Revenue Won |
|----------|-------|--------------------|--------------------|----|-------------|

#### By Competitor
| Lost To | Deals Lost | ARR Lost | Your Win Rate Against | Trend | Worst Segments |
|---------|------------|----------|----------------------|-------|----------------|

### Loss Drivers (by Revenue Impact)

| Rank | Reason | Deals | ARR Lost | % of Losses | Trend | Evidence |
|------|--------|-------|----------|-------------|-------|----------|
| 1 | {reason} | {N} | ${X} | {Y}% | {growing/stable/fading} | {Strong/Moderate} |

### Deal Velocity

| Stage Transition | Won (median days) | Lost (median days) | Delta |
|-----------------|-------------------|--------------------|-------|

**Stall point:** Deals die at {stage}. Median dwell time for lost deals: {X} days vs. {Y} days for won deals.

### Pricing

- **Sweet spot:** ${X}-${Y} ({Z}% win rate)
- **Ceiling:** Above ${X}, win rate drops to {Y}%
- **What prospects say:** "{quote}" [Call: {title}, {date}]

### Feature Gaps (by Revenue Impact)

| Rank | Gap | Deals | ARR Lost | On Roadmap? | Evidence |
|------|-----|-------|----------|-------------|----------|

### Retention by Segment

| Tier | Logo Retention | GDR | Avg $ at Churn | % of Total Churned $ |
|------|---------------|-----|---------------|---------------------|

**Churn concentration:** [State which tier produces the most churned dollars relative to its share of churned logos. A tier with 25% of churned logos but 80% of churned dollars has concentrated risk.]

**Retention validates acquisition:** [Explicitly state whether the segment recommended for focus also has better retention. If it doesn't, flag the contradiction and reconsider the recommendation. Example: "SMB wins at 55% and retains at 90% GDR. Enterprise wins at 8% and retains at 82% GDR. The full-funnel case for SMB is validated."]

### Quarter-over-Quarter Trends

| Quarter | Won | Lost | DQ | Win Rate | Avg ARR Won | Top Loss Reason | Top Competitor |
|---------|-----|------|----|----------|-------------|-----------------|----------------|

### Data Gaps

| Gap | What You Can't Know | How to Fix | Priority |
|-----|---------------------|------------|----------|

### Methodology
- **CRM:** {type}, {N} records, queried {date}
- **Calls:** {type}, {N} transcripts + {M} summaries, {date range}
- **Interviews:** {N} files from {date range}
- **Enrichment:** {Revelio Labs: {N} accounts matched | Not enabled}
- **ML model:** {Accuracy {X}%, AUC {Y} | Skipped}
- **Lookback:** {days} days ({start} to {end})
```

### Layer 3: Deep Dive

An analyst or consultant reads this for the complete evidence. This layer must include a Pattern Reliability section.

```markdown
---

## Deep Dive

### Pattern Reliability

Rate every finding from the analysis. This is the analytical credibility check.

| Finding | Deals | Quarters | Outlier-Adjusted? | Repeatable? | Rating |
|---------|-------|----------|-------------------|-------------|--------|
| {finding} | {N} | {Q} | {Yes: metric holds without top deal / No / N/A} | {Yes/No} | {Repeatable Pattern / Directional Signal / Insufficient Data} |

**Rating criteria:**
- **Repeatable Pattern**: 5+ deals, persists across 2+ quarters, holds after removing outliers
- **Directional Signal**: 5+ deals but only 1 quarter of data, OR pattern weakens when outliers removed
- **Insufficient Data**: <5 deals, regardless of how strong the signal looks

### Predictive Model Results
{Only if XGBoost ran}

**Model performance:** Accuracy {X}%, AUC {Y} (on {N} test observations)

#### Top Predictors of Win vs. Loss
| Rank | Feature | Importance | Direction | Aligns with Segments? |
|------|---------|------------|-----------|----------------------|

#### Correlations
| Variable | Correlation | Interpretation |
|----------|-------------|----------------|

#### Interaction Effects
{Variables that behave differently across segments}

#### Hidden Drivers
{Features ranking high in XGBoost but not prominent in segment analysis}

### Ideal Customer Profile (Complete)

#### Firmographic Profile
| Dimension | Ideal Range | Win Rate | Sample Size | XGBoost Rank |
|-----------|-------------|----------|-------------|--------------|

#### Behavioral Signals
{Deal source, contract type, engagement patterns with specific win rates}

#### Anti-ICP
| Characteristic | Win Rate | Why (Evidence) |
|---------------|----------|----------------|

#### Velocity Benchmarks
| Metric | ICP Deals | Non-ICP Deals | Delta |
|--------|-----------|---------------|-------|

#### vs. Existing ICP
{What data confirms, contradicts, and adds vs. knowledge/icp.md}

### Segment Deep Dives

#### {Tier 1 / Enterprise}
{Full data table, top loss reasons, competitors, evidence block with direct quotes}

{Repeat for each tier, vertical, lifecycle type}

### Competitive Intelligence (Detailed)

#### {Competitor Name}
- **Deals lost:** {N} (${X} ARR)
- **Segments:** {where they win}
- **Their advantage:** {from evidence}
- **Your advantage:** {from evidence}
- **Call evidence:** "{quote}" [Call: {title}, {date}]
- **Interview evidence:** "{quote}" [Interview: {source}]

{Repeat for each major competitor}

#### Incumbent Provider Displacement
| Current Provider | Won (Displaced) | Lost (Failed) | Displacement Rate |
|-----------------|-----------------|---------------|-------------------|

### Product Gap Details

#### {Gap Name}
- **Deals affected:** {N}
- **ARR lost:** ${X}
- **Segments:** {most affected}
- **On roadmap:** {Yes/No}
- **CRM evidence:** {specific deals and field values}
- **Call evidence:** "{quote}" [Call: {title}, {date}]
- **Interview evidence:** "{quote}" [Interview: {source}]

{Repeat for each top gap}

### Evidence Appendix

#### Source Coverage
| Deal | CRM | Calls | Interviews | Enrichment |
|------|-----|-------|------------|------------|

#### Calls Analyzed
{List: title, date, linked deal, duration}

#### Interview Sources
{List: file name, company, date, type}

#### Data Freshness
- CRM: queried {date}, {N} records
- Calls: {API pull date or export date}, {N} calls
- Interviews: {N} files, {date range}
- Enrichment: {Revelio snapshot date}
```

---

## Step 8: Generate Standalone ICP File

Save to: `{portfolio_path}/knowledge/icp-from-analysis-{YYYY-MM-DD}.md`

```markdown
# Data-Driven ICP: {Company Name}
**Generated:** {date}
**Based on:** {N} deals ({date range}), {N} calls, {N} interview sources
**Model quality:** XGBoost AUC = {score} | {or "No predictive model"}

## Best-Fit Customer Profile

### Firmographics
{Each dimension: specific range, win rate, sample size, confidence}

### Behavioral Signals
{Deal source, contract type, engagement patterns}

### Anti-ICP
{Characteristics to avoid, with data}

### Velocity Benchmarks
{Expected cycle times for ICP vs. non-ICP deals}

## Prescriptive Recommendations
{Under-invested winners with revenue upside}
{Fixable losers with specific actions}
{Abandon candidates with reallocation math}

## vs. Existing ICP
{Confirms / Contradicts / Adds}

## Key Evidence
{Top 3-5 Gong quotes and interview quotes that validate the profile}

## Methodology
{Data sources, sample sizes, model quality, lookback window}
```

---

## Step 9: Google Doc Export

**Skip this step in standalone repo mode.** Google Doc export requires the gws CLI which is not included in this repo. The markdown report saved in Step 8 is the final output.

**Skip if `--skip-gdoc` is set (default in this repo).**

### 9a: Convert Report to HTML

Build an HTML file at `/tmp/strategic-analysis-{company}-{date}.html`.

**Base template:**
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<style>
body { font-family: Arial, sans-serif; font-size: 11pt; line-height: 1.25; color: #000; }
h1 { font-size: 26pt; font-weight: normal; margin-top: 0; margin-bottom: 4pt; }
h2 { font-size: 20pt; font-weight: bold; margin-top: 28pt; margin-bottom: 8pt; }
h3 { font-size: 14pt; font-weight: bold; margin-top: 20pt; margin-bottom: 6pt; }
p { margin-top: 0; margin-bottom: 8pt; }
table { border-collapse: collapse; width: 100%; margin-top: 8pt; margin-bottom: 24pt; font-size: 10pt; }
th { background-color: #f3f3f3; font-weight: bold; text-align: left; padding: 8px 10px; border: 1px solid #d9d9d9; }
td { padding: 8px 10px; border: 1px solid #d9d9d9; vertical-align: top; }
.subtitle { font-size: 13pt; color: #666; margin-bottom: 8pt; }
table.callout-table { margin-top: 2pt; margin-bottom: 8pt; font-size: 11pt; }
.caption { font-size: 9pt; color: #666; font-style: italic; }
ul { margin-top: 4pt; margin-bottom: 12pt; }
li { margin-bottom: 4pt; }
</style>
</head>
<body>
<!-- Convert the .md report content to HTML -->
</body>
</html>
```

**Formatting rules:**
- `<h1>` for doc title, `<h2>` for layer headers and major sections, `<h3>` for subsections
- All data tables use `<table>` with gray headers
- Evidence strength: <span style="color: #1a7a1a">Strong</span>, <span style="color: #b36b00">Moderate</span>, <span style="color: #cc1111">Emerging</span>
- Executive Summary findings in callout boxes: `<table class="callout-table"><tr><td style="background-color: #e8f5e8; border: none; padding: 12px 16px;">`
- Layer separators: `<h2>` with extra top margin

### 9b: Upload to Google Drive

```bash
CLIENT_ID=$(python3 -c "import json; d=json.load(open('$HOME/.config/gws/credentials.json')); print(d['client_id'])")
CLIENT_SECRET=$(python3 -c "import json; d=json.load(open('$HOME/.config/gws/credentials.json')); print(d['client_secret'])")
REFRESH_TOKEN=$(python3 -c "import json; d=json.load(open('$HOME/.config/gws/credentials.json')); print(d['refresh_token'])")

ACCESS_TOKEN=$(curl -s -X POST https://oauth2.googleapis.com/token \
  -d "client_id=$CLIENT_ID" \
  -d "client_secret=$CLIENT_SECRET" \
  -d "refresh_token=$REFRESH_TOKEN" \
  -d "grant_type=refresh_token" | python3 -c "import json,sys; print(json.load(sys.stdin).get('access_token',''))")

curl -s -X POST "https://www.googleapis.com/upload/drive/v3/files?uploadType=multipart" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -F "metadata={\"name\": \"Strategic Analysis: {Company} ({date})\", \"mimeType\": \"application/vnd.google-apps.document\"};type=application/json" \
  -F "file=@/tmp/strategic-analysis-{company}-{date}.html;type=text/html"
```

Extract the `id` from the response. The Google Doc URL is: `https://docs.google.com/document/d/{id}/edit`

If the upload fails (expired token, missing credentials), keep the .md file and say: "Google Doc creation failed (this is expected in standalone repo mode). The markdown report is saved at {path}."

---

## Step 10: Present Findings

> Strategic analysis complete for **{Company}**.
>
> **Top findings:**
> 1. **{Finding 1}** {with specific number}
> 2. **{Finding 2}** {with specific number}
> 3. **{Finding 3}** {with specific number}
>
> **Your ICP today:** {one-sentence data-driven description}
>
> **Where to invest:** {one-sentence prescriptive recommendation with revenue upside}
>
> **Biggest data gap:** {what's missing and what it prevents knowing}
>
> **Reports saved:**
> - Full analysis: `{portfolio_path}/deliverables/strategic-analysis-{date}.md`
> - ICP document: `{portfolio_path}/knowledge/icp-from-analysis-{date}.md`
> - Google Doc: {URL or "skipped"}

---

## Error Handling

| Scenario | Action |
|----------|--------|
| CRM auth expired (SF) | Stop. "Run `sf org login web --alias {org}`" |
| CRM auth expired (HubSpot) | Stop. "Check HubSpot MCP private app token" |
| Gong API 401/403 | Warn, switch to `--skip-calls` |
| Gong API 429 | Wait per `Retry-After`, retry |
| Fathom API failure | Try API key fallback, then manual mode |
| No deals returned | Expand `--days`, check stage mappings |
| <30 deals | Warn small sample, proceed |
| Interview files missing | Proceed with CRM + calls, note in evidence appendix |
| Python missing (ML) | Auto-set `--skip-ml`, note in data gaps |
| XGBoost install fails | Auto-set `--skip-ml`, note in data gaps |
| Field not found | Skip dimension, note in data gaps |
| HubSpot rate limit | Wait 100ms between requests, retry with backoff |
| SOQL too long | Split into multiple queries |
| Google Doc upload fails | Keep .md, warn (expected in standalone mode) |
| Segment <3 deals | Include but flag "Small sample (N={count})" |
| Free text loss reasons | Cluster via Claude, present for confirmation |
| Snowflake MCP unavailable | Disable enrichment, note in data gaps |
| Revelio match rate low | Report: "Matched X of Y accounts. Unmatched accounts won't have enrichment data." |

---

## Adding a New Company

### Salesforce Company

1. **Connect Salesforce:**
   ```bash
   sf org login web --alias my-org --set-default
   ```

2. **Add to companies.json:**
   ```json
   "{CompanyName}": {
     "crm": {
       "type": "salesforce",
       "sf_org": "my-org",
       "field_mapping": "auto"
     },
     "call_recorder": {
       "type": "gong"
     },
     "portfolio_path": "./",
     "interview_paths": {
       "knowledge_files": []
     }
   }
   ```

3. **Run:** `/strategic-analysis --company {CompanyName}` -- field discovery auto-runs on first use.

### HubSpot Company

1. **Set up HubSpot MCP** with a private app token.

2. **Add to companies.json:**
   ```json
   "{CompanyName}": {
     "crm": {
       "type": "hubspot",
       "field_mapping": "auto"
     },
     "call_recorder": {
       "type": "fathom"
     },
     "portfolio_path": "./",
     "interview_paths": {
       "churn_interviews": "",
       "market_research": "",
       "knowledge_files": [],
       "gong_exports": ""
     }
   }
   ```

3. **Run:** `/strategic-analysis --company {CompanyName}` -- field discovery auto-runs.

### Call Recorder Options

- **Gong:** Set `GONG_ACCESS_KEY` and `GONG_ACCESS_SECRET` in `~/.env`
- **Fathom:** Set `FATHOM_API_KEY` in `~/.env`
- **Manual:** Set `"call_recorder": {"type": "manual"}` -- you'll be prompted to paste transcripts

### Enrichment

To enable Revelio Labs enrichment, ensure Snowflake MCP is connected and run with `--enrich`. No per-company config needed; enrichment works by matching account names to the Revelio Labs company database.
