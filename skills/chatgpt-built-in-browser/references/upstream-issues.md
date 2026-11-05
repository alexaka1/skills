# Browser access issues and recovery evidence

Checked November 5, 2026. These are public reports, not maintainer confirmation that every symptom shares a cause. Some concern external browsers; they inform diagnosis but do not expand this skill beyond the built-in browser. Status is a dated snapshot.

## Closest initialization and lifecycle reports

For the shared activation/reset procedure and its stopping conditions, read [recovery](recovery.md). The table records report-specific distinctions.

| Issue | Reported symptom |
| --- | --- |
| [#57721 — mobile-created macOS chats](https://github.com/openai/codex/issues/57721) (open) | Browser denied until the same chat is initialized through the desktop. |
| [#58921 — paired desktop connection](https://github.com/openai/codex/issues/58921) (open) | macOS client/Ubuntu host: browser works on the host; remote chat fails until opened there. |
| [#58752 — phone-created Linux chat](https://github.com/openai/codex/issues/58752) (open) | Denial disappears after desktop interaction without changing the permission. |
| [#57190 — existing macOS chat after restart](https://github.com/openai/codex/issues/57190) (open) | Existing desktop chat fails while other chats work; report discusses missing identity registration during resume. |
| [#56542 — Windows mid-session failure](https://github.com/openai/codex/issues/56542) (open) | IAB stops working; report includes a missing route log and recovery after restart. |
| [#56609 — macOS account switch](https://github.com/openai/codex/issues/56609) (open) | Existing chat loses browser access after a reported account switch. |

The [macOS recovery comment](https://github.com/openai/codex/issues/57721#issuecomment-6603872823) reports that supported navigation to the same chat restored access without a desktop message or permission change. It describes a missing executor mapping returning false before the Work setting is read.

The [Windows/IAB comment](https://github.com/openai/codex/issues/57721#issuecomment-6497348601) documents successful proactive activation, a same-turn retry that still failed after resume, and a later-turn success following reset. The reset's independent effect was not isolated. These observations support a bounded attempt, not a guaranteed repair.

Our local inspection/reproductions against desktop build 26.1029.61003 (24614) separately found: session identity registration can be omitted for both remote and desktop startup paths; a rejected authorization promise is retained per session/turn; and authorized IAB access can still lack its exact chat/window route. Hidden IAB page reads succeeded after supported same-chat activation, including after another chat was selected. Continuous foreground presence is therefore not universally required. The exact internal branch behind every historical denial was not recorded.

The current #57190 description mentions a candidate change capturing authenticated identity when startup/resume is sent, with focused checks passing and native end-to-end acceptance unverified. Do not describe this candidate as a released permanent fix. An instruction skill cannot repair the application's registration or cache implementation.

## Same error text, distinct or unconfirmed triggers

| Issue | Evidence and distinction | Agent response |
| --- | --- | --- |
| [#54989 — enabled Browser toggle on Windows](https://github.com/openai/codex/issues/54989) (open) | Same error; restart reportedly helps. A Browser toggle screenshot alone does not establish every effective Work permission. | Run bounded recovery; report the error as ambiguous rather than telling the user they disabled it. |
| [#56645 — Windows settings error](https://github.com/openai/codex/issues/56645) (open) | Same message despite reported browser/extension permissions. Exact cause unconfirmed. | Use bounded recovery; distinguish app-level access from extension and OS permissions. |
| [#59181 — Windows desktop blocked](https://github.com/openai/codex/issues/59181) (open) | Both external-browser calls and IAB selection fail; eventual recovery does not establish a cause. | Verify a real IAB page read after recovery; do not claim unexplained success proves a fix. |
| [#58295 — Windows host experience/mode](https://github.com/openai/codex/issues/58295) (open) | User-observed host ChatGPT/Codex mode change affects access, including remote requests. | Report mode as a possible distinct trigger if evidenced. Do not switch modes automatically or assert a registration cause. |
| [#58809 — macOS mid-chat loss](https://github.com/openai/codex/issues/58809) (open) | Initial declined/site-policy errors differ from later Work denial; user toggling Work browser access restores it. | Keep website denials separate; respect effective policy. Do not toggle permissions as automatic recovery. |
| [#56734 — local-site permission discrepancy](https://github.com/openai/codex/issues/56734) (closed) | Linux report of allowance in UI versus retained per-chat denial; exact authoritative control uncertain. | Report the denied origin and discrepancy without editing policy files or using another browser. |
| [#59370 — SSH workspace/tool availability](https://github.com/openai/codex/issues/59370) (open) | Windows/custom-provider setup can open an IAB pane but lacks browser-control tools and shows restricted availability. | Treat missing tools or explicit restrictions as a capability boundary. A rendered pane does not prove agent access. |

## Related failures surfaced during the investigation

These are included for completeness; the session-activation workaround is not established as their solution.

| Issue | Distinct failure | Agent response |
| --- | --- | --- |
| [#59482 — Chrome plugin missing](https://github.com/openai/codex/issues/59482) (open) | Plugin-directory/catalog problem; report also mentions unavailable built-in control. | Check whether supported IAB tools are exposed; report missing capability. Do not install Chrome as fallback. |
| [#59229 — bundled Browser plugin lookup](https://github.com/openai/codex/issues/59229) (open) | Missing-plugin UI error plus disabled discovery; native Chrome succeeds separately. | Report catalog/bootstrap failure separately. Native Chrome success does not authorize leaving IAB scope. |
| [#53986 — shared generated marketplace](https://github.com/openai/codex/issues/53986) (open) | Local/SSH sessions reportedly overwrite a bundled catalog, leaving only Visualize. | Report missing plugin/bootstrap capability; do not copy bundled files or rewrite the catalog as skill recovery. |
| [#23302 — older mobile-started bridge failure](https://github.com/openai/codex/issues/23302) (open) | Legacy Chrome client reports an untrusted/missing native bridge, a different error and interface. | Preserve trust checks; do not import legacy browser clients or spoof bridge identity. |
| [#52969 — Windows kernel asset initialization](https://github.com/openai/codex/issues/52969) (open) | Computer Use fails before app selection with asset-write/path error (`os error 3`). | Report runtime initialization failure; do not mislabel it as an off browser setting or assume repeated reset fixes it. |

## Product reference

The [official Browser documentation](https://learn.chatgpt.com/docs/browser) describes the desktop built-in profile, `@Browser`, agent website permissions, and the separate cloud Work browser. Follow current tool documentation for API details; do not hard-code installed runtime bundle paths from this investigation.
