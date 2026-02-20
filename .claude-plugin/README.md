# kotlin-swift-interopedia Claude Plugin

A Claude Code plugin that provides a reference skill for **Kotlin-Swift interoperability** via the Kotlin/Native Objective-C bridge. Use it in any KMP project with an iOS target to get instant, accurate guidance on all 63 documented interop features.

## What It Does

When you invoke the skill, Claude has access to:

- **Feature Support Matrix** — quick lookup of support levels (✅ ⚠️ 🔧 🐛 ❌) for all 63 features
- **7 condensed reference files** — code-heavy summaries organized by category
- **63 full documentation articles** — complete explanations with all edge cases

## Installation

```sh
claude --plugin-dir /path/to/kotlin-swift-interopedia
```

Or add it to your project's `.claude/settings.json`:

```json
{
  "pluginDirs": ["/path/to/kotlin-swift-interopedia"]
}
```

## Skill Invocation

Once the plugin is loaded, invoke the skill with:

Ask a question — Claude will automatically use the skill when you mention Kotlin, Swift, KMP, iOS interop, SKIE, or KMP-NativeCoroutines.

## Categories Covered (63 features)

| Category | Features |
|----------|----------|
| Overview / Interop Basics | 11 features: classes, top-level functions/properties, types, collections, exceptions, API visibility, annotations, KDoc |
| Functions and Properties | 8 features: member functions, constructors, properties, lambda args, returning functions |
| More About Functions | 8 features: overloads, default arguments, value classes, vararg, inline, DSL receivers |
| Types | 6 features: integer mapping, optional types, collections with basic/custom types, mutable collections, Unit/Nothing |
| Classes and Interfaces | 13 features: abstract, annotation, data, enum, fun interface, inline/value, inner, interfaces, objects, companion, open, sealed, sealed interfaces |
| Coroutines | 2 features: suspend functions, flows (with SKIE and KMP-NativeCoroutines variants) |
| Extensions | 6 features: extension functions/properties over usual and platform classes, companion object extensions |
| Generics | 8 features: generic classes/functions, bounded, covariant, contravariant, reified, star projection, generic interfaces |

## Reference Files

| File | Contents |
|------|----------|
| `references/overview.md` | Interop basics, annotations, API visibility, KDoc |
| `references/functions.md` | All function/property patterns, overloads, lambdas, defaults |
| `references/types.md` | Integer mapping table, Char, optional types, collections |
| `references/classes-and-interfaces.md` | Classes, enums, sealed, interfaces, SKIE patterns |
| `references/coroutines.md` | Suspend functions and Flows, KMP-NativeCoroutines, SKIE |
| `references/extensions.md` | Extension functions and properties for usual/platform classes |
| `references/generics.md` | Generic classes/functions, variance, limitations |

## Source Repository

[github.com/kotlin-hands-on/kotlin-swift-interopedia](https://github.com/kotlin-hands-on/kotlin-swift-interopedia)
