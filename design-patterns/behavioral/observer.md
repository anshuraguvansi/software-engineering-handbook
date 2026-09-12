# Observer Pattern

The Observer Pattern is a behavioral design pattern. It creates a one-to-many subscription relationship so dependents are notified when a subject publishes an event or changes state.

## Intent

Use Observer when:

- several independent components react to the same event
- publishers should not depend on concrete subscribers
- subscribers must be added or removed at runtime
- event delivery should replace scattered direct calls

## Problem

Suppose an order service must update analytics and send a receipt after an order is placed. Direct calls couple the service to every reaction.

### Swift
```swift
func placeOrder(id: String) {
    print("Saving \(id)")
    EmailService().sendReceipt(orderId: id)
    AnalyticsService().recordPurchase(orderId: id)
}
```

### Go
```go
func placeOrder(id string) {
	println("Saving " + id)
	EmailService{}.SendReceipt(id)
	AnalyticsService{}.RecordPurchase(id)
}
```

### TypeScript
```typescript
function placeOrder(id: string): void {
  console.log(`Saving ${id}`)
  new EmailService().sendReceipt(id)
  new AnalyticsService().recordPurchase(id)
}
```

### Python
```python
def place_order(order_id: str) -> None:
    print(f"Saving {order_id}")
    EmailService().send_receipt(order_id)
    AnalyticsService().record_purchase(order_id)
```

This design has a few problems:

- the publisher knows every downstream action
- adding a reaction requires changing the order workflow
- optional integrations cannot subscribe or unsubscribe dynamically
- a failing dependency can become tangled with the primary operation

## Solution

Define an observer contract and let the subject maintain subscriptions. After completing its work, the subject publishes an event through that contract.

### Swift
```swift
struct OrderPlaced { let orderId: String }

protocol OrderObserver: AnyObject {
    func orderPlaced(_ event: OrderPlaced)
}

final class OrderService {
    private var observers: [any OrderObserver] = []

    func subscribe(_ observer: any OrderObserver) { observers.append(observer) }

    func placeOrder(id: String) {
        print("Saving \(id)")
        let event = OrderPlaced(orderId: id)
        observers.forEach { $0.orderPlaced(event) }
    }
}

final class ReceiptSender: OrderObserver {
    func orderPlaced(_ event: OrderPlaced) { print("Receipt for \(event.orderId)") }
}

final class PurchaseAnalytics: OrderObserver {
    func orderPlaced(_ event: OrderPlaced) { print("Recorded \(event.orderId)") }
}

let orders = OrderService()
orders.subscribe(ReceiptSender())
orders.subscribe(PurchaseAnalytics())
orders.placeOrder(id: "order-1")
```

### Go
```go
package main

type OrderPlaced struct { OrderID string }

type OrderObserver interface { OrderPlaced(OrderPlaced) }

type OrderService struct { observers []OrderObserver }

func (s *OrderService) Subscribe(observer OrderObserver) {
	s.observers = append(s.observers, observer)
}

func (s *OrderService) PlaceOrder(id string) {
	println("Saving " + id)
	event := OrderPlaced{OrderID: id}
	for _, observer := range s.observers { observer.OrderPlaced(event) }
}

type ReceiptSender struct{}
func (ReceiptSender) OrderPlaced(event OrderPlaced) { println("Receipt for " + event.OrderID) }

type PurchaseAnalytics struct{}
func (PurchaseAnalytics) OrderPlaced(event OrderPlaced) { println("Recorded " + event.OrderID) }

func main() {
	orders := &OrderService{}
	orders.Subscribe(ReceiptSender{})
	orders.Subscribe(PurchaseAnalytics{})
	orders.PlaceOrder("order-1")
}
```

### TypeScript
```typescript
type OrderPlaced = { orderId: string }

interface OrderObserver {
  orderPlaced(event: OrderPlaced): void
}

class OrderService {
  private readonly observers = new Set<OrderObserver>()

  subscribe(observer: OrderObserver): () => void {
    this.observers.add(observer)
    return () => this.observers.delete(observer)
  }

  placeOrder(id: string): void {
    console.log(`Saving ${id}`)
    for (const observer of this.observers) observer.orderPlaced({ orderId: id })
  }
}

class ReceiptSender implements OrderObserver {
  orderPlaced(event: OrderPlaced): void { console.log(`Receipt for ${event.orderId}`) }
}

class PurchaseAnalytics implements OrderObserver {
  orderPlaced(event: OrderPlaced): void { console.log(`Recorded ${event.orderId}`) }
}

const orders = new OrderService()
orders.subscribe(new ReceiptSender())
orders.subscribe(new PurchaseAnalytics())
orders.placeOrder("order-1")
```

### Python
```python
from dataclasses import dataclass
from typing import Protocol


@dataclass(frozen=True)
class OrderPlaced:
    order_id: str


class OrderObserver(Protocol):
    def order_placed(self, event: OrderPlaced) -> None:
        pass


class OrderService:
    def __init__(self) -> None:
        self.observers: list[OrderObserver] = []

    def subscribe(self, observer: OrderObserver) -> None:
        self.observers.append(observer)

    def place_order(self, order_id: str) -> None:
        print(f"Saving {order_id}")
        event = OrderPlaced(order_id)
        for observer in list(self.observers):
            observer.order_placed(event)


class ReceiptSender:
    def order_placed(self, event: OrderPlaced) -> None:
        print(f"Receipt for {event.order_id}")


class PurchaseAnalytics:
    def order_placed(self, event: OrderPlaced) -> None:
        print(f"Recorded {event.order_id}")


orders = OrderService()
orders.subscribe(ReceiptSender())
orders.subscribe(PurchaseAnalytics())
orders.place_order("order-1")
```

## Why This Is Better

- publishers depend only on a small observer contract
- reactions can be added and removed independently
- one event can fan out to multiple consumers
- observers and publication behavior can be tested separately

## Relationship With SOLID

- **Single Responsibility Principle:** the subject owns its workflow while observers own individual reactions.
- **Open/Closed Principle:** new observers can be added without changing the publisher.
- **Dependency Inversion Principle:** the publisher depends on an observer abstraction.

## When To Use It

- UI views react to model changes
- domain events trigger independent in-process actions
- plugins or integrations subscribe dynamically
- one component must notify an unknown number of dependents

## When Not To Use It

- there is one required collaborator and a direct call makes that dependency clearer
- strict ordering or atomic completion across subscribers is essential
- a durable message broker is required for cross-process delivery

## Notes

- Define whether notifications are synchronous or asynchronous and how failures are handled.
- Provide unsubscription when subscriber lifetimes differ from the publisher; otherwise references can leak.
- Avoid exposing a mutable subject and forcing observers to pull ambiguous state. Prefer small, immutable event payloads.
- Observer describes in-process relationships; publish-subscribe commonly adds a broker so publishers and subscribers do not know each other.
