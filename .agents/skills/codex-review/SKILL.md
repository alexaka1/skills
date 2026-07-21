---
name: codex-review
description: Ask Codex CLI (gpt-5.6-sol) for an independent code review of uncommitted changes, a branch diff, a commit, or a specific implementation. This is how gpt-5.6-sol is invoked for review work. Use when the user asks Claude to have Codex or gpt-5.6-sol review work, when the model-selection rubric calls for a gpt-5.6-sol review perspective, or when Codex should audit a diff, find bugs or regressions, or compare Claude's implementation against requirements. For a review by Claude itself, use the normal review process instead.
---

# Codex Review

Use Codex as an independent reviewer when the user wants a second-pass review or when a change is broad enough that another agent's perspective is useful.

Prefer Claude's normal review process for small local checks. Do not delegate review just to avoid reading the code yourself. Treat Codex's output as evidence, not authority.

## Workflow

1. Identify the review target: uncommitted changes, base branch, commit SHA, PR checkout, or specific files.
2. Create a temporary artifact directory for the Codex report.
3. Run `codex review` with a focused review prompt.
4. Read Codex's report and verify important claims against the code before presenting them.

Use one of these command shapes:

```bash
ARTIFACT_DIR="$(mktemp -d "${TMPDIR:-/tmp}/codex-review.XXXXXX")"
REPORT="$ARTIFACT_DIR/report.md"
PROMPT="$ARTIFACT_DIR/prompt.md"

# Review staged, unstaged, and untracked changes with a custom review prompt.
# (No scope flag: passing a prompt defaults the scope to uncommitted changes.)
codex -C "$PWD" -m gpt-5.6-sol -c model_reasoning_effort="high" review - < "$PROMPT" > "$REPORT"

# Review current branch against a base branch (Codex's built-in review stance).
codex -C "$PWD" -m gpt-5.6-sol -c model_reasoning_effort="high" review --base main > "$REPORT"

# Review a single commit (Codex's built-in review stance).
codex -C "$PWD" -m gpt-5.6-sol -c model_reasoning_effort="high" review --commit <sha> > "$REPORT"
```

Always pass `-m gpt-5.6-sol -c model_reasoning_effort="high"` — review work runs on gpt-5.6-sol with high reasoning, regardless of the user's Codex defaults.

**Scope flags and a custom prompt are mutually exclusive.** `--uncommitted`, `--base`, and `--commit` cannot be combined with a `[PROMPT]` positional (the CLI errors out). So:

- For a **custom review prompt**, omit the scope flag and let it default to the uncommitted changes (first shape above). A `-` positional reads the prompt from stdin.
- For a **base-branch or commit** review, use the scope flag with no prompt; Codex applies its built-in review instructions, which already report severity, file/line, failure mode, and fix direction.

## Review Prompt

Ask Codex to use a code-review stance:

```text
Review these changes for bugs, regressions, missing tests, security issues, and requirement mismatches.

Prioritize findings over summary. For each finding include:
- severity
- file and line reference
- concrete failure mode
- suggested fix direction

Do not edit files. If there are no substantive findings, say so and name any residual test gaps.
```

Add task-specific context when useful: requirements, risky areas, expected behavior, relevant tests, or files Claude is unsure about.

## Reporting Back

Before relaying a Codex finding, inspect the cited code or diff enough to decide whether the finding is real. In the user-facing response, separate confirmed issues from Codex suggestions you did not verify.

If Codex finds nothing, say that clearly and mention what review target it inspected.

If `codex` is not installed or the command fails, report the error and offer to review the changes directly instead.
