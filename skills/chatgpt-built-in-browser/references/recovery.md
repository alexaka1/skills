# Recover IAB access

Use this procedure for `Local browser use is disabled in Work settings` or `Browser is not available: iab`. It permits one supported recovery attempt in the affected chat.

## Why recovery can help

Missing session identity registration can yield the Work-settings error before settings are read. IAB can separately lack the chat's window route. A rejected authorization check can remain cached for the current session/turn. Same-chat desktop activation can initialize registration or routing; resetting CUA's JavaScript runtime allows fresh handles and an access recheck.

Evidence: [#57721](https://github.com/openai/codex/issues/57721), its [macOS activation report](https://github.com/openai/codex/issues/57721#issuecomment-6603872823), its [Windows/IAB recovery observations](https://github.com/openai/codex/issues/57721#issuecomment-6497348601), and [#57190](https://github.com/openai/codex/issues/57190). These observations support a workaround, not guaranteed recovery or a shipped permanent fix.

## Procedure

1. Capture the exact error. A specific website denial, account boundary, locked computer, or explicit policy restriction ends this recovery path; report that constraint. The generic Work-settings message alone does not establish that the user disabled browsing.
2. Resolve the current chat ID from trusted session metadata and discover the dedicated app tool `navigate_to_codex_page` and its schema. Call it once with `{threadId: currentChatId}` to activate the same chat on its host. Continue only after successful navigation. If the ID or tool is unavailable, report that limitation; never guess an ID or use another chat. Use the dedicated tool rather than driving ChatGPT/Codex through native computer use or sending a desktop message.
3. After activation succeeds, call `mcp__cua_repl.js_reset` once. Reinitialize with exactly one documented entrypoint, read the returned instructions, and reacquire IAB/tab handles. Reset discards JavaScript bindings; it grants no permissions.
4. Retry the intended IAB page read once. A successful page read restores the browsing workflow. Otherwise report the exact remaining error and completed recovery steps, then end the retry loop. Preserve established facts instead of asking the absent user to repeat checks.

## Limits

An already-resumed chat may skip registration during activation. Reset alone may not clear every denial; the Windows report's later-turn success did not isolate its effect. A later user turn can recheck access, but do not manufacture turns to force retries. Describe unresolved access as unresolved; registration is a possible cause unless directly verified.

Recovery changes only same-chat activation and CUA runtime state. Do not restart the app, toggle permissions, change accounts, edit policy/session files or executor identities, patch bundles, or bypass access checks. Report any need for product repair separately.

For other failure classes or source attribution, consult [the complete issue index](upstream-issues.md).
