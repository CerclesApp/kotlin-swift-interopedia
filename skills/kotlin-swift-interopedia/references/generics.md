# Generics Reference

## Key Limitation: No Basic Type Generics

Kotlin/Swift generics interop goes through Objective-C, which **does not support generics over scalar/basic types**. You cannot use:
- `Int` / `Int8` / `Int16` / `Int32` / `Int64`
- `Float` / `Double`
- `String`
- `Bool`

as generic type arguments in Swift. Use `NSNumber`, `NSString`, or custom class wrappers instead.

## Generic Classes

```kotlin
class StateHolderWithoutAny<T>(data: T) {
    val myState = data
    fun pullState(): T = myState
}
```

Without bound, `T` is treated as `Any?` in Swift — requires force unwrap:
```swift
let result: NSString = StateHolderWithoutAny<NSString>(data: "'222'").pullState()!
```

**Fix:** Add `T : Any` bound in Kotlin to make it non-nullable:
```kotlin
class StateHolderWithAny<T : Any>(data: T) {
    val myState = data
    fun pullState(): T = myState
}
```
```swift
let result: NSString = StateHolderWithAny<NSString>(data: "'222'").pullState()
// No force unwrap needed
```

## Generic Functions

Without bound, `T` is `Any?`. Must use explicit type casts and no type inference:

```kotlin
fun <T> convert(data: T): T = data
```
```swift
let result1: Int = GenericFunctionsKt.convert(data: 12) as! Int
let result2: String = GenericFunctionsKt.convert(data: "'222'") as! String
```

With `T : Any` bound — becomes `Any` (non-optional), but still requires cast:
```kotlin
fun <T : Any> strictedGeneric(data: T): T = data
```
```swift
let result = GenericFunctionsKt.strictedGeneric(data: "hello") as! String
```

## Bounded Generics

Bounds are **not enforced** in Swift. The `.h` file loses the type restriction:

```kotlin
open class ForStricted
class ChildStricted : ForStricted()
class StrictedGeneric<T : ForStricted>(val data: T)
```
```swift
// Kotlin won't allow StrictedGeneric("123"), but Swift compiles this:
let _ = StrictedGeneric(data: NSString("1122"))  // No type error in Swift!
```

## Covariant Generics (`out T`)

Covariance (`out`) is noted in the `.h` file but **lost in Swift**. Requires explicit type cast:

```kotlin
class OutGeneric<out T>(data: T) {
    val myState = data
    fun pullState(): T = myState
}
// In Kotlin: OutGeneric<String> can be assigned to OutGeneric<Any>
```
```swift
// Swift doesn't honor covariance, must cast manually:
private func outGenericUsage(generic: OutGeneric<NSString>) {
    let _: OutGeneric<AnyObject> = generic as! OutGeneric<AnyObject>
}
```

## Contravariant Generics (`in T`)

Contravariance (`in`) is noted in the `.h` file but **doesn't compile** as expected. Requires type cast:

```kotlin
class InGenericItem<in T>
// In Kotlin: InGenericItem<SuperClass> can be used as InGenericItem<Child>
```
```swift
// Direct assignment doesn't compile; use type cast:
inGenericUsage(generic: InGenericItem<ChildClass>() as! InGenericItem<SuperClass>)

private func inGenericUsage(generic: InGenericItem<SuperClass>) {
    let _: InGenericItem<ChildClass> = generic as! InGenericItem<ChildClass>
}
```

## Reified Functions

**Not supported.** Crashes at runtime with `"unsupported call of reified inlined function"`.

```kotlin
inline fun<reified T> reifiedExample(marks: Int): T { /* ... */ }
```
```swift
// Compiles, but crashes at runtime:
let c = ReifiedFunctionsKt.reifiedFunction(marks: 23)
// Error: unsupported call of reified inlined function
```

Adding `@Throws(IllegalStateException::class)` converts the crash to a catchable NSError, but the function still fails.

## Star Projection

Use `MyGeneric<AnyObject>` with type cast:

```kotlin
class MyGeneric<T : Any>(val data: T) {
    fun someStarProjection(myGeneric: MyGeneric<*>) { }
}
```
```swift
let starProj = MyGeneric(data: NSNumber(12))
starProj.someStarProjection(myGeneric: starProj as! MyGeneric<AnyObject>)
starProj.someStarProjection(myGeneric: MyGeneric<AnyObject>(data: NSString("111")))
```

## Generic Interfaces

**Not supported.** Generic type information is **lost** in the `.h` file:

```kotlin
interface SocketConverter<T : Any> {
    fun convert(element: String): T
}
```
The `.h` file shows `convert(element:)` returning `id` (any object), not `T`.

## Summary

| Feature | Support | Workaround |
|---------|---------|-----------|
| Generic classes | Partial | Use `T : Any` bound; avoid basic type parameters |
| Generic functions | Partial | Explicit casts; use `T : Any` for non-nullable |
| Bounded generics | None | Bounds not enforced; document expected types |
| Covariant `out T` | Partial | Explicit `as!` cast |
| Contravariant `in T` | Partial | Explicit `as!` cast |
| Reified functions | None | Avoid; use alternative patterns |
| Star projection | Partial | Use `MyGeneric<AnyObject>` with cast |
| Generic interfaces | None | Use non-generic protocols with concrete types |
