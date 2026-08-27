---
name: dotnet-build-sarif
description: Get SARIF from `dotnet build` via the compiler ErrorLog property, using a per-project relative path so later projects do not overwrite earlier logs. Use when collecting compiler or analyzer findings as SARIF, interpreting suppressed warnings, or when a user asks for ErrorLog, SARIF 2.1, or suppression-aware build diagnostics.
---

# Dotnet Build SARIF

Produce a SARIF 2.1 log of compiler and analyzer diagnostics from `dotnet build`. Suppressed diagnostics stay in the log. They are marked suppressed; they are not omitted.

## Produce the log

Pass the compiler `ErrorLog` property on the build command. Request SARIF 2.1 explicitly — the compiler default is SARIF 1.0.

```bash
dotnet build MySolution.slnx -v:minimal -p:ErrorLog='dotnet-build-error.sarif%2cversion=2.1'
```

Equivalent MSBuild:

```bash
dotnet msbuild MySolution.slnx -v:minimal -p:ErrorLog='dotnet-build-error.sarif%2cversion=2.1'
```

You can also set it in a project file:

```xml
<ErrorLog>dotnet-build-error.sarif,version=2.1</ErrorLog>
```

Rules:

- Use a **relative** filename such as `dotnet-build-error.sarif`. Each project writes next to that project (typically the project directory). Prefer a name already gitignored in the repo if one exists.
- Do **not** use a single absolute path for `ErrorLog`. Every project would write the same file, and later projects would overwrite earlier findings.
- Escape the comma before `version=2.1` as `%2c` when the value goes through an MSBuild `-p:` property. A literal comma is treated as an MSBuild property separator. A semicolon also works as the ErrorLog argument separator (`file.sarif;version=2.1`) and avoids the comma problem.
- Valid `version` values: `1`, `2`, `2.1`. `2` and `2.1` both mean SARIF 2.1.0.
- Run `dotnet clean` first unless the user asked for an incremental build. Incremental builds can skip a compile and leave a stale or missing log.

For a solution, enumerate projects with `dotnet sln <solution> list` if you need to collect the per-project files afterward.

## What the compiler writes

Each compile that honors `ErrorLog` writes one SARIF document. Active and suppressed diagnostics both appear under `runs[].results[]`.

A result that is **not** suppressed has no `suppressions` property (or an empty array):

```json
{
  "ruleId": "CS0219",
  "level": "warning",
  "message": { "text": "The variable 'unused' is assigned but its value is never used" }
}
```

A result that **is** suppressed is still present. The marker is a non-empty `suppressions` array:

```json
{
  "ruleId": "CS0219",
  "level": "warning",
  "message": { "text": "The variable 'alsoUnused' is assigned but its value is never used" },
  "suppressions": [
    {
      "kind": "inSource",
      "properties": {
        "suppressionType": "Pragma Directive"
      }
    }
  ]
}
```

Treat a result as suppressed when `suppressions` exists and has length greater than zero. Do not treat absence from the console build output as absence from SARIF — `#pragma warning disable`, `[SuppressMessage]`, and diagnostic suppressors still emit a result.

### Suppression object fields

| Field | Meaning |
| --- | --- |
| `kind` | Compilers emit `inSource` for pragma, `[SuppressMessage]`, and `DiagnosticSuppressor`. |
| `justification` | Present only for `[SuppressMessage(..., Justification = "...")]` with a non-null justification. |
| `properties.suppressionType` | `Pragma Directive`, `SuppressMessageAttribute`, or a `DiagnosticSuppressor { ... }` string. |

### Rule metadata is separate

Rule descriptors under `runs[].tool.driver.rules[]` can also carry suppression hints. These do **not** replace the per-result marker:

- `properties.isEverSuppressed` — `"true"` if the rule was suppressed in source or disabled by options for part or all of the compilation.
- `properties.suppressionKinds` — `inSource` and/or `external`.
  - `inSource`: at least one reported diagnostic was suppressed in source (and that diagnostic still appears in `results` with a `suppressions` array).
  - `external`: the diagnostic ID was disabled by `/nowarn`, a ruleset, globalconfig, editorconfig, and similar. Those diagnostics typically **do not** appear as results.

## Interpret findings

- Default: honor suppressions. Count and report **active** results (`suppressions` missing or empty) unless the user asks to include suppressed diagnostics.
- If the user says "include suppressed", "ignore suppressions", or equivalent, include results that have a `suppressions` marker and say so.
- When reporting a count, say whether it is active-only, suppressed-only, or all results in the raw log.
- Console `dotnet build` output omits in-source-suppressed warnings. SARIF does not. Use the marker, not the console, to decide.
