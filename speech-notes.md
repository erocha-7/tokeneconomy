# The Token Economy — Speech Notes
*Five Levels · Balanced for chat users and builders · Jun 2026*

---

## Opening (before clicking any tab)

> "Tokens are the currency of AI. Every word you send, every file you upload, every tool you connect — it all converts to tokens. The model never reads your message for free. And most of the waste isn't obvious. That's what these five levels are about."

Set the frame: this isn't about cutting corners. It's about understanding where your budget goes so you can spend it on the work that matters.

---

## Tab 1 · Ingestion
*The Text Purist · ~2 min · keep it brief*

The core idea here is simple: **PDFs are expensive because Claude reads each page as an image, not just text.**

A 4,500-word document is roughly 6,000 tokens as plain text. As a PDF, each page gets rendered — so you're paying for the text *and* a visual scan of the layout. For scans or image-heavy reports, it gets far worse.

**The habit that kills tokens:** dragging PDFs into a chat when a plain-text paste would do the same job. And once a file is in the thread, it's re-sent every turn.

**The fix is boring and effective:** convert to Markdown before the call. For teams building with the API, add a lightweight ingestion step that extracts and strips layout first.

> *"This one's intuitive. The next four are where it gets interesting."*

---

## Tab 2 · Threads
*The Thread Architect · ~6 min · this is the core insight*

### Set it up

> "Here's something most people don't realise. There's no memory inside a thread. Every time you hit send, the model re-reads the entire conversation from the beginning. A long thread isn't a history — it's a recurring bill."

Move the slider to around 12 turns and let the numbers land.

### Walk through the demo

Point at the **context tape** — the bar showing "Your instructions" versus "Thread overhead."

> "At 12 turns, your original instructions already hold less than half the model's attention. The rest is overhead from the back-and-forth. Keep going..."

Drag to 30–40 turns.

> "Now your instructions are buried. The model isn't ignoring you — you've just drowned your brief under accumulated noise. This is what we call **context rot**. It *feels* like Claude forgot, but you filled the window with junk."

**For the chat users:** click the "Chat User" button. Let them read the copy. The message is: split your work into an exploration chat and an execution chat. Never let one thread do everything.

**For the builders:** click "Builder." The message shifts to: every token in a long thread re-enters the payload on every call. Latency climbs, structured outputs go flaky. Summarise and flush around turn 10–15, not at the limit.

### Point out the "big window" objection proactively

The UI surfaces it — quote it directly:

> "Someone will always say: 'but the window is 500K now, why does any of this matter?' It does matter. A bigger window doesn't fix attention dilution. You can fit more in, but quality degrades long before you hit the limit. Big doesn't mean focused."

### The Architect's Move (the flow diagram)

Three nodes: **Exploration → State Anchor → Execution.**

> "Think of it like this. You have two modes of working with AI. Exploration is cheap, messy, iterative — let it sprawl. But the moment you know what you want to build, you don't carry that mess forward. You ask for a summary — the decisions, the constraints, the format rules. Maybe 600 tokens. Then you open a fresh thread, paste that anchor at the top, and execute cleanly."

> "In Claude Code specifically: drop into plan mode first, then `/compact` proactively — at 50–60% full, not at the limit — then `/clear` for the fresh thread."

### The comparison cards

> "Same outcome, sliced — roughly 50% fewer tokens through the model. And more importantly: no drift."

---

## Tab 3 · Workspace
*The Silent Tax · ~5 min*

### Hook

> "Level 3 is about something that happens before you type a single word."

Click into the tab. Let the meter register — it starts with GitHub and Jira already toggled on.

> "See that number? You've already spent — look at it — over 70,000 tokens. GitHub's MCP server is 55,000 tokens. Jira is 17,000. That's Anthropic's published figure. And this tax is paid on every turn, before your prompt even arrives."

### The connector toggles

Toggle tools on and off and watch the meter move. Start turning on Salesforce, Drive, Slack, Notion.

