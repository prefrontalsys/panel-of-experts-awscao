# Panel of Experts — Design Notes

## The Heterogeneous-Models Commitment

Running five panelists on Claude (or all on GPT, or all on Gemini) reduces the panel to a self-consistency check on a single model family. You get the appearance of disagreement — panelists performing different roles — but the disagreement is shallow because the underlying statistical priors are shared.

Provider diversity is what makes the panel epistemically interesting. The disagreements we want are:

- Empiricist (Gemini) flagging that a claim Claude generated has no citations it can verify
- Devil's Advocate (GPT/Codex) identifying a failure mode that didn't appear in Claude's training
- Theorist (Claude) proposing a mechanism GPT's training distribution doesn't emphasize

These are cross-family disagreements. They don't reduce to "one model is wrong and the other is right" — they reduce to "these models have different priors, and the disagreement surfaces information that any single model's prior would hide."

## Why Four Panelists + Moderator

The four roles (Empiricist, Theorist, Devil's Advocate, Synthesist) are not arbitrary. They map to four stages of a mature argument:

1. **What is true?** (Empiricist) — evidence and calibration
2. **What structure explains it?** (Theorist) — mechanism and hypothesis
3. **Where does this break?** (Devil's Advocate) — failure modes and counterexamples
4. **What do we now know?** (Synthesist) — integration, explicit residual uncertainty

Fewer than four collapses stages. More than four introduces redundancy without new epistemic work. The moderator is the fifth role because orchestration is itself a distinct activity — doing it badly (turn management, groupthink detection, premature convergence) ruins the other four.

## Dialog Protocol Rationale

**Round 1 is parallel, non-blocking.** Each panelist gets the framed proposition without seeing the others' responses. This is the anchoring prevention step. If you serialize round 1, the first response biases everything downstream.

**Round 2 is sequential, blocking.** The moderator picks specific claims from round 1 and dispatches them to the panelist most likely to disagree. Blocking handoff is correct here because the challenge response depends on seeing the challenge.

**Round 3 is synthesis.** The Synthesist reads the full transcript and produces structure. Critical: the Synthesist does NOT take a side. Flattening into a recommendation is the default failure mode of panel-like systems; explicit classification of disagreement type (factual / definitional / value) is the countermeasure.

**Round 4 is optional — Devil's Advocate final pass.** If the synthesis feels too clean, run one more adversarial turn. If the attack is credible, the synthesis gets updated to include the residual uncertainty. This is the "what did we miss?" check.

## Known Failure Modes (to watch for)

1. **Immediate convergence in round 1.** If all panelists agree in their independent takes, the topic is probably not controversial enough to benefit from the panel. Or the framing is bad. Or the panelists are too similar in stance.
2. **Devil's Advocate performing contrarianism.** Attacking the weakest version of an argument. Strawmen. If you see this, the panelist prompt needs tightening.
3. **Synthesist summarizing instead of synthesizing.** Bullet-point recaps of each panelist without integration. The distinguishing feature of real synthesis is the disagreement classification.
4. **Moderator arguing the substance.** Moderator's role is process. If they start contributing substantive arguments, they lose the ability to orchestrate neutrally.
5. **Provider collapse.** If CAO falls back to the same provider for multiple panelists (auth failure, missing CLI), the whole point of the panel disappears. The launcher should verify providers are actually distinct before dispatching.

## Open Questions

- **Should the Empiricist have web-search tools?** Almost certainly yes. But CAO's tool-restrictions system is not yet wired into this repo's install scripts.
- **Should there be a "Skeptic" role distinct from "Devil's Advocate"?** Skeptic would challenge the whole framing; Devil's Advocate challenges within the framing. v0 merges these — may need to split.
- **Human-in-the-loop**: Can a user interject mid-panel? CAO's `send_message` mechanism supports this, but the moderator prompt doesn't currently anticipate it.
- **Panel memory**: Should panels remember prior sessions? Currently stateless. Adding persistence would allow the panel to build on its prior deliberations — but also risks calcification.
