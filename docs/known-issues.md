# Known Issues

## BLOCKER: CAO banner-detection fails for claude_code workers

**Status**: Punted 2026-04-23 pending upstream fix. Project is blocked on this for any panelist using `claude_code` provider.

**Symptom**: `cao launch --provider claude_code ...` fails with:

```
500 Server Error: Internal Server Error
{"detail":"Failed to create session: Claude Code initialization timed out after 30 seconds"}
```

The same failure occurs for in-session `cao_assign` calls that spawn new `claude_code` workers.

**Evidence** (from `~/.aws/cli-agent-orchestrator/logs/cao_*.log`):

```
10:58:33  Created tmux session
10:58:36,433  Shell ready
10:58:36,492  send_keys: claude --dangerously-skip-permissions ...
10:58:36,860  Claude Code idle prompt detected, no prompts needed   ← claude IS ready ~370ms after send_keys
10:59:07,479  ERROR - Claude Code initialization timed out after 30 seconds
10:59:07,533  Killed tmux session
```

Claude Code boots in under a second. The inner `_handle_startup_prompts` loop detects the idle REPL prompt correctly. The OUTER loop in `ClaudeCodeProvider.initialize()` (at `cli_agent_orchestrator/providers/claude_code.py:277-295`) then runs for the full 30s without matching, and the init fails.

**Hypothesis**: The outer loop requires BOTH (a) banner markers in `new_content` (regex looks for `──────` horizontal rule, or the literal strings "Claude Code" / "bypass permissions" / "trust this folder") AND (b) `get_status()` returning IDLE/COMPLETED. With `--dangerously-skip-permissions` (passed unconditionally by CAO), claude may not print the banner markers the outer loop expects. The inner loop returns early on idle detection; the outer loop keeps waiting for a banner that never appears.

**Uncertainty**: This hypothesis is not independently verified. A prior successful launch of `code_supervisor` on `claude_code` (terminal e3bc3e33 in session cao-1021d701) booted fine on the same CAO version. Something about the `panel_moderator` profile (prompt length? option combination? timing?) is specific to this failure. Establishing the precise root cause requires a proper CAO dev bench, which this project does not have.

**Decision**: Do not patch AWS's installed code. File upstream issue; wait for fix. Keep the v0 repo as a complete reference implementation that will start working when CAO's banner detection is fixed upstream.

**What does NOT work today**:
- `cao launch --agents panel_moderator --provider claude_code ...`
- `bin/panel "<topic>"`
- In-session `cao_assign(agent_name="panel_moderator")` from any existing CAO supervisor on claude_code

**What MIGHT work today** (untested):
- Rerouting panelists to non-claude_code providers (`gemini_cli`, `codex`, `copilot_cli`). The banner bug is in `claude_code.py`; other providers have different init paths.
- Moderator on `codex` or `gemini_cli` is a design compromise — Claude is the strongest synthesizer — but would unblock v0 if tested successfully.

## Built-in CAO roles restrict Write/Edit

CAO's built-in roles (`supervisor`, `reviewer`, `developer`) are defined in `cli_agent_orchestrator/constants.py` as fixed allowed-tools sets. `supervisor` has only `@cao-mcp-server, fs_read, fs_list` — no write access. `developer` is the most permissive built-in (`@builtin, fs_*, execute_bash, @cao-mcp-server`).

**Workaround applied in this repo**:
- Panelist profiles use `role: developer` (has `fs_*` and `execute_bash`)
- Moderator uses `role: supervisor` + explicit `allowedTools` in frontmatter that include `fs_write` and `execute_bash`

## `role: worker` is not a built-in role

CAO has exactly three built-in roles: `supervisor`, `reviewer`, `developer`. An unknown role name (`worker`, `panelist`, anything else) produces:

```
Unknown role 'X' — falling back to unrestricted. Define custom roles in settings.json under 'roles'.
```

**Workarounds**: (a) use `role: developer`, or (b) define a custom role in `~/.aws/cli-agent-orchestrator/settings.json`:

```json
{
  "roles": {
    "panelist": ["@cao-mcp-server", "fs_read", "fs_list", "fs_write"]
  }
}
```
