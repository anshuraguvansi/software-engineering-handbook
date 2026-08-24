## Open/Closed Principle

A class should be open for extension and closed for modification. This means we should design in a way that we can extend functionality without modifying the existing class.

Closed for modification does not mean no change is ever allowed. It means when we add a new behavior, we should prefer adding new code instead of changing already tested code.

Let’s understand this better with an example: let’s continue with the Payment manager example:


### Swift
```swift
class PaymentManager {
    func makeCashPayment(amount: Double) throws {
        // handle cash payment
    }

    func makeVisaPayment(amount: Double) throws {
        // handle visa payment using Visa SDK
    }
}
```

### Go
```golang
type PaymentManager struct {
    // Payment manager structure.
}

func (pm *PaymentManager) MakeCashPayment(amount float64) error {
    // handle cash payment
    return nil
}

func (pm *PaymentManager) MakeVisaPayment(amount float64) error {
    // handle visa payment using Visa SDK
    return nil
}
``` 

### TypeScript
```typescript
class PaymentManager {
    makeCashPayment(amount: number): void {
        // handle cash payment
        // throw new Error("Cash payment failed");
    }

    makeVisaPayment(amount: number): void {
        // handle visa payment using Visa SDK
    }
}
``` 

### Python
```python
class PaymentManager:
    def make_cash_payment(self, amount) -> None:
        # handle cash payment
        # raise Exception("Cash payment failed")
        pass

    def make_visa_payment(self, amount) -> None:
        # handle visa payment using Visa SDK
        pass
``` 

Now let's say we want to support UPI as a new payment method. For that, we have to modify the existing `PaymentManager` class, which means this implementation is violating the Open/Closed Principle.

Now let’s see how we can fix the Open/Closed Principle violation in this class.

### Swift
```swift
protocol PaymentMethod {
    func makePayment(amount: Double) throws 
}

final class CashPaymentMethod: PaymentMethod {
    func makePayment(amount: Double) throws {
        // handle cash payment
    }
}

final class VisaPaymentMethod: PaymentMethod {
    func makePayment(amount: Double) throws {
         // handle visa payment using Visa SDK
    }
}

class PaymentManager {
    func pay(using method: PaymentMethod, amount: Double) throws {
        try method.makePayment(amount: amount)
    }
}
```

### Go
```golang
type PaymentMethod interface {
	MakePayment(amount float64) error
}

type VisaPaymentMethod struct {
}

func (vp *VisaPaymentMethod) MakePayment(amount float64) error {
	// handle visa payment using Visa SDK
	return nil
}

type CashPaymentMethod struct {
	// cash payment method
}

func (cp *CashPaymentMethod) MakePayment(amount float64) error {
	// handle cash payment
	return nil
}

type PaymentManager struct {
    // Payment manager.
}

func (pm *PaymentManager) PayUsing(method PaymentMethod, amount float64) error {
    return method.MakePayment(amount)
}
```

### TypeScript
```typescript
interface PaymentMethod {
  makePayment(amount: number): void
}

class CashPayment implements PaymentMethod {
  makePayment(amount: number): void {
    // make cash payment logic
  }
}

class VisaPayment implements PaymentMethod {
  makePayment(amount: number): void {
    // make visa payment logic
  }
}

class PaymentManager {
  payUsing(method: PaymentMethod, amount: number): void {
    method.makePayment(amount)
  }
}
```
### python
```python
from abc import ABC, abstractmethod

class PaymentMethod(ABC):
    @abstractmethod
    def make_payment(self, amount) -> None:
        pass

class CashPayment(PaymentMethod):
    def make_payment(self, amount) -> None:
        #  cash payment logic
        pass

class VisaPayment(PaymentMethod):
    def make_payment(self, amount) -> None:
        #  visa payment logic
        pass

class PaymentManager:
    def pay_using(self, method: PaymentMethod, amount) -> None:
        method.make_payment(amount)
```

Now you see `PaymentManager` does only one job: it orchestrates execution. So if we add UPI as a new payment method, there will not be any change in `PaymentManager`; we will create a new payment method for UPI.

### Swift
```swift
final class UPIPaymentMethod: PaymentMethod {
    func makePayment(amount: Double) throws {
         // handle UPI payment
    }
}
```

### Go
```golang
type UPIPaymentMethod struct {
}

func (up *UPIPaymentMethod) MakePayment(amount float64) error {
	// handle UPI payment
	return nil
}
```

### TypeScript
```typescript
class UPIPaymentMethod implements PaymentMethod {
  makePayment(amount: number): void {
    // make UPI payment logic
  }
}
```

### Python
```python
class UPIPayment(PaymentMethod):
    def make_payment(self, amount) -> None:
        #  UPI payment logic
        pass
``` 