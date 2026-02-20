# Extensions Reference

## Extension Functions

### Over Usual Class (Custom Type)

Works naturally — callable directly on a class instance:

```kotlin
class UsualClass {}
fun UsualClass.extensionFunction() { println("Successful call") }
```
```swift
UsualClass().extensionFunction()  // Direct call, no wrapper needed
```

### Over Platform Class (String, Int, etc.)

Extension on a platform/primitive type generates a **wrapper class** (`FileNameKt`). The receiver becomes a parameter:

```kotlin
// ExtensionFunctionOverPlatformClass.kt
fun String.extensionFunctionOverStringClass() { println(this) }
```
```swift
// Cannot call as "123".extensionFunctionOverStringClass()
ExtensionFunctionOverPlatformClassKt.extensionFunctionOverStringClass("123")
```

## Extension Properties

### Over Usual Class

Works naturally — accessible on a class instance:

```kotlin
val ExtensionPropertyUsualClass.extensionProperty: String get() = "123"
```
```swift
ExtensionPropertyUsualClass().extensionProperty  // Direct access
```

### Over Platform Class

Like extension functions on platform types, a wrapper class is used. The receiver becomes a parameter:

```kotlin
// ExtensionPropertyPlatformClass.kt
val String.myExtensionProperty: String get() = "789"
```
```swift
// Cannot call as "123".myExtensionProperty
ExtensionPropertyPlatformClassKt.myExtensionProperty("123")
```

## Extension Properties on Companion Objects

### Usual Class Companion

Accessible through `.companion`:

```kotlin
class ExtensionPropertiesCompanionObjectUsualClass {
    companion object {}
}
val ExtensionPropertiesCompanionObjectUsualClass.Companion.EXT_PROP: String get() = "456"
```
```swift
ExtensionPropertiesCompanionObjectUsualClass.companion.EXT_PROP
```

### Platform Class Companion (e.g., String.Companion)

**Not accessible from Swift.** The property appears in the `.h` Objective-C header but cannot be used from Swift:

```kotlin
val String.Companion.MY_CONST_VAL: String get() = "123"
```
```swift
// Cannot access — visible in .h but inaccessible in Swift
```

## Summary

| Extension Type | Swift Access Pattern |
|---------------|---------------------|
| Func on usual class | `UsualClass().extensionFunction()` |
| Func on platform class | `FileKt.extensionFunction(receiver)` |
| Prop on usual class | `UsualClass().extensionProperty` |
| Prop on platform class | `FileKt.extensionProperty(receiver)` |
| Prop on companion of usual class | `MyClass.companion.EXT_PROP` |
| Prop on companion of platform class | ❌ Inaccessible |
