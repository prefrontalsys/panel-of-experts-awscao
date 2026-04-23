# Panel of Experts

Heterogeneous multi-model deliberation system built on [CAO (CLI Agent Orchestrator)](https://github.com/awslabs/cli-agent-orchestrator). A moderator coordinates four specialized panelists, each running on a deliberately different model family, to surface genuine disagreement and produce structured position documents instead of flattened consensus.

Originating concept: [simpleminded.bot — ChatGPT Panel of Experts](https://simpleminded.bot/posts/chatgpt-panel-of-experts) (2025-07-15 vault note). CAO is the execution substrate the original concept was waiting for.

## Why heterogeneous models

Running five panelists on the same model collapses the disagreement — they share training data, alignment tuning, and failure modes. Provider diversity is not a nice-to-have; it is the point. The panel's epistemic value comes from the disagreement surface between model families.

| Panelist | Provider (v0) | Rationale |
|---|---|---|
| Moderator | `claude_code` | Strong at orchestration + synthesis |
| Empiricist | `gemini_cli` | Fresh web context, fact-checking strength |
| Theorist | `claude_code` | Hypothesis generation, structural reasoning |
| Devil's Advocate | `codex` | Different model family for genuine disagreement |
| Synthesist | `claude_code` | Integrative, terse structured output |

(Provider assignments are v0 — refine based on observed panel behavior.)

## Architecture

```
User → Moderator (supervisor profile)
           ├── cao_assign → Empiricist (parallel, round 1)
           ├── cao_assign → Theorist (parallel, round 1)
           ├── cao_assign → Devil's Advocate (parallel, round 1)
           │
           ├── cao_handoff → sequential challenges (round 2)
           │
           └── cao_handoff → Synthesist (round 3, final)
```

Inter-agent communication uses CAO's native two-mode protocol:

- **Handoff (blocking)**: moderator dispatches, auto-captures return
- **Assign (non-blocking)**: moderator dispatches in parallel, panelists call `send_message` to return

Payload format: markdown, with absolute path references for large artifacts. Transcripts are written incrementally to `transcripts/<topic-slug>/`.

## Repository Layout

```
panel-of-experts/
├── README.md                  # this file
├── profiles/                  # CAO agent profiles (installable .md files)
│   ├── panel_moderator.md
│   ├── empiricist.md
│   ├── theorist.md
│   ├── devils_advocate.md
│   └── synthesist.md
├── bin/
│   ├── panel-install          # installs all 5 profiles via `cao install`
│   └── panel                  # launches a panel session for a topic
├── docs/
│   └── design-notes.md        # why heterogeneous, dialog protocol details
└── transcripts/               # panel session outputs (gitignored)
    └── <topic-slug>/
        ├── framing.md
        ├── round-N-<panelist>.md
        ├── transcript.md
        └── synthesis.md
```

## Installation

Prerequisites: CAO installed (`uv tool install cli-agent-orchestrator`), and each panelist's CLI available on PATH (`claude`, `codex`, `gemini`).

```bash
git clone <repo-url> ~/panel-of-experts
cd ~/panel-of-experts
./bin/panel-install
```

## Usage

```bash
./bin/panel "Should memory-augmented LLMs replace retrieval-augmented generation for long-context tasks?"
```

The launcher starts a CAO session named `panel-<timestamp>` with the moderator agent, which orchestrates the four panelists. Watch live via:

```bash
tmux attach -t panel-<timestamp>
```

Transcript lands in `transcripts/<topic-slug>/`.

## Design Status

- **v0 (current)**: 5 profiles, 4-round protocol, manual launch. Uses CAO's 7 built-in providers.
- **Known limitation**: CAO's provider list is hardcoded (7 CLIs). Expanding to arbitrary CLIs (Ollama, local models, custom wrappers) requires forking CAO or upstreaming a pluggable-provider PR. See `Agent/Memory/idea_cao_pluggable_providers.md` in the vault.
- **Future**: Provider-rotation per topic (assign empiricist to whichever CLI has strongest current web access), persistent panel memory across sessions, human-in-the-loop mid-panel injection.

## License

TBD — likely MIT to match CAO.
