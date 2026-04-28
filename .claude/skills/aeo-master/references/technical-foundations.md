# Technical Foundations for AI Visibility

## Table of Contents
- [Schema Markup Implementation](#schema-markup-implementation)
- [AI Crawler Access](#ai-crawler-access)
- [Content Structure for AI Citation](#content-structure-for-ai-citation)
- [Entity Signals](#entity-signals)
- [Platform-Specific Technical Notes](#platform-specific-technical-notes)

---

## Schema Markup Implementation

Structured data shows a +73% selection rate for AI citations. Use JSON-LD format — it separates structured data from HTML, making it cleaner for AI systems to parse.

### Essential Schema Types for B2B SaaS

**Organization (every site):**
```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Company Name",
  "legalName": "Company Legal Name Inc.",
  "foundingDate": "2020-01-01",
  "url": "https://company.com",
  "logo": "https://company.com/logo.png",
  "sameAs": [
    "https://linkedin.com/company/name",
    "https://g2.com/products/name",
    "https://crunchbase.com/organization/name",
    "https://twitter.com/name"
  ],
  "description": "One-sentence product description with category"
}
```

**Product (product pages):**
```json
{
  "@context": "https://schema.org",
  "@type": "SoftwareApplication",
  "name": "Product Name",
  "applicationCategory": "BusinessApplication",
  "operatingSystem": "Web",
  "offers": {
    "@type": "Offer",
    "price": "99",
    "priceCurrency": "USD"
  },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.5",
    "reviewCount": "250",
    "bestRating": "5"
  }
}
```

**FAQ (any page with Q&A content):**

FAQ content with proper FAQPage schema achieves a 71% citation rate vs. 58% without (Stacked GTM, Webflow case study, 2026). This makes FAQPage the highest-leverage schema type for citation lift.

```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [{
    "@type": "Question",
    "name": "Question text here?",
    "acceptedAnswer": {
      "@type": "Answer",
      "text": "Direct answer text here."
    }
  }]
}
```

**Article (blog posts, thought leadership):**
Include `author`, `datePublished`, `dateModified`, `publisher`, and `headline`.

**BreadcrumbList (site-wide):**
Helps AI models understand site hierarchy and content relationships.

### Validation

Test schema using Google's Rich Results Test or Schema.org validator. Broken or invalid schema is worse than no schema — AI systems may misparse it.

---

## AI Crawler Access

### robots.txt Configuration

Ensure these crawlers are NOT blocked:

| Crawler | Platform | User-Agent |
|---------|----------|------------|
| GPTBot | ChatGPT/OpenAI | `GPTBot` |
| ClaudeBot | Claude/Anthropic | `ClaudeBot` |
| PerplexityBot | Perplexity | `PerplexityBot` |
| Google-Extended | Gemini/Google AI | `Google-Extended` |
| Bingbot | ChatGPT Search (via Bing) | `Bingbot` |

Example `robots.txt` allowing all AI crawlers:
```
User-agent: GPTBot
Allow: /

User-agent: ClaudeBot
Allow: /

User-agent: PerplexityBot
Allow: /

User-agent: Google-Extended
Allow: /
```

**Common mistake:** Many sites block all unknown bots by default. Check for blanket `Disallow: /` rules that catch AI crawlers.

### llms.txt

A proposed standard (not adopted by any major AI platform) that provides AI-specific crawling guidance. A 300K-domain study (SE Ranking, 2026) and Search Engine Journal analysis found no statistically significant correlation between having llms.txt and receiving AI citations. Google's John Mueller stated no AI system currently uses it. Low-cost to add (~30 min) but has no proven impact. Do not prioritize over schema markup, entity signals, content structure, or brand presence.

### JavaScript Rendering

AI crawlers often struggle with heavy client-side rendering (SPAs, React apps without SSR). If the site relies on JavaScript to render content, ensure server-side rendering (SSR) or static generation (SSG) is in place for key pages.

---

## Content Structure for AI Citation

### How AI Retrieves Content (The Passage-Level Shift)

Google AI Mode doesn't rank pages — it dissects content into individual passages and retrieves them independently. Key confirmed mechanisms (iPullRank, via Google public documentation):

- **Query fan-out:** One user query expands into dozens of synthetic sub-queries. Your content competes at the passage level across queries you'll never see tracked.
- **Passage-level retrieval:** Individual paragraphs are retrieved, compared, and ranked — not whole pages.
- **Reasoning chains:** The system constructs logical inference steps, selecting passages that support specific reasoning checkpoints.

This means every section of your content needs to work as a standalone answer, not just as part of a larger narrative.

### Heading Hierarchy

Pages with clean H1 → H2 → H3 hierarchy are more easily parsed for passage-level extraction. This is a widely accepted best practice across the industry.

- One H1 per page (the page title)
- H2s for major sections (these are what AI systems use as passage anchors)
- H3s for subsections within H2s
- Don't skip levels (H1 → H3 with no H2)

### Answer-First Writing

Write 40-60 word direct answers immediately under H2 headings. AI systems extract these as self-contained snippets.

Growth Memo's analysis of 1.2M ChatGPT responses found that 44.2% of citations come from the first 30% of a document (the "ski ramp" pattern). **Caveat:** This research is ChatGPT-only — citation patterns may differ on Google AI Mode, Claude, and Perplexity.

**Bad:**
```
## Our Pricing Model
At Company, we believe in transparent and fair pricing that scales
with your business. We've designed our plans to accommodate teams
of all sizes. Here's how it works...
[pricing details buried 200 words later]
```

**Good:**
```
## Pricing
Company starts at $99/month for teams up to 10 users, scaling to
$299/month for unlimited users. All plans include core features —
the difference is user count and support tier.
```

### Self-Contained Sections

Each H2 section should make sense extracted on its own. This is the single highest-confidence recommendation based on confirmed passage-level retrieval in Google AI Mode (iPullRank) and the ChatGPT citation research (Growth Memo). Don't rely on context from earlier sections to understand later ones.

### Entity Density

Cited ChatGPT content averages 20.6% entity density (named tools, competitors, people, brands) versus 5-8% in standard English text. Name specific things rather than writing generically — "Salesforce and HubSpot" beats "leading CRM platforms."

**Caveat:** This is correlational, not causal, and based on ChatGPT data only (Growth Memo, 1.2M responses). Still, it aligns with the broader principle that specific, verifiable claims perform better than vague ones across all platforms.

### Citation Triggers (Princeton GEO Study)

Three strategies consistently boost AI visibility by 30-40%:

1. **Cite sources** — Reference specific studies, reports, or data sources
2. **Add quotations** — Include expert quotes with attribution
3. **Include statistics** — Use specific numbers, percentages, and data points

Content with all three performs dramatically better than content with none. (This is one of the better-sourced claims in the AEO space.)

### Freshness Signals

- Add "Last updated: [date]" to cornerstone content — pages with a visible "Last Updated" timestamp see citation rates move from 42% to 61% (Stacked GTM, 2026). Pages updated within 3 months are cited 2x more often
- Include the current year in titles and headers where relevant
- Refresh cornerstone content quarterly — fresh content is favored, especially by Perplexity (real-time search) and ChatGPT (search mode)
- Named statistics (with source and year) improve AI visibility by 41% — this compounds with the Princeton GEO findings on citation triggers

---

## Entity Signals

### The Entity Home (About Page)

The company's About page serves as the primary entity definition for AI models. It should explicitly state:
- What the company does (product category)
- Who it serves (target market, company size, role)
- Key differentiators (2-3 specific things, not generic claims)
- Founding date and headquarters
- Team/leadership with linked profiles

### Cross-Platform Consistency

AI models cross-reference entity information across sources. Inconsistencies create confusion and reduce citation confidence.

Ensure identical information on:
- Company website (About page)
- G2 profile
- Crunchbase
- LinkedIn company page
- Clutch / industry directories
- Wikipedia (if applicable)

Key fields to keep consistent: company name, founding date, headquarters, product category, target market description.

### sameAs Properties

The Organization schema `sameAs` property explicitly links your entity across platforms. This helps AI models confirm "this is the same company" when they see mentions on different sites.

---

## Platform-Specific Technical Notes

### Perplexity
- Real-time web search for every query
- Most responsive to fresh content — new pages can get cited within days
- Clean HTML structure matters more than domain authority
- FAQ and comparison formats perform particularly well
- **Sourcing durability caveat:** Perplexity changes sourcing methodology periodically — a page cited immediately after publication may lose citations weeks later without the underlying content changing. Seer Interactive's GEO experiment (2026) found their correction article was cited immediately, but citations dropped to zero within weeks when Perplexity shifted methodology. Build centralized reference pages rather than relying on single blog posts for reputation correction

### ChatGPT
- Hybrid: training data (parametric knowledge) + Bing search when search is active
- Training data lags months behind — current content affects search mode, not base knowledge
- Bing SEO signals matter when ChatGPT uses search
- Brand authority and recognition from training data heavily influence base recommendations
- Growth Memo's citation research (1.2M responses) found front-loaded, entity-rich, definition-heavy content gets cited more — correlational, not causal

### Google AI Mode
- **This is the biggest platform shift.** AI Mode fundamentally differs from traditional Google search (iPullRank analysis, confirmed via Google public documentation):
  - **Query fan-out:** Expands one query into dozens of synthetic sub-queries
  - **Passage-level retrieval:** Retrieves individual passages, not whole pages
  - **Personalization:** User behavioral profiles shape results (inferred from Google patents — medium confidence)
  - **Pairwise ranking:** Passages may compete head-to-head via LLM comparison (inferred from Google patents — medium confidence)
- Ranking #1 for a core query only gives ~25% chance of appearing in AI Mode (iPullRank/ZipTie data)
- AI Overviews have cut organic clicks by ~34.5% (Ahrefs data cited by iPullRank)
- Traditional SEO is necessary but not sufficient — content must be modular and passage-ready

### Google AI Overviews (Legacy)
- Pulls from Google's existing index
- Traditional SEO signals (domain authority, backlinks, page speed) still matter
- Featured snippet content is often reused
- Being absorbed into AI Mode over time

### Claude
- Primarily relies on training data with limited web search
- Highest authority bar — strongly weights Tier 1 sources (65%)
- Most conservative about citations
- Being mentioned in authoritative publications matters more than on-page optimization

### Gemini
- Uses Google's index plus broader web signals
- Directory presence and review site listings weighted more than other platforms
- Balanced approach between real-time retrieval and authority signals

### Crawler Personalities (Trakkr Study 006, March 2026)

Each AI crawler reads your site differently. Across 11.4M tracked crawler visits and 882 brands:

| Crawler | Total Visits | Reading Pattern |
|---------|-------------|-----------------|
| **GPTBot (OpenAI)** | 461K | Product-page scanner: 84% product pages, 12% category, 3% other |
| **Meta AI** | 77K | Pure product crawler: 96% product pages |
| **OpenAI Search** | 21K | Lighter product focus: 77% product pages, 17% other, 4% blog |
| **ClaudeBot (Anthropic)** | 2K | Most diverse reader: 62% other, 16% product, 10% homepage |

Key implication: GPTBot and Meta AI are obsessed with product pages — but product pages have only 0.49x citation efficiency (read twice as often as recommended). ClaudeBot is the only crawler that reads broadly across content types. This means:

- **For ChatGPT visibility:** Focus on making product pages citable — add expert context, use-case framing, structured data. GPTBot already reads them; the gap is making them recommendation-worthy
- **For Claude visibility:** Invest in content diversity — blog, about, use-case pages. ClaudeBot rewards breadth
- **For all platforms:** Blog content (1.4x efficiency) and use-case pages (1.9x) consistently outperform product pages regardless of which crawler reads them

### Cross-Platform Caveat

Citation patterns vary significantly across platforms. Most published research is ChatGPT-specific (Growth Memo) or inference-based (iPullRank's Google patent analysis). SparkToro's research (2,961 responses across ChatGPT, Claude, and Google AI) confirmed that AI responses are wildly inconsistent — same prompt rarely produces the same brand list. Track visibility on each platform separately rather than assuming one set of optimizations works everywhere.
