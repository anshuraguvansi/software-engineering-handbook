# Decorator Pattern

The Decorator Pattern is a structural design pattern. It adds behavior to an object by wrapping it with another object that exposes the same interface.

## Intent

Use a decorator when:

- behavior should be added to selected objects at runtime
- combining features through subclasses would create too many class combinations
- each added responsibility should remain separate and reusable
- callers should continue using the same interface after behavior is added

## Problem

Suppose an application sends notifications by email. Some notifications also need logging, retry handling, and metrics. Creating a subclass for every combination quickly becomes difficult to maintain.

For example, `LoggingEmailNotifier`, `RetryingEmailNotifier`, and `LoggingRetryingEmailNotifier` all duplicate behavior and do not scale when new notification channels or features are introduced.

### Swift
```swift
final class EmailNotifier {
    func send(message: String) {
        print("Email: \(message)")
    }
}

final class LoggingEmailNotifier {
    func send(message: String) {
        print("Sending notification")
        print("Email: \(message)")
    }
}

final class RetryingEmailNotifier {
    func send(message: String) {
        for _ in 1...3 {
            print("Email: \(message)")
        }
    }
}

final class LoggingRetryingEmailNotifier {
    func send(message: String) {
        print("Sending notification")
        for _ in 1...3 {
            print("Email: \(message)")
        }
    }
}
```

### Go
```go
package main

type EmailNotifier struct{}

func (n *EmailNotifier) Send(message string) {
	println("Email:", message)
}

type LoggingEmailNotifier struct{}

func (n *LoggingEmailNotifier) Send(message string) {
	println("Sending notification")
	println("Email:", message)
}

type RetryingEmailNotifier struct{}

func (n *RetryingEmailNotifier) Send(message string) {
	for attempt := 0; attempt < 3; attempt++ {
		println("Email:", message)
	}
}
```

### TypeScript
```typescript
class EmailNotifier {
  send(message: string): void {
    console.log(`Email: ${message}`)
  }
}

class LoggingEmailNotifier {
  send(message: string): void {
    console.log("Sending notification")
    console.log(`Email: ${message}`)
  }
}

class RetryingEmailNotifier {
  send(message: string): void {
    for (let attempt = 0; attempt < 3; attempt += 1) {
      console.log(`Email: ${message}`)
    }
  }
}
```

### Python
```python
class EmailNotifier:
    def send(self, message: str) -> None:
        print(f"Email: {message}")


class LoggingEmailNotifier:
    def send(self, message: str) -> None:
        print("Sending notification")
        print(f"Email: {message}")


class RetryingEmailNotifier:
    def send(self, message: str) -> None:
        for _ in range(3):
            print(f"Email: {message}")
```

This design has a few problems:

- every feature combination needs another concrete class
- adding a new notification channel multiplies the number of subclasses
- shared logging and retry logic is duplicated across implementations
- choosing behavior at runtime is difficult because the combination is fixed in the class name

## Solution

Define a common interface for the core object and all decorators. Each decorator stores another object with that interface, performs its extra work, and delegates to the wrapped object.

Decorators can be stacked in any needed combination. The caller still sees one `Notifier`.

### Swift
```swift
protocol Notifier {
    func send(message: String) throws
}

final class EmailNotifier: Notifier {
    func send(message: String) throws {
        print("Email: \(message)")
    }
}

final class LoggingNotifier: Notifier {
    private let wrapped: any Notifier

    init(wrapping notifier: any Notifier) {
        wrapped = notifier
    }

    func send(message: String) throws {
        print("Sending notification")
        try wrapped.send(message: message)
        print("Notification sent")
    }
}

final class RetryingNotifier: Notifier {
    private let wrapped: any Notifier
    private let maxAttempts: Int

    init(wrapping notifier: any Notifier, maxAttempts: Int = 3) {
        wrapped = notifier
        self.maxAttempts = maxAttempts
    }

    func send(message: String) throws {
        var lastError: Error?

        for _ in 1...maxAttempts {
            do {
                try wrapped.send(message: message)
                return
            } catch {
                lastError = error
            }
        }

        throw lastError!
    }
}

let notifier: any Notifier = LoggingNotifier(
    wrapping: RetryingNotifier(wrapping: EmailNotifier())
)

do {
    try notifier.send(message: "Your order has shipped")
} catch {
    print("Notification failed: \(error)")
}
```

