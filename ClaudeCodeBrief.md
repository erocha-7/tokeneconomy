# Build brief — Token Economy: single tabbed demo

## Objective
Combine the 5 standalone Level demos (Levels 1–5) in this folder into **one tabbed HTML experience** I can open once and click through 5 tabs to present live. Apply the per-level change notes below before delivering. Do not break any existing interactivity.

---

## Inputs (in this folder)
- 5 working interactive HTML demos — one per Level. **Inventory them yourself** (`ls`, open each) rather than assuming filenames.
- `token-economy-master-verified-notes.md` — verified copy and figures (re-verified 10 Sep 2026). **Do not invent numbers; source any new text from this file.**
- `CHANGELOG-2026-09.md` — what changed since the June 2026 run, which claims were cut, and what new material is worth a slide. Read this before touching any figure.
- This brief is the complete specification. No other notes file exists — all change instructions are written below.

---

## Architecture — do this, don't improvise

**Default: iframe tabs (preserves each demo's JS/CSS exactly — zero collision risk).**
- One `index.html` shell: shared header + 5-tab bar + content area.
- Each tab lazy-loads its Level file in an `<iframe>` (only the active tab's `src` is set, to avoid loading all 5 at once).
- The 5 demo files stay in the folder; the shell references them.

**Single-file variant (only if I explicitly ask for it):** inline everything into one `index.html`, scope CSS under `#lvlN`, wrap each Level's JS in an IIFE, init on tab activation. Higher collision risk — don't do it unless I ask.

---

## Shared shell spec
- **Header:** "The Token Economy" + tagline "Five levels. One session."
- **Tab bar:** `1 · Ingestion`, `2 · Threads`, `3 · Workspace`, `4 · Cost`, `5 · Agents`
- **Design system:** Level 2 is the visual template for the whole series (warm-ink dark background, Bricolage Grotesque display, IBM Plex Mono for data, amber/teal accent pair). Read Level 2's CSS variables and replicate them in the shell header and in any rebuilt Level pages.
- **Audience toggle label (global):** everywhere a Level says "Business Owner" change it to **"Chat User"**. "Builder" stays as-is.
- **Footer:** "The Token Economy · re-verified Sep 2026"
- Proper `role="tablist"` / `tab` / `tabpanel`, arrow-key nav, visible focus ring.
- Works offline — no build step, fonts degrade to system stack.
- Responsive; `prefers-reduced-motion` respected; no localStorage/sessionStorage.

---

## Per-level change notes
*Apply every item below. My notes win over the existing file wherever they conflict.*

### Level 1 — Document Ingestion
1. **Restyle to match Level 2.** Read Level 2's CSS (colours, type scale, card/section layout) and apply it to Level 1 so they look like the same series.
2. **New section after "The Difference":** add a clearly-labelled fun section with heading **"Meanwhile, your context window…"** containing exactly these 4 GIFs in order, each with the caption shown. Display them in a 2×2 grid, captions below each GIF, tone is light:

   - `https://media2.giphy.com/media/WDvg3UX4PiHio/giphy.gif` — caption: *"You, uploading a PDF to ask one question."*
   - `https://media1.giphy.com/media/CNAhQuDceLwwo/giphy.gif` — caption: *"Claude, processing your 47-page formatting metadata."*
   - `https://media4.giphy.com/media/q7kofYLObTVUk/giphy.gif` — caption: *"Your token limit, watching the thread grow."*
   - `https://media3.giphy.com/media/4mBzKXTeRmEOA/giphy.gif` — caption: *"Same doc, pasted as Markdown."*

   Use `<img>` tags with `loading="lazy"`, `width="100%"`, and `alt` text matching the caption. The section sits at the very bottom of Level 1, after all educational content.

### Level 2 — Thread Dynamics *(the visual template — get this right first)*
1. **Audience toggle labels:** "Business Owner" → **"Chat User"**. "Builder" stays.
2. **"Why you care" subheadings:** update them to map to the new labels — "For Chat Users" and "For Builders" (not "For Chat Users & Business Owners").
3. **"Accumulated cruft" label on the context tape:** replace with something more professional. Suggested: **"Accumulated noise"** or **"Thread overhead"** — pick whichever reads better in context. Do not use "cruft" or "croft".
4. This file is the design reference. After changes, confirm its CSS variables are clean and reusable before touching the other levels.

### Level 3 — Workspace Hygiene
1. **Apply Level 2 template** — same CSS variables, same header style, same section rhythm.
2. **Audience toggle:** "Business Owner" → **"Chat User"**. Same label propagation as Level 2.

### Level 4 — Cost Controller
1. **Apply Level 2 template** — same CSS variables and layout style.
2. **Audience toggle:** same label change.
3. **Clarity overhaul — this is the most important change for this level.** The current page is mechanically correct but the core message (when and why caching pays off vs. costs more) is not landing. Do the following:
   - Add a **plain-language explainer section** before the calculator that answers in two sentences: *"What is prompt caching and when should I use it?"* Source the explanation from `token-economy-master-verified-notes.md` (the cache-write premium, the re-read threshold, the TTL mechanics).
   - Add a **visual "Cache Payoff" diagram** — a simple before/after showing: call 1 (write cost, no saving) → calls 2–N (read discount kicks in). Make it clear the first call is more expensive, not less.
   - Make the calculator's **cache toggle more prominent** — label it "Cache on/off" and show a one-line consequence next to it ("First call costs more. Every repeat call saves 90%.").
   - The **"Show the token math" expander** is good — keep it, but open it by default on desktop so the builder audience sees the rate table without hunting.

### Level 5 — Agentic Systems
1. **Apply Level 2 template.**
2. **Audience toggle:** same label change.
3. **Remove the "The shift this level is about" section** (the table comparing "what the lesson said to build" vs "what ships now"). Replace it with a new section called **"Governing Agents in Practice"** that is forward-looking and action-oriented: what do you actually *do* to stop a loop, scope a subagent, gate an irreversible action. Short, practical, numbered steps — not a before/after comparison.
4. **Add a worked customer-support case study** as the centrepiece of this level. The scenario: an AI agent that routes incoming customer support tickets. Show the uncontrolled version (agent reads the customer's full lifetime history → token explosion, no stop-guard, cost runaway) vs the governed version (pre-summarised last-30-days context, `maxTurns`, `max_budget_usd`, `SubagentStop` hook, confirmation gate before any refund or escalation). Make it interactive if possible — a simple toggle between "ungoverned" and "governed" run, with the token count and cost changing visibly. Source figures from `token-economy-master-verified-notes.md`.

---

## Hard constraints
- Every existing demo's interactivity works unchanged (sliders, toggles, calculators, the audience-lens toggle inside each demo).
- Verified figures in `token-economy-master-verified-notes.md` are the source of truth — do not change any numbers without sourcing them there.
- No redesign beyond what's specified. Integration + my change notes only.

---

## Workflow
1. Inventory the folder; read each HTML and confirm which Level it is.
2. Read `token-economy-master-verified-notes.md`.
3. **Show me a short plan** — architecture choice, tab labels, a bullet per Level of what you'll change — and **wait for my OK** before building.
4. Build Level 2 first (it's the template). Confirm CSS variables are clean. Then build the shell. Then patch Levels 1, 3, 4, 5 in order.
5. Verify: open `index.html`, click all 5 tabs, confirm each demo still runs. Report any compromises.

## Done =
`index.html` (tabbed shell) + 5 updated Level files, all in the same folder. Opens offline. All 5 tabs run their demos intact. Change notes applied. One-line note of anything that needed compromise.

## Out of scope
New levels, new features, changed verified figures, anything not in these notes.