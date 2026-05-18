# Round 2 — Cross-Examination

**Purpose**: Surface direct disagreements by routing each panelist's most interesting claim to the panelist most likely to disagree with it.

**Your actions this round**:

1. For each panelist's round-1 response (in `<topic-slug>/round-1-<panelist>.md`):
   - Identify the single most interesting claim — the one that is (a) load-bearing for their position, (b) likely contested by another panelist.
   - Pick the panelist most likely to disagree with that claim. Heuristics:
     - Empirical claims → Devil's Advocate or Theorist
     - Theoretical claims → Empiricist
     - Adversarial/skeptical claims → Theorist or Empiricist
   - Use `cao_handoff` (blocking) to send the claim to the chosen panelist with framing: "The {original_panelist} claimed: '...'. What's your strongest counter?"
   - Wait for response. Write to `<topic-slug>/round-2-<challenger>-vs-<original>.md`.
2. After each challenge response, use `cao_handoff` to return to the ORIGINAL panelist with the challenger's response, asking for a rebuttal. Write to `<topic-slug>/round-2-<original>-rebuttal.md`.

**Decision points**:

- If round 1 showed immediate convergence, this round MUST include the Devil's Advocate attacking the consensus. Convergence-without-challenge is the panel's failure mode.
- If a panelist concedes immediately on a challenge, dispatch to the same challenger with a different claim — concessions are useful data but don't generate new content.
- Cap this round at 3-4 challenge cycles. More than that is diminishing returns; the same disagreements will recur in synthesis.

**Anti-patterns to avoid**:

- Dispatching the same panelist back-to-back without giving them the new response (loses thread).
- Letting the panelists argue without the moderator framing each turn (becomes unstructured chat).
- Picking weak claims to challenge (strawmen waste the panel's time — same rule applies as in the Devil's Advocate role).
