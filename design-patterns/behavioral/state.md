# State Pattern

The State Pattern is a behavioral design pattern. It lets an object change its behavior when its internal state changes by delegating state-specific rules to separate objects.

## Intent

Use State when:

- behavior depends on an object's lifecycle state
- large conditionals repeat the same state checks
- transitions have rules that belong with state-specific behavior
- invalid operations should be rejected consistently

## Problem

Suppose an order can be pending, paid, shipped, or cancelled. A service that branches on strings for every operation becomes difficult to extend and can permit invalid transitions.

### Swift
```swift
func ship(_ order: Order) {
    if order.status == "paid" { order.status = "shipped" }
    else { print("Cannot ship") }
}
```

### Go
```go
func (o *Order) Ship() {
	if o.Status == "paid" { o.Status = "shipped" } else { println("Cannot ship") }
}
```

### TypeScript
```typescript
ship(): void {
  if (this.status === "paid") this.status = "shipped"
  else throw new Error("Cannot ship")
}
```

### Python
```python
def ship(self) -> None:
    if self.status == "paid":
        self.status = "shipped"
    else:
        raise ValueError("Cannot ship")
```

This design has a few problems:

- state checks are duplicated across operations
- transition rules are scattered throughout the class
- string values allow invalid or unknown states
- adding a state requires editing many branches

## Solution

Define a common state interface and move state-dependent operations into concrete state objects. The order is the context; it delegates behavior and provides a controlled transition method.

### Swift
```swift
protocol OrderState {
    var name: String { get }
    func pay(_ order: Order)
    func ship(_ order: Order)
}

final class Order {
    fileprivate var state: any OrderState = PendingState()
    var status: String { state.name }
    func pay() { state.pay(self) }
    func ship() { state.ship(self) }
    fileprivate func transition(to state: any OrderState) { self.state = state }
}

struct PendingState: OrderState {
    let name = "pending"
    func pay(_ order: Order) { order.transition(to: PaidState()) }
    func ship(_ order: Order) { print("Pay before shipping") }
}

struct PaidState: OrderState {
    let name = "paid"
    func pay(_ order: Order) { print("Already paid") }
    func ship(_ order: Order) { order.transition(to: ShippedState()) }
}

struct ShippedState: OrderState {
    let name = "shipped"
    func pay(_ order: Order) { print("Already shipped") }
    func ship(_ order: Order) { print("Already shipped") }
}

let order = Order()
order.pay()
order.ship()
```

### Go
```go
package main

type OrderState interface {
	Name() string
	Pay(*Order)
	Ship(*Order)
}

type Order struct { state OrderState }
func NewOrder() *Order { return &Order{state: PendingState{}} }
func (o *Order) Status() string { return o.state.Name() }
func (o *Order) Pay() { o.state.Pay(o) }
func (o *Order) Ship() { o.state.Ship(o) }

type PendingState struct{}
func (PendingState) Name() string { return "pending" }
func (PendingState) Pay(o *Order) { o.state = PaidState{} }
func (PendingState) Ship(*Order) { println("Pay before shipping") }

type PaidState struct{}
func (PaidState) Name() string { return "paid" }
func (PaidState) Pay(*Order) { println("Already paid") }
func (PaidState) Ship(o *Order) { o.state = ShippedState{} }

type ShippedState struct{}
func (ShippedState) Name() string { return "shipped" }
func (ShippedState) Pay(*Order) { println("Already shipped") }
func (ShippedState) Ship(*Order) { println("Already shipped") }

func main() {
	order := NewOrder()
	order.Pay()
	order.Ship()
}
```

### TypeScript
```typescript
interface OrderState {
  readonly name: string
  pay(order: Order): void
  ship(order: Order): void
}

class Order {
  private state: OrderState = new PendingState()
  get status(): string { return this.state.name }
  pay(): void { this.state.pay(this) }
  ship(): void { this.state.ship(this) }
  transitionTo(state: OrderState): void { this.state = state }
}

class PendingState implements OrderState {
  readonly name = "pending"
  pay(order: Order): void { order.transitionTo(new PaidState()) }
  ship(_order: Order): void { throw new Error("Pay before shipping") }
}

class PaidState implements OrderState {
  readonly name = "paid"
  pay(_order: Order): void { throw new Error("Already paid") }
  ship(order: Order): void { order.transitionTo(new ShippedState()) }
}

class ShippedState implements OrderState {
  readonly name = "shipped"
  pay(_order: Order): void { throw new Error("Already shipped") }
  ship(_order: Order): void { throw new Error("Already shipped") }
}

const order = new Order()
order.pay()
order.ship()
```

### Python
```python
from __future__ import annotations
from typing import Protocol


class OrderState(Protocol):
    name: str
    def pay(self, order: Order) -> None: ...
    def ship(self, order: Order) -> None: ...


class Order:
    def __init__(self) -> None:
        self._state: OrderState = PendingState()

    @property
    def status(self) -> str:
        return self._state.name

    def pay(self) -> None:
        self._state.pay(self)

    def ship(self) -> None:
        self._state.ship(self)

    def transition_to(self, state: OrderState) -> None:
        self._state = state


class PendingState:
    name = "pending"
    def pay(self, order: Order) -> None: order.transition_to(PaidState())
    def ship(self, order: Order) -> None: raise ValueError("Pay before shipping")


class PaidState:
    name = "paid"
    def pay(self, order: Order) -> None: raise ValueError("Already paid")
    def ship(self, order: Order) -> None: order.transition_to(ShippedState())


class ShippedState:
    name = "shipped"
    def pay(self, order: Order) -> None: raise ValueError("Already shipped")
    def ship(self, order: Order) -> None: raise ValueError("Already shipped")


order = Order()
order.pay()
order.ship()
```

## Why This Is Better

- transition rules live beside state-specific behavior
- the context no longer contains repeated state conditionals
- invalid operations are handled consistently by each state
- new states can be introduced with localized changes

## Relationship With SOLID

- **Single Responsibility Principle:** each state owns the rules for one lifecycle stage.
- **Open/Closed Principle:** state behavior can be extended with new implementations.
- **Dependency Inversion Principle:** the context delegates through the state abstraction.

## When To Use It

- workflows have explicit lifecycle stages and guarded transitions
- behavior changes substantially across states
- the same state conditionals appear in several methods
- state-specific rules are growing independently

## When Not To Use It

- an enum and a small transition function express the lifecycle clearly
- states contain no behavior and are only labels
- transitions are mostly data-driven and better modeled as a table or state-machine library

## Notes

- Decide whether the context or each state owns transitions. Keep that choice consistent.
- Make illegal transitions explicit through errors or result types instead of silently ignoring them.
- Concrete state objects can be shared when they are immutable and contain no context-specific data.
- State and Strategy have similar structures, but states represent lifecycle modes and may initiate transitions; strategies are normally selected by a client.