### Go
```go
package main

import "fmt"

type Notifier interface {
	Send(message string) error
}

type EmailNotifier struct{}

func (n *EmailNotifier) Send(message string) error {
	fmt.Println("Email:", message)
	return nil
}

type LoggingNotifier struct {
	wrapped Notifier
}

func (n *LoggingNotifier) Send(message string) error {
	fmt.Println("Sending notification")
	if err := n.wrapped.Send(message); err != nil {
		return err
	}
	fmt.Println("Notification sent")
	return nil
}

type RetryingNotifier struct {
	wrapped     Notifier
	maxAttempts int
}

func (n *RetryingNotifier) Send(message string) error {
	var lastErr error
	for attempt := 0; attempt < n.maxAttempts; attempt++ {
		if err := n.wrapped.Send(message); err == nil {
			return nil
		} else {
			lastErr = err
		}
	}
	return lastErr
}

func main() {
	notifier := &LoggingNotifier{
		wrapped: &RetryingNotifier{
			wrapped:     &EmailNotifier{},
			maxAttempts: 3,
		},
	}
	_ = notifier.Send("Your order has shipped")
}
```

### TypeScript
```typescript
interface Notifier {
  send(message: string): void
}

class EmailNotifier implements Notifier {
  send(message: string): void {
    console.log(`Email: ${message}`)
  }
}

class LoggingNotifier implements Notifier {
  constructor(private readonly wrapped: Notifier) {}

  send(message: string): void {
    console.log("Sending notification")
    this.wrapped.send(message)
    console.log("Notification sent")
  }
}

class RetryingNotifier implements Notifier {
  constructor(
    private readonly wrapped: Notifier,
    private readonly maxAttempts = 3,
  ) {}

  send(message: string): void {
    for (let attempt = 0; attempt < this.maxAttempts; attempt += 1) {
      try {
        this.wrapped.send(message)
        return
      } catch (error) {
        if (attempt === this.maxAttempts - 1) throw error
      }
    }
  }
}

const notifier: Notifier = new LoggingNotifier(
  new RetryingNotifier(new EmailNotifier()),
)
notifier.send("Your order has shipped")
```

### Python
```python
from typing import Protocol


class Notifier(Protocol):
    def send(self, message: str) -> None:
        pass


class EmailNotifier:
    def send(self, message: str) -> None:
        print(f"Email: {message}")


class LoggingNotifier:
    def __init__(self, wrapped: Notifier) -> None:
        self.wrapped = wrapped

    def send(self, message: str) -> None:
        print("Sending notification")
        self.wrapped.send(message)
        print("Notification sent")


class RetryingNotifier:
    def __init__(self, wrapped: Notifier, max_attempts: int = 3) -> None:
        self.wrapped = wrapped
        self.max_attempts = max_attempts

    def send(self, message: str) -> None:
        for attempt in range(self.max_attempts):
            try:
                self.wrapped.send(message)
                return
            except ConnectionError:
                if attempt == self.max_attempts - 1:
                    raise


notifier: Notifier = LoggingNotifier(RetryingNotifier(EmailNotifier()))
notifier.send("Your order has shipped")
```

## Why This Is Better

- features can be combined without creating a subclass for every combination
- each decorator has one focused responsibility
- behavior can be selected and ordered at runtime
- new decorators work with every existing component that supports the interface
- core components stay small because cross-cutting behavior lives in wrappers

## Relationship With SOLID

- **Single Responsibility Principle:** logging, retrying, and delivery each have separate responsibilities.
- **Open/Closed Principle:** add a new decorator without changing the core notifier or existing decorators.
- **Liskov Substitution Principle:** decorators implement the same interface as the objects they wrap.
- **Dependency Inversion Principle:** callers depend on `Notifier`, not on a concrete delivery channel or decorator stack.

## When To Use It

- optional features must be combined in different ways at runtime
- subclass combinations are growing quickly
- logging, caching, validation, authorization, retrying, compression, or metrics should wrap a core operation
- existing components need extra behavior without changing their source code

## When Not To Use It

- only one fixed behavior is needed for every instance
- the extra behavior belongs directly in the component's primary responsibility
- a simple function, middleware pipeline, or language feature is clearer
- many nested decorators would make behavior and debugging difficult to follow

## Notes

- Decorator order matters. `Logging(Retrying(Email))` logs one overall operation, while `Retrying(Logging(Email))` logs every attempt.
- Decorators should preserve the expected behavior of the interface. Unexpectedly swallowing errors or changing return values can violate substitution.
- Middleware, HTTP interceptors, and stream wrappers are common real-world forms of the Decorator Pattern.
- Keep the wrapped interface narrow so decorators remain easy to compose and test.
