# Adapter Pattern

The Adapter Pattern is a structural design pattern. It converts one interface into another interface that the client code expects.

## Intent

Use an adapter when:

- existing code needs to work with an incompatible library or service
- you cannot change the interface provided by a dependency
- callers should use one stable application interface
- conversion rules belong at the boundary, not throughout business logic

## Problem

Suppose an order service expects a payment processor with a `charge` operation that accepts an amount in dollars. A third-party gateway already exists, but it exposes `makePayment`, requires the amount in cents, and uses a different result type.

If the order service calls the gateway directly, third-party details leak into the business layer. Every caller must know how to convert money, interpret statuses, and handle the gateway-specific API.

### Swift
```swift
final class LegacyPaymentGateway {
    func makePayment(cents: Int, currencyCode: String) -> String {
        print("Charging \(cents) \(currencyCode)")
        return "approved"
    }
}

final class OrderService {
    private let gateway = LegacyPaymentGateway()

    func placeOrder(total: Double) {
        let cents = Int(total * 100)
        let status = gateway.makePayment(cents: cents, currencyCode: "USD")

        if status == "approved" {
            print("Order placed")
        }
    }
}
```

### Go
```go
package main

type LegacyPaymentGateway struct{}

func (g *LegacyPaymentGateway) MakePayment(cents int, currencyCode string) string {
	return "approved"
}

type OrderService struct {
	gateway *LegacyPaymentGateway
}

func (s *OrderService) PlaceOrder(total float64) {
	cents := int(total * 100)
	status := s.gateway.MakePayment(cents, "USD")

	if status == "approved" {
		println("Order placed")
	}
}
```

### TypeScript
```typescript
class LegacyPaymentGateway {
  makePayment(cents: number, currencyCode: string): "approved" | "declined" {
    return "approved"
  }
}

class OrderService {
  private gateway = new LegacyPaymentGateway()

  placeOrder(total: number): void {
    const cents = Math.round(total * 100)
    const status = this.gateway.makePayment(cents, "USD")

    if (status === "approved") {
      console.log("Order placed")
    }
  }
}
```

### Python
```python
class LegacyPaymentGateway:
    def make_payment(self, cents: int, currency_code: str) -> str:
        return "approved"


class OrderService:
    def __init__(self) -> None:
        self.gateway = LegacyPaymentGateway()

    def place_order(self, total: float) -> None:
        cents = round(total * 100)
        status = self.gateway.make_payment(cents, "USD")

        if status == "approved":
            print("Order placed")
```

This design has a few problems:

- business code depends directly on a third-party API
- money and status conversions are repeated by every caller
- replacing the gateway requires changes throughout the application
- testing requires the concrete gateway instead of a small application-facing abstraction

## Solution

Define the interface that the application needs and create an adapter that implements it. The adapter contains the incompatible dependency and translates calls, data, and results at one boundary.

The order service now depends only on its own `PaymentProcessor` abstraction.

### Swift
```swift
import Foundation

enum PaymentResult {
    case approved
    case declined
}

protocol PaymentProcessor {
    func charge(amount: Decimal) -> PaymentResult
}

final class LegacyPaymentGateway {
    func makePayment(cents: Int, currencyCode: String) -> String {
        print("Charging \(cents) \(currencyCode)")
        return "approved"
    }
}

final class LegacyPaymentAdapter: PaymentProcessor {
    private let gateway: LegacyPaymentGateway

    init(gateway: LegacyPaymentGateway) {
        self.gateway = gateway
    }

    func charge(amount: Decimal) -> PaymentResult {
        let cents = NSDecimalNumber(decimal: amount * 100).intValue
        let status = gateway.makePayment(cents: cents, currencyCode: "USD")
        return status == "approved" ? .approved : .declined
    }
}

final class OrderService {
    private let paymentProcessor: PaymentProcessor

    init(paymentProcessor: PaymentProcessor) {
        self.paymentProcessor = paymentProcessor
    }

    func placeOrder(total: Decimal) {
        guard paymentProcessor.charge(amount: total) == .approved else {
            print("Payment declined")
            return
        }

        print("Order placed")
    }
}
```

