# Abstract Factory Pattern

The Abstract Factory Pattern is a creational design pattern. It creates families of related objects without exposing concrete classes to client code.

## Intent

Use an abstract factory when:

- you need to create multiple related objects together
- those objects must stay compatible with one another
- client code should not depend on concrete implementations
- you want to switch entire product families in one place

## Problem

Suppose we are building a checkout system that supports multiple payment providers. Each provider needs a related set of objects:

- a payment processor
- a refund processor
- a receipt formatter

If the checkout flow directly creates Stripe, PayPal, or other concrete objects, it becomes tightly coupled to one vendor family and can accidentally mix incompatible components.

### Swift
```swift
final class CheckoutService {
    func completeCheckout(amount: Double, provider: String) {
        let paymentProcessor: Any
        let refundProcessor: Any
        let receiptFormatter: Any

        if provider == "stripe" {
            paymentProcessor = StripePaymentProcessor()
            refundProcessor = StripeRefundProcessor()
            receiptFormatter = StripeReceiptFormatter()
        } else {
            paymentProcessor = PayPalPaymentProcessor()
            refundProcessor = PayPalRefundProcessor()
            receiptFormatter = PayPalReceiptFormatter()
        }

        print("Created checkout objects for \(provider): \(paymentProcessor), \(refundProcessor), \(receiptFormatter)")
        print("Charging amount: \(amount)")
    }
}

final class StripePaymentProcessor {}
final class StripeRefundProcessor {}
final class StripeReceiptFormatter {}

final class PayPalPaymentProcessor {}
final class PayPalRefundProcessor {}
final class PayPalReceiptFormatter {}
```

### Go
```go
package main

import "fmt"

type StripePaymentProcessor struct{}
type StripeRefundProcessor struct{}
type StripeReceiptFormatter struct{}

type PayPalPaymentProcessor struct{}
type PayPalRefundProcessor struct{}
type PayPalReceiptFormatter struct{}

type CheckoutService struct{}

func (s *CheckoutService) CompleteCheckout(amount float64, provider string) {
	var paymentProcessor any
	var refundProcessor any
	var receiptFormatter any

	if provider == "stripe" {
		paymentProcessor = &StripePaymentProcessor{}
		refundProcessor = &StripeRefundProcessor{}
		receiptFormatter = &StripeReceiptFormatter{}
	} else {
		paymentProcessor = &PayPalPaymentProcessor{}
		refundProcessor = &PayPalRefundProcessor{}
		receiptFormatter = &PayPalReceiptFormatter{}
	}

	fmt.Printf("Created checkout objects for %s: %T, %T, %T\n", provider, paymentProcessor, refundProcessor, receiptFormatter)
	fmt.Println("Charging amount:", amount)
}
```

### TypeScript
```typescript
class StripePaymentProcessor {}
class StripeRefundProcessor {}
class StripeReceiptFormatter {}

class PayPalPaymentProcessor {}
class PayPalRefundProcessor {}
class PayPalReceiptFormatter {}

class CheckoutService {
  completeCheckout(amount: number, provider: string): void {
    let paymentProcessor: unknown
    let refundProcessor: unknown
    let receiptFormatter: unknown

    if (provider === "stripe") {
      paymentProcessor = new StripePaymentProcessor()
      refundProcessor = new StripeRefundProcessor()
      receiptFormatter = new StripeReceiptFormatter()
    } else {
      paymentProcessor = new PayPalPaymentProcessor()
      refundProcessor = new PayPalRefundProcessor()
      receiptFormatter = new PayPalReceiptFormatter()
    }

    console.log(
      `Created checkout objects for ${provider}:`,
      paymentProcessor,
      refundProcessor,
      receiptFormatter,
    )
    console.log(`Charging amount: ${amount}`)
  }
}
```

