# Factory Pattern

The Factory Pattern is a creational design pattern. It centralizes object creation so client code depends on abstractions instead of concrete implementations.

## Intent

Use a factory when:

- object creation involves branching logic
- client code should not know concrete class names
- setup rules may change over time
- you want to keep business logic focused on behavior, not construction

## Problem

Suppose we are building an order system that sends shipment notifications. At first we support only email. Later, the system also needs SMS, push, and other channels.

If the business service directly creates concrete notification objects, it becomes tightly coupled to implementation details.

### Swift
```swift
final class OrderService {
    func shipOrder(orderId: String, channel: String) {
        print("Shipping order: \(orderId)")

        if channel == "email" {
            let notifier = EmailNotification()
            notifier.send(message: "Your order has shipped")
        } else if channel == "sms" {
            let notifier = SMSNotification()
            notifier.send(message: "Your order has shipped")
        }
    }
}

final class EmailNotification {
    func send(message: String) {
        print("Email: \(message)")
    }
}

final class SMSNotification {
    func send(message: String) {
        print("SMS: \(message)")
    }
}
```

### Go
```go
package main

import "fmt"

type EmailNotification struct{}

func (n *EmailNotification) Send(message string) {
	fmt.Println("Email:", message)
}

type SMSNotification struct{}

func (n *SMSNotification) Send(message string) {
	fmt.Println("SMS:", message)
}

type OrderService struct{}

func (s *OrderService) ShipOrder(orderID string, channel string) {
	fmt.Println("Shipping order:", orderID)

	if channel == "email" {
		notifier := &EmailNotification{}
		notifier.Send("Your order has shipped")
	} else if channel == "sms" {
		notifier := &SMSNotification{}
		notifier.Send("Your order has shipped")
	}
}
```

### TypeScript
```typescript
class EmailNotification {
  send(message: string): void {
    console.log(`Email: ${message}`)
  }
}

class SMSNotification {
  send(message: string): void {
    console.log(`SMS: ${message}`)
  }
}

class OrderService {
  shipOrder(orderId: string, channel: string): void {
    console.log(`Shipping order: ${orderId}`)

    if (channel === "email") {
      const notifier = new EmailNotification()
      notifier.send("Your order has shipped")
    } else if (channel === "sms") {
      const notifier = new SMSNotification()
      notifier.send("Your order has shipped")
    }
  }
}
```

### Python
```python
class EmailNotification:
    def send(self, message: str) -> None:
        print(f"Email: {message}")


class SMSNotification:
    def send(self, message: str) -> None:
        print(f"SMS: {message}")


class OrderService:
    def ship_order(self, order_id: str, channel: str) -> None:
        print(f"Shipping order: {order_id}")

        if channel == "email":
            notifier = EmailNotification()
            notifier.send("Your order has shipped")
        elif channel == "sms":
            notifier = SMSNotification()
            notifier.send("Your order has shipped")
```

This design has a few problems:

- the business layer is tightly coupled to concrete classes
- adding a new notification type requires modifying existing workflow code
- creation logic gets repeated in multiple places
- testing becomes harder because construction happens inside the method

## Solution

Introduce:

- a common product abstraction
- concrete implementations of that abstraction
- a factory responsible for creating the right implementation

Now the business service asks for a notification object without caring which concrete class gets created.

### Swift
```swift
protocol Notification {
    func send(message: String)
}

final class EmailNotification: Notification {
    func send(message: String) {
        print("Email: \(message)")
    }
}

final class SMSNotification: Notification {
    func send(message: String) {
        print("SMS: \(message)")
    }
}

final class PushNotification: Notification {
    func send(message: String) {
        print("Push: \(message)")
    }
}

enum NotificationChannel {
    case email
    case sms
    case push
}

protocol NotificationFactory {
    func create(channel: NotificationChannel) -> Notification
}

final class DefaultNotificationFactory: NotificationFactory {
    func create(channel: NotificationChannel) -> Notification {
        switch channel {
        case .email:
            return EmailNotification()
        case .sms:
            return SMSNotification()
        case .push:
            return PushNotification()
        }
    }
}

final class OrderService {
    private let factory: NotificationFactory

    init(factory: NotificationFactory) {
        self.factory = factory
    }

    func shipOrder(orderId: String, channel: NotificationChannel) {
        print("Shipping order: \(orderId)")
        let notifier = factory.create(channel: channel)
        notifier.send(message: "Your order has shipped")
    }
}
```

