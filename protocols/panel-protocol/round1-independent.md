# Round 1 — Independent Takes

**Purpose**: Get each panelist's unbiased first response. Prevent anchoring.

**Your actions this round**:

1. Frame the user's question as a debatable proposition. Write the framing to `<topic-slug>/framing.md`.
2. Dispatch the framed proposition to **every panelist listed in `dispatch` above** via `cao_assign` (parallel, non-blocking). Each panelist receives the SAME message — they must not see each other's responses.
3. Wait for all panelists to return their results via `send_message`. Their responses arrive in your inbox asynchronously.
4. Write each panelist's response to `<topic-slug>/round-1-<panelist-name>.md`.

**Anti-patterns to avoid this round**:

- Sending different framings to different panelists. The framing is the experimental variable to hold constant.
- Serializing the dispatch. If you `cao_handoff` instead of `cao_assign`, you're letting each panelist see prior responses — that destroys the independent-takes property.
- Editorializing in the dispatch message. Send the framed proposition and nothing else.

**Watch for** (flag in transcript, don't act on yet):

- Immediate convergence across panelists. If all three say the same thing in round 1, the framing is either too narrow or the topic isn't actually contentious. Note in transcript; round 2's devil's advocate will be asked to attack the consensus.
- Off-topic responses. If a panelist misunderstands the framing, decide whether to re-prompt or note the misunderstanding in transcript.
