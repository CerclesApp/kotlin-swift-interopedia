# Coroutines Reference

## Suspend Functions

### Default Interop (Callback)

Suspend functions become completion handler callbacks:

```kotlin
class ThingRepository {
    suspend fun getThing(succeed: Boolean): Thing {
        delay(100.milliseconds)
        if (succeed) return Thing(0) else error("oh no!")
    }
}
```
```swift
ThingRepository().getThing(succeed: true, completionHandler: { thing, error in
    // do something
})
```

### Experimental: async/await (Kotlin 1.5.30+)

```swift
Task {
    do {
        let thing = try await ThingRepository().getThingSimple(succeed: true)
        print("Thing is \(thing).")
    } catch {
        print("Found error: \(error)")
    }
}
```

**Problem:** No cancellation support. Even cancelled Tasks still complete:
```swift
Task {
    let thing = try await ThingRepository().getThingSimple(succeed: true)
    print("This IS printed even if Task is cancelled")
}.cancel()
```

### KMP-NativeCoroutines

Library: [github.com/rickclephas/KMP-NativeCoroutines](https://github.com/rickclephas/KMP-NativeCoroutines)

Supports: async/await, Combine, RxSwift. Provides real cancellation.

```kotlin
@NativeCoroutines
suspend fun getThing(succeed: Boolean): Thing { /* ... */ }
```
```swift
Task {
    do {
        let result = try await asyncFunction(for: ThingRepository().getThing(succeed: true))
        print("Got result: \(result)")
    } catch {
        print("Failed with error: \(error)")
    }
}
```

Cancellation works — the suspended function fails with `CancellationError`:
```swift
Task {
    let result = try await asyncFunction(for: ThingRepository().getThing(succeed: true))
    // Not reached if cancelled
}.cancel()
```

### SKIE

Library: [skie.touchlab.co](https://skie.touchlab.co/)

No annotation needed on Kotlin side. Directly compatible with Swift async/await.

```kotlin
// No annotation needed
suspend fun getThing(succeed: Boolean): Thing { /* ... */ }
```
```swift
Task {
    let result = try await ThingRepository().getThing(succeed: true)
    print("Got result: \(result)")
}
```

Cancellation: Task cancellation stops the coroutine immediately, nothing printed:
```swift
Task {
    let result = try await ThingRepository().getThing(succeed: true)
    // Not reached - Task cancelled
}.cancel()
```

## Flows

### Default Interop (Callback + Type Loss)

Flows require a manual `FlowCollector`. Generic type argument is **lost** — must use `Any?`.

```kotlin
class NumberFlowRepository {
    fun getNumbers(): Flow<Int> = flow {
        for (i in 1..10) { emit(i); delay(1.seconds) }
    }
}
```
```swift
class AnyCollector : Kotlinx_coroutines_coreFlowCollector {
    func emit(value: Any?) async throws {
        print("Got number: \(value!)")
    }
}

Task {
    try await NumberFlowRepository().getNumbers().collect(collector: AnyCollector())
}
```

**Problem:** No type safety (everything is `Any?`). No cancellation support.

### KMP-NativeCoroutines

```kotlin
class NumberFlowRepository {
    @NativeCoroutines
    fun getNumbers(): Flow<Int> = flow { /* ... */ }
}
```
```swift
Task {
    do {
        let sequence = asyncSequence(for: NumberFlowRepository().getNumbers())
        for try await number in sequence {
            print("Got number: \(number)")  // number is typed Int
        }
    } catch { print("Failed: \(error)") }
}
```

Cancellation: Flow stops emitting, no error thrown.

### SKIE

SKIE converts Flow to typed Swift `AsyncSequence`. No annotations needed.

```kotlin
// No annotation needed
fun getNumbers(): Flow<Int> = flow { /* ... */ }
```
```swift
Task {
    for await it in NumberFlowRepository().getNumbers() {
        print("Got number: \(it)")  // typed
    }
}
```

Cancellation: `AsyncSequence` processing stops immediately when Task is cancelled.

## Summary: Which Library to Use?

| Need | Recommendation |
|------|---------------|
| Coroutines + Flows only | Either SKIE or KMP-NativeCoroutines |
| Also want enum/sealed/default arg improvements | SKIE |
| Combine/RxSwift integration | KMP-NativeCoroutines (built-in adapters) |
| No annotation overhead in Kotlin | SKIE |
| Minimal Gradle changes | KMP-NativeCoroutines (annotation-based selection) |

### Setup Links
- KMP-NativeCoroutines: https://github.com/rickclephas/KMP-NativeCoroutines#installation
- SKIE: https://skie.touchlab.co/Installation
