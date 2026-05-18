# Known Issues

## RESOLVED — CAO banner-detection fails for claude_code workers

**Status**: Resolved 2026-05-17 against CAO main (commit at-or-after `eda4da2`).

**Original symptom (2026-04-23)**: `cao launch --provider claude_code ...` failed with:

```
500 Server Error: Internal Server Error
{"detail":"Failed to create session: Claude Code initialization timed out after 30 seconds"}
```

The inner `_handle_startup_prompts` loop detected the idle REPL prompt within ~370ms, but the outer init loop in `ClaudeCodeProvider.initialize()` timed out at 30s — apparently because the banner markers it scanned for (`──────`, "Claude Code", "bypass permissions", "trust this folder") never appeared with `--dangerously-skip-permissions`.

**Resolution**: CAO main now ships `_ensure_skip_bypass_prompt_setting()` which writes `skipDangerousModePermissionPrompt: true` into `~/.claude/settings.json` before launching claude. The bypass-permissions dialog no longer interrupts startup, so the banner renders reliably and the outer loop matches as expected. Test: `cao launch --agents panel_moderator --auto-approve --headless "ping"` completes in ~5s with the moderator responding "pong" — verified on Claude Code v2.1.41+ and CAO main as of 2026-05-17.

## ACTIVE (intentional upstream design, surprising in practice) — `cao launch --yolo` skips frontmatter `provider:`

**Symptom**: launching a profile that declares `provider: claude_code` in YAML frontmatter with the `--yolo` flag results in the agent launching on `kiro_cli` (the CAO default) instead. The warning banner reads `Agent 'X' launching UNRESTRICTED on kiro_cli` regardless of what the profile specifies.

**Root cause**: in `cli_agent_orchestrator/cli/commands/launch.py`, provider-resolution from frontmatter is gated inside the `else` branch of the permission-resolution conditional:

```python
if yolo:
    resolved_allowed_tools = ["*"]
elif allowed_tools:
    resolved_allowed_tools = list(allowed_tools)
else:
    profile = load_agent_profile(agents)
    ...
    if provider is None:
        provider = resolve_provider(agents, DEFAULT_PROVIDER)
```

When `--yolo` is passed, the profile is never loaded, `resolve_provider` is never called, and the subsequent fallback `if provider is None: provider = DEFAULT_PROVIDER` makes kiro_cli the launch target — regardless of the profile's declared provider.

**This is upstream-documented behavior, not a bug**. PR [#196](https://github.com/awslabs/cli-agent-orchestrator/pull/196) (MERGED 2026-04-22) explicitly placed the resolution inside the `else` branch and documented in its description: *"Fall back to DEFAULT_PROVIDER when --provider was not given and profile resolution didn't set it (yolo, --allowed-tools, or missing profile)."* The rationale is that `--yolo` implies "unrestricted; don't bother loading the profile." Frontmatter `provider:` is collateral damage of that decision.

**Practical impact for this repo**: panel-of-experts requires per-panelist provider routing to maintain its cross-family disagreement property. Launching with `--yolo` silently violates that property — every panelist would run on kiro_cli. This is exactly the homogeneous-models failure mode the project exists to prevent.

**Workarounds (in priority order)**:
1. **Use `--auto-approve` instead of `--yolo`**. Takes the profile-loading path, frontmatter provider is honored, frontmatter `allowedTools` restrictions are enforced (which `--yolo` would also override with `["*"]`). For panel-of-experts this is strictly better than yolo regardless of the provider issue.
2. **Pass `--provider <X>` explicitly with `--yolo`** if yolo is required for some reason.

**Possible upstream feature request**: argue that `provider:` is a separate concern from `allowedTools` and the two should be resolved independently. Run `resolve_provider()` outside the yolo/allowed_tools conditional. Not yet filed; the current behavior is intentional per PR #196, so a discussion or feature request would be the right shape, not a bug report.

## NOTE — `cao install --provider X` is misleading

The `cao install <file> --provider <X>` flag accepts a provider name at install time but does NOT persist that selection anywhere `cao launch` reads from. The provider is read solely from the profile's YAML frontmatter at launch time (and only in the non-yolo path, per the entry above).

**Recommendation**: always set `provider:` in the profile's YAML frontmatter. Treat the `cao install --provider` flag as informational only. This repo's `bin/panel-install` passes `--provider` for compatibility but the v0 profiles also declare `provider:` in frontmatter so launch resolution works correctly without depending on the install flag.

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
