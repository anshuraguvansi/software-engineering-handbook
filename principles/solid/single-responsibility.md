## Single Responsibility Principle

A class/function/module should have a single reason to change. This means there should be one actor that can drive the change.

Let’s understand this better with an example: let’s design a payment manager that can handle multiple payment types.


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

This implementation is violating the Single Responsibility Principle. Why? Because now there are two actors that can bring change to this class: one is when the finance department changes cash reconciliation logic, and the other is any change in the Visa SDK.

**Actor means the source of a change request, like the finance team, compliance team, or SDK vendor.**

You might think, what is the problem if there are two actors driving change? For a small app or feature, this might work. But in a big and complex project, it becomes challenging because a change in cash payment logic might require extra testing of the Visa flow as well, just to check side effects. Readability also becomes a problem. Think of having 6-7 payment methods; it becomes difficult to understand and make changes, especially for a new team member.

Now let’s see how we can fix the SRP violation in this class.

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

### Python
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
    def make_payment(self, method: PaymentMethod, amount) -> None:
        method.make_payment(amount)
``` 

Now you see `PaymentManager` does only one job: it orchestrates execution. It does not know how Cash or Visa works. It only knows that they work. By separating Cash and Visa into their own classes, the code becomes much easier to test, read, and modify.