### Python
```python
class StripePaymentProcessor:
    pass


class StripeRefundProcessor:
    pass


class StripeReceiptFormatter:
    pass


class PayPalPaymentProcessor:
    pass


class PayPalRefundProcessor:
    pass


class PayPalReceiptFormatter:
    pass


class CheckoutService:
    def complete_checkout(self, amount: float, provider: str) -> None:
        if provider == "stripe":
            payment_processor = StripePaymentProcessor()
            refund_processor = StripeRefundProcessor()
            receipt_formatter = StripeReceiptFormatter()
        else:
            payment_processor = PayPalPaymentProcessor()
            refund_processor = PayPalRefundProcessor()
            receipt_formatter = PayPalReceiptFormatter()

        print(
            f"Created checkout objects for {provider}: "
            f"{payment_processor}, {refund_processor}, {receipt_formatter}"
        )
        print(f"Charging amount: {amount}")
```

This design has a few problems:

- the checkout flow is tightly coupled to vendor-specific classes
- adding a new provider requires touching business workflow code
- related objects can be created inconsistently in different places
- tests become harder because construction and behavior are mixed together

## Solution

Introduce:

- abstract product interfaces for each role
- concrete products grouped by family
- an abstract factory that creates the full family
- concrete factories for each provider or environment

Now the checkout service depends on abstractions and receives a factory that guarantees compatible objects.

### Swift
```swift
protocol PaymentProcessor {
    func charge(amount: Double)
}

protocol RefundProcessor {
    func refund(paymentId: String)
}

protocol ReceiptFormatter {
    func format(amount: Double) -> String
}

final class StripePaymentProcessor: PaymentProcessor {
    func charge(amount: Double) {
        print("Stripe charge: \(amount)")
    }
}

final class StripeRefundProcessor: RefundProcessor {
    func refund(paymentId: String) {
        print("Stripe refund: \(paymentId)")
    }
}

final class StripeReceiptFormatter: ReceiptFormatter {
    func format(amount: Double) -> String {
        "Stripe receipt for \(amount)"
    }
}

final class PayPalPaymentProcessor: PaymentProcessor {
    func charge(amount: Double) {
        print("PayPal charge: \(amount)")
    }
}

final class PayPalRefundProcessor: RefundProcessor {
    func refund(paymentId: String) {
        print("PayPal refund: \(paymentId)")
    }
}

final class PayPalReceiptFormatter: ReceiptFormatter {
    func format(amount: Double) -> String {
        "PayPal receipt for \(amount)"
    }
}

protocol PaymentGatewayFactory {
    func createPaymentProcessor() -> PaymentProcessor
    func createRefundProcessor() -> RefundProcessor
    func createReceiptFormatter() -> ReceiptFormatter
}

final class StripeFactory: PaymentGatewayFactory {
    func createPaymentProcessor() -> PaymentProcessor {
        StripePaymentProcessor()
    }

    func createRefundProcessor() -> RefundProcessor {
        StripeRefundProcessor()
    }

    func createReceiptFormatter() -> ReceiptFormatter {
        StripeReceiptFormatter()
    }
}

final class PayPalFactory: PaymentGatewayFactory {
    func createPaymentProcessor() -> PaymentProcessor {
        PayPalPaymentProcessor()
    }

    func createRefundProcessor() -> RefundProcessor {
        PayPalRefundProcessor()
    }

    func createReceiptFormatter() -> ReceiptFormatter {
        PayPalReceiptFormatter()
    }
}

final class CheckoutService {
    private let factory: PaymentGatewayFactory

    init(factory: PaymentGatewayFactory) {
        self.factory = factory
    }

    func completeCheckout(amount: Double) {
        let paymentProcessor = factory.createPaymentProcessor()
        let receiptFormatter = factory.createReceiptFormatter()

        paymentProcessor.charge(amount: amount)
        print(receiptFormatter.format(amount: amount))
    }
}
```

