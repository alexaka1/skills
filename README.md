# Agent Skills

A [Cursor plugin](https://cursor.com/docs/plugins.md) that packages agent skills for common development workflows.

This repo follows the [plugin-template](https://github.com/cursor/plugin-template) **single-plugin** layout: plugin contents live at the repository root, with one `.cursor-plugin/plugin.json` and no `.cursor-plugin/marketplace.json`.

## Included

| Skill | Description |
|-------|-------------|
| [codex-computer-use](skills/codex-computer-use/SKILL.md) | Delegate local app verification and GUI interaction to Codex CLI |
| [codex-implementation](skills/codex-implementation/SKILL.md) | Delegate scoped code changes to Codex CLI, then review and verify the result |
| [codex-review](skills/codex-review/SKILL.md) | Ask Codex CLI for an independent code review |
| [dotnet-build-sarif](skills/dotnet-build-sarif/SKILL.md) | Collect compiler and analyzer findings as SARIF 2.1 from `dotnet build` via ErrorLog |
| [file-based-csharp](skills/file-based-csharp/SKILL.md) | Create, run, and publish single-file C# programs without project files (.NET 10+) |

## Install locally

```bash
ln -s /absolute/path/to/skills ~/.cursor/plugins/local/alexaka1-skills
```

Reload Cursor (**Developer: Reload Window**) and confirm the skills appear under **Customize**.

On Teams/Enterprise, admins must enable **Allow Local Plugin Imports**.

`.agents/skills` is a symlink to `skills/`, so agents in this checkout still load these as project skills.

## Marketplace

Submit the repository at [cursor.com/marketplace/publish](https://cursor.com/marketplace/publish).

## Validate

```bash
node scripts/validate-template.mjs
```

## License

[MIT](LICENSE)
