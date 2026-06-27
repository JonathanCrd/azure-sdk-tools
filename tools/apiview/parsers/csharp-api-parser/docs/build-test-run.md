---
title: Build, Test and Run
tags: [apiview, csharp-parser, build, test, ci]
aliases: [Building, Running, CI, Tests]
---

# 9. Build, Test & Run

> [!summary]
> The parser is a standard .NET 8 console app packaged as a global tool. This note covers building it,
> running it against a `.nupkg`, the test suite, and how CI ships it.

## Solution layout

```text
tools/apiview/parsers/csharp-api-parser/
├─ csharp-api-parser.sln           # solution
├─ ci.yml                          # Azure Pipelines definition
├─ CSharpAPIParser/               # the tool
│  ├─ CSharpAPIParser.csproj
│  ├─ Program.cs                   # CLI + orchestration  → [[processing-pipeline]]
│  └─ TreeToken/CodeFileBuilder.cs # translator           → [[codefilebuilder]]
└─ CSharpAPIParserTests/           # xUnit tests
   ├─ CodeFileTests.cs
   ├─ ProgramTests.cs
   ├─ CodeFileBuilderTests.cs
   └─ TestData.cs                  # JSON schema for validation
```

## Project configuration (`CSharpAPIParser.csproj`)

| Setting | Value | Why |
|---|---|---|
| `TargetFramework` | `net8.0` | Runtime. |
| `OutputType` | `Exe` | Console app. |
| `PackAsTool` | `true` | Ships as a .NET global tool. |
| `PackageId` | `Microsoft.ApiView.CsharpParser` | NuGet id. |
| `ToolCommandName` | `CSharpAPIParserForAPIView` | The command name once installed. |
| `Nullable` / `ImplicitUsings` | `enable` | Modern C# defaults. |

**Package references:** `NuGet.Protocol` (download dependency packages), `System.CommandLine` (CLI).

**Project references:**

- `src/dotnet/APIView/APIView/APIView.csproj` — token model, `CompilationFactory`, ordering, `Analyzer`.
- `src/dotnet/Azure.ClientSdk.Analyzers/Azure.ClientSdk.Analyzers/Azure.ClientSdk.Analyzers.csproj` —
  the SDK guideline analyzers.

> [!note] Building requires the sibling projects
> Because of those `ProjectReference`s, you build from within the azure-sdk-tools repo (the references
> are relative paths up to `src/dotnet/...`). Building the parser pulls in the APIView library and the
> analyzers.

## Build

```powershell
# from the parser folder
dotnet build .\csharp-api-parser.sln
```

## Run

```powershell
dotnet run --project .\CSharpAPIParser\CSharpAPIParser.csproj -- `
  --packageFilePath C:\path\to\Azure.Storage.Blobs.12.21.2.nupkg `
  --outputDirectoryPath C:\path\to\out `
  --outputFileName Azure.Storage.Blobs
```

Or as an installed global tool:

```powershell
dotnet tool install --global Microsoft.ApiView.CsharpParser
CSharpAPIParserForAPIView --packageFilePath .\Some.Package.1.0.0.nupkg --outputDirectoryPath .\out
```

- `--packageFilePath` is **required** and must exist.
- Omitting `--outputFileName` uses the assembly/nuspec name.
- The positional `runAnalysis` argument defaults to `true`; pass `false` to skip the
  [[analysis-and-diagnostics|analyzers]].
- Output is `<outputDirectoryPath>\<name>.json`. The tool prints the path on success and writes errors
  to `stderr` (and returns non-zero) on failure.

Full CLI semantics: [[processing-pipeline#Command-line interface]].

## Tests

The test project (`CSharpAPIParserTests`) targets **net9.0**, uses **xUnit**, and references the
**real Azure packages** it parses (via `<PackageReference>` to `Azure.Template`, `Azure.Storage.Blobs`,
`Azure.Core`, and others). The static constructor in `CodeFileTests` loads each assembly, runs
`CompilationFactory.GetCompilation`, and builds a `CodeFile` once for reuse across tests.

```powershell
dotnet test .\csharp-api-parser.sln
```

What the suite verifies:

| Test (file) | Verifies |
|---|---|
| `TestPackageMetadata` (`CodeFileTests`) | Package name/version/language and top-level line count. |
| `TestClassReviewLineWithoutBase` / `WithBase` | Class lines render with/without base types correctly. |
| `TestMultipleKeywords` | `public readonly struct ... : IEquatable<...>` shape. |
| `TestApiReviewLine` / `MoreParams` | Method signatures (incl. generics, defaults) render exactly. |
| `NoDuplicateLineIds` | No duplicate `LineId`s (mirrors `VerifyCodeFile`). → [[codefilebuilder#Validation]] |
| `TestAllClassesHaveEndOfContextLine` | Every type is followed by an `IsContextEndLine` `}`. |
| `TestHiddenAPI` | `[EditorBrowsable(Never)]`/protected members still emit with correct text. |
| `TestAPIReviewContent` | Full `GetApiText()` output matches a golden string. → [[token-model#Serialization]] |
| `TestCodeFileJsonSchema` | Serialized `CodeFile` validates against the JSON schema in `TestData`. |
| `TestNavigationNodeHasRenderingClass` | Navigation tokens carry render classes. |
| `VerifyAttributeHAsRelatedLine` | Attribute lines link back to their owner via `RelatedToLine`. |
| `SelectSpecificVersion_Picks_Single_Version` (`ProgramTests`) | NuGet range → single version. |
| `GetPackageVersion_StripsBuildMetadata` (`CodeFileBuilderTests`) | `1.2.3+sha` → `1.2.3`. |

> [!tip] Updating golden tests
> `TestAPIReviewContent` and the metadata/line-count assertions pin exact output. If you intentionally
> change rendering, update these expectations **and** bump `CodeFileBuilder.CurrentVersion`
> (see [[codefilebuilder]]).

## CI / release (`ci.yml`)

- Triggers on changes under `tools/apiview/parsers/csharp-api-parser` (for `main`, `feature/*`,
  `release/*`, `hotfix/*`) and on PRs to those branches.
- Extends the shared template `eng/pipelines/templates/stages/archetype-sdk-tool-dotnet.yml` with
  `PackageDirectory` pointing at `CSharpAPIParser`.
- The release stage is enabled only on `main` (`SkipReleaseStage: false`), which packs and publishes
  the `Microsoft.ApiView.CsharpParser` tool.

Per the repo's pipeline naming convention: public build = `tools - csharp-api-parser - ci`,
internal build = `tools - csharp-api-parser`.

## Next

Unfamiliar with a term? See the [[glossary]].
