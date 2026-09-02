# Facade Pattern

The Facade Pattern is a structural design pattern. It provides one simple interface to a group of complex subsystems.

## Intent

Use a facade when:

- clients must coordinate several services to complete a common task
- a subsystem has too many classes, operations, or ordering rules for most callers
- you want a stable application-facing API while internals can change
- the common workflow should be easier to discover and use correctly

## Problem

Suppose placing an order requires checking inventory, charging a payment method, creating a shipment, and sending a confirmation. Without a facade, every caller must understand and coordinate the same workflow.

The controller or client becomes responsible for subsystem ordering, failure handling, and rollback decisions. Repeating that orchestration in multiple places produces inconsistent checkout behavior.

### Swift
```swift
final class InventoryService {
    func reserve(productId: String, quantity: Int) -> Bool { true }
}

final class PaymentService {
    func charge(customerId: String, amountInCents: Int) -> Bool { true }
}

final class ShippingService {
    func createShipment(productId: String, quantity: Int) -> String {
        "shipment-123"
    }
}

final class NotificationService {
    func sendOrderConfirmation(customerId: String, shipmentId: String) {}
}

let inventory = InventoryService()
let payment = PaymentService()
let shipping = ShippingService()
let notifications = NotificationService()

if inventory.reserve(productId: "book-1", quantity: 1),
   payment.charge(customerId: "customer-1", amountInCents: 2999) {
    let shipmentId = shipping.createShipment(productId: "book-1", quantity: 1)
    notifications.sendOrderConfirmation(customerId: "customer-1", shipmentId: shipmentId)
}
```

### Go
```go
package main

type InventoryService struct{}

func (s *InventoryService) Reserve(productID string, quantity int) bool { return true }

type PaymentService struct{}

func (s *PaymentService) Charge(customerID string, amountInCents int) bool { return true }

type ShippingService struct{}

func (s *ShippingService) CreateShipment(productID string, quantity int) string {
	return "shipment-123"
}

type NotificationService struct{}

func (s *NotificationService) SendOrderConfirmation(customerID string, shipmentID string) {}

func main() {
	inventory := &InventoryService{}
	payment := &PaymentService{}
	shipping := &ShippingService{}
	notifications := &NotificationService{}

	if inventory.Reserve("book-1", 1) && payment.Charge("customer-1", 2999) {
		shipmentID := shipping.CreateShipment("book-1", 1)
		notifications.SendOrderConfirmation("customer-1", shipmentID)
	}
}
```

### TypeScript
```typescript
class InventoryService {
  reserve(productId: string, quantity: number): boolean {
    return true
  }
}

class PaymentService {
  charge(customerId: string, amountInCents: number): boolean {
    return true
  }
}

class ShippingService {
  createShipment(productId: string, quantity: number): string {
    return "shipment-123"
  }
}

class NotificationService {
  sendOrderConfirmation(customerId: string, shipmentId: string): void {}
}

const inventory = new InventoryService()
const payment = new PaymentService()
const shipping = new ShippingService()
const notifications = new NotificationService()

if (inventory.reserve("book-1", 1) && payment.charge("customer-1", 2999)) {
  const shipmentId = shipping.createShipment("book-1", 1)
  notifications.sendOrderConfirmation("customer-1", shipmentId)
}
```

### Python
```python
class InventoryService:
    def reserve(self, product_id: str, quantity: int) -> bool:
        return True


class PaymentService:
    def charge(self, customer_id: str, amount_in_cents: int) -> bool:
        return True


class ShippingService:
    def create_shipment(self, product_id: str, quantity: int) -> str:
        return "shipment-123"


class NotificationService:
    def send_order_confirmation(self, customer_id: str, shipment_id: str) -> None:
        pass


inventory = InventoryService()
payment = PaymentService()
shipping = ShippingService()
notifications = NotificationService()

if inventory.reserve("book-1", 1) and payment.charge("customer-1", 2999):
    shipment_id = shipping.create_shipment("book-1", 1)
    notifications.send_order_confirmation("customer-1", shipment_id)
```

This design has a few problems:

