# /marketstorm

Run a structured multi-agent competitive market analysis. Discovers competitors, mines their weaknesses, consolidates the feature landscape, evaluates demand, and surfaces adjacent market opportunities. Domain-agnostic — works for any product category.

---

## Pre-flight

**Token cost:** A full run with 15–25 competitors spawns one subagent per competitor — a significant token spend. Recommend the full 15–25 for the most complete picture; suggest capping at 10–12 for a faster, lighter pass. Ask the user which they prefer before starting.

If the user provided arguments (`$ARGUMENTS`), treat them as:
- A path to a context file — read it
- A brief product description — use it directly as project context

If no arguments were provided, search the current project for context files (`project-vision.md`, `docs/project-vision.md`, `BACKLOG.md`, `CLAUDE.md`). Read what you find.

---

## Step 0 — Load project context

Extract and hold onto:

- The core concept (what does this product do, in plain terms?)
- The target users
- The key values or philosophy (what does it stand for? what does it deliberately *not* do?)
- The main features planned or in scope

If any of these are missing, ask before proceeding.

---

## Step 1 — Competitor discovery

Search for competitive products in the space. Cast a wide net. Aim for 15–25 products across three categories:

- **Direct competitors** — products targeting the same users with meaningfully overlapping features or purpose
- **Partial competitors** — products that address one significant dimension of what this project does, even if their overall scope differs
- **Workflow neighbors** — tools that users of this product would likely already use, consider alongside it, or replace it with — even if they solve a different problem

For each product, record: name, URL, one-sentence description, and category. Cap at 25. Skip products that only share keywords but have no meaningful overlap with what this project does.

---

## Step 2 — Per-competitor analysis (parallel subagents)

For each discovered competitor, spawn one subagent. **All subagents must be launched in a single message as simultaneous Agent tool calls. Do not launch them sequentially.**

**Subagent brief** (customize per competitor):

> Analyze **[COMPETITOR NAME]** ([URL]) as a potential competitor to **[PROJECT NAME]**.
>
> **Project context:**
> [Insert: core concept, target users, key values, planned features]
>
> **Your tasks:**
>
> 1. **Product summary** — What does [COMPETITOR NAME] do and who is it for? (2–3 sentences)
>
> 2. **Overlap with our project** — What features, values, target users, or positioning does this product share with our project? List each as a discrete, named element (e.g., "voice input", "offline mode", "writer-owned data").
>
> 3. **Gaps** — What does our project offer (or plan to offer) that this product does not? List each as a discrete element.
>
> 4. **User complaints and unmet needs** — Search for reviews on app stores, G2, Capterra, Reddit, and relevant forums. Use primary sources — actual user reviews and forum posts, not the competitor's own marketing copy or summary articles. What do users wish it did differently? What frustrates them? What do they keep asking for? List the top 3–5 complaints as discrete items. Quote briefly where the signal is strong.
>
> 5. **Positioning notes** — Anything notable about pricing model, philosophical stance, tone, or target niche worth flagging for competitive context.
>
> Return your findings using this exact structure — the consolidation agent will parse it by field name:
>
> **Product:** [COMPETITOR NAME]
> **Overlap:** [element name]; [element name]; ...
> **Gaps:** [element name]; [element name]; ...
> **Complaints:** [complaint summary (with brief quote if the signal is strong)]; [complaint summary]; ...
> **Positioning:** [1 sentence on pricing, stance, or niche]
>
> Use concise named elements (2–5 words each) for Overlap, Gaps, and Complaints — not full sentences. The consolidation agent merges by element name.

---

## Step 3 — Element consolidation

A consolidation agent receives structured summaries of all per-competitor outputs. **Pass structured summaries — not full prose outputs.** For each competitor provide:
- Product name
- Overlap elements (list)
- Gap elements (list)
- Top complaints (list, 3–5 items)

The consolidation agent's job: merge duplicates and near-duplicates into single elements with consistent, descriptive names, then produce a unified element matrix.

**Output — a table:**

| Element | Competitors with it (X/N) | In our project? | User complaint signal |
|---|---|---|---|

Aim for 20–40 elements. If the raw list exceeds 50, merge the most closely related items — prefer fewer, more meaningful elements over an exhaustive but noisy list.

Save this matrix to `marketstorm-matrix.md` in the project root before proceeding to Step 4.

---

## Step 4 — Demand evaluation

