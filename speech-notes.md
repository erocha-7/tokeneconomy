# The Token Economy — Speech Notes
*Five Levels · Balanced for chat users and builders · **re-verified Sep 2026***

> **Before you present:** read `CHANGELOG-2026-09.md`. Nine figures moved since the June run and four claims were cut. The riskiest habit is muscle memory — the June numbers are still in your head.

---

## Opening (before clicking any tab)

> "Tokens are the currency of AI. Every word you send, every file you upload, every tool you connect — it all converts to tokens. The model never reads your message for free. And most of the waste isn't obvious. That's what these five levels are about."

Set the frame: this isn't about cutting corners. It's about understanding where your budget goes so you can spend it on the work that matters.

**Optional opener if you presented this before:** "I gave this talk in June. Nine of my numbers have since changed, one of them by 5×, and a new tokenizer quietly made every token estimate in the room about 30% too low. That's the actual lesson: the economy moves."

---

## Tab 1 · Ingestion
*The Text Purist · ~3 min*

The core idea: **PDFs are expensive because Claude reads each page as text *and* as an image.**

Two charges per page, not one — the docs list them separately. Text runs **1,500–3,000 tokens per page** depending on density, and the rendered page image is billed **on top** under the standard vision calculation. For scans, worse.

**The habit that kills tokens:** dragging PDFs into a chat when a plain-text paste would do the same job. And once a file is in the thread, it's re-sent every turn.

**The fix is boring and effective:** convert to Markdown before the call. For teams building with the API, add an ingestion step that extracts, OCRs scans, and strips layout first.

### NEW — the tokenizer tax (this is your best new material)

> "Before we go further, a correction to my own talk. Claude 4.7 and later — that's Opus 4.7, 4.8, Opus 5, Sonnet 5 — use a new tokenizer. It produces about **30% more tokens for exactly the same text**. Sonnet 4.6 and Haiku 4.5 still use the old one."

Land the three consequences:
1. "A 4,500-word document used to be about 6,000 tokens. On Opus 5 it's closer to **7,800**."
2. "Which means token counts **aren't comparable across model generations any more**. If you benchmark Haiku against Opus 5 on the same input, you're not even counting the same way."
3. "So the only honest unit left is **cost per completed task**. Not tokens. Not price per million."

> *"If you built a token budget before this year, it's under-counting. Re-baseline it."*

---

## Tab 2 · Threads
*The Thread Architect · ~6 min · this is the core insight*

### Set it up

> "There's no memory inside a thread. Every time you hit send, the model re-reads the entire conversation from the beginning. A long thread isn't a history — it's a recurring bill."

Move the slider to around 12 turns and let the numbers land.

### Walk through the demo

Point at the **context tape** — "Your instructions" versus "Thread overhead."

> "At 12 turns, your original instructions already hold less than half the model's attention. The rest is overhead from the back-and-forth."

Drag to 30–40 turns.

> "Now your instructions are buried. The model isn't ignoring you — you've drowned your brief under accumulated noise. This is **context rot**. And that's Anthropic's own term — it's in the docs: 'as token count grows, accuracy and recall degrade.'"

**For the chat users:** split your work into an exploration chat and an execution chat. Never let one thread do everything. Add the surface detail: **chat interfaces can drop the oldest turns first** — so in a very long chat, the thing you're relying on may already be gone.

**For the builders:** every token in a long thread re-enters the payload on every call. Latency climbs, structured outputs go flaky. Summarise and flush around turn 10–15, not at the limit.

### The "big window" objection — UPDATED

The June version said 500K. **Don't say that — it isn't a published figure.** Use this instead:

> "Someone always says: 'but the window is a million now, why does any of this matter?' And they're right about the number — 1M is the **default** on Opus 5, Sonnet 5, and the 4.6-and-later models. No beta header, no long-context surcharge. Capacity got cheap."

