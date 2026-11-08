---
name: windows-direct-execution
description: Launch Windows CLI programs directly and diagnose Codex "blocked by policy" execution rejections on Windows.
---

# Windows direct execution

## Launch existing programs

- Submit the executable path and its arguments directly to the normal execution tool.
- Run multiple programs through separate tool calls. Keep orchestration outside the PowerShell command text.
- For interactive input, enable the tool's supported stdin mechanism before launch. Keep credentials out of command arguments and files.
- Use the project's existing test runner when available.

## Avoid the verified false positive

Verified against stable [Codex 0.166.0](https://github.com/openai/codex/releases/tag/rust-v0.166.0): the Windows classifier can falsely flag a complex PowerShell wrapper containing both `UseShellExecute = false` and an unrelated HTTP(S) URL. Its [fallback classifier](https://github.com/openai/codex/blob/rust-v0.166.0/codex-rs/shell-command/src/command_safety/windows_dangerous_commands.rs#L104-L131) matches text without understanding that shell execution is disabled or that the URL is merely metadata. Check the installed version before applying this diagnosis to another failure.

Prefer direct invocation over constructing an inline PowerShell `ProcessStartInfo` launcher for an existing executable.

Keep `UseShellExecute = false` in legitimate .NET code that redirects input/output. Do not remove correct process settings, disguise keywords, or change security settings to suppress classification.

## Diagnose `blocked by policy`

Treat this as a generic Codex rejection whose cause needs evidence. Establish the rejecting component before attributing the failure to Windows or PowerShell policy, an automatic reviewer, or a blanket executable restriction.

1. Record the exact redacted command, literal error, Codex version, whether the process started, and the effective turn's `approval_policy`, `sandbox_mode`, and `approvals_reviewer`. Distinguish effective values from saved configuration defaults.
2. Inspect both explicit execution rules and the installed version's built-in classifier. In the verified release, [`codex execpolicy check`](https://github.com/openai/codex/blob/rust-v0.166.0/codex-rs/execpolicy/src/execpolicycheck.rs#L41-L50) omits the heuristic fallback; empty `matchedRules` therefore does not clear the classifier. Reproduce suspected false positives with offline classifier inputs rather than executing the rejected script.
3. Explain only the decision path supported by that evidence. In the reproduced case, the dangerous classification plus `approval_policy = "never"` yielded [`Forbidden`](https://github.com/openai/codex/blob/rust-v0.166.0/codex-rs/core/src/exec_policy.rs#L800-L820), and the [orchestrator rejected it before process creation](https://github.com/openai/codex/blob/rust-v0.166.0/codex-rs/core/src/tools/orchestrator.rs#L200-L202). The automatic reviewer need not run. `danger-full-access` did not bypass classification: `never` disabled approval prompts rather than approving every command. Changing only the reviewer would leave this rejection path intact.

The verified release uses the same [generic error text](https://github.com/openai/codex/blob/rust-v0.166.0/codex-rs/core/src/exec_policy.rs#L1129-L1136) for other dangerous classifications; the wording alone cannot identify this particular false positive.
