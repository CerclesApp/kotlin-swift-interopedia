# Functions and Properties Reference

## Member Functions

```kotlin
class UsualClassFunction {
    fun someFunction() { }
}
```
```swift
let myClass = UsualClassFunction()
myClass.someFunction()
```

## Constructors

All constructor parameters must be named in Swift:

```kotlin
class UsualClassConstructor(val param: String)
```
```swift
let _ = UsualClassConstructor(param: "123")  // param name required
```

## Member Properties

```kotlin
class UsualClassValProperty(val param: String) {
    val property: String get() = "123"
}
class UsualClassPropertyMutable(var param: String) {
    var property: String = "123"
}
```
```swift
// val → read-only
let myClass = UsualClassValProperty(param: "123")
let _ = myClass.param
let _ = myClass.property

// var → mutable
let myMutable = UsualClassPropertyMutable(param: "123")
myMutable.param = "467"
myMutable.property = "775"
```

## Top-Level Properties

Top-level properties are accessed via a `FileNameKt` wrapper:

```kotlin
// TopLevelProperty.kt
val topLevelProperty = "Some value"
// TopLevelPropertyMutable.kt
var topLevelPropertyMutable = "Some value"
```
```swift
let _ = TopLevelPropertyKt.topLevelProperty            // read-only
TopLevelPropertyMutableKt.topLevelPropertyMutable = "Changed"
```

## Lambda Arguments

Works well. Trailing closure syntax supported for single-lambda functions:

```kotlin
fun funcWithLambda(calculation: () -> Int): Int = 100 + calculation()
fun funcWithParametrizedLambda(parametrizedLambda: (String) -> String): String =
    parametrizedLambda("paramForLambda")
fun funcWithUnitLambda(unitLambda: () -> Unit) { unitLambda() }
```
```swift
let result1 = FunctionWithLambdaArgsKt.funcWithLambda(calculation: { return 2 })
let result2 = FunctionWithLambdaArgsKt.funcWithLambda { return 30 }  // trailing closure

let result3 = FunctionWithLambdaArgsKt.funcWithParametrizedLambda { arg in
    return "arg: \(arg)"
}
FunctionWithLambdaArgsKt.funcWithUnitLambda { print("called") }
```

## Functions Returning Lambdas

```kotlin
fun returnLambda(): () -> Unit = { println("Lambda") }
fun returnParametrizedLambda(): (String) -> Unit = { println("arg: $it") }
```
```swift
FunctionReturnsLambdaKt.returnLambda()()
FunctionReturnsLambdaKt.returnParametrizedLambda()("123")
```

## Functions with Overloads

When multiple functions share the same parameter names, Swift appends underscores to disambiguate:

```kotlin
fun overloadFunction(param: Int) { }
fun overloadFunction(param: Long) { }
fun overloadFunction(param: Float) { }
fun overloadFunction(param: Double) { }
fun overloadFunction(param: String) { }
fun overloadFunction(param: Boolean) { }
```
```swift
FunctionsWithOverloadsKt.overloadFunction(param: true)     // Bool
FunctionsWithOverloadsKt.overloadFunction(param_: 2.0)     // Double
FunctionsWithOverloadsKt.overloadFunction(param__: 2.0)    // Float
FunctionsWithOverloadsKt.overloadFunction(param___: 2)     // Int32
FunctionsWithOverloadsKt.overloadFunction(param____: 4)    // Int64
FunctionsWithOverloadsKt.overloadFunction(param_____: "x") // String
```

**Fix:** Use different parameter names across overloads:
```kotlin
fun anotherOverload(intParam: Int) { }
fun anotherOverload(longParam: Long) { }
fun anotherOverload(floatParam: Float) { }
```
```swift
FunctionsWithOverloads2Kt.anotherOverload(intParam: 1)
FunctionsWithOverloads2Kt.anotherOverload(longParam: 1)
FunctionsWithOverloads2Kt.anotherOverload(floatParam: 1.0)
```

## Functions with Default Arguments

**Without SKIE:** All arguments must always be specified.

```kotlin
class FunctionWithDefaultArgumentsClass {
    fun defaultParamsFunction(funcParam1: String, funcParam2: Int = 30): String = "def"
}
```
```swift
// Must specify all args, default values ignored
FunctionWithDefaultArgumentsClass().defaultParamsFunction(funcParam1: "1", funcParam2: 100)
```

**With SKIE:** Use `@DefaultArgumentInterop.Enabled` annotation:

```kotlin
class FunctionWithDefaultArgumentsClass(val arg1: Int = 1) {
    @DefaultArgumentInterop.Enabled
    fun functionWithDefaultArgument(arg2: Int = 2) { println(arg2) }
}
```
```swift
let defaultArguments = FunctionWithDefaultArgumentsClass(arg1: 123)
defaultArguments.functionWithDefaultArgument()  // arg2 uses default
```

## Constructors with Default Arguments

Same limitation as functions — all args required in Swift. SKIE does NOT add default arg support to constructors.

```kotlin
class ConstructorWithDefaultArgumentsClass(
    val param1: String,
    val param2: Int = 300,
    val param3: Boolean = false
)
```
```swift
// All params required
 ConstructorWithDefaultArgumentsClass(param1: "123", param2: 500, param3: false)
```

## Functions Expecting Lambda with Receiver

Extension function in argument becomes a lambda with the receiver as first parameter:

```kotlin
fun funcWithExtension(extension: UsualClassExample.() -> Unit) { /* ... */ }
```
```swift
FunctionWithExtensionKt.funcWithExtension(extension: { usualClassExample in
    usualClassExample.param1 = "changed"
})
```

## Functions with Receivers (DSL Pattern)

DSL-style extension functions in arguments become regular function parameters. Less idiomatic in Swift:

```kotlin
fun experiments(block: ExperimentsDsl.() -> Unit = {}) { /* ... */ }
```
```swift
dsl.experiments { e in
    e.enable(experiment: Experiment(key: "key1", description: "desc1"))
}
```

## Functions with Value Class Parameter

Value class itself is not in the .h file. The argument is expanded to its underlying primitive:

```kotlin
@JvmInline value class ValueClassExample(val t: Int)
fun valueClassUsageExample(v: ValueClassExample): String = "${v.t}"
```
```swift
// Type of v is Int32, not ValueClassExample
FunctionWithValueClassParameterKt.valueClassUsageExample(v: 40)
```

## Functions with Vararg Parameter

`vararg` maps to `KotlinArray<T>`, not Swift variadic syntax:

```kotlin
fun funcWithVararg(vararg item: String) { println(item.joinToString()) }
```
```swift
let arr = KotlinArray<NSString>(size: 10, init: { index in "\(index)" as NSString })
FunctionWithVarargParameterKt.funcWithVararg(item: arr)
```

## Inline Functions

Present in .h file and callable, but **not inlined** — they become regular function calls:

```kotlin
inline fun inlineFunction(action: () -> Unit) { action() }
```
```swift
InlineFunctionKt.inlineFunction { print("Inside") }  // Called as regular function
```
