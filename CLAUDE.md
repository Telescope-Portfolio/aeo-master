# AEO Master Document Generator

This project produces a comprehensive Answer Engine Optimization (AEO) report for your company. It tells you exactly how visible your brand is across AI platforms (ChatGPT, Perplexity, Google AI, Claude, etc.), what content gaps exist, and what to build first.

## How to Run

Type `/aeo-master` in Claude Code. The skill reads your company config from `config/company.md`, scrapes your website, pulls your Scrunch AI visibility data, and produces a full report.

## Prerequisites

- **Claude Code** installed and working
- **Scrunch account** with your brand configured and prompts tracked (the skill reads your visibility data from the Scrunch API)
- **Firecrawl API key** set as the `FIRECRAWL_API_KEY` environment variable, or Firecrawl MCP configured in Claude Code (used to scrape your website pages for the audit)
- **Scrunch API key** set as the `SCRUNCH_API_TOKEN` environment variable

## Output

The report saves to `outputs/aeo-master-YYYY-MM.md`. It includes:
- AI visibility scorecard across all tracked platforms
- Full site audit with quality ratings per page
- Content gaps mapped to zero-visibility AI queries
- Priority questions your brand should own
- Content production plan with specs for each piece
- Operating cadence sized to your team
- Measurement framework with Scrunch API queries

## Configuration

Edit `config/company.md` with your company details before running. The more complete your config, the better the output. At minimum you need: company name, website URL, at least one competitor, and your Scrunch workspace ID.

## What This Doesn't Do

- No CRM or Gong integration. The skill uses publicly available data and your Scrunch monitoring data.
- No Google Doc export. Output is markdown. Copy it into whatever tool your team uses.
- No automated content creation. It tells you what to write, not writes it for you.
