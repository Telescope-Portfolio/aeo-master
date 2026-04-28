# AEO Master Document Generator

A Claude Code skill that produces a full Answer Engine Optimization report for your B2B SaaS company. You fill out a config file about your company, run one command, and get back a 16-section document that tells you exactly how visible (or invisible) your brand is across AI platforms, what content gaps are costing you pipeline, and what to publish first.

## What You Get

The output is a single markdown document with these sections:

1. **Executive Summary** with your overall visibility score and the one metric to watch
2. **Situation and Background** grounded in your company's specific context
3. **Scrunch Visibility Data** with scorecards by platform, buyer journey stage, and competitor
4. **Full Site Audit** scoring every key page on your site for AI extractability
5. **Content Gaps by Type** (educational gaps, vertical gaps, competitive gaps)
6. **Priority Questions to Own** ranked by purchase intent
7. **Content Production Plan** with word counts, opening paragraphs, and specs for each piece
8. **Answer Ownership Map** showing where each answer should live (dedicated page, blog post, FAQ, etc.)
9. **Writing Rules for Canonical Answers** so your team knows the format that gets cited
10. **Question Mining Playbook** for finding the next questions to answer
11. **Third-Party Presence Strategy** covering G2, review sites, and roundup articles
12. **Technical Foundations** (schema markup, crawl access, freshness signals, extractability)
13. **Platform Awareness** showing which AI platforms to prioritize and why
14. **Operating Cadence and Ownership** sized to your actual team (weekly, monthly, quarterly tasks)
15. **Measurement Framework** with Scrunch API queries you can run to track progress
16. **Phase Roadmap** with timelines for the next 6 months

## Prerequisites

You need three things:

- **Claude Code** installed and working on your machine
- **A Scrunch account** with your brand configured, competitors added, and prompts being tracked. The skill pulls your visibility data from the Scrunch API. If you don't have Scrunch set up yet, talk to your investor or Scrunch rep.
- **A Firecrawl API key.** Firecrawl is used to scrape your website pages for the content audit. Sign up at [firecrawl.dev](https://firecrawl.dev) and grab an API key.

## Setup (5 Minutes)

**1. Clone the repo**

```bash
git clone https://github.com/your-org/aeo-master.git
cd aeo-master
```

**2. Set your API keys**

Add these to your shell profile (`.zshrc`, `.bashrc`, etc.) or export them in your terminal:

```bash
export SCRUNCH_API_TOKEN="your-scrunch-api-key"
export FIRECRAWL_API_KEY="your-firecrawl-api-key"
```

If you use Firecrawl as an MCP server in Claude Code instead of the CLI, that works too. The skill will use whichever is available.

**3. Fill out your company config**

Open `config/company.md` and fill in the blanks. At minimum you need:
- Company name and website URL
- 2-4 competitors with one-line descriptions
- Your Scrunch workspace ID (find it in your Scrunch dashboard under Settings > API)
- 3-5 specific differentiators (not "great product" but "dedicated CSM from day one" or "94% payment recovery rate")

The optional sections (verticals, buyer objections) make the output significantly better if you fill them in.

**4. Run it**

Open the project in Claude Code and type:

```
/aeo-master
```

The skill will scrape your site, pull your Scrunch data, cross-reference the gaps, and produce the report.

## Example Output

See `examples/stay-aeo-master.md` for a real report produced for a Shopify subscription platform. It shows the level of detail you can expect.

## How Long Does It Take?

About 15-20 minutes. Most of that time is spent on the Firecrawl site scrape (auditing 8-15 pages) and the Scrunch API pulls. The analysis and document assembly happens quickly after that.

## What This Doesn't Do

A few things to know going in:

- **No CRM integration.** The skill doesn't pull from Salesforce, HubSpot, or Gong. It works with your website content and Scrunch monitoring data. If you have win/loss data or call recordings, you can add insights from those to the config file manually.
- **No automated content writing.** The output tells you what to write, with detailed specs. It doesn't write the content for you.
- **No Google Doc export.** Output is a markdown file in the `outputs/` folder. Copy it into Google Docs, Notion, or wherever your team works.
- **No historical trending.** The report is a point-in-time snapshot. Run it monthly to track progress.
- **Not a substitute for Scrunch setup.** You need prompts configured in Scrunch before this skill has data to work with. If your Scrunch workspace is empty, the report will be thin.
