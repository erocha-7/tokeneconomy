# The Token Economy — September 2026 Refresh
*What changed since the June 2026 deck, what to drop, and what's new enough to be worth a slide.*

Verified 10 September 2026 against [platform.claude.com](https://platform.claude.com/docs/en/about-claude/pricing) and [code.claude.com](https://code.claude.com/docs/en/agent-sdk/typescript). Every number here is in `token-economy-master-verified-notes.md` with its source.

---

## TL;DR for the presenter

The June deck is **structurally sound and factually stale**. The five-level frame holds up; the thesis of Level 5 ("the primitives ship, your job is governance") aged well. But nine specific figures moved, and **four claims should be cut outright** because a technical audience can now contradict them from the docs.

If you only fix five things before presenting:

1. **Sonnet 4.6 $3/$15 → Sonnet 5 $2/$10.** Your default-tier recommendation got 33% cheaper. That's a good-news slide, not just a correction.
2. **Haiku's cache minimum is 4,096, not 2,048** — and Opus 5's is **512**. Your Level 4 caveat is directionally right and numerically wrong in both directions.
3. **The ~30% tokenizer increase on Claude 4.7+.** This is new material that didn't exist in June, it undercuts every token estimate in the deck, and it is the single most interesting thing you can tell a room that already thinks it understands tokens.
4. **"GitHub MCP = 55,000 tokens" is wrong** — 55K is a five-server stack. Overstating one connector by ~5× is exactly the claim someone will check live.
5. **`max_budget_usd` → `maxBudgetUsd`** in the TypeScript SDK, and it trips on a *client-side estimate*, not a hard billing ceiling.

---

## 1. Corrections — numbers that moved

| # | June deck said | Now verified | Where it appears |
|---|---|---|---|
| 1 | Sonnet 4.6 is the default tier at **$3 / $15** | **Sonnet 5 at $2 / $10** (introductory pricing was made permanent; the scheduled 1 Sep 2026 rise to $3/$15 **was cancelled**) | L4 table, L4 calculator, L5 cost meter, L1 model picker |
| 2 | Opus **4.x** at $5 / $25 | **Opus 5** at $5 / $25 — price unchanged, name and default behaviour changed | L4, L5, L1 |
| 3 | Cache minimum **1,024** / **2,048 on Haiku** | **512** Opus 5 · **1,024** Sonnet 5 / Opus 4.8 · **2,048** Opus 4.7 · **4,096 Haiku 4.5** and Opus 4.6 | L4 explainer, L4 chips, L4 lever copy, L4 JS model table |
| 4 | Cache read is **0.1× input, "the 90% off"** | Still 0.1× — **except Fable 5.1 at 0.025× ($0.25/M, 97.5% off)** | L4 explainer + footnote |
| 5 | Context window **≈500K chat / up to 1M Code & API**, plan-dependent | **1M is the default** on Opus 5, Opus 4.8/4.7/4.6, Sonnet 5, Sonnet 4.6 — **no beta header, no long-context premium**. Haiku 4.5 / Sonnet 4.5 = 200K. **The 500K chat figure is not published** — docs only say chat surfaces can roll off oldest-first | L2 objection shield, L2 footnote, L3 window scale |
| 6 | GitHub MCP server **= 55,000 tokens (93 tools)** | **≈55,000 tokens is a five-server stack** (GitHub, Slack, Sentry, Grafana, Splunk) | L3 meter narrative, speech notes |
| 7 | Tool Search: accuracy **79.5% → 88.1%** (Opus 4.5) | **Not in current docs.** Docs now state accuracy degrades **above 30–50 available tools**. The >85% token reduction **is** still published | L3 architect lens, speech notes |
| 8 | PDF page ≈ **1,500–3,000 tokens/page** all-in | **1,500–3,000 text tokens/page PLUS image tokens** — two separate charges | L1 explainer |
| 9 | Agent SDK guard **`max_budget_usd`** (mixed with camelCase `maxTurns`) | **TypeScript: `maxBudgetUsd`** · **Python: `max_budget_usd`**. Pick one language per slide. It stops on the SDK's **client-side cost estimate**, not a billing hard stop | L5 governance panel, L5 footnote, speech notes |

## 2. Claims to cut

- **"GitHub's MCP server is 55,000 tokens."** → "A typical five-server MCP stack is ~55,000 tokens."
- **"79.5% to 88.1% accuracy."** → "Selection accuracy degrades past 30–50 tools; Tool Search cuts definition tokens by over 85%."
- **"The window is 500K in chat."** → "1M by default on the API and Claude Code; chat surfaces roll off oldest-first."
- **"Jira MCP ≈ 17,000 tokens."** → Demote to illustrative; tell people to run `/context`.
- **⚠️ "Separate Agent SDK credit pool, effective 15 Jun 2026."** → **Could not be re-verified.** Check your own plan's billing page before repeating it.

## 3. New material worth a slide

Ranked by how much it will land with a room that saw the June version.

### 3.1 The tokenizer tax — best new content in the refresh *(Level 1)*
Claude **4.7 and later** (Opus 4.7/4.8/5, Sonnet 5, Fable 5/5.1) use a newer tokenizer producing **≈30% more tokens for the same text**. Sonnet 4.6, Opus 4.6 and Haiku 4.5 use the previous one.

Why it's a great slide: it inverts the audience's assumption that newer = cheaper, and it retroactively invalidates every token budget in the room.
- "4,500 words ≈ 6,000 tokens" becomes **≈7,800** on Opus 5.
- A Haiku-vs-Opus 5 routing comparison **isn't apples-to-apples** — different tokenizers on the same input.
- Therefore: **cost per completed task** is the only honest unit.

### 3.2 The effort dial beats model downgrading *(Level 4)*
`output_config.effort` — `low`/`medium`/`high`/`xhigh`/`max`, GA, default `high`. The guidance is counter-intuitive and quotable: **lower effort on a newer model often beats high effort on the previous generation**, so measure "Opus 5 at `low`" *before* building a cascade to a cheaper model. And a multi-model cascade **forfeits cache reuse** — caches are model-scoped. One model, one cache namespace.

New lever order for the Level 4 close: **free wins (caching, hygiene, batch) → effort → model choice.**

### 3.3 Compaction is not free *(Level 2)*
The June deck said "automatic compaction near the limit — the platform handles it." Sharper and more honest now:
- On the API it's **opt-in beta** (`compact-2026-01-12`), trigger **default 150K input tokens**, minimum 50K.
- It runs **an extra sampling pass that is billed**, itemised in `usage.iterations`.
- Fits the deck's existing "caching is cost relief, not a cure" theme perfectly: **every context-management feature has a price; the question is whether it's cheaper than the rot.**

### 3.4 Agents have a second meter *(Level 5)*
**Managed Agents: tokens at standard rates + $0.08 per session-hour**, charged only while status is `running` (idle is free). Replaces container-hour billing. **No Batch discount** — sessions are stateful.
Self-hosted alternative: code execution gives every org **1,550 free container-hours/month**, then $0.05/hr, and is **free** when paired with current web search / web fetch.

This is the first time the deck can say: *"agentic cost is no longer only a token question."*

### 3.5 Two kinds of budget — only one is enforced *(Level 5)*
- **Task budgets** (beta, min 20,000 tokens): server injects a countdown the model can see, so it **paces itself and lands gracefully**. **Advisory.**
- **Managed Agents session budgets**: **dollar-denominated, platform-enforced.**

Line for the slide: **task budgets prevent bad endings; session budgets prevent bad bills.**

### 3.6 Context awareness is now model-dependent *(Level 2)*
Sonnet 5 / 4.6 / 4.5 and Haiku 4.5 receive an injected `<system_warning>Token usage: X/Y; Z remaining</system_warning>` after each tool call — they track their own budget. **Opus 4.7+ and Fable do not**, and need an explicit task budget instead. Nice inversion: the cheaper models are the self-aware ones.

### 3.7 Caching mechanics worth adding *(Level 4)*
- **Payback is exact:** 5-min write (1.25×) pays back after **one** read; 1-hr write (2×) after **two**.
- **TTL runs from request start**, not response end — a 4-minute streamed response leaves ~1 minute to land the follow-up.
- **Mid-conversation system messages** (Opus 5 / 4.8 / Fable — *not* Sonnet 5): append `{"role":"system"}` to `messages[]` instead of editing top-level `system`, and the cached prefix survives. Editing `system` mid-conversation throws the whole cache away.
- **Tool Search composes with caching:** `defer_loading` leaves the prefix untouched.

### 3.8 Two new priced levers *(Level 4)*
- **Fast mode** (Opus 5 / 4.8, research preview, first-party only): **$10/$50 — 2× price for up to 2.5× output speed.** Own rate limit; no Batch; switching speed invalidates cache.
- **`inference_geo: "us"`**: **1.1× on every token category.** Data residency has a 10% price tag — name it in the budget.

### 3.9 Tool overhead is now published per model *(Level 3)*
With ≥1 tool present: **Opus 5 = 286 tokens** (`auto`/`none`), Sonnet 5 = 354, Haiku 4.5 = 496. Bash adds ~325 on Opus 5; computer-use toolset ~4,500; browser-use ~6,600. Useful because it's small — it lets you make the point that **definitions, not the tool-use scaffolding, are the tax.**

### 3.10 Sourcing the 39% properly *(Level 5)*
Memory tool + context editing = **39%** over baseline; **context editing alone = 29%**; **84% fewer tokens** in extended workflows. Source: Anthropic's context-management blog, **internal agentic-search eval** — not a docs benchmark. Cite it that way and add the 84%, which is the more striking number for a token talk.

---

## 4. Structural change: Level 3 now models a 1M window — DONE

**Level 3's capacity bar was the weakest of its three arguments, so it has been demoted rather than hidden.** At a 200K window, 55K of connector definitions was ~27% of your room — a visceral bar. The track now models the **1M default** (Opus 5, Sonnet 5, Opus 4.6+, Sonnet 4.6), where the same stack is **~5.5%** and every connector switched on still stays **under 14%**.

That is deliberately unflattering to the bar, and **that is the point.** The argument is now ordered:

1. Those 55K tokens are **re-read and re-billed on every turn** — cost scales with conversation length, not capacity. Window size does not touch this.
2. Selection still degrades **above 30–50 tools**, regardless of how much room you have.
3. Capacity is the *third* reason now, not the first — and the platform took it away from you.

What changed in the demo to keep it coherent at 1M:
- Window `200000` → `1000000`; scale reads 0 / 250K / 500K / 750K / 1M.
- **Hero framing moved from window fraction to absolute load.** The warn state now trips on tokens resident (≥75K), not on a window percentage that can no longer reach 50%.
- **Business-lens bullets rewritten.** The old copy branched on window fraction and would have said "plenty of room" in every state — true at 1M, but it argues against the level. It now leads with the recurring charge.
- **Reclaim readout re-based:** "% of the window handed back" (a rounding error at 1M) → **"% of the pre-prompt load removed."** Both fixes on now reads a satisfying **88%**.
- **Removed a fabricated metric.** The architect lens displayed "Tool-select accuracy 80%" from a formula descended from the retired 79.5%→88.1% pair — an invented number shown as fact, and inconsistent with cutting that claim from the sourcing note. It now reports **tools available against the documented 30-tool threshold**.
- **Tool Search modelled from the published figure:** was a flat 500-token residual (unsourced); now a 15% residual, which is what ">85% reduction" actually says.

This is a stronger, more defensible Level 3 than the June version — and it pre-empts the "but I have a million tokens" heckle instead of walking into it.

---

## 5. Files updated in this refresh

| File | Status |
|---|---|
| `token-economy-master-verified-notes.md` | Rewritten — new figures table, all five levels revised, per-level change log |
| `CHANGELOG-2026-09.md` | New — this file |
| `speech-notes.md` | Updated — talking points, model tiers, Q&A re-answered |
| `Level1.html` | Model picker + rates → Opus 5 / Sonnet 5 / Haiku 4.5; tokenizer note |
| `Level2.html` | 500K-chat claim → 1M default + rolling FIFO |
| `Level3.html` | Rescaled to the **1M default window** (see section 4); 55K re-attributed to a five-server stack; fabricated accuracy metric replaced with the documented 30-tool threshold |
| `Level4.html` | Rate table, calculator model data, cache minimums, Fable 5.1 cache-read exception |
| `Level5.html` | Cost meter models, SDK flag names, real-vs-illustrative footnote |
| `index.html` | Footer date |
| `ClaudeCodeBrief.md` | Footer string + source-of-truth date reference |

**Resolved decisions:**
- **Level 3's bar is rescaled to the 1M default** — see section 4.
- **Fable 5.1 stays a footnote; no fourth routing tier.** The routing ladder answers "what is the cheapest *sufficient* model?" and Fable answers a different question — hardest long-horizon work, correctness over cost. Putting it on the ladder would imply it's the top rung of a cost decision, which is the opposite of the level's argument. It now has one deliberate footnote in the Level 4 footer (what it is, $10/$50, why it's excluded, and its 0.025× cache-read quirk) plus a Q&A answer in the speech notes. Three tiers stay on the diagram: Haiku 4.5 → Sonnet 5 → Opus 5.

**No open items.**
