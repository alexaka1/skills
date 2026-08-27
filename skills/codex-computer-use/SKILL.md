---
name: codex-computer-use
description: >-
  Ask Codex CLI (gpt-5.6-sol) to run local app verification that needs computer use,
  browser automation, simulators, screenshots, app launching, or independent
  runtime inspection. This is how gpt-5.6-sol is invoked for computer-use work. Use
  when the user asks Claude to test a flow, verify UI behavior, inspect a
  running app, capture screenshots, or report confirmation and feedback about
  implemented behavior that benefits from computer use functionality.
---

# Codex Computer Use

You cannot see the user's screen or click anything — but the Codex CLI can. Codex has a
built-in computer-use capability (screenshots + accessibility tree + mouse/keyboard) and
already knows how to operate GUIs. Your job is only to:

1. Write a precise task brief.
2. Run `codex exec` with it.
3. Read Codex's report back and act on it.

Treat Codex like a capable hands-and-eyes subagent. Describe *outcomes*, not clicks —
don't micromanage how it should find buttons or which coordinates to use.

## Run a task

```bash
codex exec --skip-git-repo-check \
  -m gpt-5.6-sol -c model_reasoning_effort="low" \
  -o "$SCRATCHPAD/codex-report.md" \
  'Use your computer-use capability (do not substitute shell commands,
   AppleScript, or screencapture). In TextEdit: <what to accomplish>.
   <scope and boundaries>.
   <reporting instructions — see below>.'
```

- Always pass `-m gpt-5.6-sol` — computer-use work runs on gpt-5.6-sol regardless of the
  user's Codex defaults. Default to `-c model_reasoning_effort="low"`; use `"medium"` only
  for genuinely complex work. Never use a reasoning level higher than `"medium"`.
- `codex exec` is **non-interactive**: it cannot ask the user (or you) anything mid-run.
  Everything it needs — permissions, decisions, fallback behavior — must be in the brief.
- `-o <file>` writes Codex's final message to a file; read that for the result. stdout
  streams progress, which is useful to watch for errors. Add `--json` if you need
  machine-readable events.
- Run from the user's project directory when the task relates to it (Codex may need to
  read files); otherwise `--skip-git-repo-check` lets it run anywhere.
- Expect runs to take 1–5 minutes; set a generous Bash timeout (≥300s).

## Writing the brief

- **Mandate computer use — it is not optional for Codex.** You invoked this skill because
  you already decided the task needs GUI interaction; don't leave the method open or Codex
  will burn minutes trying shell commands, AppleScript, or `screencapture` first (and those
  often fail). Open every brief with an explicit directive: "Use your computer-use
  capability" plus "do not substitute shell commands, AppleScript, or screenshots via the
  terminal."
- **Name the target app up front.** Computer use operates per-app: "…in Safari: …". If
  you don't know which app, start with "list the running apps" as its own quick task.
- **State the goal and the stop condition.** "Type the text and leave the document
  unsaved" beats a click-by-click script — Codex sees the actual screen and you don't,
  so your imagined UI steps may not match reality.
- **Scope it.** Say what is out of bounds: "Do not touch any other app or window. Do not
  save, send, or close anything unless told."
- **Handle blockage explicitly.** "If any step fails or something unexpected appears
  (dialog, login prompt, missing element), do not improvise — stop and report exactly
  what you saw." Without this, a blocked non-interactive Codex may wander.
- **Demand a structured report**, since the final message is your only return channel:

  ```
  End your final message with:
  STATUS: done | blocked | failed
  DETAILS: what you did and what the screen showed, quoting exact on-screen text
  ```

  If you need specific data off the screen (a value, an error message, a list of tabs),
  ask for it verbatim — Codex saw it, you didn't.

## Multi-step control

Prefer one well-specified brief — each `codex exec` round-trip is slow. But when you need
to inspect intermediate state before deciding the next action, continue the same Codex
session so it keeps its context (app state, what it already did):

```bash
codex exec resume --last -m gpt-5.6-sol -c model_reasoning_effort="low" \
  "Now that the dialog is open: <next instruction>. Same reporting format."
```

## Safety: approvals go in the brief, or nowhere

Two layers matter:

1. **You → user.** Before delegating anything with real-world side effects — deleting
   data, sending messages/emails, submitting forms, purchases, logins, changing system
   settings — make sure the user actually asked for it or confirmed it. A vague request
   ("clean up my desktop") is not approval for destructive specifics.
2. **Codex's own policy.** Codex follows a confirmation policy for risky GUI actions and,
   being unable to prompt in exec mode, will stop rather than do them — *unless the
   initial prompt clearly pre-approves them*. So when the user has approved a risky step,
   say so explicitly in the brief: "The user has approved sending this exact message in
   Slack." Never claim an approval the user didn't give — that's the safety model both
   sides rely on.

## When it fails

The most common failure is on stderr/stdout even when Codex reports politely:

> `Computer Use approval denied via MCP elicitation for app 'com.google.Chrome'`

This means the target app is not on the user's "Always allow" list — headless Codex
auto-denies the per-app permission prompt. **This is not retryable by you.** The user
must approve that app once via the Codex desktop app; walk them through it using
[references/setup.md](references/setup.md), then rerun the same command.

For anything else (tools missing entirely, plugin not installed, macOS permission
errors), diagnose with [references/setup.md](references/setup.md).

## When NOT to use this

- **Terminal work** — Codex refuses to automate terminals (it has shell access, and so do
  you). Use Bash directly.
- **Fine-grained browser scripting** (form-by-form, selector-level control) when a
  dedicated browser automation skill/tool is available (e.g. agent-browser) — those are
  faster and more precise. Codex is the right choice when the task spans native apps,
  needs arbitrary screen state, or is a verify-this-flow check where Codex's independent
  judgment is the point.
- **Anything scriptable** — file operations, app launches (`open -a`), AppleScript-friendly
  automation. GUI driving is the slowest tool; use it when nothing else can do the job.