### Go
```go
package main

import "fmt"

type Notification interface {
	Send(message string)
}

type EmailNotification struct{}

func (n *EmailNotification) Send(message string) {
	fmt.Println("Email:", message)
}

type SMSNotification struct{}

func (n *SMSNotification) Send(message string) {
	fmt.Println("SMS:", message)
}

type PushNotification struct{}

func (n *PushNotification) Send(message string) {
	fmt.Println("Push:", message)
}

type NotificationFactory interface {
	Create(channel string) (Notification, error)
}

type DefaultNotificationFactory struct{}

func (f *DefaultNotificationFactory) Create(channel string) (Notification, error) {
	switch channel {
	case "email":
		return &EmailNotification{}, nil
	case "sms":
		return &SMSNotification{}, nil
	case "push":
		return &PushNotification{}, nil
	default:
		return nil, fmt.Errorf("unsupported notification channel: %s", channel)
	}
}

type OrderService struct {
	factory NotificationFactory
}

func NewOrderService(factory NotificationFactory) *OrderService {
	return &OrderService{factory: factory}
}

func (s *OrderService) ShipOrder(orderID string, channel string) error {
	fmt.Println("Shipping order:", orderID)

	notifier, err := s.factory.Create(channel)
	if err != nil {
		return err
	}

	notifier.Send("Your order has shipped")
	return nil
}
```

### TypeScript
```typescript
interface Notification {
  send(message: string): void
}

class EmailNotification implements Notification {
  send(message: string): void {
    console.log(`Email: ${message}`)
  }
}

class SMSNotification implements Notification {
  send(message: string): void {
    console.log(`SMS: ${message}`)
  }
}

class PushNotification implements Notification {
  send(message: string): void {
    console.log(`Push: ${message}`)
  }
}

type NotificationChannel = "email" | "sms" | "push"

interface NotificationFactory {
  create(channel: NotificationChannel): Notification
}

class DefaultNotificationFactory implements NotificationFactory {
  create(channel: NotificationChannel): Notification {
    switch (channel) {
      case "email":
        return new EmailNotification()
      case "sms":
        return new SMSNotification()
      case "push":
        return new PushNotification()
    }
  }
}

class OrderService {
  constructor(private readonly factory: NotificationFactory) {}

  shipOrder(orderId: string, channel: NotificationChannel): void {
    console.log(`Shipping order: ${orderId}`)
    const notifier = this.factory.create(channel)
    notifier.send("Your order has shipped")
  }
}
```

### Python
```python
from abc import ABC, abstractmethod


class Notification(ABC):
    @abstractmethod
    def send(self, message: str) -> None:
        pass


class EmailNotification(Notification):
    def send(self, message: str) -> None:
        print(f"Email: {message}")


class SMSNotification(Notification):
    def send(self, message: str) -> None:
        print(f"SMS: {message}")


class PushNotification(Notification):
    def send(self, message: str) -> None:
        print(f"Push: {message}")


class NotificationFactory(ABC):
    @abstractmethod
    def create(self, channel: str) -> Notification:
        pass


class DefaultNotificationFactory(NotificationFactory):
    def create(self, channel: str) -> Notification:
        if channel == "email":
            return EmailNotification()
        if channel == "sms":
            return SMSNotification()
        if channel == "push":
            return PushNotification()
        raise ValueError(f"Unsupported notification channel: {channel}")


class OrderService:
    def __init__(self, factory: NotificationFactory) -> None:
        self.factory = factory

    def ship_order(self, order_id: str, channel: str) -> None:
        print(f"Shipping order: {order_id}")
        notifier = self.factory.create(channel)
        notifier.send("Your order has shipped")
```

## Why This Is Better

- `OrderService` focuses on business workflow
- creation logic is centralized in one place
- new product types are easier to introduce
- tests can inject a fake or stub factory

## Relationship With SOLID

Factory often works well with SOLID principles:

- Open/Closed Principle: new implementations can be added with minimal changes to client code
- Single Responsibility Principle: creation logic moves out of business services
- Dependency Inversion Principle: high-level modules can depend on factory and product abstractions

## When To Use It

Factory is useful when:

- object creation is conditional
- constructors are complex
- different environments need different implementations
- you want to hide third-party SDK details from core business logic

## When Not To Use It

Do not introduce a factory just for the sake of abstraction.

If construction is a single line and unlikely to change, a direct constructor call is usually simpler and better.

## Notes

- A simple factory centralizes creation in one class or function.
- Factory Method is a more specific pattern where subclasses decide what to create.
- Factory is about object creation, not object behavior after creation.
