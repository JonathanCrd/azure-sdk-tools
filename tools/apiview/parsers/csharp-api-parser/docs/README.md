---
title: C# API Parser — Documentation Home
tags: [apiview, csharp-parser, index, moc]
aliases: [C# API Parser Docs, CSharpAPIParser Home]
---

# C# API Parser — Documentation

> [!info] What is this?
> This folder documents the **C# API Parser** (`CSharpAPIParser`), the .NET tool that turns a
> compiled C# NuGet package (`.nupkg`) into an **APIView token file** (`*.json`). APIView renders
> that token file as a reviewable, navigable view of a library's public API surface.

This is a **map of content (MOC)**. Start with [[overview]] and follow the links. Every note is
self-contained and cross-linked so it works well in Obsidian and is easy for an agent to traverse.

## Reading order

1. [[overview|1. Overview]] — what the tool does and where it fits.
2. [[architecture|2. Architecture]] — the components and how data flows between them.
3. [[processing-pipeline|3. Processing Pipeline]] — the end-to-end run, step by step (`Program.cs`).
4. [[codefilebuilder|4. CodeFileBuilder]] — the heart of the parser: Roslyn symbols → tokens.
5. [[token-model|5. Token Model]] — the output objects: `CodeFile`, `ReviewLine`, `ReviewToken`.
6. [[compilation-and-dependencies|6. Compilation & Dependencies]] — loading the assembly with Roslyn.
7. [[symbol-ordering|7. Symbol Ordering]] — how members are sorted for a stable, readable review.
8. [[analysis-and-diagnostics|8. Analysis & Diagnostics]] — the Azure SDK lint rules run on the API.
9. [[build-test-run|9. Build, Test & Run]] — building, running, packaging, CI, and tests.

## Quick reference

- [[class-reference|Class Reference]] — one-line summary of every important type.
- [[glossary|Glossary]] — Roslyn and APIView terms used throughout these docs.

## At a glance

| | |
|---|---|
| **Project** | `tools/apiview/parsers/csharp-api-parser` |
| **Output type** | Console app (`Exe`), packaged as a .NET global tool |
| **Tool command** | `CSharpAPIParserForAPIView` |
| **NuGet package id** | `Microsoft.ApiView.CsharpParser` |
| **Target framework** | `net8.0` |
| **Input** | A C# package file (`.nupkg`) |
| **Output** | An APIView token file (`<name>.json`) following the tree-style `CodeFile` schema |
| **Core dependency** | Roslyn (`Microsoft.CodeAnalysis.CSharp`) for symbol analysis |
| **Sibling project** | `src/dotnet/APIView/APIView` — provides the token model, `CompilationFactory`, ordering, and analyzers |

## How the pieces connect (one diagram)

```mermaid
flowchart LR
    nupkg[".nupkg package"] --> prog["Program.cs<br/>(CLI + unpack)"]
    prog -->|dll + xml + deps| cf["CompilationFactory<br/>(Roslyn)"]
    cf -->|IAssemblySymbol| cfb["CodeFileBuilder"]
    cfb -->|walks symbols| tokens["CodeFile<br/>(ReviewLine / ReviewToken tree)"]
    cfb -.->|runs| analyzer["Analyzer<br/>(SDK lint rules)"]
    analyzer -.->|CodeDiagnostic[]| tokens
    tokens -->|SerializeAsync| json["<name>.json"]
    json --> apiview["APIView web<br/>(renders the review)"]
```

See [[architecture]] for the detailed version of this diagram and a component-by-component walkthrough.

## Related upstream docs

- `tools/apiview/parsers/CONTRIBUTING.md` — the cross-language token schema (CodeFile / ReviewLine / ReviewToken / CodeDiagnostic) that every APIView parser must emit.
