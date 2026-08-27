---
name: file-based-csharp
description: Create, run, and publish single-file C# programs without project files. Use when building scripts, utilities, prototypes, or learning C# with .NET 10+. Triggers on file-based C# apps, single-file programs, dotnet run with .cs files, or shebang scripts.
---

# File-Based C# Programs

File-based apps let you build, run, and publish .NET applications from a single `*.cs` file without a traditional project file (`*.csproj`). Requires **.NET 10 SDK or later**.

## Quick Start

Create a file named `hello.cs`:

```csharp
Console.WriteLine("Hello, world!");
```

Run it:

```bash
dotnet run hello.cs
# or shorthand:
dotnet hello.cs
```

## Supported Directives

Place directives at the top of the C# file, after the optional shebang.

### `#:package` - Add NuGet Packages

```csharp
#:package Newtonsoft.Json@13.0.3
#:package Serilog@3.1.1
#:package Spectre.Console@*  // Latest version
```

When using central package management (`Directory.Packages.props`), version can be omitted.

### `#:property` - Set MSBuild Properties

```csharp
#:property TargetFramework=net10.0
#:property PublishAot=false
#:property OutputPath=./output
```

### `#:project` - Reference Other Projects

```csharp
#:project ../SharedLibrary/SharedLibrary.csproj
```

### `#:sdk` - Specify SDK (default: Microsoft.NET.Sdk)

```csharp
#:sdk Microsoft.NET.Sdk.Web
#:sdk Aspire.AppHost.Sdk@13.0.2
```

## CLI Commands

### Run

```bash
dotnet run hello.cs
dotnet run --file hello.cs
dotnet hello.cs

# Pass arguments
dotnet run hello.cs -- arg1 arg2

# Pipe code from stdin
echo 'Console.WriteLine("test");' | dotnet run -
```

### Build

```bash
dotnet build hello.cs
dotnet build hello.cs --output ./bin
```

Output goes to temp directory by default. Override with `#:property OutputPath=./output`.

### Clean

```bash
dotnet clean hello.cs
dotnet clean file-based-apps          # Clean all cached file-based apps
dotnet clean file-based-apps --days 7 # Only unused for 7+ days
```

### Publish

Native AOT is enabled by default for optimized self-contained executables.

```bash
dotnet publish hello.cs
dotnet publish hello.cs --output ./publish
```

To disable AOT: `#:property PublishAot=false`

### Pack as .NET Tool

```bash
dotnet pack hello.cs
```

`PackAsTool=true` by default. Disable with `#:property PackAsTool=false`.

### Convert to Traditional Project

```bash
dotnet project convert hello.cs
```

Creates a directory with a `.csproj` and cleaned `.cs` file.

### Restore Dependencies

```bash
dotnet restore hello.cs
```

## Unix Shebang Support

Make file-based apps directly executable on Unix:

```csharp
#!/usr/bin/env dotnet
#:package Spectre.Console@0.49.1

using Spectre.Console;

AnsiConsole.MarkupLine("[green]Hello, World![/]");
```

Then:

```bash
chmod +x hello.cs
./hello.cs
```

**Note:** Use `LF` line endings (not `CRLF`). No BOM in the file.

## Program Structure

File-based apps are regular C# programs with these characteristics:

- Use **top-level statements** or classic `Main` method
- Declare types (classes, records, structs) after top-level statements
- Access `args` for command-line arguments
- All C# syntax is valid

### Example with Types

```csharp
#!/usr/bin/env dotnet
#:package System.CommandLine@2.0.0

using System.CommandLine;

// Top-level statements first
var option = new Option<int>("--count", () => 1);
var root = new RootCommand { option };

root.SetHandler((count) =>
{
    for (int i = 0; i < count; i++)
        Console.WriteLine($"Hello #{i + 1}");
}, option);

return await root.InvokeAsync(args);

// Type declarations come after
public record Config(string Name, int Value);
```

## Launch Profiles

Create `hello.run.json` next to `hello.cs`:

```json
{
  "profiles": {
    "dev": {
      "commandName": "Project",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development"
      }
    }
  }
}
```

Run with profile:

```bash
dotnet run hello.cs --launch-profile dev
```

## User Secrets

```bash
dotnet user-secrets set "ApiKey" "secret-value" --file hello.cs
dotnet user-secrets list --file hello.cs
```

## Directory Layout Best Practices

**Avoid** placing file-based apps inside project directories:

```
# Bad
MyProject/
  MyProject.csproj
  scripts/
    utility.cs  # Inherits project settings

# Good
MyProject/
  MyProject.csproj
scripts/
  utility.cs  # Isolated
```

**Be mindful** of implicit MSBuild files (`Directory.Build.props`, `Directory.Build.targets`, `nuget.config`, `global.json`) in parent directories - they affect file-based apps.

## Implicit Build Files

File-based apps respect:

- `Directory.Build.props` - MSBuild properties
- `Directory.Build.targets` - Build logic
- `Directory.Packages.props` - Central package management
- `nuget.config` - Package sources
- `global.json` - SDK version

## Build Caching

The SDK caches builds based on source content, directives, and SDK version. If changes don't trigger rebuilds:

```bash
dotnet clean hello.cs && dotnet build hello.cs
```

## Complete Example

```csharp
#!/usr/bin/env dotnet
#:package Colorful.Console@1.2.15
#:package System.CommandLine@2.0.0

using System.CommandLine;
using System.CommandLine.Parsing;

Option<int> delayOption = new("--delay")
{
    Description = "Delay between lines in milliseconds.",
    DefaultValueFactory = _ => 100
};

Argument<string[]> messagesArg = new("messages")
{
    Description = "Text to render as ASCII art."
};

var root = new RootCommand("ASCII Art Generator");
root.Options.Add(delayOption);
root.Arguments.Add(messagesArg);

var result = root.Parse(args);
if (result.Errors.Count > 0)
{
    foreach (var error in result.Errors)
        Console.Error.WriteLine(error.Message);
    return 1;
}

int delay = result.GetValue(delayOption);
var messages = result.GetValue(messagesArg) ?? [];

foreach (string msg in messages)
{
    Colorful.Console.WriteAscii(msg);
    await Task.Delay(delay);
}

return 0;
```

Run it:

```bash
dotnet run ascii.cs -- "Hello" "World" --delay 500
```

## References

- [File-based apps documentation](https://learn.microsoft.com/en-us/dotnet/core/sdk/file-based-apps)
- [Tutorial: Build file-based C# programs](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/file-based-programs)
