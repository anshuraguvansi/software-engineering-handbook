## Liskov-Substitution Principle

Subtypes must be substitutable for their base type without altering the correctness of the program. Any code written against the base type/protocol should work correctly with any conforming subtype.

LSP violations are almost never about method signatures (the compiler enforces those). They're about behavioral contracts: preconditions, postconditions, side effects, and failure semantics that the type system can't check.

Let's understand this with an example. In our previous payment method, let's say we got a new requirement to provide support for two-factor authentication for the payment.

```swift
protocol PaymentMethod {
    /// Throws if payment cannot be completed for any reason.
    /// Returns normally only on confirmed success.
    func makePayment(amount: Double) throws
}

final class TwoFactorPaymentDecorator: PaymentMethod {
    private let wrapped: PaymentMethod

    func makePayment(amount: Double) throws {
        guard userConfirmed2FA() else { return }   // silently does nothing on failure
        try wrapped.makePayment(amount: amount)
    }
}
```

```golang
type PaymentMethod interface {
    // Returns an error if payment cannot be completed for any reason.
    // Returns nil only on confirmed success.
    MakePayment(amount float64) error
}

type TwoFactorPaymentDecorator struct {
    wrapped PaymentMethod
}

func (tf *TwoFactorPaymentDecorator) MakePayment(amount float64) error {
    if userConfirmed2FA() {
        _ = tf.wrapped.MakePayment(amount)
    }
    return nil // silently does nothing on failure
}
```

```typescript 
interface PaymentMethod {
  makePayment(amount: number): void
}

class TwoFactorPaymentDecorator implements PaymentMethod {
    private wrapped: PaymentMethod

    constructor(wrapped: PaymentMethod) {
        this.wrapped = wrapped
    }

    makePayment(amount: number): void {
        if userConfirmed2FA() {
            this.wrapped.makePayment(amount)
        }
        // silently does nothing on failure
    }
}
```

If you look at the `TwoFactorPaymentDecorator`, you may have noticed that in case of 2FA failure the method silently returns, which doesn't honor the contract. To fix this, the method should throw an error.

```swift
protocol PaymentMethod {
    /// Throws if payment cannot be completed for any reason.
    /// Returns normally only on confirmed success.
    func makePayment(amount: Double) throws
}

final class TwoFactorPaymentDecorator: PaymentMethod {
    private let wrapped: PaymentMethod
    private let twoFactorService: TwoFactorService

    func makePayment(amount: Double) throws {
        guard twoFactorService.confirm() else {
            throw PaymentError.twoFactorDeclined   // contract honored -- caller can react
        }
        try wrapped.makePayment(amount: amount)
    }
}
```

```golang
type PaymentMethod interface {
    // Returns an error if payment cannot be completed for any reason.
    // Returns nil only on confirmed success.
    MakePayment(amount float64) error
}

type TwoFactorPaymentDecorator struct {
    wrapped PaymentMethod
}

func (tf *TwoFactorPaymentDecorator) MakePayment(amount float64) error {
    if !userConfirmed2FA() {
        return ErrTwoFactorDeclined // contract honored -- caller can react
    }
    return tf.wrapped.MakePayment(amount)
}
```

```typescript 
interface PaymentMethod {
  makePayment(amount: number): void
}

class TwoFactorPaymentDecorator implements PaymentMethod {
    private wrapped: PaymentMethod

    constructor(wrapped: PaymentMethod) {
        this.wrapped = wrapped
    }

    makePayment(amount: number): void {
        if !userConfirmed2FA() {
            throw new Error("ErrTwoFactorDeclined") // contract honored -- caller can react
        }
        this.wrapped.makePayment(amount)
    }
}
```