---
name: chatgpt-built-in-browser
description: Invoke when the user requests @Browser (iab) or browsing inside the ChatGPT/Codex app, or when iab reports disabled Work settings or an unavailable browser.
---

# ChatGPT/Codex built-in browser

## Scope

Use only the desktop app's built-in browser through the supported `mcp__cua_repl` API. Select `iab`, or a provider verified by current inventory as type `iab`, including on a paired host whose tools are exposed to this chat. Use no external browser, extension, cloud browser, native-app browser control, standalone browser automation, or HTTP scraping fallback. An explicit user change of browser scope ends this workflow.

## Browse

1. Discover CUA's current tools. If exposed, initialize with exactly one documented entrypoint and read its returned instructions before continuing. For a known authorized URL, use `await cua.createBrowserTab("iab", url, { visible: false })`; show it when requested. Reuse a tab only after establishing that it belongs to IAB. If tools are absent, follow the failure guidance below.
2. Read the target page through the documented tab API before acting. Inventory or tab creation alone does not establish usable access. The built-in profile has its own sign-in session.
3. Complete the authorized task and verify the requested result in the page. Finish CUA with its required completion-summary call.

## Access failure

For disabled-Work-settings or unavailable-IAB errors, read [recovery](references/recovery.md) before declaring a block. Missing session registration or routing can cause these messages; supported same-chat activation followed by one runtime reset may restore access without the user at the computer.

For missing tools, plugin/bootstrap errors, or explicit policy denials, use [the issue index](references/upstream-issues.md) to distinguish the failure and its supported response. It contains all 18 reports from the investigation. Keep browser scope unchanged throughout recovery.
