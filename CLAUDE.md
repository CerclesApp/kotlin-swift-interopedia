# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is the **Kotlin-Swift Interopedia** — a reference project documenting how Kotlin language features behave when consumed from Swift via Kotlin/Native's Objective-C bridge. It is not directly interoperable with Swift, but through an Objective-C bridge that the Kotlin/Native compiler generates.

The repo has three parts:
- `docs/` — Markdown articles documenting each language feature's interoperability
- `kotlin-swift-interopedia-samples/` — A KMP playground app (standard interop via KMP-NativeCoroutines)
- `kotlin-swift-interopedia-samples-skie/` — Same concept but using the [SKIE](https://skie.touchlab.co/) library for improved Swift interop

## Building the Sample Apps

Both sample projects are Kotlin Multiplatform (KMP) projects with an iOS app. They use Gradle for the Kotlin/shared module and Xcode for the iOS app.

**Build the shared Kotlin framework** (run from within the respective project directory):
```sh
cd kotlin-swift-interopedia-samples
./gradlew :shared:embedAndSignAppleFrameworkForXcode
# or for iOS simulator
./gradlew :shared:iosSimulatorArm64Binaries
```

**Open the iOS app in Xcode:**
```sh
open kotlin-swift-interopedia-samples/iosApp/iosApp.xcodeproj
```

The iOS app must be built through Xcode after the shared framework is assembled. There is no Android app module — the `composeApp` subproject referenced in `settings.gradle.kts` provides Android support.

## Project Architecture

### Shared Module (`shared/`)
- Package: `com.jetbrains.swiftinteropplayground`
- Contains Kotlin source files organized by interop category (same categories as `docs/`): `generics/`, `types/`, `moreaboutfunctions/`, `classes/`, `coroutines/`, `extensions/`, etc.
- The shared module compiles to a static XCFramework (`baseName = "shared"`) imported by the iOS app as `import shared`
- Targets: `iosX64`, `iosArm64`, `iosSimulatorArm64`, and `androidTarget`
- Opts into experimental annotations: `kotlin.experimental.ExperimentalObjCName` and `kotlin.experimental.ExperimentalObjCRefinement` across all source sets

### Standard Sample (`kotlin-swift-interopedia-samples/`)
- Uses **KMP-NativeCoroutines** (v1.0.0-ALPHA-21) for improved suspend function / Flow interop from Swift
- Uses **KSP** for annotation processing
- `-Xexport-kdoc` compiler flag is set so KDoc comments appear in Xcode

### SKIE Sample (`kotlin-swift-interopedia-samples-skie/`)
- Uses **SKIE** (v0.5.6) plugin (`co.touchlab.skie`) instead of KMP-NativeCoroutines
- Only contains Kotlin/Swift examples for features SKIE improves: enum classes, sealed classes, sealed interfaces, suspend functions, flows, and default arguments
- Adds `skieConfigAnnotations` as a dependency for SKIE configuration annotations

### iOS App (`iosApp/`)
- SwiftUI-based app
- Entry point: `InteropSamples.swift` — defines `InteropSection` and `InteropSample` structs, and organizes all samples into sections matching the docs categories
- Each category has a subfolder (e.g., `Classes/`, `Generics/`, `Types/`) with `*Example.swift` files that call into the shared Kotlin framework
- `ResultView.swift` renders sample output; results are either printed to the console or captured via `outputMarker = "Sample output: "`

### Documentation (`docs/`)
- One Markdown file per language feature, organized into subdirectories matching the interop categories
- Each article follows a consistent structure: explanation, Kotlin code, Swift usage, and (where applicable) a SKIE-improved version
- All articles link back to `README.md` as the table of contents

## Adding a New Interop Article

1. Add Kotlin source to `shared/src/commonMain/kotlin/com/jetbrains/swiftinteropplayground/<category>/`
2. Add the corresponding Swift example to `iosApp/iosApp/<Category>/<Feature>Example.swift`
3. Register the sample in `InteropSamples.swift` inside the appropriate `*Section()` function
4. Create the documentation article in `docs/<category>/<Feature Name>.md`
5. Add the article link to `README.md`'s table for that category
6. If SKIE improves the feature, mirror the changes in `kotlin-swift-interopedia-samples-skie/`
