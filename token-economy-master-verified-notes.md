# The Token Economy — Master Verified Notes (L1–L5)
*Single source of truth · all rates, mechanisms & feature names verified against Anthropic docs, June 2026*

This consolidates the five separately-verified levels into one document so the deck, the talking points, and the HTML demos all cite the **same** numbers. Where a figure is illustrative rather than measured, it's flagged — don't put those on a slide as fact.

---

## Verified figures at a glance
*The load-bearing numbers, in one place. If a slide disagrees with this table, the slide is wrong.*

| Claim | Verified value (Jun 2026) | Note |
|---|---|---|
| 4,500-word doc as plain text | ≈ 6,000 tokens (~4 chars/token) | Not 100,000 — that's the inflated original |
| PDF page, multimodal | ≈ 1,500–3,000 tokens/page (~2,250 mid) | Each page is rendered as an **image** |
| Native-text PDF → Markdown saving | ≈ 3–10× (scans: far more) | Don't quote a fixed multiple; demo the live number |
| Context window — Claude.ai chat | ≈ **500K** (Opus 4.6–4.8, Sonnet 4.6); else 200K | Surface-specific |
| Context window — Claude Code / API | up to **1M** (Opus 4.6+, Sonnet 4.6, Fable 5) | Plan-dependent |
| Compact proactively at | ≈ **50–60%** full | Not at the limit, where auto-compact picks for you |
| GitHub MCP server | ≈ 55,000 tokens (93 tools) | Anthropic published |
| Jira MCP | ≈ 17,000 tokens | Anthropic published; tool defs seen up to 134K pre-optimisation |
| Tool Search | GA, default-on, ≈ **85%** token reduction, ≈500-token overhead | Tool-selection accuracy 79.5% → 88.1% (Opus 4.5) |
| Pricing /M — Haiku 4.5 | $1.00 in / $5.00 out | |
| Pricing /M — Sonnet 4.6 | $3.00 in / $15.00 out | The recommended default tier |
| Pricing /M — Opus 4.x | $5.00 in / $25.00 out | **Version it** — legacy Opus was $15/$75 |
| Cache **read** | 0.1× input (the "90% off") → Haiku $0.10 / Sonnet $0.30 / Opus $0.50 | |
| Cache **write** | 1.25× input (5-min) / 2× (1-hr) → Haiku $1.25 / Sonnet $3.75 / Opus $6.25 | Caching only wins once the block is **re-read** |
| Cache minimum | 1,024 tokens (Opus 4 / Sonnet 4); **2,048** (Haiku) | Small blocks on Haiku silently fail to cache |
| Cache TTL | 5-min default (resets on each read); 1-hr option at 2× write | |
| Batch API | 50% off; stacks with caching → up to ~95% | For non-realtime jobs |
| Agent SDK guards | `maxTurns`, `max_budget_usd`, `SubagentStop`, `PreToolUse`, `permissionMode` | Shipped primitives, not patterns to build |
| Agent SDK telemetry | `total_cost_usd` + per-model token usage on every result | |
| Memory tool + context editing | ≈ 39% lift on long agentic tasks | |

**Illustrative, never quote as measured:** the original "100,000 tokens / 20× saving" (L1), "$2,000 vs $250/month" (L4), and any hard turn-count rule like "80% of window" (L2) or "3–5 turns good / 10+ = failure" (L5). These are directional, not constants.

---

## Level 1 — Document Ingestion & File Formats
**Concept · The Text Purist.** Feed the model clean text/Markdown instead of raw PDFs, because every PDF page is processed as an **image**, not just text.

**Habit that kills tokens.** Dropping raw, multi-page PDFs (especially scans or image-heavy reports) into a chat to do something a plain-text paste would do just as well.

**Why you care — chat users & business owners.** On claude.ai, PDFs get multimodal processing — Claude reads each page as text *and* renders it as an image — so a document balloons past its word count. A native-text PDF worth ~6,000 tokens of words can run several times that once imaged; scans are worse and hit the context ceiling fast. Pasting the same content as text/Markdown skips the page-images. And once a file is in the thread it's re-sent every turn — re-uploading just duplicates the cost.

**Why you care — coders & architects.** Through the API, PDFs are billed as extracted text **plus ~1,500–3,000 tokens per page** of image. That inflates payloads, adds latency, and — un-cached — multiplies cost per call. Converting to Markdown removes the per-page image charge outright.

**Example.** *Business:* summarising 10 support logs — paste raw text or convert to `.md` instead of uploading 10 PDFs (~3–10× drop for native-text; far more for scans). *Technical:* a lightweight ingestion step that parses `.docx/.pdf/.xlsx`, **OCRs scanned PDFs**, strips layout, and emits clean Markdown before the call.

