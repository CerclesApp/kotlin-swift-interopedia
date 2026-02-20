---
name: kotlin-swift-interopedia
description: >
  Reference guide for Kotlin-Swift interoperability via the Kotlin/Native
  Objective-C bridge. Use when working on KMP projects with an iOS target,
  or when questions involve Kotlin, Swift, KMP, iOS interop, SKIE, or
  KMP-NativeCoroutines. Covers all 63 documented features with support
  levels, type mappings, and workaround patterns.
triggers:
  - kotlin swift interop
  - KMP iOS
  - kotlin native swift
  - SKIE
  - KMP-NativeCoroutines
  - shared module swift
  - kotlin objective-c
---

# Kotlin-Swift Interopedia Reference

## How Interopedia Works

Kotlin is **not directly interoperable with Swift**. Instead:

```
Kotlin/Native → Objective-C headers (.h file) → Swift import
```

The Kotlin/Native compiler generates Objective-C headers that Swift imports via `import shared`. This means:
- Some Kotlin features have no Objective-C equivalent and are lost or degraded
- Swift sees Kotlin types through their Objective-C bridge representation
- Libraries like **SKIE** and **KMP-NativeCoroutines** improve specific pain points

## Feature Support Levels

| Level | Meaning |
|-------|---------|
| ✅ Works as expected | Kotlin feature maps cleanly to Swift |
| ⚠️ Workaround needed | Usable but requires boilerplate or type mapping |
| 🔧 Community solution | Use SKIE or KMP-NativeCoroutines for good interop |
| 🐛 Use with care | Partial support, type safety reduced |
| ❌ Not supported | Feature unavailable or crashes at runtime |

## Feature Support Matrix

### Overview / Interop Basics
| Feature | Support | Notes |
|---------|---------|-------|
| Classes and functions | ✅ | `SimpleKotlinClass().simpleKotlinFunction()` |
| Top-level functions | ⚠️ | Via `FileNameKt.functionName()` wrapper class |
| Top-level properties | ⚠️ | Via `FileNameKt.propertyName` wrapper class |
| Types (simple + custom) | ✅ | Pass as args, return from functions |
| Collections | ⚠️ | Mapped (see types reference) |
| Exceptions | ⚠️ | Must use `@Throws`; undeclared exceptions crash app |
| Public API visibility | ✅ | `public` visible; `internal` hidden from Swift |
| @ObjCName annotation | ✅ | Rename Kotlin constructs for Swift (experimental) |
| @HiddenFromObjC | ✅ | Hide Kotlin declarations from Swift (experimental) |
| @ShouldRefineInSwift | ✅ | Mark for Swift-side wrapper (experimental) |
| KDoc comments | ✅ | Export with `-Xexport-kdoc`; partial Xcode support |

### Functions and Properties
| Feature | Support | Notes |
|---------|---------|-------|
| Member functions | ✅ | `MyClass().myFunction()` |
| Constructors | ✅ | All params must be named in Swift |
| Read-only member properties | ✅ | `val` → read-only Swift property |
| Mutable member properties | ✅ | `var` → mutable Swift property |
| Top-level val properties | ⚠️ | Via `FileKt.propName` |
| Top-level var properties | ⚠️ | Via `FileKt.propName`, mutable |
| Functions with lambda args | ✅ | Works naturally, trailing closure syntax supported |
| Functions returning lambda | ✅ | Returns Swift function type |

### More About Functions
| Feature | Support | Notes |
|---------|---------|-------|
| Functions with overloads | ⚠️ | Same param names get underscores: `param`, `param_`, `param__` |
| Functions with default arguments | ⚠️ | All args required; SKIE adds `@DefaultArgumentInterop.Enabled` |
| Constructors with default arguments | ⚠️ | All args always required in Swift |
| Functions expecting lambda with receiver | ⚠️ | Receiver becomes first parameter |
| Functions with receivers (DSL) | ⚠️ | Extension in arg becomes `(ReceiverType) -> Unit` |
| Functions with value class parameter | ❌ | Value class not in .h; arg becomes primitive type |
| Functions with vararg | ❌ | Mapped to `KotlinArray<T>`, not Swift variadic |
| Inline functions | ❌ | Present in .h but not inlined; loses inline semantics |

### Types
| Feature | Support | Notes |
|---------|---------|-------|
| Basic types (String, Bool) | ✅ | Direct mapping |
| Integer types | ⚠️ | Kotlin Int→Int32, Long→Int64, etc. (see types reference) |
| Float/Double | ✅ | Direct mapping |
| Char | ⚠️ | Mapped to `unichar`; awkward to use |
| Optional basic types | ⚠️ | Requires `KotlinInt?`, `KotlinBoolean?`, etc. wrappers |
| Collections with custom types | ✅ | No extra mapping needed |
| Collections with basic types | ⚠️ | Requires `KotlinInt`, `KotlinFloat`, etc. wrappers |
| Mutable collections | ⚠️ | Use `NSMutableArray`, `KotlinMutableSet`, etc. |
| Unit / Nothing | ✅ | `KotlinUnit`, `KotlinNothing` in Swift |

