# Agent Skills

A [Cursor plugin marketplace](https://github.com/cursor/plugin-template) of individually installable agent skills.

Each skill is its own plugin under `plugins/`. Add this GitHub repo as a marketplace, then install only the skills you want. Cloud Agents pick up **team** and **project** plugins; they do not sync skills from your laptop.

## Plugins

| Plugin | Description |
|--------|-------------|
| [codex-computer-use](plugins/codex-computer-use/skills/codex-computer-use/SKILL.md) | Delegate local app verification and GUI interaction to Codex CLI |
| [codex-implementation](plugins/codex-implementation/skills/codex-implementation/SKILL.md) | Delegate scoped code changes to Codex CLI, then review and verify the result |
| [codex-review](plugins/codex-review/skills/codex-review/SKILL.md) | Ask Codex CLI for an independent code review |
| [dotnet-build-sarif](plugins/dotnet-build-sarif/skills/dotnet-build-sarif/SKILL.md) | Collect compiler and analyzer findings as SARIF 2.1 from `dotnet build` via ErrorLog |
| [file-based-csharp](plugins/file-based-csharp/skills/file-based-csharp/SKILL.md) | Create, run, and publish single-file C# programs without project files (.NET 10+) |

## Install one skill

### Team or Cloud Agents

1. In the [Cursor dashboard](https://cursor.com/dashboard), open **Plugins** → **Add Marketplace**.
2. Import this GitHub repository.
3. Leave plugins on **Default Off**, then install only the skills you need.

Cloud Agents load team-installed plugins automatically.

### Local (this machine)

Symlink a single plugin folder. The folder name should match the plugin `name`:

```bash
ln -s /absolute/path/to/skills/plugins/file-based-csharp ~/.cursor/plugins/local/file-based-csharp
```

Reload Cursor (**Developer: Reload Window**) and confirm the skill under **Customize**.

On Teams/Enterprise, admins must enable **Allow Local Plugin Imports**.

### Copy into a project (Cloud Agents without a team marketplace)

Copy one skill folder into the repo the Cloud Agent will clone:

```bash
mkdir -p /path/to/app/.cursor/skills
cp -R plugins/file-based-csharp/skills/file-based-csharp /path/to/app/.cursor/skills/
```

## Add a skill

See [docs/add-a-plugin.md](docs/add-a-plugin.md).

## Validate

```bash
node scripts/validate-template.mjs
```

## License

[MIT](LICENSE)