**Say-it-out-loud caveats.** Mechanism is **page-as-image**, not "fonts/headers/metadata bloat." claude.ai ≠ API: in chat you hit **context + usage limits**; on the API you pay per **token**. "100,000 tokens" and "20×" are not constants — demo the live number.

*Corrected vs original: removed the 100k/20× fixed figures, fixed the cost mechanism to per-page image tokens, split claude.ai vs API, added OCR clause.*

---

## Level 2 — Thread Dynamics & Conversation Sprawl
**Concept · The Thread Architect.** Inside a single thread there's **no free recall** — by default the model re-reads the entire conversation every turn. A long thread isn't memory; it's a recurring tax, and your instructions occupy a shrinking share of the window as junk piles up.

**Habit that kills tokens.** Letting one chat sprawl across 30–50 turns — brainstorming, execution and editing crammed into one window.

**Why you care — chat users & business owners.** As the thread grows, your instructions hold a smaller slice of attention. Past a point the model loses the thread of your constraints — **context rot** (drift) — and ignores formatting, tone, requirements. It feels like Claude "forgot," but you drowned the instructions under accumulated **cruft**. Summarise and start a clean thread before that happens.

**Why you care — coders & architects.** Re-sending an ever-growing history inflates the input payload on *every* call — latency up, structured output less predictable. Prompt caching cuts the **dollar** cost ~90% but does **not** undo the attention dilution: a cached long thread still drifts.

**Example.** *Business:* split into an **Exploration** chat (thinking) and an **Execution** chat (final work); when exploration concludes, summarise it, open a **fresh** thread seeded with the summary, execute in <15 turns. *Technical:* don't maintain raw history — summarise after ~10 turns, flush, inject the summary as an anchor. In Claude Code that's **plan mode (Shift+Tab) → `/compact` → `/clear`**, with `/context` to watch usage. Compact proactively; auto-compact fires near the limit and often keeps the wrong things.

**Say-it-out-loud caveats.** "No persistent memory" is too blunt — say "no free recall **inside a thread**" (Claude.ai now has cross-chat memory). The window is surface-specific (≈500K chat / up to 1M Code & API), so "just use the big window" doesn't save you: quality degrades before the limit. Use real terms — *cruft* (not "croft"), *context rot/drift* (not "psychosis").

*Corrected vs original: fixed "croft"→cruft, dropped "psychosis"; tightened the memory framing; added window sizes + the "big window won't save you" rebuttal; named the real commands. Exploration/Execution split kept intact.*

---

## Level 3 — Tool & Workspace Hygiene
**Concept · The Workspace Hygienist.** Load only the files, connectors and plugins the immediate task needs. **Crucial 2026 update:** the platform now auto-prunes tool *definitions* (Tool Search) — but it does **not** prune your *workspace*.

**Habit that kills tokens.** Leaving connectors toggled on in a Project, stale reference files attached, and a sprawling thread running — none of which Tool Search touches.

**Why you care — chat users & business owners.** Connectors are a silent tax paid before you type: definitions load into the system prompt every session. A heavy stack is real money — GitHub MCP ≈ 55K tokens, Jira ≈ 17K — but a light setup (1–2 connectors) barely feels it. *Don't let anyone leave thinking every chat bleeds 50K.*

**Why you care — coders & architects.** Over-loaded tool context measurably hurts selection accuracy. With Tool Search, Opus 4.5 went **79.5% → 88.1%** while cutting tool token usage ~85%.

**Example.** *Business:* one clean Project per task; upload the single relevant file, not the whole shared drive. *Technical:* you no longer build a tool classifier — **Tool Search ships it** (GA, default-on). Your job is the part it doesn't automate: dropping stale files and replacing the long thread with a summary anchor.

**Say-it-out-loud caveats.** The punchline: *"The platform optimises tool definitions. Hygiene is the part you still own."* The "up to 50,000" figure cuts both ways — undersells enterprise, oversells casual users. GitHub/Jira/Tool-Search numbers are Anthropic-published; other connector values are illustrative. Anyone can run `/context` in Claude Code for their real breakdown.

*Corrected vs original: swapped the invented "50,000" framing for sourced figures; reframed the "build a dynamic tool assembly" counterpart as the now-shipped Tool Search; separated platform-automated vs your-job hygiene.*

---

## Level 4 — Stable Context Caching & Strategic Model Routing
**Concept · The Cost Controller.** Match the task to the cheapest sufficient model tier and cache static assets to claim the cache-read discount.

**Habit that kills tokens.** Defaulting every task to Opus, and re-sending unchanged reference docs on every call.

