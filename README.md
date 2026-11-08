# Skills

A collection of agent skills for common development workflows.

## Available Skills

| Skill | Description |
|-------|-------------|
| [codex-computer-use](skills/codex-computer-use/SKILL.md) | Delegate local app verification and GUI interaction to Codex CLI |
| [codex-implementation](skills/codex-implementation/SKILL.md) | Delegate scoped code changes to Codex CLI, then review and verify the result |
| [codex-review](skills/codex-review/SKILL.md) | Ask Codex CLI for an independent code review |
| [dotnet-build-sarif](skills/dotnet-build-sarif/SKILL.md) | Collect compiler and analyzer findings as SARIF 2.1 from `dotnet build` via ErrorLog |
| [file-based-csharp](skills/file-based-csharp/SKILL.md) | Create, run, and publish single-file C# programs without project files (.NET 10+) |
| [hydration-safe-browser-values](skills/hydration-safe-browser-values/SKILL.md) | Paint time and browser-only values correctly before React hydrates, with no flicker or mismatch |
| [upstream-dotnet-repro](skills/upstream-dotnet-repro/SKILL.md) | Create a minimal standalone .NET repro and prepare an upstream issue for user review |

## "I don't trust closed source software anymore."

— [Theo Browne](https://x.com/theo/status/2040518687007056114)

| Skill | Description |
|-------|-------------|
| [chatgpt-built-in-browser](skills/chatgpt-built-in-browser/SKILL.md) | Use the ChatGPT/Codex built-in browser and recover from unavailable-browser errors |
| [windows-direct-execution](skills/windows-direct-execution/SKILL.md) | Launch Windows CLI programs directly and avoid a verified Codex PowerShell classifier false positive |

## Usage

Skills live under `skills/` and are picked up automatically by agents when relevant to a task.