### Go
```go
package main

import "fmt"

type PaymentProcessor interface {
	Charge(amount float64)
}

type RefundProcessor interface {
	Refund(paymentID string)
}

type ReceiptFormatter interface {
	Format(amount float64) string
}

type StripePaymentProcessor struct{}

func (p *StripePaymentProcessor) Charge(amount float64) {
	fmt.Println("Stripe charge:", amount)
}

type StripeRefundProcessor struct{}

func (p *StripeRefundProcessor) Refund(paymentID string) {
	fmt.Println("Stripe refund:", paymentID)
}

type StripeReceiptFormatter struct{}

func (f *StripeReceiptFormatter) Format(amount float64) string {
	return fmt.Sprintf("Stripe receipt for %.2f", amount)
}

type PayPalPaymentProcessor struct{}

func (p *PayPalPaymentProcessor) Charge(amount float64) {
	fmt.Println("PayPal charge:", amount)
}

type PayPalRefundProcessor struct{}

func (p *PayPalRefundProcessor) Refund(paymentID string) {
	fmt.Println("PayPal refund:", paymentID)
}

type PayPalReceiptFormatter struct{}

func (f *PayPalReceiptFormatter) Format(amount float64) string {
	return fmt.Sprintf("PayPal receipt for %.2f", amount)
}

type PaymentGatewayFactory interface {
	CreatePaymentProcessor() PaymentProcessor
	CreateRefundProcessor() RefundProcessor
	CreateReceiptFormatter() ReceiptFormatter
}

type StripeFactory struct{}

func (f *StripeFactory) CreatePaymentProcessor() PaymentProcessor {
	return &StripePaymentProcessor{}
}

func (f *StripeFactory) CreateRefundProcessor() RefundProcessor {
	return &StripeRefundProcessor{}
}

func (f *StripeFactory) CreateReceiptFormatter() ReceiptFormatter {
	return &StripeReceiptFormatter{}
}

type PayPalFactory struct{}

func (f *PayPalFactory) CreatePaymentProcessor() PaymentProcessor {
	return &PayPalPaymentProcessor{}
}

func (f *PayPalFactory) CreateRefundProcessor() RefundProcessor {
	return &PayPalRefundProcessor{}
}

func (f *PayPalFactory) CreateReceiptFormatter() ReceiptFormatter {
	return &PayPalReceiptFormatter{}
}

type CheckoutService struct {
	factory PaymentGatewayFactory
}

func NewCheckoutService(factory PaymentGatewayFactory) *CheckoutService {
	return &CheckoutService{factory: factory}
}

func (s *CheckoutService) CompleteCheckout(amount float64) {
	paymentProcessor := s.factory.CreatePaymentProcessor()
	receiptFormatter := s.factory.CreateReceiptFormatter()

	paymentProcessor.Charge(amount)
	fmt.Println(receiptFormatter.Format(amount))
}
```

### TypeScript
```typescript
interface PaymentProcessor {
  charge(amount: number): void
}

interface RefundProcessor {
  refund(paymentId: string): void
}

interface ReceiptFormatter {
  format(amount: number): string
}

class StripePaymentProcessor implements PaymentProcessor {
  charge(amount: number): void {
    console.log(`Stripe charge: ${amount}`)
  }
}

class StripeRefundProcessor implements RefundProcessor {
  refund(paymentId: string): void {
    console.log(`Stripe refund: ${paymentId}`)
  }
}

class StripeReceiptFormatter implements ReceiptFormatter {
  format(amount: number): string {
    return `Stripe receipt for ${amount}`
  }
}

class PayPalPaymentProcessor implements PaymentProcessor {
  charge(amount: number): void {
    console.log(`PayPal charge: ${amount}`)
  }
}

class PayPalRefundProcessor implements RefundProcessor {
  refund(paymentId: string): void {
    console.log(`PayPal refund: ${paymentId}`)
  }
}

class PayPalReceiptFormatter implements ReceiptFormatter {
  format(amount: number): string {
    return `PayPal receipt for ${amount}`
  }
}

interface PaymentGatewayFactory {
  createPaymentProcessor(): PaymentProcessor
  createRefundProcessor(): RefundProcessor
  createReceiptFormatter(): ReceiptFormatter
}

class StripeFactory implements PaymentGatewayFactory {
  createPaymentProcessor(): PaymentProcessor {
    return new StripePaymentProcessor()
  }

  createRefundProcessor(): RefundProcessor {
    return new StripeRefundProcessor()
  }

  createReceiptFormatter(): ReceiptFormatter {
    return new StripeReceiptFormatter()
  }
}

class PayPalFactory implements PaymentGatewayFactory {
  createPaymentProcessor(): PaymentProcessor {
    return new PayPalPaymentProcessor()
  }

  createRefundProcessor(): RefundProcessor {
    return new PayPalRefundProcessor()
  }

  createReceiptFormatter(): ReceiptFormatter {
    return new PayPalReceiptFormatter()
  }
}

class CheckoutService {
  constructor(private readonly factory: PaymentGatewayFactory) {}

  completeCheckout(amount: number): void {
    const paymentProcessor = this.factory.createPaymentProcessor()
    const receiptFormatter = this.factory.createReceiptFormatter()

    paymentProcessor.charge(amount)
    console.log(receiptFormatter.format(amount))
  }
}
```