> "But that makes this level *more* important, not less. **Capacity got cheap. Attention didn't.** You can fit more in; quality still degrades long before the limit. And one more thing people miss: cached tokens still *occupy* the window. Caching changes what you pay, not whether it counts."

### The Architect's Move (the flow diagram)

Three nodes: **Exploration → State Anchor → Execution.**

> "Exploration is cheap, messy, iterative — let it sprawl. But the moment you know what you want to build, you don't carry that mess forward. Ask for a summary — decisions, constraints, format rules. Maybe 600 tokens. Open a fresh thread, paste that anchor at the top, execute cleanly."

> "In Claude Code: plan mode first, then `/compact` proactively — at 50–60% full, not at the limit — then `/clear`. `/context` shows you where you actually are."

### NEW — compaction is not free

> "The platform will now do this for you server-side. But be precise about it: on the API it's opt-in beta, it triggers at **150,000 input tokens** by default, and — this is the part nobody mentions — **it costs an extra sampling pass that you get billed for.** It shows up as its own line in the usage response."

> "Which is the theme of this whole talk: every context-management feature has a price. The question is whether it's cheaper than the rot."

### NEW — context awareness (one-liner, for builders)

> "Small twist: Sonnet 5 and Haiku 4.5 now get told how much context they have left after every tool call — they can pace themselves. Opus doesn't. The cheaper models are the self-aware ones; for Opus you set an explicit task budget."

---

## Tab 3 · Workspace
*The Silent Tax · ~5 min*

### Hook

> "Level 3 is about something that happens before you type a single word."

Click into the tab. Let the meter register.

> "See that number? That's tool definitions loading before you've asked anything. A typical five-server MCP stack — GitHub, Slack, Sentry, Grafana, Splunk — is **about 55,000 tokens** of definitions before Claude does any work. That's Anthropic's published figure."

⚠️ **Say "five-server stack," not "GitHub."** The June version attributed all 55K to GitHub alone — that overstates one connector by roughly 5× and it's checkable from the docs in about ten seconds.

### The connector toggles

> "Every connector you leave switched on loads its definitions into the request every session. You're not paying when you *use* the tool — you're paying whether you use it or not. It's rent, not pay-per-use."

### The argument order — UPDATED for a 1M window

The capacity bar is a weaker argument than it was. **Lead with dollars, then accuracy, then capacity:**

> "First: those tokens are re-read and **re-billed on every single turn**. That's the cost argument, and window size doesn't touch it."

> "Second: accuracy. The docs are explicit — tool selection degrades once you're past **30 to 50 available tools**. That's a reasoning problem, not a capacity problem."

> "Third — and this is where I'll correct myself from June — capacity. 55K out of a 200K window was 27% of your room. Out of 1M it's 5%. So if your only argument was 'you're running out of space,' the platform just took that argument away from you. The other two still stand."