> "Every connector you leave switched on in a project loads its tool definitions into the system prompt. You're not paying when you use the tool — you're paying whether you use it or not. It's rent, not pay-per-use."

**For chat users:** the message is about room. More context spent on connectors means less room for the actual answer. Toggle the "Chat User" lens — the bullets explain it in plain terms.

**For architects:** toggle to "Architect." Tool-selection accuracy is shown. When you pile up tool definitions, the model has to choose between more options — and it gets it wrong more often. Anthropic measured this: from 79.5% to 88.1% accuracy improvement with Tool Search active.

### The Fix section

Click **Enable Tool Search** — watch the meter collapse.

> "This is now a platform feature. Tool Search is GA and on by default in Claude Code. Instead of loading all 93 GitHub tools upfront — 55,000 tokens — the model loads about 500 tokens of lookup overhead and fetches definitions on demand. That's an 85% reduction. You didn't have to build this. It ships."

Then click **Prune the workspace**.

> "But Tool Search only handles the tool definitions side. The stale files you left in a project? The 40-turn chat history? That part is still yours to own. The platform optimises tool definitions. Workspace hygiene is the part you still own."

### The takeaway (three steps)

Point at the checklist at the bottom:

1. Audit before you prompt — one clean project per task
2. Let Tool Search handle definitions — don't rebuild what's already there
3. Own the rest — prune files and threads every session

---

## Tab 4 · Cost
*The Cost Controller · ~6 min · the pricing mechanics*

### Frame it

> "Everything we've covered so far has been about reducing tokens. Level 4 is about pricing strategy — making sure that when tokens do flow, you're not paying Opus rates for a job Haiku could do."

### Model routing — three tiers, not two

> "There's a temptation to think of model choice as Haiku versus Opus. The real answer is three tiers."

Draw this out verbally:
- **Haiku 4.5** — $1/M in, $5/M out. Routing, classification, extraction. Fast and cheap.
- **Sonnet 4.6** — $3/M in, $15/M out. The default workhorse. Most tasks live here.
- **Opus 4.x** — $5/M in, $25/M out. Reserve for reasoning that genuinely justifies the premium.

> "The mistake people make is treating Opus as the default and everything else as a downgrade. Sonnet is the right default. You *escalate* to Opus when the task earns it."

*(For technical rooms: note that legacy Opus was $15/$75 — this generation is dramatically cheaper. Always version the model name when citing a price.)*

### Prompt caching — honest mechanics

> "Caching is often oversold as pure upside. Let me be precise about how it actually works."

Point at the payoff diagram on screen:

- Cache **write** costs a premium: 1.25× on the 5-minute cache, 2× on the 1-hour option.
- Cache **read** is where the savings are: 0.1× input — roughly 90% off.
- So for Sonnet: write is $3.75/M, read is $0.30/M.

> "Caching only wins once you re-read the cached block enough times to amortise the write cost. A one-shot prompt that you cache once and never repeat? You lost money. The case for caching is: stable content, read many times. A 50-page sales playbook loaded once into Project Knowledge and referenced across hundreds of conversations — that's the winning pattern."

**For chat users:** Project Knowledge is the implementation of this. Load your reference material once; only pay active tokens on the variable bit you type.

**For builders:** static system prompts, long reference documents, shared instruction sets — these are cache candidates. The Batch API adds another layer: 50% off for non-realtime jobs, and it stacks with caching toward ~95% savings on bulk processing.

### Haiku caveat worth flagging

> "One thing that trips builders: Haiku's cache minimum is 2,048 tokens, not 1,024. Small blocks on Haiku silently fail to cache. If you're wondering why your caching isn't working on Haiku — check the block size."

---

## Tab 5 · Agents
*The Systems Designer · ~8 min · the most complex level*

### Pivot from "build it" to "govern it"

> "Everything in the first four levels applies when you're doing the work yourself. Level 5 is about what happens when you hand that work to an agent — and the agent hands it to more agents."

> "The framing that used to be common: 'here are five disciplines you need to implement to govern agentic systems.' The framing that's true now: most of that plumbing ships. The work moved up the stack — from engineering to governance."

