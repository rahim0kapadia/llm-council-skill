---
name: llm-council
description: Run a high-stakes decision through five independent advisory lenses and synthesize a direct recommendation. Trigger only when the user explicitly asks for council/war-room/stress-test/pressure-test or presents a real costly tradeoff. Do not trigger for factual lookups, routine triage, or simple execution.
tool_call_budget: 100
---

# LLM Council

Use this for expensive-to-be-wrong decisions, not for routine debugging or status work.

## Context Budget

- Do not read global `CLAUDE.md`, whole memory folders, transcripts, histories, or rule directories by default.
- Gather only the 1-3 files the user explicitly referenced or the smallest project files needed to understand the decision.
- If the task is operational triage, run the triage first and council only the unresolved decision.
- Keep all advisor prompts compact. Do not paste large files into every sub-agent.

## Decision Flow

1. Frame the question in one neutral paragraph:
   - decision to make
   - relevant constraints
   - what is at stake
   - evidence already gathered

2. Run five advisors in parallel:
   - Contrarian: fatal flaws, missing risks, downside.
   - First Principles: actual goal, hidden assumptions, simpler framing.
   - Expansionist: upside, adjacent opportunity, bigger leverage.
   - Outsider: clarity for someone without insider context.
   - Executor: fastest practical path and Monday-morning action.

3. Each advisor response must be 120-220 words, direct, and specific. No preamble.

4. Skip peer-review agents by default. Use peer review only when the user explicitly asks for it or the decision is irreversible/high-cost.

5. Synthesize locally unless peer review was requested.

## Output Shape

Return:

- Where the council agrees.
- Where the council clashes.
- Blind spots.
- Recommendation.
- First action.

Generate HTML/transcript artifacts only when the user asks for artifacts. Otherwise keep the answer in chat.

## Guardrails

- If the user says "maybe council" during a status/triage request, do status/triage first. Council only the remaining strategic decision.
- Never spawn 10 agents automatically. Default is 5 advisors total.
- If a council compact happens, resume from the framed question and already-returned advisor outputs instead of restarting the council.

## Roast Mode (`/roast <idea>`)

For a *business idea / product / offer* go-no-go (not a code/architecture decision). Pulls Claude out of agreement mode and forces a kill-or-ship verdict. Triggered by `/roast` or "roast this idea".

1. **Ask 3 scoping questions first, then wait for answers:**
   - Who exactly is the target buyer?
   - What is your unfair edge / why you?
   - Constraints — budget, time-to-first-dollar, hard nos?

2. **Run the 5 advisors in parallel, with two swaps from the default set:**
   - Contrarian: fatal flaws, why this dies. (same)
   - Expansionist: biggest upside / adjacent leverage. (same)
   - First Principles: pure logic, no outside context. (same)
   - **Deep Researcher** (replaces Outsider): pull real market size + competitor pricing off the web (WebSearch). Cite URLs — no fabricated numbers (zero-hallucination-policy).
   - **Buyer** (replaces Executor): role-play the named ICP. State plainly "would I pay $X? y/n" + the top objection that stops the purchase.

3. **Judge synthesizes ONE verdict** in this shape:

   - **Verdict:** green-light / reshape / kill — confidence 0-100%
   - **Per-persona score:** Contrarian _/10 · Expansionist _/10 · First-Principles _/10 · Researcher _/10 · Buyer _/10
   - **Biggest risk** (one line) · **Biggest upside** (one line)
   - **Cheapest 48-hour test:** the single smallest experiment that validates demand before any code is written (e.g. "DM 20 of the ICP, ask for pre-orders"). Must be runnable in 48h with the stated budget.

Researcher numbers cite a source; Buyer/scores are explicitly the persona's judgment, not fact.
