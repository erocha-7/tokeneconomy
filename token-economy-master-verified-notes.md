# The Token Economy — Master Verified Notes (L1–L5)
*Single source of truth · all rates, mechanisms & feature names re-verified against Anthropic docs, **10 September 2026***

This consolidates the five separately-verified levels into one document so the deck, the talking points, and the HTML demos all cite the **same** numbers. Where a figure is illustrative rather than measured, it's flagged — don't put those on a slide as fact.

> **Sept 2026 refresh.** This document was rebuilt from the June 2026 version against live docs. The model lineup changed (Claude 5 family), **Sonnet got cheaper**, the **cache minimums all moved**, and a **new tokenizer** broke cross-generation token comparisons. Every figure below carries its source. See `CHANGELOG-2026-09.md` for a diff of what changed and why, plus the three claims from the June deck you should now drop.

**Primary sources:** [Pricing](https://platform.claude.com/docs/en/about-claude/pricing) · [Prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) · [Context windows](https://platform.claude.com/docs/en/build-with-claude/context-windows) · [Compaction](https://platform.claude.com/docs/en/build-with-claude/compaction) · [Tool search](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool) · [PDF support](https://platform.claude.com/docs/en/build-with-claude/pdf-support) · [Memory tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool) · [Agent SDK](https://code.claude.com/docs/en/agent-sdk/typescript)

---

## Verified figures at a glance
*The load-bearing numbers, in one place. If a slide disagrees with this table, the slide is wrong.*

| Claim | Verified value (10 Sep 2026) | Note |
|---|---|---|
| 4,500-word doc as plain text | ≈ 6,000 tokens on the **old** tokenizer; ≈ **7,800** on Opus 5 / Sonnet 5 | The ~4 chars/token rule of thumb is now generation-specific — see tokenizer row |
| **Tokenizer change** | Claude **4.7 and later** produce **≈30% more tokens** for the same text | Opus 4.7/4.8/5, Sonnet 5, Fable 5/5.1. Sonnet 4.6, Opus 4.6 and Haiku 4.5 use the previous tokenizer |
| PDF page, multimodal | ≈ **1,500–3,000 text tokens/page**, **plus** image tokens on top | Docs list text and image costs as two separate charges — the June deck under-counted by omitting the image half |
| Native-text PDF → Markdown saving | Large but variable (scans: far more) | Don't quote a fixed multiple; demo the live number with `count_tokens` |
| Context window — API / Claude Code | **1M** default on Opus 5, Opus 4.8/4.7/4.6, Sonnet 5, Sonnet 4.6, Fable 5/5.1 | **No beta header, no long-context premium.** Haiku 4.5 & Sonnet 4.5 = 200K |
| Max output | **128K tokens** on every 1M-context model | Streaming required at that size |
| Context window — claude.ai chat | Chat surfaces can manage the window **rolling first-in-first-out** | Docs footnote. **Don't quote a chat-specific token number** — it isn't published |
| Compact proactively at | ≈ **50–60%** full | Rule of thumb, not a platform constant |
| API compaction trigger (server-side) | default **150,000 input tokens**; min configurable **50,000** | Beta `compact-2026-01-12` |
| Compaction cost | **Not free** — an extra sampling pass, billed; itemised in `usage.iterations` | The single most-missed fact about compaction |
| Multi-server MCP stack (GitHub, Slack, Sentry, Grafana, Splunk) | ≈ **55,000 tokens** of definitions before any work | Anthropic-published. **This is a five-server figure, not GitHub alone** |
| Tool-selection degradation | Accuracy drops once you exceed **30–50 available tools** | Current docs give this threshold, not a % pair |
| Tool Search reduction | **over 85%**, loading only the **3–5 tools** needed | Server-side tool; not separately metered |
| Tool Search trigger | ≥ **10 tools**, or tool defs > **10K tokens** | Docs' own "when to use" bar |
| Tool-use system prompt overhead | Opus 5 **286** tok (`auto`) / 406 (`any`,`tool`); Sonnet 5 354/474; Haiku 4.5 496/588 | Paid whenever ≥1 tool is present |
| Pricing /M — Haiku 4.5 | $1.00 in / $5.00 out | 200K context |
| Pricing /M — **Sonnet 5** | **$2.00 in / $10.00 out** | The recommended default tier — **cheaper than Sonnet 4.6 was** |
| Pricing /M — Sonnet 4.6 | $3.00 in / $15.00 out | Previous generation, still served |
| Pricing /M — **Opus 5** | **$5.00 in / $25.00 out** | Same as Opus 4.8/4.7/4.6. **Version it** — Opus 4.1 and Opus 4 were $15/$75 |
| Pricing /M — Fable 5.1 | $10.00 in / $50.00 out | Most capable widely-released model; above Opus tier |
| Cache **read** | 0.1× input → Haiku $0.10 / Sonnet 5 $0.20 / Opus 5 $0.50 | **Fable 5.1 is the exception: 0.025× = $0.25/M (97.5% off)** |
| Cache **write** | 1.25× input (5-min) / 2× (1-hr) → Sonnet 5 $2.50 / $4.00; Opus 5 $6.25 / $10.00 | Caching only wins once the block is **re-read** |
| **Cache minimum** | **512** (Opus 5, Fable 5/5.1) · **1,024** (Sonnet 5, Sonnet 4.6, Opus 4.8) · **2,048** (Opus 4.7) · **4,096** (**Haiku 4.5**, Opus 4.6/4.5) | **All changed since June.** Haiku is now the *highest* floor, not 2,048 |
| Cache TTL | 5-min default, refreshed free on each read; 1-hr option at 2× write | Lifetime runs from **request start**, so a 4-min response eats 4 of the 5 minutes |
| Batch API | 50% off; stacks with caching | Opus 5 $2.50/$12.50 · Sonnet 5 $1.00/$5.00 · Haiku $0.50/$2.50 |
| Fast mode (Opus 5 / 4.8) | **$10 in / $50 out** — 2× price for up to 2.5× output speed | Research preview, Claude API only; no Batch, no Bedrock/Vertex/Foundry |
| US-only inference (`inference_geo:"us"`) | **1.1× on every token category** | Compliance has a line-item price |
| Effort control | `output_config.effort`: `low`/`medium`/`high`/`xhigh`/`max`, default `high` | GA. The first quality-trading cost lever after caching |
| Web search / web fetch | Search **$10 per 1,000 searches** + tokens; **fetch free** + tokens | Fetched page ≈ 2,500 tok (10 kB); research PDF ≈ 125,000 tok (500 kB) |
| Code execution | **1,550 free hours/org/month**, then **$0.05/hr/container**, 5-min minimum | Free when paired with `web_search_20260209`+ / `web_fetch_20260209`+ |
| **Managed Agents runtime** | tokens at standard rates **+ $0.08 per session-hour** (`running` only) | New billing dimension; replaces container-hour billing. No batch discount |
| Agent SDK guards | TS `maxTurns`, `maxBudgetUsd`, `permissionMode`, hooks `PreToolUse` / `SubagentStop` | Python: `max_turns`, `max_budget_usd`. **Pick one language per slide** |
| Agent SDK telemetry | `total_cost_usd` + per-model token usage on every result | `maxBudgetUsd` trips on a **client-side estimate**, not a billing-system ceiling |
| Task budgets | `output_config.task_budget`, min **20,000** tokens; beta `task-budgets-2026-03-13` | **Advisory** pacing signal so the agent lands gracefully — not an enforced cap |
| Context awareness | Sonnet 5 / 4.6 / 4.5 and Haiku 4.5 get an **injected remaining-budget warning** after each tool call | Opus 4.7+ and Fable do **not** — give them a task budget instead |
| Memory tool + context editing | **39%** lift (both) · **29%** (context editing alone) · **84%** fewer tokens in extended workflows | Anthropic internal agentic-search eval, context-management blog. Not a docs-page benchmark |

**Illustrative, never quote as measured:** "$2,000 vs $250/month" (L4), any hard turn-count rule like "80% of window" (L2) or "3–5 turns good / 10+ = failure" (L5), and per-connector token values other than the published 55K multi-server figure. These are directional, not constants.

**Dropped since June — do not say these on stage:**
1. ~~"GitHub's MCP server is 55,000 tokens (93 tools)"~~ → the 55K is a **five-server** stack. Attributing it to GitHub alone overstates one connector by ~5×.
2. ~~"Tool Search takes accuracy from 79.5% to 88.1%"~~ → that pair came from an engineering blog measured on **Opus 4.5** and is no longer in the docs. Use the **30–50 tool** degradation threshold instead.
3. ~~"Context window is 500K in chat"~~ → not a published figure. Say **1M by default on the API/Code**, and that chat surfaces roll off oldest-first.
4. ~~"Jira MCP ≈ 17,000 tokens"~~ → demote to illustrative; re-measure with `/context`.

---

## Level 1 — Document Ingestion & File Formats
**Concept · The Text Purist.** Feed the model clean text/Markdown instead of raw PDFs, because every PDF page is processed as **both text and an image**, not just text.

**Habit that kills tokens.** Dropping raw, multi-page PDFs (especially scans or image-heavy reports) into a chat to do something a plain-text paste would do just as well.

**Why you care — chat users & business owners.** On claude.ai, PDFs get multimodal processing — Claude reads each page as text *and* renders it as an image — so a document balloons past its word count. Pasting the same content as text/Markdown skips the page-images entirely. And once a file is in the thread it's re-sent every turn — re-uploading just duplicates the cost.

**Why you care — coders & architects.** Through the API a PDF is billed as extracted text — **1,500–3,000 tokens per page** by density — **plus** image tokens for the rendered page, charged under the standard vision calculation. Two charges, not one. Converting to Markdown removes the per-page image charge outright. Limits: **32 MB** per request and **600 pages** (100 when the request's context window is under 1M).

**NEW for Sept 2026 — the tokenizer changed under you.** Claude **4.7 and later** models (Opus 4.7/4.8/5, Sonnet 5, Fable 5/5.1) use a newer tokenizer that emits **≈30% more tokens for identical text**. Sonnet 4.6, Opus 4.6 and Haiku 4.5 still use the previous one. Consequences to say out loud:
- The old "4,500 words ≈ 6,000 tokens" heuristic becomes **≈7,800 tokens** on Opus 5 / Sonnet 5.
- **Token counts are no longer comparable across model generations.** A per-token price cut can be partly eaten by a token-count rise, so compare **cost per completed task**, not tokens.
- Any spreadsheet, budget, or context-budget estimate built before the switch is under-counting on the new models. Re-baseline with the token-counting endpoint.

**Example.** *Business:* summarising 10 support logs — paste raw text or convert to `.md` instead of uploading 10 PDFs. *Technical:* a lightweight ingestion step that parses `.docx/.pdf/.xlsx`, **OCRs scanned PDFs**, strips layout, and emits clean Markdown before the call.

**Say-it-out-loud caveats.** Mechanism is **page-as-text-plus-image**, not "fonts/headers/metadata bloat." claude.ai ≠ API: in chat you hit **context + usage limits**; on the API you pay per **token**. No fixed multiple for the PDF→Markdown saving — demo the live number.

*Changed vs June: added the ~30% tokenizer rise and its three consequences; corrected the per-page cost to text **plus** image rather than 1,500–3,000 all-in; added the 32 MB / 600-page limits; dropped the fixed "3–10×" saving multiple.*

---

## Level 2 — Thread Dynamics & Conversation Sprawl
**Concept · The Thread Architect.** Inside a single thread there's **no free recall** — by default the model re-reads the entire conversation every turn. A long thread isn't memory; it's a recurring tax, and your instructions occupy a shrinking share of the window as junk piles up.

**Habit that kills tokens.** Letting one chat sprawl across 30–50 turns — brainstorming, execution and editing crammed into one window.

**Why you care — chat users & business owners.** As the thread grows, your instructions hold a smaller slice of attention. Past a point the model loses the thread of your constraints — **context rot** — and ignores formatting, tone, requirements. It feels like Claude "forgot," but you drowned the instructions under accumulated noise. Note the surface difference: chat interfaces can drop the **oldest** turns first (rolling first-in-first-out), so in a long chat the material you're relying on may already be gone.

**Why you care — coders & architects.** Re-sending an ever-growing history inflates the input payload on *every* call — latency up, structured output less predictable. Prompt caching cuts the **dollar** cost ~90% but does **not** undo attention dilution: a cached long thread still drifts. Context rot is named in Anthropic's own docs — "as token count grows, accuracy and recall degrade" — so curating context matters as much as having room for it.

**Three platform features that changed this level since June:**
- **Server-side compaction** (beta `compact-2026-01-12`) summarises earlier context automatically when input crosses a threshold — **default 150,000 tokens, minimum configurable 50,000**. It is **not free**: compaction runs an extra sampling pass that is billed and itemised in `usage.iterations`. Append the whole `response.content` back — including compaction blocks — or you silently lose the state.
- **Context editing** (beta `context-management-2025-06-27`) *clears* rather than summarises: `clear_tool_uses_20250919` for old tool results, `clear_thinking_20251015` for thinking blocks.
- **Context awareness.** Sonnet 5, Sonnet 4.6, Sonnet 4.5 and Haiku 4.5 now receive an injected remaining-budget warning after each tool call, so they pace themselves. **Opus 4.7+ and Fable don't** — those need an explicit task budget instead.

**Example.** *Business:* split into an **Exploration** chat (thinking) and an **Execution** chat (final work); when exploration concludes, summarise it, open a **fresh** thread seeded with the summary, execute in <15 turns. *Technical:* don't maintain raw history — summarise, flush, inject the summary as an anchor. In Claude Code that's **plan mode (Shift+Tab) → `/compact` → `/clear`**, with `/context` to watch usage. Compact proactively; auto-compact fires near the limit and often keeps the wrong things.

**Say-it-out-loud caveats.** Say "no free recall **inside a thread**" — Claude.ai has cross-chat memory. On the "just use the big window" objection: the window is **1M by default** on Opus 5 / Sonnet 5 / Opus 4.6+ / Sonnet 4.6 with **no beta header and no long-context premium** — and that makes the rebuttal *more* important, not less: capacity got cheap, attention didn't. Cached prefixes still **occupy** the window; caching changes what you pay, not whether it counts.

*Changed vs June: replaced the unsourced "500K in chat" with the documented rolling-FIFO behaviour and the 1M API default; added compaction's real trigger threshold **and its billing cost**; added context editing and context awareness; noted that cached tokens still consume the window.*

---

## Level 3 — Tool & Workspace Hygiene
**Concept · The Workspace Hygienist.** Load only the files, connectors and plugins the immediate task needs. The platform now auto-prunes tool *definitions* (Tool Search) — but it does **not** prune your *workspace*.

**Habit that kills tokens.** Leaving connectors toggled on in a Project, stale reference files attached, and a sprawling thread running — none of which Tool Search touches.

**Why you care — chat users & business owners.** Connectors are a silent tax paid before you type: definitions load into the request every session. A heavy stack is real money — a typical five-server MCP setup (**GitHub, Slack, Sentry, Grafana, Splunk**) runs **≈55,000 tokens** of definitions before Claude does any work. A light setup (1–2 connectors) barely registers. *Don't let anyone leave thinking every chat bleeds 55K.*

**Why you care — coders & architects.** Over-loaded tool context measurably hurts selection accuracy: the docs put the degradation threshold at **more than 30–50 available tools**. Tool Search typically cuts definition tokens by **over 85%**, loading only the **3–5 tools** a request needs. Even the base overhead is now published per model: with at least one tool present you pay a tool-use system prompt of **286 tokens on Opus 5** (`auto`/`none`), 354 on Sonnet 5, 496 on Haiku 4.5 — plus tool definitions on top. The bash tool adds ~325 on Opus 5; the computer-use toolset ~4,500; browser-use ~6,600.

**The honest reframe for a 1M window.** 55K of definitions is ~5.5% of a 1M window instead of ~27% of a 200K one — so the *"you're out of room"* argument got weaker, and **the Level 3 demo now models the 1M default so the slide can't overclaim.** The *cost* argument did not weaken: those tokens are re-read and re-billed on **every turn**, so spend scales with conversation length rather than capacity, and the accuracy penalty above 30–50 tools is unaffected by window size. **Argument order on stage: dollars → accuracy → capacity.** Say the capacity part as a concession you're making, not a threat you're issuing — it lands better and it's what the numbers support.

**Example.** *Business:* one clean Project per task; upload the single relevant file, not the whole shared drive. *Technical:* you no longer build a tool classifier — **Tool Search ships it** as a server-side tool (`tool_search_tool_regex_20251119` or `..._bm25_20251119`). Reach for it at **≥10 tools or >10K tokens of definitions**; keep your 3–5 most-used tools non-deferred; up to **10,000** deferred tools per request; it is **not billed as a separate server tool**. Bonus: `defer_loading` leaves the cached prefix untouched, so **Tool Search and prompt caching compose** — deferred definitions expand inline instead of invalidating your cache.

**Say-it-out-loud caveats.** The punchline still holds: *"The platform optimises tool definitions. Hygiene is the part you still own."* Be precise about default-on: Tool Search is **on by default in Claude Code**, but on the API it's **opt-in** — you declare the search tool and set `defer_loading: true` yourself. At least one tool must stay non-deferred or you get a 400. Anyone can run `/context` in Claude Code for their real breakdown.

*Changed vs June: re-attributed the 55K figure to a five-server stack (it is not GitHub alone); replaced the retired 79.5%→88.1% accuracy pair with the documented 30–50 tool threshold; added the per-model tool-use system prompt overhead; added the 1M-window honest reframe; corrected "GA default-on" to default-on in Claude Code / opt-in on the API; added that `defer_loading` preserves caching.*

---

## Level 4 — Stable Context Caching & Strategic Model Routing
**Concept · The Cost Controller.** Match the task to the cheapest sufficient model tier, cache static assets to claim the cache-read discount, and — new this generation — **turn the effort dial down before you reach for a smaller model**.

**Habit that kills tokens.** Defaulting every task to Opus, re-sending unchanged reference docs on every call, and running everything at maximum effort.

**Why you care — chat users & business owners.** Running simple work on top-tier models burns priority usage. Loading a playbook as Project Knowledge (cached) means you only spend active tokens on the variable bit you paste in.

**Why you care — coders & architects.** Cache **read** = 0.1× input (the 90% off). But cache **write** costs a premium (1.25× on 5-min, 2× on 1-hr), so caching only nets out once the static block is **re-read** enough to amortise the write: **one read** pays back the 5-minute write, **two reads** pay back the 1-hour write. A one-shot prompt loses money.

**Routing, repriced for Sept 2026 — three tiers, and the middle one got cheaper:**
- **Haiku 4.5** — $1 / $5. Routing, classification, extraction. 200K context, old tokenizer.
- **Sonnet 5** — **$2 / $10**. The default workhorse. **Cheaper than Sonnet 4.6's $3/$15**, with a 1M window.
- **Opus 5** — $5 / $25. Reasoning that justifies the premium. 1M window.
- *(Deliberately **not** a fourth tier:* **Fable 5.1** *— $10 / $50, 1M window. Routing asks "what is the cheapest **sufficient** model?"; Fable answers a different question — the hardest long-horizon work, correctness over cost. Reach for it when Opus 5 at `max` effort has already failed, not as the top rung of a cost decision.)*

The mistake is still treating Opus as the default and everything else as a downgrade. Sonnet is the right default; you *escalate* when the task earns it.

**NEW — the effort dial is the first lever, not model choice.** `output_config.effort` takes `low`/`medium`/`high`/`xhigh`/`max` (GA, default `high`). Guidance worth repeating verbatim:
- Order of levers: **free wins first** (caching, input hygiene, batch), then **effort**, then model choice.
- **Lower effort on a newer model often beats high effort on the previous generation** — so measure "Opus 5 at `low`" before you build a cascade down to a smaller model.
- A multi-model cascade **forfeits cache reuse** — caches are model-scoped. One model, one cache namespace.
- Coding and long-horizon agentic work repay high effort (`xhigh` is Claude Code's default); chat, classification and high-volume routes often hold quality at `low`/`medium`.
- Judge **cost per completed task**, not per request. A cheaper request that needs three retries isn't cheaper.
- Changing top-level `effort` mid-conversation **invalidates the messages cache**.

**NEW — three more cost mechanics that weren't on the June slide:**
- **Mid-conversation system messages** (Opus 5, Opus 4.8, Fable 5/5.1 — **not Sonnet 5**): append `{"role":"system", ...}` to `messages[]` instead of editing the top-level `system` field, and the cached prefix survives. Editing `system` mid-conversation throws the whole cache away.
- **Fast mode** (Opus 5 / 4.8, research preview, first-party API only): up to **2.5× output speed at 2× the price** — $10/$50. Its own rate limit; switching speed invalidates the cache; not available with Batch.
- **`inference_geo: "us"`** puts a **1.1× multiplier on every token category**. If you're pinning inference to the US for compliance, that's a 10% line item — name it in the budget.

**Example.** *Business:* sales rep's 50-page playbook loaded once as static Project Knowledge, not pasted every prompt. *Technical:* route **Haiku** (extract/classify) → **Sonnet 5** (default workhorse) → **Opus 5** (hard reasoning); cache static blocks; **Batch API** (50% off) for non-realtime jobs stacks with caching.

**Say-it-out-loud caveats.** **Version the Opus number** ("Opus 5, $5/M" — Opus 4.1 and Opus 4 were $15/$75). **The cache minimums all moved and Haiku is now the worst offender: 4,096 tokens on Haiku 4.5**, 1,024 on Sonnet 5, and just **512 on Opus 5** — small blocks below the floor silently fail to cache, with no error. **Fable 5.1 breaks the "90% off" line**: its cache reads are 0.025× ($0.25/M), i.e. 97.5% off. TTL default is 5 min and is measured **from request start**, so a 4-minute streamed response leaves you ~1 minute to land the follow-up. "$2,000 vs $250/month" remains illustrative.

*Changed vs June: Sonnet 4.6 $3/$15 → **Sonnet 5 $2/$10**; Opus 4.x → Opus 5 (price unchanged); **all cache minimums corrected** (Haiku 2,048 → 4,096; Opus 1,024 → 512); added the Fable 5.1 cache-read exception; added the effort dial as the primary lever with the cascade/cache-namespace warning; added mid-conversation system messages, fast mode and the `inference_geo` multiplier; added the TTL-from-request-start subtlety and the 1-read/2-read payback rule.*

---

## Level 5 — Agentic Systems & Governance
**Concept · The Systems Designer.** *Thesis holds:* the orchestration primitives **ship**. The five disciplines survive, but the work is **composition and governance** — flags you set, not plumbing you build. What's new since June is that agents now have **their own billing dimension**.

**Habit that kills tokens.** Running autonomous agents on unbounded loops — dumping whole folders into context, cycling without telemetry — and calling it "agentic."

**Why you care — chat users & business owners.** An ungoverned loop can burn a budget overnight. Set a ceiling before you set it running.

**Why you care — coders & architects.** Runaway recursion, rate-limit walls, and degradation from drowning in un-indexed context — all now addressable with shipped features rather than hand-rolled scaffolding.

**The five disciplines → what ships now.**
- *Index references* → built-in retrieval + the **memory tool** (`memory_20250818`, client-side, all Claude 4+ models) & context editing — **39%** lift with both, **29%** from context editing alone, and **84%** fewer tokens in extended workflows (Anthropic internal agentic-search eval)
- *Pre-process context* → **server-side compaction** (beta, default trigger 150K input tokens) — remembering it costs an extra billed sampling pass
- *Scope each agent* → **subagents** with their own context window, tools, and model
- *Stop the loop* → **`maxTurns`** · **`maxBudgetUsd`** · **`SubagentStop`** hook
- *Measure the burn* → **`total_cost_usd`** + per-model token usage on every result

**Get the flag names right — they're language-specific.** TypeScript: `maxTurns`, `maxBudgetUsd`, `permissionMode` (`'default' | 'plan' | 'bypassPermissions' | 'auto'`), `hooks` keyed by `PreToolUse` / `SubagentStop`. Python: `max_turns`, `max_budget_usd`. Mixing camelCase and snake_case on one slide is the kind of thing a technical room catches. And be honest about what the budget cap is: **`maxBudgetUsd` stops the query when the SDK's own client-side cost estimate crosses your number** — it's compared against the same estimate as `total_cost_usd`, with documented accuracy caveats. It is a guardrail, not a billing-system hard stop.

**NEW — budgets now come in two flavours, and only one is enforced:**
- **Task budgets** (beta `task-budgets-2026-03-13`; Opus 5, Opus 4.8/4.7, Sonnet 5, Fable 5/5.1) — `output_config.task_budget = {type: "tokens", total: N}`, minimum **20,000**. The server injects a countdown the model can see, so it **paces itself and lands gracefully** instead of being guillotined by `max_tokens`. **Advisory.**
- **Managed Agents session budgets** — hard, **dollar-denominated, platform-enforced** caps on a single session. **Enforced.**

Different tools for different fears: task budgets stop *bad endings*, session budgets stop *bad bills*.

**NEW — Managed Agents adds a second meter.** If Anthropic runs the loop and hosts the sandbox, you pay **tokens at standard rates plus $0.08 per session-hour**, metered only while the session status is `running` — `idle`, `rescheduling` and `terminated` time is free. Session runtime **replaces** code-execution container-hour billing; you aren't charged both. The **Batch discount does not apply** (sessions are stateful and interactive). For the self-hosted path, the code-execution tool gives every org **1,550 free container-hours a month**, then $0.05/hour with a 5-minute minimum per execution — and it's **free entirely** when paired with the current web search / web fetch tools.

**Two governance moves the original missed.** A **confirmation gate** before any irreversible action (`permissionMode` / human-in-the-loop), and **least privilege** enforced deterministically by a **`PreToolUse`** hook that can *deny* a tool call — not a system-prompt request the model can drift past. Mixed-model pattern: **Opus 5 orchestrator + Sonnet 5 subagents** (but see the cache-namespace caveat in L4 — a mixed roster forfeits cache reuse across models, so it has to earn the split).

**Say-it-out-loud caveats.** Drop the "Nate's 5 Commandments" branding for a technical room — keep the disciplines, map each to its SDK feature. Turn-count rules ("3–5 good, 10+ = failure") are **heuristics, not constants**; the enforced limit is whatever you set in `maxTurns`. The complexity router is Level 4 material — frame it here as one node. **The 39% / 29% / 84% figures come from an Anthropic blog post's internal eval, not a docs benchmark page — cite them as "Anthropic's internal agentic-search eval," not as a published standard.** ⚠️ **The June deck's claim about a separate Agent SDK credit pool for subscription plans (dated 15 Jun 2026) could not be re-verified in the current docs — check your own plan's billing page before repeating it on stage.**

*Changed vs June: fixed `max_budget_usd` → `maxBudgetUsd` for the TypeScript SDK and split the flag names by language; added the honest framing that the budget cap trips on a client-side estimate; added task budgets vs Managed Agents session budgets (advisory vs enforced); added Managed Agents' $0.08/session-hour meter and the code-execution free tier; sourced the 39% figure precisely and added the 29%/84% companions; flagged the unverified Agent SDK credit-pool claim.*

---

## Cross-level cleanup (applies to the whole deck)
- **Terminology:** *context rot* is Anthropic's own term and appears in the docs — use it. Avoid invented vocabulary; a technical room docks it instantly.
- **Always version model names** when a price or window is attached (Opus 5, Sonnet 5, Haiku 4.5) — and note that Opus 5 and Opus 4.8/4.7/4.6 all sit at $5/$25, so "Opus is expensive" needs a version to mean anything.
- **Surface discipline:** state whether you mean **Claude.ai chat**, **Claude Code**, or the **API** — windows, billing and limits differ across all three.
- **Generation discipline (new):** with the 4.7+ tokenizer emitting ~30% more tokens, always say which model a token count came from. Cross-generation token comparisons are now invalid by default.
- **No fake constants:** replace every hard "magic number" (80% of window, 20× saving, 3–5 turns) with the live demo figure or an explicit "rule of thumb."
- **Caching is cost relief, not a cure** — it discounts re-sent tokens; it doesn't stop context rot, and cached tokens still occupy the window.
- **Cheaper per token ≠ cheaper per job.** Between the tokenizer change, effort levels, and retry loops, the only honest unit is **cost per completed task**.

*Open decision (still on the table): this is five verified levels plus five demos. The remaining leverage move is packaging "The Token Economy" as one named, owned deliverable — a talk, lead magnet, or workshop — rather than five chat outputs.*
