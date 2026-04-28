# AEO Master Document Generator

This project produces a comprehensive Answer Engine Optimization (AEO) report for your company. It pulls your CRM deal data and call transcripts, measures your AI visibility across 8+ platforms via Scrunch, audits your website for content gaps, and produces a 16-section action plan grounded in actual win/loss data and buyer language.

## How to Run

Type `/aeo-master` in Claude Code. The skill will:
1. Run a strategic analysis pulling your CRM and call recording data (or use manual config if CRM isn't available)
2. Pull your Scrunch AI visibility baseline
3. Scrape and audit your website
4. Cross-reference everything and produce the report

## Prerequisites

- **Scrunch account** with your brand configured and prompts tracked
- **Firecrawl MCP** or API key configured in Claude Code (used to scrape your website)
- **CRM access** (Salesforce or HubSpot) for win/loss data — optional but strongly recommended
- **Call recording access** (Gong or Fathom) for buyer language — optional but strongly recommended

Set these environment variables:
```bash
export SCRUNCH_API_TOKEN="your-scrunch-api-key"
export FIRECRAWL_API_KEY="your-firecrawl-api-key"  # if using CLI, not MCP
```

## Skills Included

- **aeo-master** — The main skill. Orchestrates everything and produces the report.
- **strategic-analysis** — Pulls CRM deal data and call transcripts to produce win rates, loss reasons, and buyer language. Runs automatically as a prerequisite.

## Output

The report saves to `outputs/aeo-master-YYYY-MM.md`. It includes:
- Win/loss grounded priority rankings (not generic templates)
- AI visibility scorecard across all tracked platforms
- Full site audit with quality ratings per page
- Content gaps mapped to zero-visibility AI queries and revenue impact
- Content production plan with specs built from actual buyer language
- Operating cadence sized to your team
- Measurement framework with Scrunch API queries

## Configuration

Edit `config/company.md` with your company details before running. The more complete your config, the better the output.

If you have CRM and call recording access, the skill will pull data automatically. If you don't, fill in the "Competitive Win/Loss Data" section of the config manually with what you know from your sales team.