### The five disciplines, mapped to what ships

Walk through the console / live agent simulation if available:

1. **Index references** → the Memory tool + context editing. Anthropic measured ~39% lift on long agentic tasks.
2. **Pre-process context** → automatic compaction near the window limit. The platform handles it.
3. **Scope each agent** → subagents with their own context window, tool set, and model tier. Opus orchestrator + Sonnet subagents is the practical pattern.
4. **Stop the loop** → `maxTurns`, `max_budget_usd`, `SubagentStop`. These are flags you set, not code you write.
5. **Measure the burn** → `total_cost_usd` on every result, plus per-model token breakdown. Telemetry is built in.

### The runaway problem (for non-technical rooms)

> "An ungoverned agent loop can burn your entire monthly budget overnight. Not because of a bug — because nothing told it to stop. The Agent SDK now has a hard ceiling: `max_budget_usd`. Set it. It's not optional."

> "And as of June 15, 2026, Claude.ai subscription users have a separate Agent SDK credit pool — so agentic usage doesn't eat into your regular chat allowance."

### Two governance moves that matter

> "Beyond the basic guards, there are two moves that I think are underappreciated."

**Confirmation gates:** before any irreversible action — deleting a file, sending an email, writing to a database — a well-governed agent stops and asks. This is `permissionMode` in the SDK. Don't skip it.

**Least privilege via `PreToolUse`:** a system-prompt instruction saying "don't use the delete tool" is a request the model can drift past under pressure. A `PreToolUse` hook that programmatically *denies* the tool call is deterministic. It can't be talked out of it.

> "The difference between 'please don't' and 'you can't' matters a lot when an agent is 15 turns deep and things have gone sideways."

### For technical rooms: the mixed-model pattern

> "Opus as the orchestrator, Sonnet as the subagents. The orchestrator does the reasoning — planning, decomposition, deciding what to delegate. The subagents do the execution — fast, cheap, scoped. You get the quality of Opus where it counts and the cost of Sonnet everywhere else."

### Close

> "The discipline-level frame survives. What changed is that you no longer have to build the plumbing from scratch. Your job is composition and governance — setting the right bounds, wiring the right hooks, and making sure the thing can stop itself when it needs to."

---

## Closing (after all five tabs)

> "Five levels. One through five, the stakes get higher — from a file format choice to autonomous systems running without you in the loop."

> "The common thread across all of them: the waste isn't random. It has a shape. Once you see the shape, you can design around it."

> "Tokens aren't going away. The models will get more efficient over time, but the costs will scale with the ambition of what you're building. Understanding the economy is how you stay ahead of it."

---

## Q&A prompts (likely questions)

**"Isn't caching free? Why do you say it costs more on first write?"**
> Cache write costs 1.25× on 5-minute, 2× on 1-hour. You amortise that over re-reads. If you never re-read the block, you net nothing.

**"What's the practical window for when I should start a new thread?"**
> Rule of thumb: 50–60% full, not at the limit. In Claude Code, run `/context` to see your usage. Auto-compact fires near the limit and often keeps the wrong things — proactive compaction keeps the right ones.

**"Can I cache in Claude.ai chat or only via API?"**
> Project Knowledge in Claude.ai implements caching implicitly — you load stable content once, and it's cached. Explicit cache control with TTL options is an API feature.

**"How do I know which model to use for a given task?"**
> Default to Sonnet 4.6. If the task is classification, extraction, routing — try Haiku first. If the task is multi-step reasoning, ambiguous planning, or high-stakes decisions — that's Opus territory. Let complexity justify the upgrade.

**"What's the biggest mistake people make with agents?"**
> No stop condition. Setting `maxTurns` and `max_budget_usd` takes 30 seconds. Not setting them is how you wake up to a depleted budget and a confused agent still running.

---

*Verified figures: all numbers in these notes match the master-verified-notes.md. Illustrative figures are flagged — don't quote them as measurements.*