### Classes and Interfaces
| Feature | Support | Notes |
|---------|---------|-------|
| Abstract classes | ⚠️ | No Xcode hints; crash if abstract method not overridden |
| Annotation classes | ❌ | Not exported to .h file |
| Data classes | 🐛 | `copy`→`doCopy`, `equals`→`isEquals`; no destructuring |
| Enum classes | ⚠️ | Static object, not Swift enum; requires `default:` case; SKIE → real Swift enum |
| Fun interfaces | ❌ | No anonymous class syntax in Swift |
| Inline/value classes | ❌ | Not in .h file |
| Inner classes | ⚠️ | `OuterClass.InnerClass(OuterClass(param:))` syntax |
| Interfaces | ✅ | Becomes `@protocol`; `val` becomes `var` in Xcode stubs |
| Objects (singletons) | ⚠️ | Access via `.shared`: `MyObject.shared.property` |
| Companion objects | ⚠️ | Access via `.companion`: `MyClass.companion.CONST` |
| Open classes | ✅ | Inherit, override `open` methods; overriding `final` crashes |
| Sealed classes | ⚠️ | Requires `default:` case; SKIE → `onEnum(of:)` pattern |
| Sealed interfaces | ⚠️ | Unrelated protocols; no exhaustive switch; SKIE improves |

### Coroutines
| Feature | Support | Notes |
|---------|---------|-------|
| Suspend functions | 🔧 | Callback by default; async/await experimental; use SKIE or KMP-NativeCoroutines |
| Flows | 🔧 | Callback with `Any?` type; use SKIE (AsyncSequence) or KMP-NativeCoroutines |

### Extensions
| Feature | Support | Notes |
|---------|---------|-------|
| Extension function over usual class | ✅ | `UsualClass().extensionFunction()` |
| Extension properties over usual class | ✅ | `UsualClass().extensionProperty` |
| Extension function over platform class | ⚠️ | Via wrapper: `FileKt.extensionFunction(receiver)` |
| Extension properties over platform class | ⚠️ | Via wrapper: `FileKt.propertyName(receiver)` |
| Extension props for companion of usual class | ⚠️ | Via `.companion.EXT_PROP` |
| Extension props for companion of platform class | ❌ | In .h file but inaccessible from Swift |

### Generics
| Feature | Support | Notes |
|---------|---------|-------|
| Generic classes | 🐛 | No basic type generics; `T` is `Any?`; use `<T: Any>` bound |
| Generic functions | 🐛 | `T` is `Any?`, no type inference; explicit casts required |
| Bounded generics | ❌ | Bound info lost in .h; Swift doesn't enforce |
| Contravariant generics (`in T`) | 🐛 | Requires explicit type cast |
| Covariant generics (`out T`) | 🐛 | Covariance lost; requires type cast |
| Reified functions | ❌ | Crashes at runtime: "unsupported call of reified inlined function" |
| Star projection | 🐛 | Use `MyGeneric<AnyObject>` with type cast |
| Generic interfaces | ❌ | Generic type info lost in .h |

## Category Quick Reference

Condensed summaries are in the `references/` directory:

- **[overview.md](references/overview.md)** — Interop basics, annotations, API visibility, KDoc
- **[functions.md](references/functions.md)** — All function/property patterns, overloads, lambdas, defaults
- **[types.md](references/types.md)** — Integer mapping table, Char, optional types, collections
- **[classes-and-interfaces.md](references/classes-and-interfaces.md)** — Classes, enums, sealed, interfaces, SKIE patterns
- **[coroutines.md](references/coroutines.md)** — Suspend functions and Flows, KMP-NativeCoroutines, SKIE
- **[extensions.md](references/extensions.md)** — Extension functions and properties for usual/platform classes
- **[generics.md](references/generics.md)** — Generic classes/functions, variance, limitations

## How to Use This Skill

When answering a question about Kotlin-Swift interop:

1. **Check the feature support matrix** above to determine if the feature works, needs a workaround, or is unsupported
2. **Consult the relevant reference file** in `references/` for condensed patterns with code examples
3. **Read the original doc** from the Documentation Index for the full explanation with all edge cases
4. **Recommend SKIE** for: enum classes, sealed classes, sealed interfaces, suspend functions, flows, default arguments
5. **Recommend KMP-NativeCoroutines** as an alternative to SKIE for coroutines (suspend + flows)
6. **Always note** that the bridge is Objective-C, so Swift-only features (protocols with PATs, etc.) don't apply on the Kotlin side
