# Citation Efficiency by Page Type

Source: Trakkr Study 006 — "AI Crawls Your Product Pages. It Cites Your Blog." (March 2026)
Dataset: 337K AI citations, 11.4M crawler visits, 882 brands, 58% of URLs classified into 13 page types.

## The Core Metric

**Citation efficiency** = how well a page type converts crawler attention into actual AI citations.
- **1.0x** = cited in proportion to crawl traffic
- **Above 1.0x** = content outperforms (cited more than crawled)
- **Below 1.0x** = AI reads the pages but doesn't recommend them

## The Full Matrix

| Page Type | Crawl Share | Citation Share | Citation Efficiency |
|-----------|------------|----------------|-------------------|
| Review / Directory | 0.3% | 1.2% | **4.8x** |
| About / Contact | 0.4% | 1.7% | **4.6x** |
| Resource / Report | 0.4% | 1.4% | **3.8x** |
| Service / Use Case | 1.4% | 2.6% | **1.9x** |
| Blog / Editorial | 14.4% | 20.2% | **1.4x** |
| Documentation | 0.7% | 0.8% | 1.1x |
| Integration | 0.4% | 0.4% | 1.0x |
| Homepage | 10.7% | 7.9% | 0.7x |
| Legal | 0.4% | 0.2% | 0.6x |
| Comparison | 0.4% | 0.2% | 0.6x |
| Product Pages | 13.0% | 6.3% | **0.5x** |
| Category / Browse | 1.9% | 0.9% | 0.5x |
| Help / FAQ | 1.9% | 0.4% | **0.2x** |

The five overperformers share a trait: they provide context, opinion, or authority rather than product information. AI uses them as evidence for recommendations, not as the recommendations themselves.

## The Big Three

### Blog / Editorial (1.4x — "The Workhorse")
- One in five AI citations (20.2%) lands on a blog post — no other page type comes close
- Highest repeat rate: once AI cites a post, it averages 8.0 different queries
- Authoritative, data-rich editorial earns both first citations and repeat appearances

### Product Pages (0.49x — "The Laggard")
- AI crawlers are obsessed with catalogs: 13% of all visits go to product pages
- Only 6.3% of citations point there — read twice as often as recommended
- Product pages need expert context, use-case framing, and structured data to close the gap

### Help / FAQ (0.21x — "The Ghost")
- Lowest citation efficiency of any page type
- 1.9% of crawl traffic but just 0.4% of citations
- AI reads FAQ for context but never surfaces it as an answer
- Use FAQ for UX and support cost reduction, not AI visibility

## Outlier Page Types

- **Review / Directory (4.8x):** G2, Capterra, Trustpilot — almost never crawled but frequently cited as authority signals
- **About / Contact (4.6x):** AI cites "who you are" far more than expected — a key AI trust signal, not just a human-facing page
- **Resource / Report (3.8x):** Whitepapers, templates, research reports earn citations far beyond crawl share
- **Service / Use Case (1.9x):** Pages that frame the product around what it solves — not what it does — get cited nearly 2x more than crawled
- **Comparison (0.6x):** Surprisingly low despite high SEO value — AI engines may avoid brand-created comparisons, preferring third-party sources for impartiality

## Crawler Personalities

Each AI crawler has a distinct reading style. "AI-optimized" is not one strategy — it's four.

| Crawler | Total Visits | Reading Pattern |
|---------|-------------|-----------------|
| **GPTBot (OpenAI)** | 461K | Product-page scanner: 84% product pages, 12% category, 3% other |
| **Meta AI** | 77K | Pure product crawler: 96% product pages |
| **OpenAI Search** | 21K | Lighter product focus: 77% product pages, 17% other, 4% blog |
| **ClaudeBot (Anthropic)** | 2K | Most diverse reader: 62% other, 16% product, 10% homepage |

- Optimizing for ChatGPT means product page structure
- Optimizing for Claude means content diversity
- There is no single "AI-optimized" strategy — only platform-specific ones

## Owned vs. Third-Party Citations

- Only **1.8%** of AI citations point to your own domain; **98.2%** cite third-party sources
- When AI cites your content, sentiment averages **76/100** — a +18 point lift vs. third-party citations (58/100)
- Wikipedia alone captures **9.6%** of all third-party citations
- Your homepage is the stickiest citation: once AI starts citing it, it averages **35x repeat appearances**

### What AI cites when it cites your domain:
| Page Type | Share of Owned Citations |
|-----------|------------------------|
| Blog / Editorial | 19.3% |
| Product Pages | 7.9% |
| Homepage | 6.6% |
| Service / Use Case | 3.7% |
| About / Contact | 2.8% |

## Methodology Notes

- **Citation data:** 337,362 unique URL-brand citation pairs from AI-generated responses across 882 brands (Trakkr citation snapshot pipeline, multiple AI providers)
- **Classification:** Domain-aware URL classifier (v3) using path pattern matching + domain overrides. 58% classified into 13 page types; 42% remain "other"
- **Normalization:** Citation efficiency scores are brand-normalized so each brand contributes equally regardless of volume
- **Limitations:** 42% "other" classification; snapshot-based (reflects point-in-time, not permanent rankings)