- every caller must know which subsystem methods to call and in what order
- error handling and compensating actions are duplicated across callers
- changes to the checkout workflow require changes in many places
- clients become tightly coupled to low-level subsystem APIs

## Solution

Create a facade that owns the common workflow. The facade coordinates the subsystem calls, exposes a task-focused API, and handles failure rules in one place.

The subsystems remain available for advanced use cases, but ordinary callers only need the facade.

### Swift
```swift
final class InventoryService {
    func reserve(productId: String, quantity: Int) -> Bool { true }
    func release(productId: String, quantity: Int) {}
}

final class PaymentService {
    func charge(customerId: String, amountInCents: Int) -> Bool { true }
}

final class ShippingService {
    func createShipment(productId: String, quantity: Int) -> String? {
        "shipment-123"
    }
}

final class NotificationService {
    func sendOrderConfirmation(customerId: String, shipmentId: String) {}
}

final class CheckoutFacade {
    private let inventory: InventoryService
    private let payment: PaymentService
    private let shipping: ShippingService
    private let notifications: NotificationService

    init(
        inventory: InventoryService,
        payment: PaymentService,
        shipping: ShippingService,
        notifications: NotificationService
    ) {
        self.inventory = inventory
        self.payment = payment
        self.shipping = shipping
        self.notifications = notifications
    }

    func placeOrder(
        customerId: String,
        productId: String,
        quantity: Int,
        amountInCents: Int
    ) -> Bool {
        guard inventory.reserve(productId: productId, quantity: quantity) else {
            return false
        }

        guard payment.charge(customerId: customerId, amountInCents: amountInCents) else {
            inventory.release(productId: productId, quantity: quantity)
            return false
        }

        guard let shipmentId = shipping.createShipment(productId: productId, quantity: quantity) else {
            inventory.release(productId: productId, quantity: quantity)
            return false
        }

        notifications.sendOrderConfirmation(customerId: customerId, shipmentId: shipmentId)
        return true
    }
}

let checkout = CheckoutFacade(
    inventory: InventoryService(),
    payment: PaymentService(),
    shipping: ShippingService(),
    notifications: NotificationService()
)
checkout.placeOrder(customerId: "customer-1", productId: "book-1", quantity: 1, amountInCents: 2999)
```

### Go
```go
package main

type InventoryService struct{}

func (s *InventoryService) Reserve(productID string, quantity int) bool { return true }
func (s *InventoryService) Release(productID string, quantity int)      {}

type PaymentService struct{}

func (s *PaymentService) Charge(customerID string, amountInCents int) bool { return true }

type ShippingService struct{}

func (s *ShippingService) CreateShipment(productID string, quantity int) (string, bool) {
	return "shipment-123", true
}

type NotificationService struct{}

func (s *NotificationService) SendOrderConfirmation(customerID string, shipmentID string) {}

type CheckoutFacade struct {
	inventory     *InventoryService
	payment       *PaymentService
	shipping      *ShippingService
	notifications *NotificationService
}

func (f *CheckoutFacade) PlaceOrder(
	customerID string,
	productID string,
	quantity int,
	amountInCents int,
) bool {
	if !f.inventory.Reserve(productID, quantity) {
		return false
	}

	if !f.payment.Charge(customerID, amountInCents) {
		f.inventory.Release(productID, quantity)
		return false
	}

	shipmentID, created := f.shipping.CreateShipment(productID, quantity)
	if !created {
		f.inventory.Release(productID, quantity)
		return false
	}

	f.notifications.SendOrderConfirmation(customerID, shipmentID)
	return true
}

func main() {
	checkout := CheckoutFacade{
		inventory:     &InventoryService{},
		payment:       &PaymentService{},
		shipping:      &ShippingService{},
		notifications: &NotificationService{},
	}
	checkout.PlaceOrder("customer-1", "book-1", 1, 2999)
}
```

