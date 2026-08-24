## Interface Segregation Principle

No client should be forced to depend on (or implement) methods it doesn't use. Prefer several small, role-specific interfaces over one large, general-purpose one.

Let's understand this with an example.

### Swift
```swift
protocol PaymentMethod {
    func makePayment(amount: Double) throws
    func refund(amount: Double) throws   // Cash can't be refunded programmatically
}

final class CashPayment: PaymentMethod {
    func makePayment(amount: Double) throws { /* ... */ }
    func refund(amount: Double) throws {
        throw PaymentError.notSupported   // forced to implement something meaningless
    }
}
```

### Go
```golang
import "errors"

type PaymentMethod interface {
    makePayment(amount float64) error
    refund(amount float64) error // Cash can't be refunded programmatically
}

type CashPayment struct{}

func (cp *CashPayment) makePayment(amount float64) error {
    // payment logic
    return nil
}

func (cp *CashPayment) refund(amount float64) error {
    // forced to implement something meaningless
    return errors.New("refund not supported for cash payments")
}
```

### TypeScript
```typescript
interface PaymentMethod {
    makePayment(amount: number): void
    refund(amount: number): void
}

class CashPayment implements PaymentMethod {
    makePayment(amount: number) {
        // payment logic
    }

    refund(amount: number) {
        // forced to implement something meaningless
        throw new Error("refund not supported for cash payments")
    }
}
```

### Python
```python
from abc import ABC, abstractmethod

class PaymentMethod(ABC):
    @abstractmethod
    def make_payment(self, amount) -> None:
        pass

    @abstractmethod
    def make_refund(self, amount) -> None:
        pass

class CashPayment(PaymentMethod):
    def make_payment(self, amount) -> None:
        # payment logic
        pass

    def make_refund(self, amount) -> None:
        raise NotImplementedError("Refund is not supported for cash payments")

```


This design violates the Interface Segregation Principle because it forces `CashPayment` to implement refund behavior that it does not support. In the case of cash, refunds are typically not handled programmatically. Let's fix this.

### Swift
```swift
protocol PaymentMethod {
    func makePayment(amount: Double) throws
}

protocol RefundablePayment: PaymentMethod {
    func refund(amount: Double) throws  
}

final class VisaPayment: RefundablePayment {
    func makePayment(amount: Double) throws { /* ... */ }
    func refund(amount: Double) throws { /* ... */ }
}

final class CashPayment: PaymentMethod {
    func makePayment(amount: Double) throws { /* ... */ }
}

final class PaymentManager {
    func processRefund(for payment: RefundablePayment, amount: Double) throws {
        try payment.refund(amount: amount)
    }
}
```

### Go
```golang
type PaymentMethod interface {
    makePayment(amount float64) error
}

type RefundablePayment interface {
    PaymentMethod // Embedding
    refund(amount float64) error
}

type VisaPayment struct{}

func (vp *VisaPayment) makePayment(amount float64) error {
    // payment logic
    return nil
}

func (vp *VisaPayment) refund(amount float64) error {
    // refund logic
    return nil
}

type CashPayment struct{}

func (cp *CashPayment) makePayment(amount float64) error {
    // payment logic
    return nil
}

type PaymentManager struct{}

func (pm *PaymentManager) processRefund(payment RefundablePayment, amount float64) error {
    return payment.refund(amount)
}

```


### TypeScript
```typescript
interface PaymentMethod {
    makePayment(amount: number): void
}

interface RefundablePayment extends PaymentMethod {
    refund(amount: number): void
}

class VisaPayment implements RefundablePayment {
    makePayment(amount: number) {
        // payment logic
    }

    refund(amount: number) {
        // refund logic
    }
}

class CashPayment implements PaymentMethod {
    makePayment(amount: number) {
        // payment logic
    }
}

class PaymentManager {
    processRefund(payment: RefundablePayment, amount: number): void {
        payment.refund(amount)
    }
}
```

### Python
```python
from abc import ABC, abstractmethod

class PaymentMethod(ABC):
    @abstractmethod
    def make_payment(self, amount) -> None:
        pass

class RefundablePayment(PaymentMethod):
    @abstractmethod
    def make_refund(self, amount) -> None:
        pass


class VisaPayment(RefundablePayment):
    def make_payment(self, amount) -> None:
        # payment logic
        pass

    def make_refund(self, amount) -> None:
        # refund logic
        pass

class CashPayment(PaymentMethod):
    def make_payment(self, amount) -> None:
        # payment logic
        pass

class PaymentManager:
    def process_refund(self, payment: RefundablePayment, amount) -> None:
        payment.make_refund(amount)

```