A single agent receives the element matrix from Step 3 and handles both rubric generation and application in one pass.

Brief:
> You have a list of market elements for [PROJECT NAME] — a [brief product description]. Your job is in two parts:
>
> **Part 1 — Build a demand rubric.** Identify specific, measurable signals for assessing whether each element is in high, medium, or low demand from real users. For each signal: explain how to assess it (what to search for, where to look), define what counts as high, medium, and low, and note any reliability limits. Focus on signals measurable via web search: community discussion frequency, app store review patterns, search trends, frequency of user requests in competitor reviews. Prioritize primary sources — forum threads, review sites, direct user quotes.
>
> **Part 2 — Apply the rubric to each element.** For each element in the matrix:
> - Apply the rubric using web search
> - **For every element scored High or Low demand: cite at least one primary source** (app store review, Reddit thread, forum post, G2/Capterra review). Do not score from inference alone.
> - Assign a **demand level**: High / Medium / Low
> - Assign a **competitive saturation level**: High / Medium / Low
> - Note the key evidence (1 sentence)
>
> **Output — the element matrix extended:**
>
> | Element | Competitors (X/N) | In our project? | Complaint signal | Demand | Saturation |
> |---|---|---|---|---|---|
>
> High Demand + Low Saturation = strategic sweet spot.

---

## Step 5 — Adjacent markets

This agent thinks laterally. It receives the full element matrix with demand scores, the project context, and the user complaint patterns.

Brief:
> You have a detailed map of what [PROJECT NAME] does, what the competitive landscape looks like, and what users across this space are frustrated by. Your job is to identify 3–5 adjacent market opportunities where the *core mechanisms* of this product could address a completely different user problem in a different context.
>
> The distinction between mechanism and surface feature matters here. If a product's surface feature is "writing companion," its underlying mechanism might be "structured conversational exploration of a complex, evolving body of work." That mechanism could apply to legal case development, academic research, or product strategy. The adjacent opportunity lives at the mechanism level, not the surface level.
>
> For each opportunity:
> - **Target market and user** — who is this for?
> - **Mechanism being applied** — what does this product do, at the mechanism level, that maps to this market?
> - **Unmet need** — what problem does this user have that isn't being solved?
> - **Opportunity signal** — evidence from the complaint data, demand analysis, or gap list that this is a real opening
>
> Push past the obvious. The best adjacent opportunities feel surprising at first but obvious in retrospect. A good test: if you described the opportunity to someone in the target market, would they immediately say "yes, we need that"?

---

## Step 6 — Final report

A final agent synthesizes all findings into a structured strategic report.

**This agent's prompt must be fully self-contained. Embed the complete element matrix with demand and saturation scores, the top user complaints, and the adjacent market findings directly in the prompt. Do not spawn this agent expecting it to read the conversation — it starts cold with no prior context.**

Report structure:

```
## Competitive Analysis: [PROJECT NAME]
*[Date] — [N] competitors analyzed*

### The Landscape
[2–3 sentences: how crowded, how mature, dominant approaches or philosophies among competitors]

### High-Demand, Low-Competition Opportunities
[Elements where Demand = High and Saturation = Low or Medium — the strategic sweet spots.
List each with a brief explanation of why it's an opportunity and what the evidence is.
Tier them if meaningful differences in strength exist.]

### Key Competitive Overlaps
[Elements where many competitors are already strong.
Note any where user satisfaction with existing solutions is low — those remain opportunities even in crowded territory.]

### What Users Are Asking For (and Not Getting)
[The most recurring complaints surfaced across competitor reviews and forums.
Real demand that the market is currently failing to satisfy.]

### Adjacent Market Opportunities
[The 3–5 adjacent markets from Step 6: target user, mechanism, unmet need, opportunity signal]

### Strategic Recommendations
[3–5 concrete takeaways: what to prioritize, what to avoid, what is table stakes,
and what to watch as the market evolves]

### Appendix: Competitors Analyzed
[Table: Competitor | Category | URL]
```

Save the complete report to `marketstorm-results.md` in the project root (or current working directory if no project root is identifiable).

---

## Handoff to Namestorm

After saving the report, ask:

> "Would you like to kick off Namestorm now? The competitive landscape, adjacent market findings, and any suite or companion product concepts identified in this analysis are good naming context — Namestorm will load them automatically."

If yes, run `/namestorm` with the current project context. The Marketstorm results will be picked up automatically in Namestorm's Step 0 context load.