**Why you care — chat users & business owners.** Running simple work on top-tier models burns priority usage. Loading a playbook as Project Knowledge (cached) means you only spend active tokens on the variable bit you paste in.

**Why you care — coders & architects.** Cache **read** = 0.1× input (the 90% off). But cache **write** costs a premium (1.25× on the 5-min cache, 2× on the 1-hr), so caching only nets out once the static block is **re-read** enough to amortise the write — a one-shot prompt loses money.

**Example.** *Business:* sales rep's 50-page playbook loaded once as static Project Knowledge, not pasted every prompt. *Technical:* a smart routing layer — **Haiku** (route/extract) → **Sonnet 4.6** (default workhorse) → **Opus 4.x** (reasoning that justifies the premium); cache static blocks; **Batch API** (50% off) for non-realtime jobs stacks with caching toward ~95% savings.

**Say-it-out-loud caveats.** **Version the Opus number** ("Opus 4.x, $5/M" — legacy was $15/M). Routing is **three-tier**, not Haiku↔Opus binary — Sonnet is the default. Cache minimum is 1,024 tokens (Opus 4 / Sonnet 4) but **2,048 on Haiku** — small blocks there silently fail. TTL default is 5 min (resets on each read); 1-hr is the option, not the default. "$2,000 vs $250/month" is illustrative.

*Corrected vs original: versioned the Opus price; added the cache-write premium (kills the "pure upside" oversell); upgraded routing to three tiers with Sonnet default; added Batch API and the Haiku 2,048 floor.*

---

## Level 5 — Agentic Systems & Governance
**Concept · The Systems Designer.** *Thesis pivot:* the orchestration primitives now **ship** in the Claude Agent SDK. The five disciplines survive, but the work moved up the stack — from plumbing to **composition and governance**. These used to be code you wrote; now they're flags you set.

**Habit that kills tokens.** Running autonomous agents on unbounded loops — dumping whole folders into context, cycling without telemetry — and calling it "agentic."

**Why you care — chat users & business owners.** An ungoverned loop can burn a budget overnight. *Partly contained now:* on subscription plans, Agent SDK / `claude -p` usage draws from a **separate Agent SDK credit pool** (effective 15 Jun 2026); on API keys, `max_budget_usd` is your hard ceiling.

**Why you care — coders & architects.** Runaway recursion, rate-limit walls, and degradation from drowning in un-indexed context — all now addressable with shipped features rather than hand-rolled scaffolding.

**The five disciplines → what ships now.**
- *Index references* → built-in retrieval + **memory tool** & context editing (~39% lift on long agentic tasks)
- *Pre-process context* → **automatic compaction** near the limit
- *Scope each agent* → **subagents** with their own context window, tools, and model
- *Stop the loop* → **`maxTurns`** · **`max_budget_usd`** · **`SubagentStop`** hook
- *Measure the burn* → **`total_cost_usd`** + per-model token usage on every result

**Two governance moves the original missed.** A **confirmation gate** before any irreversible action (`permissionMode` / human-in-the-loop), and **least privilege** enforced deterministically by a **`PreToolUse`** hook that can *deny* a tool call — not a system-prompt request the model can drift past. Mixed-model pattern: **Opus orchestrator + Sonnet subagents**.

**Say-it-out-loud caveats.** Drop the "Nate's 5 Commandments" branding for a technical room — keep the disciplines, map each to its SDK feature. Turn-count rules ("3–5 good, 10+ = failure") are **heuristics, not constants**; the enforced limit is whatever you set in `maxTurns`. The complexity router is Level 4 material — frame it here as one node, don't re-teach it.

*Corrected vs original: pivoted from "build the framework" to "govern the shipped primitives"; mapped each discipline to its Agent SDK feature; added the confirmation-gate + least-privilege governance moves and the separate Agent SDK credit pool; demoted turn-count rules to heuristics.*

---

## Cross-level cleanup (applies to the whole deck)
- **Terminology:** *cruft* not "croft"; *context rot / drift* not "psychosis." A technical room docks invented vocabulary instantly.
- **Always version model names** when a price or window is attached (Opus 4.x, Sonnet 4.6).
- **Surface discipline:** state whether you mean **Claude.ai chat**, **Claude Code**, or the **API** — windows, billing and limits differ across all three.
- **No fake constants:** replace every hard "magic number" (80% of window, 20× saving, 3–5 turns) with the live demo figure or an explicit "rule of thumb."
- **Caching is cost relief, not a cure** — it discounts re-sent tokens, it doesn't stop context rot.

*Open decision (flagged across the series): this is now five verified levels plus five demos. The remaining leverage move is packaging "The Token Economy" as one named, owned deliverable — a talk, lead magnet, or Rathúnas workshop — rather than five chat outputs. That's the dated bet still on the table.*