### Go
```go
package main

type PaymentResult string

const (
	Approved PaymentResult = "approved"
	Declined PaymentResult = "declined"
)

type PaymentProcessor interface {
	Charge(cents int) PaymentResult
}

type LegacyPaymentGateway struct{}

func (g *LegacyPaymentGateway) MakePayment(cents int, currencyCode string) string {
	return "approved"
}

type LegacyPaymentAdapter struct {
	gateway *LegacyPaymentGateway
}

func (a *LegacyPaymentAdapter) Charge(cents int) PaymentResult {
	status := a.gateway.MakePayment(cents, "USD")
	if status == "approved" {
		return Approved
	}
	return Declined
}

type OrderService struct {
	paymentProcessor PaymentProcessor
}

func (s *OrderService) PlaceOrder(totalCents int) {
	if s.paymentProcessor.Charge(totalCents) != Approved {
		println("Payment declined")
		return
	}

	println("Order placed")
}

func main() {
	processor := &LegacyPaymentAdapter{gateway: &LegacyPaymentGateway{}}
	service := OrderService{paymentProcessor: processor}
	service.PlaceOrder(4999)
}
```

### TypeScript
```typescript
type PaymentResult = "approved" | "declined"

interface PaymentProcessor {
  charge(amount: number): PaymentResult
}

class LegacyPaymentGateway {
  makePayment(cents: number, currencyCode: string): PaymentResult {
    return "approved"
  }
}

class LegacyPaymentAdapter implements PaymentProcessor {
  constructor(private readonly gateway: LegacyPaymentGateway) {}

  charge(amount: number): PaymentResult {
    const cents = Math.round(amount * 100)
    return this.gateway.makePayment(cents, "USD")
  }
}

class OrderService {
  constructor(private readonly paymentProcessor: PaymentProcessor) {}

  placeOrder(total: number): void {
    if (this.paymentProcessor.charge(total) !== "approved") {
      console.log("Payment declined")
      return
    }

    console.log("Order placed")
  }
}
```

### Python
```python
from decimal import Decimal
from typing import Protocol


class PaymentProcessor(Protocol):
    def charge(self, amount: Decimal) -> bool:
        pass


class LegacyPaymentGateway:
    def make_payment(self, cents: int, currency_code: str) -> str:
        return "approved"


class LegacyPaymentAdapter:
    def __init__(self, gateway: LegacyPaymentGateway) -> None:
        self.gateway = gateway

    def charge(self, amount: Decimal) -> bool:
        cents = int(amount * 100)
        return self.gateway.make_payment(cents, "USD") == "approved"


class OrderService:
    def __init__(self, payment_processor: PaymentProcessor) -> None:
        self.payment_processor = payment_processor

    def place_order(self, total: Decimal) -> None:
        if not self.payment_processor.charge(total):
            print("Payment declined")
            return

        print("Order placed")
```

## Why This Is Better

- business code uses a stable interface owned by the application
- conversion and compatibility logic is centralized in one place
- the third-party dependency can be replaced without changing callers
- adapters are easy to test with known input and output mappings
- tests can replace the adapter with a fake `PaymentProcessor`

## Relationship With SOLID

- **Single Responsibility Principle:** the adapter owns translation between two interfaces; business services retain their domain behavior.
- **Open/Closed Principle:** add an adapter for another provider without changing the order service.
- **Dependency Inversion Principle:** high-level code depends on `PaymentProcessor`, not the concrete third-party gateway.

## When To Use It

- integrating a legacy component, SDK, or third-party API
- migrating to a new provider while preserving the existing application interface
- normalizing several providers behind one domain-specific interface
- isolating data conversion, error mapping, or protocol differences at a system boundary

## When Not To Use It

- you own both interfaces and can make them compatible directly
- the dependency already exposes the exact interface the application needs
- the adapter would hide important behavioral differences that callers need to understand

## Notes

- Object adapters wrap an instance of the incompatible class. This is the most common form because it works in languages without multiple inheritance.
- Class adapters use inheritance to adapt an interface where the language supports it, but they are more tightly coupled.
- Keep adapters narrow and domain-focused. An adapter should translate an interface, not become a general-purpose service layer.
- Map errors, units, and data types explicitly so the application does not inherit provider-specific behavior by accident.