*(Drop the 79.5% → 88.1% accuracy pair. It came from an engineering blog measured on Opus 4.5 and isn't in the current docs.)*

### The Fix section

Click **Enable Tool Search** — watch the meter collapse.

> "Instead of loading every definition upfront, Claude searches the catalogue and loads only the **three to five tools** it needs. The docs say **over 85%** reduction. You didn't build that — it ships."

Be precise on availability:

> "It's on by default in Claude Code. On the API it's **opt-in** — you declare the search tool and mark the rest `defer_loading: true`. Worth it once you're past ten tools, or your definitions exceed 10K tokens."

Nice bonus for builders:

> "And it composes with caching — deferring a tool leaves your cached prefix untouched, so you're not trading cache hits for tool savings."

Then click **Prune the workspace**.

> "Tool Search only handles definitions. The stale files in your project, the 40-turn history? Still yours. **The platform optimises tool definitions. Workspace hygiene is the part you still own.**"

### The takeaway

1. Audit before you prompt — one clean project per task
2. Let Tool Search handle definitions — don't rebuild what ships
3. Own the rest — prune files and threads every session

---

## Tab 4 · Cost
*The Cost Controller · ~7 min · the pricing mechanics*

### Frame it

> "Everything so far has been about reducing tokens. Level 4 is about pricing strategy — making sure that when tokens do flow, you're not paying Opus rates for a job Haiku could do."

### Model routing — three tiers, and the middle one got cheaper

- **Haiku 4.5** — $1 / $5. Routing, classification, extraction. 200K window.
- **Sonnet 5** — **$2 / $10.** The default workhorse. 1M window.
- **Opus 5** — $5 / $25. Reasoning that justifies the premium. 1M window.
- *(Above the tier: **Fable 5.1** at $10 / $50 for the hardest long-horizon work.)*

> "Good news since June: the tier I recommend as your default **got cheaper**. Sonnet was $3 in, $15 out. Sonnet 5 is **$2 and $10** — and Anthropic had scheduled a rise back to $3/$15 on the first of September and then cancelled it. That's the standard price now."

> "The mistake is still treating Opus as the default and everything else as a downgrade. Sonnet is the right default. You *escalate* to Opus when the task earns it."

*(For technical rooms: Opus 4.1 and Opus 4 were $15/$75. Opus 5, 4.8, 4.7 and 4.6 are all $5/$25. Which means "Opus is expensive" is meaningless without a version number.)*

### NEW — reach for the effort dial before you downgrade the model

> "There's a lever that didn't exist the last time I gave this talk, and it belongs *before* model choice: **effort**. Low, medium, high, xhigh, max. Default is high."

The counter-intuitive part — this is the quotable bit:

> "**Lower effort on a newer model often beats high effort on the previous generation.** So before you build a routing cascade down to a cheaper model, just measure Opus 5 at low effort. You may be done."

> "And there's a hidden cost to cascades: **caches are model-scoped.** Every model you add to the chain is another cache namespace you don't get to reuse. One model, one cache."

New lever order for the close: **free wins → effort → model choice.**

### Prompt caching — honest mechanics

> "Caching is often oversold as pure upside. Let me be precise."

- Cache **write** costs a premium: 1.25× on 5-minute, 2× on 1-hour.
- Cache **read** is 0.1× input — 90% off.
- For Sonnet 5: write $2.50/M, read **$0.20/M**.

> "The payback is exact: the 5-minute write pays for itself after **one** read. The 1-hour write after **two**. A one-shot prompt you cache and never repeat? You lost money."

Two subtleties worth the extra thirty seconds:

> "The 5-minute clock starts when the **request** starts, not when the response finishes. So if your response streams for four minutes, you have about one minute to land the follow-up before the cache is gone."

> "And on Opus 5 there's a trick: if you need to inject an operator instruction mid-conversation, **append a system message to the messages array** instead of editing the top-level system prompt. Editing the system prompt throws away your entire cache. Appending doesn't. Not available on Sonnet 5 — Opus and Fable only."

### The cache minimum caveat — CORRECTED

> "One thing that trips builders, and I got this wrong in June. The minimum block size for caching **moved on every model**, and Haiku is now the worst offender: **4,096 tokens on Haiku 4.5**. Sonnet 5 is 1,024. And Opus 5 is only **512** — the most cache-friendly of the lot."

> "Below the floor, caching silently fails. No error. If your caching isn't working, check the block size against *that model's* floor."

*(One more exception if you're citing "90% off" as a law: **Fable 5.1's cache reads are 0.025× — 97.5% off**, $0.25 per million.)*

### NEW — two more priced levers

> "**Fast mode** on Opus 5: up to two-and-a-half times the output speed, at exactly double the price — $10 in, $50 out. Research preview, first-party API only, no batch. Worth it when latency is the product."

> "And if you pin inference to the US for compliance — `inference_geo: us` — that's a **1.1× multiplier on everything**. Input, output, cache reads, all of it. Data residency has a 10% price tag. Put it in the budget rather than discovering it."

**For chat users:** Project Knowledge is the implementation of caching. Load reference material once; only pay active tokens on the variable bit you type.

**For builders:** static system prompts, long reference docs, shared instruction sets — cache candidates. **Batch API** adds 50% off for non-realtime jobs and stacks with caching. Sonnet 5 on batch is **$1 / $5**.

---

## Tab 5 · Agents
*The Systems Designer · ~8 min · the most complex level*

### Pivot from "build it" to "govern it"

> "Levels one to four apply when you're doing the work. Level 5 is what happens when you hand that work to an agent — and the agent hands it to more agents."

> "The old framing: 'here are five disciplines you need to implement.' The true framing: most of that plumbing ships. The work moved up the stack — from engineering to governance."

### The five disciplines, mapped to what ships

1. **Index references** → the memory tool + context editing. **39% lift** with both, **29%** from context editing alone, and **84% fewer tokens** in extended workflows.
   - ⚠️ Source it honestly: *"that's from Anthropic's internal agentic-search eval, published on their blog — not a benchmark page."*
2. **Pre-process context** → server-side compaction. Opt-in beta, triggers at 150K, **and it's billed.**
3. **Scope each agent** → subagents with their own context window, tool set, and model tier.
4. **Stop the loop** → `maxTurns`, `maxBudgetUsd`, `SubagentStop`.
5. **Measure the burn** → `total_cost_usd` plus per-model token breakdown on every result.

### Get the flag names right — CORRECTED

> "Small thing that matters in this room: these are language-specific. TypeScript it's **`maxTurns`** and **`maxBudgetUsd`** — camelCase. Python it's `max_turns` and `max_budget_usd`. In June I had one of each on the same slide."

And be honest about the ceiling:

> "`maxBudgetUsd` stops the query when the SDK's **own client-side cost estimate** crosses your number. Same estimate as `total_cost_usd`, with documented accuracy caveats. It's a guardrail, not a billing-system hard stop. Set it anyway — but don't sell it as a wall it isn't."

### NEW — two kinds of budget, only one enforced

> "There are now two budget mechanisms and people conflate them."

- **Task budgets** — you give the agent a token ceiling, minimum 20,000, and the server injects a countdown **the model can see**. It paces itself and lands gracefully instead of being cut off mid-sentence by `max_tokens`. **Advisory.**
- **Managed Agents session budgets** — **dollar-denominated and platform-enforced.**

> "**Task budgets prevent bad endings. Session budgets prevent bad bills.**"

### NEW — agents have a second meter

> "Here's the thing that genuinely changed since June. If you let Anthropic run the loop — Managed Agents — you pay tokens at the normal rates **plus eight cents per session-hour**. Metered only while the session is actually *running*; idle time is free."

> "Which means for the first time in this talk, **agentic cost isn't only a token question.** It's tokens plus wall-clock. And the Batch discount doesn't apply — sessions are stateful and interactive."

> "If you host it yourself, code execution gives you **1,550 free container-hours a month**, then five cents an hour. And it's free entirely when paired with web search or web fetch."

### The runaway problem (for non-technical rooms)

> "An ungoverned agent loop can burn your monthly budget overnight. Not because of a bug — because nothing told it to stop. Setting a turn limit and a budget cap takes thirty seconds. Not setting them is how you wake up to a depleted budget and a confused agent still running."

*(⚠️ The June deck claimed a separate Agent SDK credit pool for subscription plans as of 15 Jun 2026. **That could not be re-verified** — check your plan's billing page before repeating it.)*

### Two governance moves that matter

**Confirmation gates:** before any irreversible action — deleting a file, sending an email, writing to a database — a well-governed agent stops and asks. `permissionMode` in the SDK.

**Least privilege via `PreToolUse`:** a system-prompt instruction saying "don't use the delete tool" is a *request* the model can drift past. A `PreToolUse` hook that programmatically **denies** the call is deterministic.

> "The difference between 'please don't' and 'you can't' matters a lot when an agent is 15 turns deep and things have gone sideways."

### For technical rooms: the mixed-model pattern

> "Opus 5 orchestrating, Sonnet 5 as the subagents. Orchestrator reasons — plans, decomposes, delegates. Subagents execute — fast, cheap, scoped."

> "But hold it to the same standard as Level 4: **a mixed roster forfeits cache reuse across models.** The split has to earn that. Measure it; don't assume it."

### Close

> "The discipline-level frame survives. What changed is that you no longer build the plumbing. Your job is composition and governance — setting the right bounds, wiring the right hooks, and making sure the thing can stop itself."

---

## Closing (after all five tabs)

> "Five levels. The stakes get higher — from a file format choice to autonomous systems running without you in the loop."

> "The common thread: the waste isn't random. It has a shape. Once you see the shape, you can design around it."

> "And one closing note. I gave this talk three months ago. Since then: the default tier got 33% cheaper, the cache minimums all moved, a new tokenizer made every token count about 30% bigger, and agents grew a second meter that bills by the hour. **The economy moves. The disciplines don't.** That's why we teach the shape and not the numbers."

---

## Q&A prompts (likely questions)

**"Isn't caching free? Why do you say it costs more on first write?"**
> Cache write costs 1.25× on 5-minute, 2× on 1-hour. You amortise over re-reads — one read pays back the 5-minute write, two reads the 1-hour. Never re-read it, and you net nothing.

**"Why did my caching stop working?"**
> Check the minimum block size for *that specific model* — they all moved. Haiku 4.5 is 4,096 tokens, Sonnet 5 is 1,024, Opus 5 is 512. Below the floor it fails silently with no error. Also check whether something in your prefix changed — a timestamp in the system prompt invalidates everything after it.

**"What's the practical window for when I should start a new thread?"**
> Rule of thumb: 50–60% full, not at the limit. `/context` in Claude Code shows your real usage. Auto-compact fires near the limit and often keeps the wrong things.

**"The window is 1M — why do I care about any of this?"**
> Two reasons that survive a big window. The tokens are re-billed every turn, so cost scales with sprawl regardless of capacity. And quality degrades before capacity does — that's context rot, and it's in Anthropic's own docs. Capacity got cheap; attention didn't.

**"Can I cache in Claude.ai chat or only via API?"**
> Project Knowledge implements it implicitly — load stable content once. Explicit cache control with TTL options is an API feature.

**"How do I know which model to use for a given task?"**
> Default to Sonnet 5. Classification, extraction, routing — try Haiku. Multi-step reasoning, ambiguous planning, high stakes — Opus 5. But try one thing first: run the *better* model at **low effort**. Lower effort on a newer model often beats high effort on the older one, and you keep a single cache namespace.

**"Is the newest model always the cheapest per job?"**
> No — and this is the trap. Claude 4.7 and later use a tokenizer that emits about 30% more tokens for identical text. A lower per-million price can be partly eaten by a higher token count. Compare cost per completed task, and re-baseline with the token-counting endpoint.

**"What's the biggest mistake people make with agents?"**
> No stop condition. `maxTurns` and `maxBudgetUsd` take thirty seconds. And know what they are: the budget cap trips on a client-side estimate, not a billing wall. If you need a hard dollar cap, that's a Managed Agents session budget.

**"Does compaction cost anything?"**
> Yes. It runs an extra sampling pass that's billed and itemised separately in the usage response. It's usually cheaper than the rot it prevents — but it isn't free, and the docs don't hide that.

---

*Verified figures: all numbers in these notes match `token-economy-master-verified-notes.md` (10 Sep 2026). Illustrative figures are flagged. Two claims carry a ⚠️ — the Agent SDK credit pool and the 39% eval attribution — read `CHANGELOG-2026-09.md` before quoting either.*
