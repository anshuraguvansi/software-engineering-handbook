# Strategy Pattern

The Strategy Pattern is a behavioral design pattern. It defines a family of interchangeable algorithms and lets a client choose one without changing its core workflow.

## Intent

Use Strategy when:

- several algorithms solve the same problem
- behavior must be selected or replaced at runtime
- conditionals are growing whenever a new variation is added
- each algorithm should be tested independently

## Problem

Suppose a checkout service calculates shipping with standard, express, or pickup rules. If every rule lives in one method, the service must change whenever a shipping option is added.

### Swift
```swift
func shippingCost(method: String, weight: Double) -> Double {
    if method == "standard" { return 5 + weight * 0.5 }
    if method == "express" { return 15 + weight * 1.5 }
    return 0
}
```

### Go
```go
func shippingCost(method string, weight float64) float64 {
	if method == "standard" { return 5 + weight*0.5 }
	if method == "express" { return 15 + weight*1.5 }
	return 0
}
```

### TypeScript
```typescript
function shippingCost(method: string, weight: number): number {
  if (method === "standard") return 5 + weight * 0.5
  if (method === "express") return 15 + weight * 1.5
  return 0
}
```

### Python
```python
def shipping_cost(method: str, weight: float) -> float:
    if method == "standard":
        return 5 + weight * 0.5
    if method == "express":
        return 15 + weight * 1.5
    return 0
```

This design has a few problems:

- one method knows every shipping rule
- adding a method requires editing existing branching logic
- algorithms cannot be composed or tested as separate units
- string-based selection can silently fall through to unintended behavior

## Solution

Define a strategy interface for the varying algorithm. Each shipping method implements that interface, and the checkout service receives the strategy it should use.

### Swift
```swift
protocol ShippingStrategy {
    func cost(forWeight weight: Double) -> Double
}

struct StandardShipping: ShippingStrategy {
    func cost(forWeight weight: Double) -> Double { 5 + weight * 0.5 }
}

struct ExpressShipping: ShippingStrategy {
    func cost(forWeight weight: Double) -> Double { 15 + weight * 1.5 }
}

struct StorePickup: ShippingStrategy {
    func cost(forWeight weight: Double) -> Double { 0 }
}

final class CheckoutService {
    private let shipping: any ShippingStrategy

    init(shipping: any ShippingStrategy) {
        self.shipping = shipping
    }

    func total(subtotal: Double, weight: Double) -> Double {
        subtotal + shipping.cost(forWeight: weight)
    }
}

let checkout = CheckoutService(shipping: ExpressShipping())
print(checkout.total(subtotal: 80, weight: 2))
```

### Go
```go
package main

type ShippingStrategy interface {
	Cost(weight float64) float64
}

type StandardShipping struct{}
func (StandardShipping) Cost(weight float64) float64 { return 5 + weight*0.5 }

type ExpressShipping struct{}
func (ExpressShipping) Cost(weight float64) float64 { return 15 + weight*1.5 }

type StorePickup struct{}
func (StorePickup) Cost(weight float64) float64 { return 0 }

type CheckoutService struct { shipping ShippingStrategy }

func (s CheckoutService) Total(subtotal, weight float64) float64 {
	return subtotal + s.shipping.Cost(weight)
}

func main() {
	checkout := CheckoutService{shipping: ExpressShipping{}}
	println(checkout.Total(80, 2))
}
```

### TypeScript
```typescript
interface ShippingStrategy {
  cost(weight: number): number
}

class StandardShipping implements ShippingStrategy {
  cost(weight: number): number { return 5 + weight * 0.5 }
}

class ExpressShipping implements ShippingStrategy {
  cost(weight: number): number { return 15 + weight * 1.5 }
}

class StorePickup implements ShippingStrategy {
  cost(_weight: number): number { return 0 }
}

class CheckoutService {
  constructor(private readonly shipping: ShippingStrategy) {}

  total(subtotal: number, weight: number): number {
    return subtotal + this.shipping.cost(weight)
  }
}

const checkout = new CheckoutService(new ExpressShipping())
console.log(checkout.total(80, 2))
```

### Python
```python
from typing import Protocol


class ShippingStrategy(Protocol):
    def cost(self, weight: float) -> float:
        pass


class StandardShipping:
    def cost(self, weight: float) -> float:
        return 5 + weight * 0.5


class ExpressShipping:
    def cost(self, weight: float) -> float:
        return 15 + weight * 1.5


class StorePickup:
    def cost(self, weight: float) -> float:
        return 0


class CheckoutService:
    def __init__(self, shipping: ShippingStrategy) -> None:
        self.shipping = shipping

    def total(self, subtotal: float, weight: float) -> float:
        return subtotal + self.shipping.cost(weight)


checkout = CheckoutService(ExpressShipping())
print(checkout.total(80, 2))
```

## Why This Is Better

- each algorithm has one focused implementation
- clients can select behavior through configuration or dependency injection
- new strategies do not require changes to the checkout workflow
- each strategy can be tested in isolation

## Relationship With SOLID

- **Single Responsibility Principle:** each strategy owns one algorithm while the context owns the workflow.
- **Open/Closed Principle:** new strategies can be added without modifying the context.
- **Dependency Inversion Principle:** the context depends on the strategy abstraction.

## When To Use It

- pricing, routing, sorting, validation, retry, or serialization rules vary
- a client must switch algorithms at runtime
- subclasses exist only to provide one varying behavior
- conditionals repeatedly select among equivalent operations

## When Not To Use It

- there is only one stable algorithm
- a small local conditional is clearer than several new types
- strategies do not share a meaningful contract

## Notes

- A strategy may be a class, closure, or first-class function; the interchangeable contract matters more than the syntax.
- Keep selection logic outside the context, such as in configuration, a factory, or the composition root.
- Strategy changes how work is performed. State changes behavior in response to an object's internal state and often controls its own transitions.
