# Singleton Pattern

The Singleton Pattern is a creational design pattern. It ensures that a class has one shared instance and provides a single access point to it.

## Intent

Use a singleton when:

- exactly one instance should coordinate a shared resource
- the instance must be accessible across the application
- creating multiple instances would cause conflicting behavior
- the instance represents application-wide configuration or infrastructure

The "one" in Singleton should have a clear scope. It may mean one instance per application process, per request, per test, or per container, depending on the system.

## Problem

Suppose several parts of an application create their own configuration store. Each instance can load a different configuration file or hold different feature-flag values.

The application no longer has one source of truth, so behavior can vary depending on which instance a component happens to use.

### Swift
```swift
final class FeatureFlags {
    private var values: [String: Bool] = [:]

    func set(_ name: String, enabled: Bool) {
        values[name] = enabled
    }

    func isEnabled(_ name: String) -> Bool {
        values[name, default: false]
    }
}

let checkoutFlags = FeatureFlags()
checkoutFlags.set("new-checkout", enabled: true)

let profileFlags = FeatureFlags()
print(profileFlags.isEnabled("new-checkout")) // false
```

### Go
```go
package main

import "fmt"

type FeatureFlags struct {
	values map[string]bool
}

func NewFeatureFlags() *FeatureFlags {
	return &FeatureFlags{values: map[string]bool{}}
}

func main() {
	checkoutFlags := NewFeatureFlags()
	checkoutFlags.values["new-checkout"] = true

	profileFlags := NewFeatureFlags()
	fmt.Println(profileFlags.values["new-checkout"]) // false
}
```

### TypeScript
```typescript
class FeatureFlags {
  private values = new Map<string, boolean>()

  set(name: string, enabled: boolean): void {
    this.values.set(name, enabled)
  }

  isEnabled(name: string): boolean {
    return this.values.get(name) ?? false
  }
}

const checkoutFlags = new FeatureFlags()
checkoutFlags.set("new-checkout", true)

const profileFlags = new FeatureFlags()
console.log(profileFlags.isEnabled("new-checkout")) // false
```

### Python
```python
class FeatureFlags:
    def __init__(self) -> None:
        self.values: dict[str, bool] = {}

    def set(self, name: str, enabled: bool) -> None:
        self.values[name] = enabled

    def is_enabled(self, name: str) -> bool:
        return self.values.get(name, False)


checkout_flags = FeatureFlags()
checkout_flags.set("new-checkout", True)

profile_flags = FeatureFlags()
print(profile_flags.is_enabled("new-checkout"))  # False
```

This design has a few problems:

- each instance has its own state, so there is no shared source of truth
- application-wide resources can be initialized more than once
- multiple connections, caches, or configuration stores can waste resources
- behavior can depend on object creation order rather than explicit configuration

## Solution

Restrict construction and expose one shared instance. Every caller uses that instance, so shared state and setup occur in one place.

### Swift
```swift
final class FeatureFlags {
    static let shared = FeatureFlags()

    private var values: [String: Bool] = [:]

    private init() {}

    func set(_ name: String, enabled: Bool) {
        values[name] = enabled
    }

    func isEnabled(_ name: String) -> Bool {
        values[name, default: false]
    }
}

FeatureFlags.shared.set("new-checkout", enabled: true)
print(FeatureFlags.shared.isEnabled("new-checkout")) // true
```

`static let` is lazily initialized and thread-safe in Swift.

### Go
```go
package main

import "sync"

type FeatureFlags struct {
	mu     sync.RWMutex
	values map[string]bool
}

var (
	featureFlagsInstance *FeatureFlags
	featureFlagsOnce     sync.Once
)

func FeatureFlagsInstance() *FeatureFlags {
	featureFlagsOnce.Do(func() {
		featureFlagsInstance = &FeatureFlags{
			values: map[string]bool{},
		}
	})

	return featureFlagsInstance
}

func (f *FeatureFlags) Set(name string, enabled bool) {
	f.mu.Lock()
	defer f.mu.Unlock()
	f.values[name] = enabled
}

func (f *FeatureFlags) IsEnabled(name string) bool {
	f.mu.RLock()
	defer f.mu.RUnlock()
	return f.values[name]
}

func main() {
	flags := FeatureFlagsInstance()
	flags.Set("new-checkout", true)
}
```

`sync.Once` guarantees one initialization even when several goroutines access the instance concurrently.

### TypeScript
```typescript
class FeatureFlags {
  private static instance: FeatureFlags | undefined
  private values = new Map<string, boolean>()

  private constructor() {}

  static getInstance(): FeatureFlags {
    if (!FeatureFlags.instance) {
      FeatureFlags.instance = new FeatureFlags()
    }

    return FeatureFlags.instance
  }

  set(name: string, enabled: boolean): void {
    this.values.set(name, enabled)
  }

  isEnabled(name: string): boolean {
    return this.values.get(name) ?? false
  }
}

const flags = FeatureFlags.getInstance()
flags.set("new-checkout", true)
```

For many TypeScript applications, exporting one module-level object is a simpler singleton because ES modules are evaluated once per module graph.

### Python
```python
class FeatureFlags:
    _instance: "FeatureFlags | None" = None

    def __new__(cls) -> "FeatureFlags":
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.values = {}
        return cls._instance

    def set(self, name: str, enabled: bool) -> None:
        self.values[name] = enabled

    def is_enabled(self, name: str) -> bool:
        return self.values.get(name, False)


checkout_flags = FeatureFlags()
checkout_flags.set("new-checkout", True)

profile_flags = FeatureFlags()
print(profile_flags.is_enabled("new-checkout"))  # True
```

In multi-threaded Python code, protect the first initialization with a lock if concurrent construction is possible.

## Why This Is Better

- one object owns the shared resource and its initialization
- callers receive consistent configuration and shared state
- expensive setup can be delayed until the instance is first used
- the access point makes the intended ownership explicit

## Relationship With SOLID

- **Single Responsibility Principle:** a singleton should manage one cohesive shared concern, not become a global service container.
- **Dependency Inversion Principle:** depend on an interface and inject the singleton where possible, rather than calling the global access point throughout business logic.
- **Open/Closed Principle:** an interface lets production code use the singleton implementation while tests or alternate environments use a replacement.

## When To Use It

- a process-wide logger, metrics registry, or immutable configuration store
- a shared cache or connection pool that must be centrally managed
- a hardware, file, or system resource that must not have competing owners
- a framework already provides a lifecycle-managed single shared instance

## When Not To Use It

- the object only seems global because dependencies have not been passed explicitly
- tests need different state or behavior for each test case
- multiple instances may be needed later for tenants, regions, users, or environments
- the singleton would hold broad mutable state used by unrelated parts of the application

## Notes

- Singleton controls the number of instances; it does not automatically make its mutable state thread-safe.
- Global access can hide dependencies and make tests order-dependent. Prefer dependency injection at application boundaries when practical.
- A dependency-injection container can manage a singleton lifecycle without requiring application code to call a global `shared` or `getInstance()` method.
- Reset or replace shared state carefully in tests to prevent state leaking between test cases.