### TypeScript
```typescript
class InventoryService {
  reserve(productId: string, quantity: number): boolean {
    return true
  }

  release(productId: string, quantity: number): void {}
}

class PaymentService {
  charge(customerId: string, amountInCents: number): boolean {
    return true
  }
}

class ShippingService {
  createShipment(productId: string, quantity: number): string | undefined {
    return "shipment-123"
  }
}

class NotificationService {
  sendOrderConfirmation(customerId: string, shipmentId: string): void {}
}

class CheckoutFacade {
  constructor(
    private readonly inventory: InventoryService,
    private readonly payment: PaymentService,
    private readonly shipping: ShippingService,
    private readonly notifications: NotificationService,
  ) {}

  placeOrder(
    customerId: string,
    productId: string,
    quantity: number,
    amountInCents: number,
  ): boolean {
    if (!this.inventory.reserve(productId, quantity)) return false

    if (!this.payment.charge(customerId, amountInCents)) {
      this.inventory.release(productId, quantity)
      return false
    }

    const shipmentId = this.shipping.createShipment(productId, quantity)
    if (!shipmentId) {
      this.inventory.release(productId, quantity)
      return false
    }

    this.notifications.sendOrderConfirmation(customerId, shipmentId)
    return true
  }
}

const checkout = new CheckoutFacade(
  new InventoryService(),
  new PaymentService(),
  new ShippingService(),
  new NotificationService(),
)
checkout.placeOrder("customer-1", "book-1", 1, 2999)
```

### Python
```python
class InventoryService:
    def reserve(self, product_id: str, quantity: int) -> bool:
        return True

    def release(self, product_id: str, quantity: int) -> None:
        pass


class PaymentService:
    def charge(self, customer_id: str, amount_in_cents: int) -> bool:
        return True


class ShippingService:
    def create_shipment(self, product_id: str, quantity: int) -> str | None:
        return "shipment-123"


class NotificationService:
    def send_order_confirmation(self, customer_id: str, shipment_id: str) -> None:
        pass


class CheckoutFacade:
    def __init__(
        self,
        inventory: InventoryService,
        payment: PaymentService,
        shipping: ShippingService,
        notifications: NotificationService,
    ) -> None:
        self.inventory = inventory
        self.payment = payment
        self.shipping = shipping
        self.notifications = notifications

    def place_order(
        self,
        customer_id: str,
        product_id: str,
        quantity: int,
        amount_in_cents: int,
    ) -> bool:
        if not self.inventory.reserve(product_id, quantity):
            return False

        if not self.payment.charge(customer_id, amount_in_cents):
            self.inventory.release(product_id, quantity)
            return False

        shipment_id = self.shipping.create_shipment(product_id, quantity)
        if shipment_id is None:
            self.inventory.release(product_id, quantity)
            return False

        self.notifications.send_order_confirmation(customer_id, shipment_id)
        return True


checkout = CheckoutFacade(
    InventoryService(),
    PaymentService(),
    ShippingService(),
    NotificationService(),
)
checkout.place_order("customer-1", "book-1", 1, 2999)
```

## Why This Is Better

- callers use one clear, task-focused API instead of coordinating multiple services
- workflow ordering and failure handling are centralized
- the facade shields most clients from changes inside the subsystem
- the common use case is easier to test and less likely to be implemented inconsistently
- advanced clients can still access individual subsystem services when needed

## Relationship With SOLID

- **Single Responsibility Principle:** the facade owns one high-level workflow while subsystems retain their focused responsibilities.
- **Open/Closed Principle:** subsystem implementations can change behind the facade without changing ordinary callers.
- **Dependency Inversion Principle:** callers can depend on a checkout interface while the facade receives subsystem dependencies through its constructor.

## When To Use It

- a common operation requires several services to work together
- a library or subsystem is difficult for most clients to use directly
- you need a stable boundary around code that is changing or being migrated
- controllers, command handlers, or UI code are repeating the same orchestration

## When Not To Use It

- the subsystem is already small and easy to use directly
- clients need many low-level operations that a single facade would only re-expose
- the facade is becoming a large "god object" that coordinates unrelated workflows

## Notes

- A facade simplifies access; it does not prevent advanced callers from using subsystem APIs directly.
- Keep facade methods focused on meaningful use cases such as `placeOrder`, not every operation offered by every subsystem.
- Facades are often application services that define a transaction or use-case boundary.
- A facade can use interfaces for its dependencies to make workflow tests independent of concrete services.
