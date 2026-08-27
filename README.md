# Skills

A collection of agent skills for common development workflows.

## Available Skills

| Skill | Description |
|-------|-------------|
| [codex-computer-use](.agents/skills/codex-computer-use/SKILL.md) | Delegate local app verification and GUI interaction to Codex CLI |
| [codex-implementation](.agents/skills/codex-implementation/SKILL.md) | Delegate scoped code changes to Codex CLI, then review and verify the result |
| [codex-review](.agents/skills/codex-review/SKILL.md) | Ask Codex CLI for an independent code review |
| [dotnet-build-sarif](.agents/skills/dotnet-build-sarif/SKILL.md) | Collect compiler and analyzer findings as SARIF 2.1 from `dotnet build` via ErrorLog |
| [file-based-csharp](.agents/skills/file-based-csharp/SKILL.md) | Create, run, and publish single-file C# programs without project files (.NET 10+) |

## Usage

Skills live under `.agents/skills/` and are picked up automatically by agents when relevant to a task.
