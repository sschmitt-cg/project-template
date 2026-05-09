# /namestorm

Run a structured multi-agent naming pipeline for a product, app, feature, or company. Generates creative candidates, screens for availability and conflicts, evaluates each from both sides, gates, summarizes, and ranks. Designed to surface non-obvious, defensible names in crowded spaces.

---

## Pre-flight

If the user provided arguments (`$ARGUMENTS`), treat them as:
- A path to a context file — read it
- A brief product description — use it directly as project context

If no arguments were provided, search the current project for context files (`project-vision.md`, `docs/project-vision.md`, `BACKLOG.md`, `CLAUDE.md`). Read what you find.

**Before generating anything:** ask whether the user would like to run `/marketstorm` first. A current competitive analysis — particularly the landscape overview and any suite or companion product concepts it surfaces — gives the naming process meaningfully richer context. If the user says yes, run the full Marketstorm pipeline first, then return here with those findings loaded as context for Step 0.

---

## Step 0 — Load project context

Extract and hold onto:

- What the product does, in plain terms
- Who it's for
- The tone or feel (professional, playful, craft-focused, minimal, voice-first, etc.)
- **Whether this name needs to anchor a product suite or family** — if so, names must also work as a prefix for companion products (e.g., "Armature Publishing," "Armature [next app]"). This constraint meaningfully narrows the candidate space and should inform generation from the start.

If a suite is required, record it explicitly as: `suite_constraint: yes — [brief description of the naming pattern]`. This value will be inserted into Step 4 pro/con briefs and Step 7 ranking criteria.

If any of these are missing, ask before proceeding.

---

## Step 1 — Generate 20 candidate names

Generate 20 candidate names. The goal is genuine creative range — not a list of obvious category-descriptor compounds. Think across these dimensions to force variety:

- **Metaphor and abstraction** — what does using this product *feel like*, not just what does it do?
- **Action and process** — verbs and gerunds that capture the experience, not the output
- **Material and craft** — physical analogues that evoke texture, quality, making
- **Unexpected juxtaposition** — two concepts that don't usually go together but click for this product
- **Coined or portmanteau words** — invented words that sound right and carry meaning
- **Short and punchy** — one or two syllables often outperform longer names in recall and memorability

Include a mix of evocative-but-abstract, descriptive-but-distinctive, and invented words. Avoid a list where all names follow the same pattern. If the name must anchor a suite, weight names that work well as prefixes.

---

## Step 2 — Availability screening

Screen each of the 20 names inline (not a subagent task). For each name, check:

1. **Direct competitor search** — search `"[name]" app` and `"[name]" software`. Is there an active product using this name in the same or adjacent space?
2. **Domain availability** — run `whois [name].app`, `whois [name].io`, and `whois [name].com` as separate Bash commands. A domain is available if the response contains "No match for", "NOT FOUND", or equivalent. Supplement with `dig +short [name].com` if whois results are ambiguous. Record each TLD as: `available`, `registered-relevant` (an active competing product), or `registered-unrelated` (parked or unrelated business). Registered-unrelated is less disqualifying than registered-relevant.
3. **App store presence** — search `"[name]" app store`. Is this name used for a prominent app?
4. **Clarity check** — can the name be spoken clearly without ambiguity? Would it be frequently misspelled? Would it be confused with an unrelated common word or brand? **For voice-first products, this check is weighted heavily — say every name aloud.**

**A name fails screening if any of these are true:**
- An active product with the same or near-identical name exists in the same or adjacent category
- All three of `.app`, `.io`, and `.com` are taken by relevant competitors
- The name has a strong unrelated existing association that would create lasting confusion

After screening, maintain a **domain status record** for each surviving name — you'll need it in Step 4:
```
[Name]: .app [available/registered-relevant/registered-unrelated], .io [...], .com [...] — best path: [e.g., "name.app is available"]
```

Keep a brief note on each eliminated name. You'll need this for the final output.

**On trademarks:** Automated screening cannot reliably surface trademark registrations. Any name that passes screening should carry a reminder that a formal USPTO (US) or WIPO (international) search is required before public use.

---

## Step 3 — Gap-fill if needed

Count surviving names. If 10 or more survive, proceed to Step 4.

If fewer than 10 survive:
- Calculate how many new names are needed: `10 − [survivors]`
- Generate exactly that many — with explicit awareness of *why* round-one names failed. If the space is saturated with craft-metaphor words, avoid that territory. If names failed for being too generic, go more invented. Incorporate the lessons.
- Run the same availability screening on new names
- Stop after this second round regardless of final count

