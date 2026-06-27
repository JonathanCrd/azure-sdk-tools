---
title: Symbol Ordering
tags: [apiview, csharp-parser, ordering]
aliases: [SymbolOrderProvider, Sorting, CodeFileBuilderSymbolOrderProvider]
---

# 7. Symbol Ordering

> [!summary]
> The order in which types and members appear in a review is **deliberate and stable**, not just
> alphabetical. `CodeFileBuilderSymbolOrderProvider` defines that order so the most important API
> (clients first, exceptions last) is easy to scan and so diffs stay quiet when unrelated members move.

- **Interface:** `ICodeFileBuilderSymbolOrderProvider`
  (`src/dotnet/APIView/APIView/Languages/ICodeFileBuilderSymbolSorter.cs`)
- **Implementation:** `CodeFileBuilderSymbolOrderProvider`
  (`src/dotnet/APIView/APIView/Languages/CodeFileBuilderSymbolSorter.cs`)
- **Namespace:** `APIView.CSharp`
- **Wired up in:** `CodeFileBuilder.SymbolOrderProvider` (a settable property defaulting to the standard
  implementation — so it can be swapped in tests or future variants).

## The contract

```csharp
public interface ICodeFileBuilderSymbolOrderProvider
{
    IEnumerable<T> OrderTypes<T>(IEnumerable<T> symbols) where T : ITypeSymbol;
    IEnumerable<ISymbol> OrderMembers(IEnumerable<ISymbol> members);
    IEnumerable<INamespaceSymbol> OrderNamespaces(IEnumerable<INamespaceSymbol> namespaces);
}
```

[[codefilebuilder|CodeFileBuilder]] calls these at every level of the walk:
`OrderNamespaces` in `Build`, `OrderTypes` in `BuildNamespace`/`BuildType`, and `OrderMembers` in
`BuildType`/`BuildExtensionMember`.

## Namespaces

Sorted alphabetically by display string (`OrderNamespaces`). Simple and predictable.

## Types — `OrderTypes`

Sorted by a 3-part key: **(type-order weight, non-public last, name)**. The weight comes from
`GetTypeOrder`:

| Rule (checked top-down) | Weight | Effect |
|---|---:|---|
| Name ends with `Client` | **-100** | Clients first — the primary entry points. |
| Name ends with `Options` | -20 | Options classes near the top. |
| Name ends with `Extensions` | +1 | Extension classes later. |
| `TypeKind == Interface` | -1 | Interfaces early. |
| `TypeKind == Enum` | 90 | Enums near the end. |
| `TypeKind == Delegate` | 99 | Delegates after enums. |
| Name ends with `Exception` | 100 | Exceptions last. |
| Nested type | 3 | Nested types after top-level non-special types. |
| _otherwise_ | 0 | Default. |

Within the same weight, **public types precede non-public**, then ties break by name.

## Members — `OrderMembers`

Sorted by **(member-order weight, non-public last, name)**, with the weight from `GetMemberOrder`:

| Rule | Weight | Effect |
|---|---:|---|
| Enum field | its **constant value** | Enum members appear in numeric order. |
| Constructor | -10 | Constructors first. |
| Field | -6 | Fields next. |
| Property | -5 | Then properties. |
| Static method | -4 | Then static methods. |
| `Object`/`ValueType` override (e.g. `ToString`, `Equals`, `GetHashCode`) | 5 | Pushed toward the end. |
| _otherwise_ (instance methods, events) | 0 | Default. |

So a typical class reads: constructors → fields → properties → static methods → instance methods →
overridden object methods, with public members ahead of non-public at each tier.

> [!note] Ordering vs. accessor folding
> Ordering operates on the members `CodeFileBuilder` chooses to emit. Property/event **accessors**
> (get/set/add/remove) are filtered out before emission (see [[codefilebuilder#Members]]), so they
> never reach the sorter as standalone entries.

## Why it matters

- **Readability:** reviewers find clients and options immediately; noise (overrides, exceptions) sinks.
- **Stable diffs:** because order is derived from kind/name (not source order or reflection order), the
  same assembly always yields the same sequence — unrelated edits don't reshuffle the file.

## Next

See the guideline checks layered on top of the surface in [[analysis-and-diagnostics]].
