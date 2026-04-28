# AEO Master Document Generator

A Claude Code skill that produces a full Answer Engine Optimization report for your B2B SaaS company. It pulls your CRM deal data, mines your call recordings for buyer language, measures your AI visibility across 8+ platforms, audits your website, and produces a 16-section action plan where every priority ranking is grounded in actual revenue data.

## What You Get

The output is a single markdown document with these sections:

1. **Executive Summary** with your overall visibility score and the one metric to watch
2. **Situation and Background** grounded in your company's specific context
3. **Scrunch Visibility Data** with scorecards by platform, buyer journey stage, and competitor
4. **Full Site Audit** scoring every key page on your site for AI extractability
5. **Content Gaps by Type** (educational gaps, vertical gaps, competitive gaps)
6. **Priority Questions to Own** ranked by revenue impact from your CRM data
7. **Content Production Plan** with word counts, opening paragraphs built from actual buyer language, and honest concession requirements
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

**Required:**
- **Claude Code** installed and working on your machine
- **A Scrunch account** with your brand configured, competitors added, and prompts being tracked. If you don't have Scrunch set up yet, talk to your investor or Scrunch rep.
- **A Firecrawl API key** for website scraping. Sign up at [firecrawl.dev](https://firecrawl.dev).

**Strongly recommended:**
- **CRM access** (Salesforce or HubSpot) so the skill can pull win rates, loss reasons, and deal data automatically
- **Call recording access** (Gong or Fathom) so the skill can mine buyer language from your sales calls

Without CRM and call recording data, the report still works (site audit + Scrunch data), but the priority rankings and content specs will be based on generic buyer question templates instead of your actual deal data. The difference is significant.

## Setup

See **[docs/setup-guide.md](docs/setup-guide.md)** for the full step-by-step walkthrough. It covers everything from installing Claude Code to configuring your CRM and call recording integrations.

**Quick version if you already have Claude Code running:**

```bash
git clone https://github.com/Telescope-Portfolio/aeo-master.git
cd aeo-master
```

Fill out `config/company.md`, then open the project in Claude Code and run `/aeo-master`.

## Example Output

See `examples/stay-aeo-master.md` for a real report produced for a Shopify subscription platform. It shows the level of detail you can expect when CRM and call recording data is available.

## How Long Does It Take?

15-20 minutes without CRM integration (Scrunch + site audit only). 30-45 minutes with CRM and call recording integration (the strategic analysis adds time for CRM queries and transcript mining). Most of the time is the Firecrawl site scrape and the strategic analysis data pull.

## What This Doesn't Do

- **No automated content writing.** The output tells you what to write, with detailed specs. It doesn't write the content for you.
- **No Google Doc export.** Output is a markdown file in the `outputs/` folder. Copy it into Google Docs, Notion, or wherever your team works.
- **No historical trending.** The report is a point-in-time snapshot. Run it monthly to track progress.
- **Not a substitute for Scrunch setup.** You need prompts configured in Scrunch before this skill has data to work with.
