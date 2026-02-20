# Types Reference

## Basic Type Mappings

### Integer Types

| Kotlin | Swift |
|--------|-------|
| `Byte` | `Int8` |
| `UByte` | `UInt8` |
| `Short` | `Int16` |
| `UShort` | `UInt16` |
| `Int` | `Int32` |
| `UInt` | `UInt32` |
| `Long` | `Int64` |
| `ULong` | `UInt64` |

**Important:** Kotlin `Int` → Swift `Int32` (NOT Swift `Int`). If you need Swift `Int`, explicit conversion is required:
```swift
let swiftInt: Int = Int(types.intType(i: Int32(swiftIntType)))
```

### String and Boolean

Direct mapping, no conversion needed:
- Kotlin `String` → Swift `String`
- Kotlin `Boolean` → Swift `Bool`

### Float/Double

Direct mapping:
- Kotlin `Float` → Swift `Float`
- Kotlin `Double` → Swift `Double`

### Char

Kotlin `Char` → Swift `unichar` (awkward to use):

```swift
let c: unichar = types.charType(c: ("a" as NSString).character(at: 0))
```

## Optional (Nullable) Basic Types

Kotlin nullable primitives (except `String?` and `Char?`) become `Kotlin*` wrapper types:

| Kotlin | Swift |
|--------|-------|
| `Byte?` | `KotlinByte?` |
| `UByte?` | `KotlinUByte?` |
| `Short?` | `KotlinShort?` |
| `UShort?` | `KotlinUShort?` |
| `Int?` | `KotlinInt?` |
| `UInt?` | `KotlinUInt?` |
| `Long?` | `KotlinLong?` |
| `ULong?` | `KotlinULong?` |
| `Float?` | `KotlinFloat?` |
| `Double?` | `KotlinDouble?` |
| `Boolean?` | `KotlinBoolean?` |
| `String?` | `String?` ← direct |
| `Char?` | `Any?` ← loses type |

**With literals or nil** — works like Kotlin:
```swift
OptionalPrimitives(optionalByte: 1, optionalInt: 1, optionalString: "123", optionalBoolean: true)
```

**With Swift typed variables** — wrap with `Kotlin*` constructor:
```swift
KotlinInt(value: intType)
KotlinBoolean(value: booleanType)
KotlinFloat(value: floatType)
```

**With optional Swift types** — check nil first:
```swift
(optionalInt != nil) ? KotlinInt(value: optionalInt!) : nil
```

## Collections with Basic Types

Elements of basic types (except `String`) become `Kotlin*` wrappers:

| Kotlin | Swift |
|--------|-------|
| `List<Int>` | `[KotlinInt]` |
| `List<Long>` | `[KotlinLong]` |
| `List<Float>` | `[KotlinFloat]` |
| `List<Double>` | `[KotlinDouble]` |
| `List<Boolean>` | `[KotlinBoolean]` |
| `List<String>` | `[String]` ← direct |
| `List<Char>` | `[Any]` ← loses type |

Passing literals works directly:
```swift
CollectionWithPrimitiveTypesKt.intList(list: [1, 2, 3])  // OK
```

Mapping from Swift `Int`:
```swift
let li: [KotlinInt] = CommonTypesKt.listType(
    list: intList.map { KotlinInt(value: Int32($0)) }
)
```

## Collections with Custom Types

No extra mapping needed:
```kotlin
fun notPrimitiveTypeList(list: List<NotPrimitiveType>): List<NotPrimitiveType> = list
```
```swift
let myList: [NotPrimitiveType] = CollectionsWithCustomTypesDataKt.notPrimitiveTypeList(list: inList)
```

## Mutable vs Immutable Collections

Regardless of Kotlin signature (`List` vs `MutableList`), both mutable and immutable can be passed from Swift. Mutability is controlled with `var`/`let`:

```swift
var mutableList: [KotlinInt] = [1, 2, 3]
let notMutableList: [KotlinInt] = [1, 2, 3]
mutableList = CommonTypesKt.listType(list: mutableList)  // OK
```

### MutableList

`MutableList` in Kotlin → `NSMutableArray` in Swift. Convert using `NSMutableArray(array:)`:

```swift
MutableImmutableCollectionsKt.mutableListType(list: NSMutableArray(array: notMutableList))
```

### MutableSet

`MutableSet` → needs `KotlinMutableSet`:
```swift
MutableImmutableCollectionsKt.mutableSetType(set: KotlinMutableSet(set: mutableSet))
// Cast back:
mutableSet = MutableImmutableCollectionsKt.mutableSetType(...) as! Set<KotlinInt>
```

### MutableMap

`MutableMap` → needs `KotlinMutableDictionary`:
```swift
MutableImmutableCollectionsKt.mutableMapType(map: KotlinMutableDictionary(dictionary: mutableMap))
// Cast back:
mutableMap = types.mutableMapType(...) as! Dictionary<String, KotlinInt>
```

## Unit and Nothing

- Kotlin `Unit` → Swift `KotlinUnit` (can be instantiated)
- Kotlin `Nothing` → Swift `KotlinNothing` (cannot be instantiated)

```swift
let example = UnitNothing()
example.unitType(p: KotlinUnit())
// example.nothingType(n: KotlinNothing())  // Won't compile
print(example.returnUnit())                 // Prints KotlinUnit
// example.returnNothing()                  // Crashes - throws exception
```
