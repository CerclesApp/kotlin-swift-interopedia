# Classes and Interfaces Reference

## Abstract Classes

Kotlin abstract classes are available in Swift but Xcode provides **no hints** to implement abstract methods. Calling an unimplemented abstract method crashes with `NSGenericException`.

```kotlin
abstract class AbstractClass(val param1: String) {
    abstract fun forOverride(): String
}
```
```swift
class ConcreteClass : AbstractClass {
    // Xcode does NOT warn that forOverride() must be overridden
    // Calling forOverride() at runtime crashes with NSGenericException
}
```

**Recommendation:** Prefer interfaces over abstract classes for Kotlin-Swift interop.

## Annotation Classes

Annotations are **not exported** to the `.h` file. Unavailable from Swift regardless of `@Retention`.

## Data Classes

Several auto-generated functions are renamed in Swift:

| Kotlin | Swift |
|--------|-------|
| `copy(...)` | `doCopy(...)` |
| `equals(...)` | `isEquals(...)` |
| `toString()` | `description` |

Destructuring (`componentN()`) is **not supported**.

```kotlin
data class DataClass(val param1: String, val param2: Int, val param3: Boolean)
```
```swift
let data = DataClass(param1: "abc", param2: 123, param3: true)
print("data = \(data)")                          // uses description
// doCopy requires ALL arguments (no default args in Swift)
```

## Enum Classes

**Without SKIE:** No real Swift enum is generated. A static object with entries is generated instead. Switch statements require a `default:` case.

```kotlin
enum class EnumClass(val type: String) {
    ENTRY_ONE("entry_one"), ENTRY_TWO("entry_two");
    companion object {
        fun findByType(type: String) = values().find { it.type == type }
    }
}
```
```swift
let e1 = EnumClass.entryOne
let _ = EnumClass.entryOne.name   // "ENTRY_ONE"
let _ = EnumClass.entryOne.type   // "entry_one"
EnumClass.companion.findByType(type: "entry_two")

switch enumClassExample {
    case .entryOne: print("entryOne")
    case .entryTwo: print("entryTwo")
    default: print("default")  // required!
}
```

**With SKIE:** Real Swift enums are generated. Switch is exhaustive (no `default:` needed).

## Fun Interfaces

Swift has no anonymous class syntax. Fun interfaces are essentially unusable from Swift in the same way.

```kotlin
fun interface FunInterfaceExample {
    fun singleFunctionInInterface(s: String): String
}
```
Swift cannot create an anonymous implementation concisely.

## Inline/Value Classes

**Not supported.** The class is not in the `.h` file. Functions taking a value class receive the underlying primitive instead (see [functions.md](functions.md)).

## Inner Classes

Minor syntax difference — outer class instance must be passed to the inner class constructor:

```kotlin
class OuterClass(val param: String) {
    inner class InnerClass { }
}
```
```swift
let _ = OuterClass.InnerClass(OuterClass(param: "1323"))
```

## Interfaces

Kotlin interfaces → Swift `@protocol`. Xcode generates stubs when implementing.

Note: `val` properties in a Kotlin interface are generated as `var` in Xcode stubs (can be manually changed to `let`).

```kotlin
interface Interfaces {
    val id: String
    fun simpleFunction(): String
    fun defaultParams(param1: String, param2: Int = 400): String
}
```
```swift
class InterfacesExample : Interfaces {
    func defaultParams(param1: String, param2: Int32) -> String { "..." }
    func simpleFunction() -> String { "..." }
    let id: String = "default"  // Changed from var to let manually
}
```

## Objects (Singletons)

Kotlin `object` → accessible via `.shared` in Swift:

```kotlin
object ObjectExample {
    const val CONST_VAL_EXAMPLE = "123"
    fun functionExample(): String = "result"
}
```
```swift
ObjectExample.shared.CONST_VAL_EXAMPLE
ObjectExample.shared.functionExample()
```

Even if created via `init()`, it remains a singleton.

## Companion Objects

Access via `.companion` in Swift:

```kotlin
class CompanionObjectClass {
    companion object {
        const val CONST_VAL_EXAMPLE = "123"
    }
}
```
```swift
CompanionObjectClass.companion.CONST_VAL_EXAMPLE
```

## Open Classes

Can inherit, override `open` methods, and access `protected` members. Overriding `final` (non-open) methods compiles but **throws NSException at runtime**.

```kotlin
open class OpenClassWithConstructorParams(val param1: String, val param2: Boolean) {
    protected val someField: String get() = "14"
    fun finalFunctionInClass() { }
    open fun functionCanBeOverridden() { }
}
```
```swift
class OpenClassExample : OpenClassWithConstructorParams {
    override func finalFunctionInClass() { }   // Compiles but crashes at runtime!
    override func functionCanBeOverridden() { }  // OK
}
let example = OpenClassExample(param1: "123", param2: true)
let _ = example.someField  // protected accessible
```

## Sealed Classes

**Without SKIE:** Generates a class hierarchy. Switch requires `default:` case (not exhaustive).

```kotlin
sealed class SealedClass {
    object Object : SealedClass()
    class Simple(val param1: String) : SealedClass()
    data class Data(val param1: String, val param2: Boolean) : SealedClass()
}
```
```swift
func example(s: SealedClass) {
    switch s {
    case is SealedClass.Object: print("object")
    case is SealedClass.Simple: print("simple")
    case is SealedClass.Data: print("data")
    default: print("Sad")  // required
    }
}
```

**Manual bridge pattern** (without SKIE):
```swift
enum SealedSwift {
    case object
    case simple(String)
    case data(String, Bool)
    public init(_ obj: SealedClass) {
        if obj is SealedClass.Object { self = .object }
        else if let obj = obj as? SealedClass.Simple { self = .simple(obj.param1) }
        else if let obj = obj as? SealedClass.Data { self = .data(obj.param1, obj.param2) }
        else { fatalError("Not synchronized") }
    }
}
```

**With SKIE:** Use `onEnum(of:)` — exhaustive, no `default:` needed:
```swift
func example(s: SealedClass) {
    switch onEnum(of: s) {
        case .object: print("object")
        case .simple(let simple): print("simple \(simple.param1)")
        case .data(let data): print("data \(data.param1) \(data.param2)")
    }
}
```

## Sealed Interfaces

**Without SKIE:** Generates unrelated protocols (not inherited from each other). No exhaustive switch possible.

```kotlin
sealed interface SealedInterfaces {
    interface First : SealedInterfaces { fun firstFunctionExample(): String }
    interface Second : SealedInterfaces { fun secondFunctionExample(): String }
}
```
```swift
func switchOnSealedInterfaces(sealedInterfaces: SealedInterfaces) {
    switch(sealedInterfaces) {
    case is SealedInterfacesFirst:
        print((sealedInterfaces as! any SealedInterfacesFirst).firstFunctionExample())
    case is SealedInterfacesSecond:
        print((sealedInterfaces as! any SealedInterfacesSecond).secondFunctionExample())
    default: print("default")  // required
    }
}
```

**With SKIE:** Same `onEnum(of:)` pattern as sealed classes.
