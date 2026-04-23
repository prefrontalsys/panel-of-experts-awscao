# Known Issues

## CAO 30-second Claude Code cold-start timeout

**Symptom**: `cao launch` with `--provider claude_code` fails with:

```
500 Server Error: Internal Server Error
{"detail":"Failed to create session: Claude Code initialization timed out after 30 seconds"}
```

**Cause**: `cli_agent_orchestrator/providers/claude_code.py` hardcodes a 30-second deadline for Claude Code to print its startup markers (`────────` separator or "Claude Code" banner text). Claude Code installations with many MCP servers, hooks, and plugins routinely exceed 30s on cold start.

**Workaround**: Patch the installed CAO to extend the deadline. Edit:

```
~/.local/share/uv/tools/cli-agent-orchestrator/lib/python3.14/site-packages/cli_agent_orchestrator/providers/claude_code.py
```

Change:

```python
deadline = time.time() + 30.0
```

to:

```python
deadline = time.time() + 90.0
```

The patch survives until you `uv tool upgrade cli-agent-orchestrator`, at which point you must re-apply it.

**Proper fix**: Upstream PR making the timeout configurable via env var (e.g., `CAO_CLAUDE_INIT_TIMEOUT`) or `settings.json`. Not yet filed.

## Built-in CAO roles restrict Write/Edit

**Symptom**: On launch, the moderator shows `Blocked: Bash, Edit, Write` — which breaks its job of writing transcripts.

**Cause**: CAO's built-in roles (`supervisor`, `reviewer`, `developer`) are defined in `cli_agent_orchestrator/constants.py` as fixed allowed-tools sets. `supervisor` has only `@cao-mcp-server, fs_read, fs_list` — no write access.

**Workaround (applied in this repo)**: Profiles declare explicit `allowedTools` in frontmatter to override role defaults. The moderator profile sets:

```yaml
role: supervisor
allowedTools:
  - "@cao-mcp-server"
  - fs_read
  - fs_list
  - fs_write
  - execute_bash
```

Panelists use `role: developer` (which already includes `fs_*` and `execute_bash`).

## `role: worker` is not a built-in role

**Symptom**: `Unknown role 'worker' — falling back to unrestricted. Define custom roles in settings.json under 'roles'.`

**Cause**: CAO has exactly three built-in roles: `supervisor`, `reviewer`, `developer`. "Worker" is a colloquial term for "non-supervisor agent" but has no built-in mapping.

**Workaround**: Panelist profiles in this repo use `role: developer`. If a tighter tool set is needed, define a custom role in `~/.aws/cli-agent-orchestrator/settings.json`:

```json
{
  "roles": {
    "panelist": ["@cao-mcp-server", "fs_read", "fs_list", "fs_write"]
  }
}
```