---

## Step 4 — Pro/con subagents (parallel, per surviving name)

For each surviving name, spawn two subagents — one pro, one con. **All subagents for all names must be launched in a single message as simultaneous Agent tool calls. Do not launch them one at a time or in batches — every pro and con agent goes in the same turn.**

**Pro subagent brief:**
> Your job is to make the strongest possible evidence-based case that "[NAME]" is the right choice for [PROJECT NAME].
>
> Project context: [insert full context from Step 0]
> Domain status: [insert from Step 2 domain status record for this name]
> Suite constraint: [insert `suite_constraint` from Step 0, or "none"]
>
> Argue for its branding strength, memorability, available domains, fit with the product concept and target users, and differentiation from competitors. If a suite constraint applies, argue for how well this name works as a prefix for companion products. Be specific. Do not invent advantages that don't exist — but make the strongest honest case you can.
>
> Keep your argument to 3–5 sentences. Return only the argument, no preamble.

**Con subagent brief:**
> Your job is to make the strongest possible evidence-based case that "[NAME]" is the wrong choice for [PROJECT NAME].
>
> Project context: [insert full context from Step 0]
> Domain status: [insert from Step 2 domain status record for this name]
> Suite constraint: [insert `suite_constraint` from Step 0, or "none"]
>
> Identify risks, weaknesses, competitive conflicts, pronunciation or spelling problems, cultural or linguistic issues, and anything that would make this name a liability. If a suite constraint applies, evaluate whether this name fails as a prefix for companion products — this is a serious weakness. Be specific. Do not invent problems that don't exist — but make the strongest honest case you can.
>
> Keep your argument to 3–5 sentences. Return only the argument, no preamble.

---

## Step 5 — Gatekeeper (per name)

This step runs inline — no subagent needed. For each name, pass the pro and con arguments to a gatekeeper with one job: **Keep** or **Cut**, with one sentence explaining the decision.

Cut names where the con argument reveals a fatal flaw — an active direct competitor, a domain situation with no viable path, or a confusion risk that can't be resolved. Keep names where weaknesses are real but manageable. When genuinely uncertain, keep — the ranker handles final prioritization.

---

## Step 6 — Balanced summarizer (per kept name)

This step runs inline — no subagent needed. For each kept name, write a 2–3 sentence balanced summary:
- What is this name's strongest quality?
- What is its main risk or limitation?
- What is the best available domain option?

Honest and useful — not promotional.

---

## Step 7 — Ranking

This step runs inline — no subagent needed. Rank all kept names from best to worst. Criteria:

- Strength of the pro argument (evidence quality, not just enthusiasm)
- Seriousness of the con argument (is the risk real or minor?)
- Domain viability (at least one clean path to a professional URL)
- **Fit with the product's stated values and tone** — a name that works against the product philosophy (e.g., a cold industrial name for a warm, personal tool) should rank lower even if technically strong on other dimensions. Weight this explicitly.
- **Voice and memorability** — easy to say aloud, easy to spell, unlikely to be confused in conversation
- Trademark risk level (flag elevated risk)
- **Suite-name viability** (if `suite_constraint` applies): does the name work as a natural prefix for companion products? A name that fails this test should rank lower even if strong on other dimensions.

For each ranked name, provide 1–2 sentences of ranking rationale — not just its position, but *why* it placed where it did relative to the names around it.

---

## Output

Present the ranked shortlist:

```
## Name Shortlist — [PROJECT NAME]

**#1 — [NAME]**
Best available domain: [domain]
[Balanced summary]
*Ranking rationale:* [1–2 sentences]

**#2 — [NAME]**
...
```

Then list all names eliminated in screening, grouped by round, with a brief reason for each.

Save the complete output to `namestorm-results.md` in the project root (or current working directory if no project root is identifiable).

Close with both of the following notices:

> **Domain note:** Whois results reflect registration status at time of screening. Verify current availability on a registrar (e.g., Namecheap, Google Domains) before committing — status can change between screening and purchase.

> **Trademark reminder:** All names above still require a formal trademark search before public use. In the US, search the USPTO database at tmsearch.uspto.gov. Internationally, search WIPO at branddb.wipo.int. Common law trademark rights can exist even without registration — a name that's clear on these databases may still carry risk if it's in active use in your market.
