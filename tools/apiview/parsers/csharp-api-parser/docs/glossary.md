---
title: Glossary
tags: [apiview, csharp-parser, glossary, reference]
aliases: [Terms, Definitions]
---

# Glossary

> [!summary]
> Roslyn and APIView terms used throughout these docs. Skim once; refer back as needed.

## APIView / token-schema terms

| Term | Meaning |
|---|---|
| **APIView** | The Azure SDK tool that renders and reviews a library's public API surface across languages. Consumes the token file this parser produces. |
| **Token file** | The `<name>.json` output. A serialized [[token-model#CodeFile|CodeFile]]. Stored in Azure blob storage and rendered by APIView. |
| **`CodeFile`** | Root output object: package metadata + a tree of review lines + diagnostics. → [[token-model#CodeFile]] |
| **`ReviewLine`** | One rendered line of the review; nests via `Children` to mirror code structure. → [[token-model#ReviewLine]] |
| **`ReviewToken`** | The smallest rendered unit (keyword, name, punctuation, literal, comment). → [[token-model#ReviewToken]] |
| **`TokenKind`** | Enum classifying a token (Keyword, TypeName, MemberName, Punctuation, …) for styling. → [[token-model#TokenKind]] |
| **`CodeDiagnostic`** | A guideline finding (id, message, target line, level) attached to the review. → [[token-model#CodeDiagnostic]] |
| **`LineId`** | Stable, unique id for a line (derived from the symbol). Anchors comments, navigation, and diffs. |
| **`NavigateToId`** | On a token, the `LineId` it hyperlinks to — used for cross-references between types. |
| **`NavigationDisplayName`** | On a token, the short label shown in APIView's navigation panel. |
| **`RelatedToLine`** | Ties a line to another (e.g. an attribute to its member) so they hide/show together in diff view. |
| **`SkipDiff`** | Marks a token to be ignored when diffing revisions (e.g. dependency versions). |
| **`IsContextEndLine`** | Marks a line that closes a scope — the `}` of a type/namespace. |
| **Tree-style vs. flat tokens** | The current schema nests lines (tree). An older "flat token list" format also exists; this parser only emits tree-style. → [[overview#What it is not]] |
| **Meta-package** | A `.nupkg` with no DLL (or no assembly symbol). The parser emits a placeholder review for it. → [[processing-pipeline#Edge cases and meta-packages]] |

## Roslyn terms

| Term | Meaning |
|---|---|
| **Roslyn** | The .NET Compiler Platform (`Microsoft.CodeAnalysis`). Provides the C# compilation and symbol model the parser walks. |
| **`IAssemblySymbol`** | Roslyn's view of a compiled assembly — the root of the symbol tree the parser traverses. |
| **`ISymbol`** | Base type for any program element (namespace, type, method, property, field, event, parameter, …). |
| **`INamespaceSymbol` / `INamedTypeSymbol` / `IMethodSymbol` / `IPropertySymbol` / `IFieldSymbol`** | Specific symbol kinds the [[codefilebuilder]] handles. |
| **`SymbolVisitor`** | Roslyn base class for visiting symbols. The [[analysis-and-diagnostics|Analyzer]] derives from it. |
| **`SymbolDisplayFormat`** | Configuration controlling how a symbol is rendered to text (nullability, defaults, generics, accessibility, …). The builder defines `_defaultDisplayFormat` and an id format. |
| **`SymbolDisplayPart`** | One piece of a formatted symbol (a keyword, a name, punctuation, a space). `MapToken` converts each part into a `ReviewToken`. → [[codefilebuilder#DisplayName and MapToken]] |
| **`Accessibility`** | A symbol's declared access (`public`, `protected`, `internal`, …). The builder reduces these to what a consumer sees. |
| **`MetadataImportOptions.Internal`** | Compilation option telling Roslyn to import internal members too — needed for `InternalsVisibleTo`/`[Friend]`. → [[compilation-and-dependencies#CompilationFactory]] |
| **`PortableExecutableReference`** | A metadata reference created from the DLL bytes; added to the compilation. |
| **`XmlDocumentationProvider`** | Supplies XML doc comments (from the package's `.xml`) so `GetDocumentationCommentXml` works. |
| **`TypedConstant`** | A compile-time constant used as an attribute argument (null, enum, `typeof`, array, primitive). Rendered by `BuildTypedConstant`. |

## NuGet terms

| Term | Meaning |
|---|---|
| **`.nupkg`** | A NuGet package — a zip containing the DLL(s), a `.nuspec`, and optional XML docs. The parser's input. |
| **`.nuspec`** | The package manifest (id, version, dependencies) inside a `.nupkg`. |
| **Version range** | A NuGet dependency constraint like `[1.0.0,)` or `(,2.0.0]`. `SelectSpecificVersion` collapses it to one version. → [[processing-pipeline#Version selection SelectSpecificVersion]] |
| **Allowlist** | The short set of foundational packages the parser resolves/references (`Azure.Core`, `System.ClientModel`, `System.Memory.Data`, `Microsoft.Bcl.AsyncInterfaces`). → [[compilation-and-dependencies#The dependency allowlist]] |
| **Transitive dependency** | A dependency of a dependency. Walked (allowlist-filtered) by `EnumerateDependenciesRecursivelyAsync`. |

## Azure SDK terms

| Term | Meaning |
|---|---|
| **`InternalsVisibleTo`** | Assembly attribute exposing internals to named assemblies; listed in the review's "Exposes internals to:" section. |
| **`[Friend]`** | Azure attribute marking an internal type/member as intentionally part of the reviewable surface. |
| **`Azure.ClientSdk.Analyzers` / `Azure.SdkAnalyzers`** | The analyzer libraries enforcing Azure SDK design guidelines, run by the [[analysis-and-diagnostics|Analyzer]]. |
| **Extension block** | A C# `extension(Receiver r) { ... }` member group. The compiler lowers it into hidden `<G>$`/`<M>$` containers, which the builder "un-lowers". → [[codefilebuilder#Extension members C 14 extension blocks]] |

## Back to start

Return to the [[README|Documentation Home]].
