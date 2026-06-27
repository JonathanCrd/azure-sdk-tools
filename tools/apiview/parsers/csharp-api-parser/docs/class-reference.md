---
title: Class Reference
tags: [apiview, csharp-parser, reference]
aliases: [Type Index, Class Index]
---

# Class Reference

> [!summary]
> A one-line-per-type index of everything that matters, with links to the deep-dive notes. Use this as
> a lookup table; use the linked pages for detail.

## In the parser project (`tools/apiview/parsers/csharp-api-parser`)

| Type | File | What it's for | More |
|---|---|---|---|
| `Program` | `CSharpAPIParser/Program.cs` | Static CLI entry point: parse args, unzip `.nupkg`, resolve deps, orchestrate the build, write JSON. | [[processing-pipeline]] |
| `Program.DependencyInfoComparer` | `Program.cs` (nested) | Equality by dependency **name** (case-insensitive) to de-dupe transitive deps. | [[compilation-and-dependencies#DependencyInfo]] |
| `CodeFileBuilder` | `CSharpAPIParser/TreeToken/CodeFileBuilder.cs` | The translator: walks the `IAssemblySymbol` and emits the `CodeFile` tree; runs analysis; validates. | [[codefilebuilder]] |
| `CodeFileBuilder.CodeFileBuilderEnumFormatter` | `CodeFileBuilder.cs` (nested) | Renders enum constant / `[Flags]` values used as attribute args. | [[codefilebuilder#Helper CodeFileBuilderEnumFormatter]] |

### Notable `Program` members

| Member | Role |
|---|---|
| `Main` / `HandlePackageFileParsing` | CLI wiring and the end-to-end routine. |
| `CreateOutputFile` | Serialize the `CodeFile` to `<name>.json`. |
| `CreateDummyCodeFile` | Build a placeholder review for meta-packages / missing symbols. |
| `ExtractNugetDependencies` | Download + extract allowlisted dependency DLLs to a temp dir. |
| `EnumerateDependenciesRecursivelyAsync` | Walk transitive (allowlisted) dependencies. |
| `SelectSpecificVersion` | Collapse a NuGet version **range** to one concrete version. |
| `IsNuget` / `IsNuspec` / `IsDll` | File-name predicates. |

### Notable `CodeFileBuilder` members

| Member | Role |
|---|---|
| `Build` | Orchestrate the whole translation. |
| `EnumerateNamespaces` / `BuildNamespace` | Namespace traversal + emission. |
| `BuildType` / `BuildBaseType` / `BuildClassModifiers` | Type emission. |
| `BuildMember` | Field/property/method/event/ctor emission. |
| `BuildExtensionMember` / `IsExtensionMemberContainer` / `IsExtensionMarkerMethod` | C# extension-block un-lowering. |
| `BuildAttributes` / `IsSkippedAttribute` / `BuildTypedConstant` | Attribute rendering. |
| `BuildDocumentation` | XML doc comments → comment tokens. |
| `DisplayName` / `MapToken` | Roslyn display parts → `ReviewToken`s (+ cross-links). |
| `IsAccessible` / `NeedsAccessibility` / `ToEffectiveAccessibility` / `IsHiddenFromIntellisense` | Visibility rules. |
| `BuildDependencies` / `BuildInternalsVisibleToAttributes` | The header sections. |
| `GetPackageVersion` / `GetLineId` / `VerifyCodeFile` | Version, ids, uniqueness gate. |

## In the referenced APIView project (`src/dotnet/APIView/APIView`)

| Type | File | What it's for | More |
|---|---|---|---|
| `CompilationFactory` | `Languages/CompilationFactory.cs` | Build a Roslyn `CSharpCompilation` and return the `IAssemblySymbol`. | [[compilation-and-dependencies]] |
| `DependencyInfo` | `Languages/DependencyInfo.cs` | `(Name, Version)` struct for a package dependency. | [[compilation-and-dependencies#DependencyInfo]] |
| `ICodeFileBuilderSymbolOrderProvider` | `Languages/ICodeFileBuilderSymbolSorter.cs` | Ordering contract for types/members/namespaces. | [[symbol-ordering]] |
| `CodeFileBuilderSymbolOrderProvider` | `Languages/CodeFileBuilderSymbolSorter.cs` | Default ordering (clients first, exceptions last, …). | [[symbol-ordering]] |
| `SymbolExtensions` | `Languages/SymbolExtensions.cs` | `GetId()` — stable symbol identifier for `LineId`/`NavigateToId`. | [[codefilebuilder#Identifiers LineId and NavigateToId]] |
| `CodeFile` | `Model/CodeFile.cs` | Root output object; serialization + text rendering. | [[token-model#CodeFile]] |
| `ReviewLine` | `Model/V2/ReviewLine.cs` | One line of the review; nests via `Children`. | [[token-model#ReviewLine]] |
| `ReviewToken` | `Model/V2/ReviewToken.cs` | One token (keyword/name/punctuation/…); factory helpers. | [[token-model#ReviewToken]] |
| `TokenKind` | `Model/V2/TokenKind.cs` | Enum of token kinds. | [[token-model#TokenKind]] |
| `CodeDiagnostic` | `Model/CodeDiagnostic.cs` | A guideline finding attached to a line. | [[token-model#CodeDiagnostic]] |
| `Analyzer` | `Analysis/Analyzer.cs` | Runs Azure SDK analyzers, collects diagnostics. | [[analysis-and-diagnostics]] |
| `SdkAnalyzerAdapter` | `Analysis/SdkAnalyzerAdapter.cs` | Bridges `Azure.SdkAnalyzers` into the pipeline. | [[analysis-and-diagnostics#SdkAnalyzerAdapter]] |

## Tests (`CSharpAPIParserTests`)

| Type | What it covers | More |
|---|---|---|
| `CodeFileTests` | End-to-end builds of real packages (Template/Storage/Core): metadata, line shapes, hidden APIs, unique ids, schema, nav, text. | [[build-test-run#Tests]] |
| `ProgramTests` | `SelectSpecificVersion` range collapsing. | [[build-test-run#Tests]] |
| `CodeFileBuilderTests` | `GetPackageVersion` build-metadata stripping. | [[build-test-run#Tests]] |
| `TestData` | The JSON schema used to validate generated token files. | [[build-test-run#Tests]] |
