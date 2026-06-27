---
title: Overview
tags: [apiview, csharp-parser, overview]
aliases: [What is the C# API Parser]
---

# 1. Overview

> [!summary]
> The C# API Parser reads a compiled `.nupkg`, uses **Roslyn** to load the library's public symbols,
> and emits a JSON **token file** describing the API surface. [APIView](https://apiview.dev) consumes
> that token file to render an API review with navigation, diffing, and SDK-guideline diagnostics.

## Why it exists

APIView is the Azure SDK team's tool for reviewing the **public API surface** of a library across
many languages (C#, Java, Python, TypeScript, Go, Swagger, etc.). Reviewers don't read source —
they read a normalized, language-aware rendering of the *shipped* API. For .NET, the artifact that
ships is a compiled assembly inside a `.nupkg`, so this parser works from **compiled metadata**, not
source code. That guarantees the review reflects exactly what consumers will see.

Each language has its own parser, but they all emit the **same token schema** (described in
`tools/apiview/parsers/CONTRIBUTING.md`). This project is the C#/.NET parser.

## What it produces

A single JSON file conforming to the **tree-style `CodeFile` schema**. It contains:

- **Package metadata** — name, version, language (`"C#"`), and parser version.
- **A tree of [[token-model|ReviewLine]] objects** — one per line of the rendered review (namespaces,
  types, members, attributes, doc comments), nested to mirror the code's structure.
- **A list of [[analysis-and-diagnostics|CodeDiagnostic]] objects** — Azure SDK guideline warnings/errors.

See [[token-model]] for the full shape of the output.

## What it is *not*

- It is **not** a source-code parser. It never reads `.cs` files; it reads a compiled `.dll`.
- It does **not** render anything. Rendering (HTML, navigation, diff) is done by APIView web.
- It only surfaces the **publicly visible API** (plus internals exposed via `InternalsVisibleTo`
  or `[Friend]`). Private implementation detail is intentionally excluded. See
  [[codefilebuilder#Accessibility and visibility]].

## How it's invoked

It's a command-line tool. The typical invocation is:

```text
CSharpAPIParserForAPIView --packageFilePath Azure.Storage.Blobs.12.21.2.nupkg \
                          --outputDirectoryPath ./out \
                          --outputFileName Azure.Storage.Blobs
```

The full CLI contract and runtime steps are documented in [[processing-pipeline]].

## Where the code lives

| Concern | Location |
|---|---|
| CLI entry point + package unpacking | `CSharpAPIParser/Program.cs` |
| Symbol → token translation | `CSharpAPIParser/TreeToken/CodeFileBuilder.cs` |
| Token model, compilation, ordering, analyzers | `src/dotnet/APIView/APIView/...` (referenced project) |
| Azure SDK lint rules | `src/dotnet/Azure.ClientSdk.Analyzers/...` (referenced project) |
| Tests | `CSharpAPIParserTests/` |

> [!note] Two `CodeFileBuilder` classes exist in the repo
> There is a legacy `CodeFileBuilder` in `src/dotnet/APIView/APIView/Languages/` that emits the *old*
> flat token format. **This parser does not use it.** The active class is
> `CSharpAPIParser.TreeToken.CodeFileBuilder` (see [[codefilebuilder]]), which emits the tree-style
> `ReviewLine` format. Don't confuse the two.

## Next

Continue to [[architecture]] to see how the components fit together.
