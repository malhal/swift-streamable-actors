# swift-streamable-actors

**Swift Streamable Actors** is a lightweight Swift macro library that brings reactive streams to Swift Actors. It allows you to observe property changes across isolation boundaries safely and efficiently using `AsyncStream`.

---

## 📋 Features

* **No Boilerplate**: Automatically generates backing storage, observer tracking, and stream factories.
* **Swift 6 Ready**: Designed for strict concurrency with proper `@Sendable` enforcement and actor isolation.
* **Memory Safe**: Automatically cleans up observers via `onTermination` when the `AsyncStream` is cancelled.
* **Non-Invasive**: Use `@StreamableIgnored` to keep internal state private or prevent constants (`let`) from being streamed.

---

## 🚀 Quick Start

### 1. Define your Actor
Apply `@Streamable` to your actor. Every stored variable (except those marked ignored) will become streamable.

```swift
import StreamableActors

@Streamable
actor IslandManager {
    var count: Int = 0
    var status: String = "Idle"
    
    @StreamableIgnored
    var secretKey: String = "ABC-123"
    
    func update() {
        count += 1
        status = "Active"
    }
}

```

### 2. Observe Changes

Access the stream using the generated static factory method. These methods follow the naming convention `propertyNameStream(for:)`.

```swift
let manager = IslandManager()

Task {
    // Access the stream via the Actor type, passing the instance
    let stream = await IslandManager.countStream(for: manager)
    
    for await newCount in stream {
        print("Count updated to: \(newCount)")
        // Updates automatically when 'count' changes inside the actor
    }
}

```

---

## 🧠 How it Works

The `@Streamable` macro transforms your actor using a **Shadow Storage** pattern. This ensures that observation logic is injected without breaking standard Swift property initialization.

### The Transformation

When you apply the macro, the following happens behind the scenes:

1. **Storage Redirection**: A property like `var count: Int` is renamed to a private backing field `_count`.
2. **Accessor Injection**: The original `count` property becomes a computed property.
* The `get` returns `_count`.
* The `set` updates `_count` and then iterates through a private registry of observers to `yield` the new value.


3. **Static Factory**: A static method is added to your actor that creates an `AsyncStream`. It generates a unique `UUID` for the subscriber and handles the registration within the actor's isolated context.
4. **Automatic Cleanup**: The `AsyncStream` is configured with an `onTermination` handler. When the stream consumer stops listening (or the Task is cancelled), it triggers a background Task to call the actor and remove the observer from the registry, preventing memory leaks.

---

## 🛠 Installation

Add the package to your `Package.swift` dependencies:

```swift
dependencies: [
    .package(url: "[https://github.com/malhal/swift-streamable-actors](https://github.com/malhal/swift-streamable-actors)", from: "1.0.0")
]

```

Then add `StreamableActors` to your target dependencies.

---

## ⚠️ Requirements

* **Swift 5.9+** (Required for Macro support)
* **macOS 13+ / iOS 16+** (Utilizes modern `AsyncStream` and `Foundation` APIs)
* **Swift 6**: Recommended for the best experience with strict concurrency checking.

---

## ⚖️ License

This project is licensed under the MIT License.