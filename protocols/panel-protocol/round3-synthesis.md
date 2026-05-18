# Round 3 — Synthesis

**Purpose**: Integrate the panel's contributions into a structured position document that honestly represents both convergence and live disagreement.

**Your actions this round**:

1. Concatenate the full transcript: framing.md + round-1-* + round-2-* into `<topic-slug>/transcript.md`.
2. Use `cao_handoff` (blocking) to dispatch the synthesist with the transcript. Frame the dispatch: "Read the attached transcript at <absolute path>. Produce a synthesis per your output format. Do NOT pick a winner; classify disagreements."
3. Wait for the synthesis. Write to `<topic-slug>/synthesis.md`.
4. Append the synthesis to `<topic-slug>/transcript.md` so the transcript is now complete-with-synthesis.

**Critical check before terminating**:

- Does the synthesis classify every live disagreement as factual / definitional / value? If not, the synthesist failed at the most useful artifact. Optionally dispatch again with that specific feedback.
- Does the synthesis flatten any disagreement that genuinely existed in the transcript? If yes, escalate to round 4 (optional adversarial pass).
- Does the synthesis claim resolution where the panel didn't reach it? Same — escalate to round 4.

**Termination**:

This is the terminal mandatory round. If the synthesis passes the checks above, the panel session is complete — write a final report to the user pointing at `<topic-slug>/synthesis.md` and stop. Do NOT run round 4 unless one of the failure conditions above triggered.
