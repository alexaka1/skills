# Setup & Troubleshooting

Read this when a computer-use delegation fails or when verifying the machine is set up.

## Quick health check

```bash
codex plugin list | grep computer-use     # expect: installed, enabled
codex features list | grep computer_use   # expect: stable  true
codex mcp list | grep computer-use        # expect an enabled MCP server entry
```

If the plugin is missing: computer use ships with the **Codex desktop app** (Codex.app),
not the bare CLI. The user installs it via Codex.app → Settings → Computer Use → Install.
The CLI picks it up automatically because the plugin registers a global `computer-use`
MCP server in `~/.codex/config.toml`.

## Smoke test (read-only, safe to run)

```bash
codex exec --skip-git-repo-check \
  -m gpt-5.6-sol -c model_reasoning_effort="low" \
  "Using your computer-use tools, list the running apps and name the frontmost one. \
   Do NOT click, type, scroll, or interact — observe only."
```

`list_apps` works without per-app approval, so this verifies the whole chain up to the
approval layer.

## Failure: approval denied

```
Computer Use approval denied via MCP elicitation for app '<bundle-id>'
```

Cause: each app must be approved by the user once ("Always allow"). Interactive Codex
surfaces a permission prompt; **headless `codex exec` auto-cancels it** (unconditional in
the exec code path — no flag or config overrides it). Approvals persist per app; once
granted, headless runs work for that app from then on.

Fix — tell the user to do this once per app:

1. Open the **Codex desktop app** (Codex.app).
2. Ask it to do something trivial with computer use in the target app, e.g.
   "Using computer use, tell me what's visible in TextEdit."
3. When the permission prompt for that app appears, choose **Always allow**.
4. Rerun the failed CLI command.

Do not attempt to write the approval store (`ComputerUseAppApprovals.json`) directly —
the format is undocumented and it bypasses a deliberate safety prompt.

## Failure: no computer-use tools at all

If Codex reports it has no computer-use capability:

- Plugin not installed/enabled → health check above.
- Feature disabled → try adding `--enable computer_use` to the command.
- Region: computer use is unavailable in the EEA, UK, and Switzerland on some accounts —
  if setup looks perfect but the capability never appears, this may be why.

## Failure: macOS permission errors

The computer-use client needs two system permissions (System Settings → Privacy &
Security):

- **Screen Recording** — to see app windows.
- **Accessibility** — to click and type.

Grant them to the Codex / Codex Computer Use helper app when prompted. These are separate
from the per-app approvals above.

## Under the hood (for debugging only)

Codex reaches the GUI through a `computer-use` MCP server exposing per-app tools:
`list_apps`, `get_app_state` (screenshot + accessibility tree), `click`, `set_value`,
`select_text`, `scroll`, `drag`, `press_key`, `type_text`, `perform_secondary_action`.
You never call these yourself — but if Codex's report mentions one failing, this is what
it's talking about. `get_app_state` is the first call that requires app approval, which
is why "approval denied" typically appears before Codex has seen anything.

Other facts that help interpret weird behavior:

- The MCP server runs **outside** Codex's sandbox, so `-s read-only` etc. don't block GUI
  actions; sandbox flags only govern Codex's shell commands.
- Screenshots go to Codex's model, not to you. If you need visual details, make Codex
  quote or describe them in its report.
- Codex blocks automating terminal apps and its own UI by design.
