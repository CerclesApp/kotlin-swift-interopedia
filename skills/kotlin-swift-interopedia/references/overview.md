# Overview: Kotlin-Swift Interop Basics

## The Bridge

Kotlin → Objective-C headers → Swift. Not direct Swift export (that's an upcoming feature).
The iOS app does `import shared` to access the compiled XCFramework.

## Classes and Functions

```kotlin
class SimpleKotlinClass {
    fun simpleKotlinFunction(): String = "Hello"
}
```
```swift
SimpleKotlinClass().simpleKotlinFunction()  // Works directly
```

## Top-Level Functions and Properties

Top-level declarations in `MyFile.kt` appear in Swift as `MyFileKt` wrapper class:

```kotlin
// TopLevelFunction.kt
fun topLevelFunction() { println("Hello") }
val topLevelProperty = "Some value"
var topLevelMutableProperty = "Mutable"
```
```swift
TopLevelFunctionKt.topLevelFunction()
TopLevelPropertyKt.topLevelProperty          // readonly
TopLevelPropertyMutableKt.topLevelMutableProperty = "Changed"
```

## Types Overview

Simple types (String, primitives) and custom types work as args and return values.
See [types.md](types.md) for the full integer mapping table.

## Collections Mapping

| Kotlin | Swift |
|--------|-------|
| `List<T>` | `Array<T>` |
| `MutableList<T>` | `NSMutableArray` |
| `Array<T>` | `KotlinArray<T>` |
| `Set<T>` | `Set<T>` |
| `Map<K,V>` | `Dictionary<K,V>` |
| `MutableMap<K,V>` | `NSMutableDictionary` |

```swift
let a: Array<KotlinInt> = CollectionsKt.getList()
CollectionsKt.set(collection: [1, 2, 3])
```

## Exceptions

All Kotlin exceptions are unchecked. In Swift all exceptions are checked.

**Rule:** Functions that may throw **must** be annotated with `@Throws`. Without it, an exception crashes the app.

```kotlin
@Throws(Exception::class)
fun riskyFunction() { throw Exception("Oops!") }
```
```swift
do {
    try MyKt.riskyFunction()
} catch {
    print(error)
}
```

Undeclared exception:
```kotlin
fun crashesApp() { throw Exception("No @Throws!") }
```
→ Calling from Swift terminates the program.

For suspend functions: if not annotated with `@Throws`, only `CancellationException` propagates as NSError.

## API Visibility

- `public` → visible in Swift (default)
- `internal` → **not exported**, invisible from Swift
- `protected` → visible from Swift (Obj-C/Swift have no protected concept); discouraged

```kotlin
internal class InternalClass { }  // Not visible in Swift
```

## Interop Annotations (Experimental)

### @ObjCName — Rename for Swift

Opt-in: `kotlin.experimental.ExperimentalObjCName`

```kotlin
@ObjCName(swiftName = "MySwiftArray")
class MyKotlinArray {
    @ObjCName("index")
    fun indexOf(@ObjCName("of") element: String): Int = 1
}
```
```swift
let array = MySwiftArray()
let index = array.index(of: "element")
```

### @HiddenFromObjC — Exclude from Swift

Opt-in: `kotlin.experimental.ExperimentalObjCRefinement`

```kotlin
@HiddenFromObjC
fun myKotlinOnlyFunction() { }  // Not visible from Swift
```
Differs from `internal`: still accessible from other Kotlin modules.

### @ShouldRefineInSwift — Wrap in Swift

Opt-in: `kotlin.experimental.ExperimentalObjCRefinement`

Marks declaration as `swift_private` (gets `__` prefix). Intended for Swift-side refinement:

```kotlin
interface Person {
    @ShouldRefineInSwift
    val namePair: Pair<String, String>
}
```
```swift
extension Person {
    var name: (firstName: String, lastName: String) {
        let namePair = __namePair
        return (namePair.first! as String, namePair.second! as String)
    }
}
```

## KDoc Comments

Enable with compiler flag in `build.gradle.kts`:
```kotlin
targets.withType<KotlinNativeTarget> {
    compilations["main"].kotlinOptions.freeCompilerArgs += "-Xexport-kdoc"
}
```
- In Xcode: Option+Double-click to see docs
- Limitations: `@property` on constructors not visible; many KDoc features unsupported
