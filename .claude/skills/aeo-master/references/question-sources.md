# Question Mining Sources — Detailed Guide

## Table of Contents
- [Conversational AI on Sales Data](#conversational-ai-on-sales-data)
- [Reddit](#reddit)
- [Sales Calls & Gong](#sales-calls--gong)
- [Customer Support & Tickets](#customer-support--tickets)
- [Social (X, LinkedIn)](#social-x-linkedin)
- [Reviews & G2](#reviews--g2)
- [Internal Search, Chat & Zero-Result Queries](#internal-search-chat--zero-result-queries)

---

## Conversational AI on Sales Data

The highest-leverage question mining pattern available today. Instead of reading transcripts one by one, deploy an AI agent on top of all sales conversations and customer interactions. This lets you query thousands of calls at once and surface patterns a human would take weeks to find.

### The Pattern

An AI agent (OpenClaw, custom RAG pipeline, or Claude on exported transcripts) sits on top of every sales call, research conversation, and customer interaction the company has ever had. You query it like a colleague: "What objections come up most?" "What questions do prospects ask before signing?" "What closes deals?"

### Why This Changes Everything

- **Speed:** Surface 100+ qualified buyer questions in a day vs. weeks of manual transcript review
- **Completeness:** Patterns across thousands of calls, not a sample of 10-20
- **Exact language:** The agent returns real buyer phrasing, not marketing's paraphrase of what buyers say
- **Evidence-based briefs:** Every content brief can include real quotes and objections, not hypothetical ones

### What to Query

| Query | What it feeds |
|-------|--------------|
| "What objections come up most in sales calls?" | Comparison pages, fear-intent content |
| "What questions do prospects ask before signing?" | Evaluation and process content |
| "What closes deals? What proof points work?" | Outcome content, case study angles |
| "What language do buyers use to describe the problem?" | Headline and opening paragraph phrasing |
| "What do buyers compare us to? How do they frame the comparison?" | Alternatives pages, competitor content |
| "What do customers misunderstand most after buying?" | Onboarding content, expectation-setting content |
| "What questions come up in the first 30 days after purchase?" | Process content, retention content |

### Tools

- **OpenClaw** — AI agent that sits in Slack, trained on all company conversations. Query via DM.
- **Custom RAG pipelines** — Build with Claude API + vector database on exported Gong/Fathom/Chorus transcripts.
- **Claude on exported transcripts** — Simplest approach. Export last 50-100 call transcripts as text, upload to Claude, query directly. Good enough for initial mining.
- **Gong Smart Trackers** — Built-in topic and objection tracking. Less flexible than a full AI agent but zero setup.
- **Fathom API** — Pull transcripts programmatically. Combine with Claude for structured extraction.

### From Questions to Scrunch Prompts

The questions you mine from sales data should flow directly into your AI visibility tracking. Use the exact buyer phrasing as Scrunch prompt text. This creates a closed loop: real buyer questions → tracked prompts → content targeting those questions → visibility measurement on the actual queries buyers use.

### If No Conversational Data Exists

If the company has no call recording (no Gong, no Fathom, no Chorus), start with:
1. Interview 3-5 sales reps. Ask: "What are the top 5 questions you get on every call?" and "What objection kills the most deals?"
2. Sit in on 5 live sales calls and take notes on buyer language
3. Pull from CRM "lost deal" notes and demo request form fields

This is slower but still better than starting with keyword tools.

---

## Reddit

Reddit is one of the highest-signal AEO sources because:
- LLMs actively ingest Reddit as a source of real-world explanation
- Questions use natural/emotional language, not SEO language
- Same uncertainty repeats across threads even when phrasing varies

### Step 1: Choose Subreddits by Buyer Maturity

Map subreddits to buyer stage, not product category:

**Early-stage uncertainty:** r/startups, r/Entrepreneur, r/marketing, r/smallbusiness
**Mid-stage evaluation:** r/SaaS, r/webdev, r/ProductManagement, r/growthmarketing
**Late-stage, post-decision:** r/sysadmin, r/devops, r/dataengineering

Look for decision stress, not brand mentions.

### Step 2: Mine Comments, Not Posts

Post titles are optimized for attention. Comments are optimized for truth.

Highest-value questions appear as:
- Follow-ups and rebuttals
- "Wish I had known this earlier" replies
- Comments buried three levels deep

Example — Post: "Is headless CMS worth it?" (low signal). Comment: "We went headless and marketing couldn't ship without engineering for six months. How do teams actually avoid that?" (gold).

### Step 3: High-Signal Question Patterns

| Pattern | Signal | AEO Value |
|---------|--------|-----------|
| **Rage posts** | Expectation gaps AI will try to explain later | High |
| **Regret posts** | Generate most reusable AI comparison answers | Very high |
| **Comparison threads** | Heavily influence AI tradeoff framing | Very high |
| **Quiet fear questions** | Surface the real blocker | High |

Look for **emotion + uncertainty** — that's where decisions live.

### Step 4: Structure the Signal

Use workflow automation (e.g., Gumloop for native Reddit scraping) to:
- Pull posts/comments from specific subreddits
- Filter by keywords, sentiment, or post type
- Pass raw text into downstream normalization

### Step 5: Apply the AEO Filter

For each Reddit question: Is this a real decision? Would AI try to answer it? Does it shape category explanation? Could our brand be credibly cited? If no → discard. If yes → backlog. Curate aggressively.

---

## Sales Calls & Gong

Highest-leverage source most marketing teams underuse. Every sales question has: a qualified buyer, proximity to a decision, and direct revenue impact.

### Pre-Price vs. Post-Price Split

**Pre-price questions** (curiosity — about fit, capability, confidence):
- "How much setup does this usually take?"
- "What kinds of teams struggle with this?"
- "Where do people underestimate the effort?"

**Post-price questions** (objections — about justification, risk, internal alignment):
- "Why is this more expensive than X?"
- "What happens if we don't fully adopt it?"
- "How do teams justify this internally?"

AI systems mirror this structure when explaining tradeoffs. Mine both intentionally.

### Objections Disguised as Curiosity

The most valuable questions sound polite:
- "How scalable is this?" (risk probe)
- "Is this overkill for a team like ours?" (fit anxiety)
- "What do companies regret not thinking about first?" (fear signal)

If your content answers the surface question but ignores the underlying fear, the answer won't stick.

### Enterprise Questions

Low volume but high influence on how AI frames enterprise-readiness:
- "How does this break at scale?"
- "What governance models actually work?"
- "Where does this fall down in complex organizations?"

If your brand never answers these publicly, AI systems infer the answer — usually not in your favor.

### Mining at Scale

Use Gong's transcripts, objection tags, topic frequency, and stage segmentation. Layer workflow automation to extract question-shaped sentences and group by stage, deal size, or outcome.

### Don't Stop at Prospect Calls

Existing customer conversations surface implementation friction, adoption challenges, and retention risks. These predict churn and shape how AI explains post-purchase reality.

---

## Customer Support & Tickets

Post-purchase truth. Support questions come from people who already chose you and are trying to make the decision feel right.

### Expectation Gaps vs. How-To Confusion

**How-to** (fix UX/docs, not AEO-critical):
- "Where do I click to do X?"
- "How do I turn this setting on?"

**Expectation gaps** (positioning problems = AEO gold):
- "I thought this would do X automatically"
- "Why is this more manual than I expected?"
- "Do most customers run into this?"

### Onboarding Questions That Predict Churn

Watch for questions that:
- Appear in the first 30 days
- Show up repeatedly across customers
- Resurface right before churn/downgrade

Examples: "Is this the intended workflow?" / "Am I using this wrong, or is this just how it works?" / "What does a successful setup actually look like?"

These are confidence questions, not beginner questions.

### Feature Confusion = Positioning Failure

If many customers misunderstand the same thing, it's not a feature problem — it's an explanation problem. Prime AEO opportunity.

### Why Support Content Converts Well

Content from support questions converts better than TOFU because it:
- Meets buyers at a moment of clarity
- Preempts regret and builds trust through honesty
- AI systems love "here's what this does, what it doesn't, and what most people misunderstand"

---

## Social (X, LinkedIn)

Upstream signal. Social shows what people wonder about when scrolling — confusion, half-formed opinions, misconceptions forming in public.

### Where the Signal Lives

**Ignore the post. Read the comments.**

Real questions appear as pushback or clarification:
- "Wait, does this mean you can't do X?"
- "How is this different from what we already do?"
- "This sounds nice, but what breaks in practice?"

When the same question appears across multiple posts → AEO-grade signal.

### Misconception Loops

Quote tweets/reposts are dangerous: someone paraphrases slightly wrong → others react to the distorted version → AI systems borrow the wrong framing because it appears more frequently.

These loops are high-priority AEO opportunities — correct the explanation before it becomes the default.

### Founder Explanations Gone Wrong

When a founder explains something and comments fill with "Can you explain more simply?" / "Who is this actually for?" — that's a signal the explanation didn't land. Those gaps are what AI systems struggle with later.

### Extract Signal Without Chasing Virality

- Track recurring questions across multiple posts, not one
- Ignore engagement metrics initially
- Look for same question phrased differently
- Focus on "how," "why," "what does this mean in practice"
- One post = nothing. Five posts saying the same thing = everything.

---

## Reviews & G2

Post-decision clarity. Reviews are delayed questions buyers wish they had asked earlier.

### Reviews Are Questions Asked Too Late

- "Great product, but I didn't realize how much setup was involved"
- "Works well once you learn it, but there's a steep curve"
- "Powerful, but probably overkill for smaller teams"

Each maps to a buyer uncertainty that wasn't addressed before purchase.

### Highest-Signal Patterns

**Mine 3-4 star reviews** (highest signal), negative reviews that still recommend, and positive reviews with caveats.

Extract: repeated "but" statements, setup/onboarding mentions, fit/non-fit language, comparison phrasing.

### Regret Signals

Look for: "I wish I had known…" / "In hindsight…" / "If you're considering this, be aware…" / "This is great, but only if…"

These signal trust gaps (misalignment, not deception).

### Comparison Language Buyers Actually Use

Buyers say: "This feels more flexible, but slower" / "Cheaper upfront, more expensive over time" / "Better for teams, worse for solo work."

This is the phrasing AI systems reuse for pros/cons. If your comparison content doesn't reflect this language, it won't get cited.

### Third-Party Signal

Third-party sources drive ~85% of brand mentions in AI-generated answers. Offsite narrative is a primary visibility signal.

### AI Amplification of Negative Reviews

LLMs are structurally programmed to present pros *and* cons. When negative information is scarce, AI searches deeper — surfacing isolated or outdated reviews and treating them as trends rather than outliers.

**Key findings from Seer Interactive's GEO experiment (2026):**
- Review directories that previously ranked on Google pages 4-5 now appear in **38% of branded AI answers**
- A single negative review from 2018, cross-posted to multiple platforms, generated **67 mentions** of a false claim across branded prompts over 3 months
- LLMs treat identical claims on multiple review sites as **corroboration, not duplication** — one unhappy client posting the same review on Clutch, AgencySpotter, and G2 reads to AI as "multiple independent sources confirm this weakness"
- Old reviews (years old) get disproportionate weight when AI can't find enough recent negative signal

**Implications for question mining:**
- Monitor branded prompts ("Tell me about [Company]", "What's [Company]'s reputation?") to see what review-sourced claims AI surfaces
- Actively soliciting recent reviews isn't vanity — it's AI reputation management. Recent positive signal dilutes the weight of old negatives
- Check which review domains get cited in AI answers about your brand — they may be sites you've never optimized for

---

## Internal Search, Chat & Zero-Result Queries

Any place a user types a question on your site expecting an explanation: search bars, help center, in-product search, chat tools (Drift, Qualified, Intercom), demo request form fields.

### What This Signal Represents

These users are **evaluating you specifically**. They believe you might be the right solution, you should have an answer, and clarification will help them decide. This is self-reported intent at the moment of friction.

### Zero-Result Questions

Not just "No results found" — also irrelevant results, chat handoffs to sales, repeated "Can you clarify…" messages, confusion in form free-text fields.

A user expected an explanation. You didn't give one. That gap creates doubt that flows to AI systems, Reddit, or competitors.

### Chat Tools Often Beat Search Bars

Chat captures natural language, emotional subtext, urgency, and repetition across users. Examples: "Is this overkill for a team our size?" / "How long does setup really take?"

### Navigation Gap vs. Messaging Gap

- **Navigation gap:** Answer exists but users can't find it → fix UX
- **Messaging gap:** Answer doesn't exist or isn't clear → fix explanation publicly

If the same question appears across chat, sales, or support — it's messaging. If it shows up repeatedly over time — it's messaging.