### Python
```python
from abc import ABC, abstractmethod


class PaymentProcessor(ABC):
    @abstractmethod
    def charge(self, amount: float) -> None:
        pass


class RefundProcessor(ABC):
    @abstractmethod
    def refund(self, payment_id: str) -> None:
        pass


class ReceiptFormatter(ABC):
    @abstractmethod
    def format(self, amount: float) -> str:
        pass


class StripePaymentProcessor(PaymentProcessor):
    def charge(self, amount: float) -> None:
        print(f"Stripe charge: {amount}")


class StripeRefundProcessor(RefundProcessor):
    def refund(self, payment_id: str) -> None:
        print(f"Stripe refund: {payment_id}")


class StripeReceiptFormatter(ReceiptFormatter):
    def format(self, amount: float) -> str:
        return f"Stripe receipt for {amount}"


class PayPalPaymentProcessor(PaymentProcessor):
    def charge(self, amount: float) -> None:
        print(f"PayPal charge: {amount}")


class PayPalRefundProcessor(RefundProcessor):
    def refund(self, payment_id: str) -> None:
        print(f"PayPal refund: {payment_id}")


class PayPalReceiptFormatter(ReceiptFormatter):
    def format(self, amount: float) -> str:
        return f"PayPal receipt for {amount}"


class PaymentGatewayFactory(ABC):
    @abstractmethod
    def create_payment_processor(self) -> PaymentProcessor:
        pass

    @abstractmethod
    def create_refund_processor(self) -> RefundProcessor:
        pass

    @abstractmethod
    def create_receipt_formatter(self) -> ReceiptFormatter:
        pass


class StripeFactory(PaymentGatewayFactory):
    def create_payment_processor(self) -> PaymentProcessor:
        return StripePaymentProcessor()

    def create_refund_processor(self) -> RefundProcessor:
        return StripeRefundProcessor()

    def create_receipt_formatter(self) -> ReceiptFormatter:
        return StripeReceiptFormatter()


class PayPalFactory(PaymentGatewayFactory):
    def create_payment_processor(self) -> PaymentProcessor:
        return PayPalPaymentProcessor()

    def create_refund_processor(self) -> RefundProcessor:
        return PayPalRefundProcessor()

    def create_receipt_formatter(self) -> ReceiptFormatter:
        return PayPalReceiptFormatter()


class CheckoutService:
    def __init__(self, factory: PaymentGatewayFactory) -> None:
        self.factory = factory

    def complete_checkout(self, amount: float) -> None:
        payment_processor = self.factory.create_payment_processor()
        receipt_formatter = self.factory.create_receipt_formatter()

        payment_processor.charge(amount)
        print(receipt_formatter.format(amount))
```

## Why This Is Better

- client code works with interfaces instead of concrete vendor classes
- related objects are created as a compatible family
- switching providers becomes a factory swap instead of scattered conditionals
- tests can inject a fake factory that returns predictable test doubles

## Relationship With SOLID

Abstract Factory often works well with SOLID principles:

- Open/Closed Principle: new product families can be added without rewriting client logic
- Single Responsibility Principle: family creation is separated from business workflow
- Dependency Inversion Principle: high-level code depends on abstract factories and product interfaces

## When To Use It

Abstract Factory is useful when:

- you need multiple related objects from the same family
- object compatibility matters
- the system supports multiple providers, themes, or environments
- you want to swap implementations at a higher level of configuration

## When Not To Use It

Do not introduce an abstract factory if you only create one simple object.

If there is no concept of a product family, a regular factory or direct construction is usually simpler.

## Notes

- Factory usually creates one product based on input.
- Abstract Factory creates a family of related products.
- A common real-world use case is switching between providers, database drivers, or UI themes.
