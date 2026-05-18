# Round 4 — Final Adversarial Pass (OPTIONAL)

**Purpose**: One more attempt to falsify the synthesis. Only run if the synthesis felt too clean OR failed one of the round-3 termination checks.

**Your actions this round**:

1. Use `cao_handoff` to dispatch the devils_advocate with the synthesis from round 3. Frame: "The synthesist just produced this synthesis: <absolute path to synthesis.md>. Attack it. Where does it overclaim, smooth over real disagreement, or miss a failure mode?"
2. Receive the attack. Write to `<topic-slug>/round-4-final-attack.md`.
3. **Decision**: Is the attack credible?
   - **Yes**: Use `cao_handoff` to send both the synthesis AND the attack back to the synthesist with: "Revise the synthesis to incorporate this attack as residual uncertainty. Do NOT capitulate; reflect the disagreement in the structure." Write revised version to `<topic-slug>/synthesis.md` (overwrite original; the original is in transcript.md).
   - **No** (attack is weak / repeats earlier challenges / strawmans): note in transcript that the adversarial pass was attempted and failed to produce credible critique. The original synthesis stands.
4. Write a final report to the user pointing at `<topic-slug>/synthesis.md` and stop.

**Anti-patterns to avoid**:

- Running round 4 unconditionally. It's expensive, slows the panel, and produces noise if the synthesis was already honest about disagreement.
- Letting the devils_advocate repeat round-2 attacks. Round 4 attacks must be ON THE SYNTHESIS specifically, not on individual panelists' positions.
- Capitulating the synthesis to the attack rather than incorporating residual uncertainty. The attack is information, not a verdict.
