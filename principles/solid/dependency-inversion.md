## Dependency Inversion Principle

High-level modules should not depend on low-level modules — both should depend on abstractions. Abstractions should not depend on details; details should depend on abstractions.

"Depend on protocols, not concrete types" is necessary but incomplete. The key idea is that both layers depend on the same abstraction, and that abstraction is owned by the policy side of the system. It may live in the high-level module itself, or in a dedicated contracts/ports module when several high-level modules need the same abstraction.

Let's understand this with an example.

### Violating Design (Abstraction Owned by Low-Level Module)

### Swift
```swift
// Inside a LoggingKit infrastructure module:
protocol LoggerProtocol {
    func log(_ message: String)
}
class FirebaseLogger: LoggerProtocol { ... }

// PaymentManager still has to `import LoggingKit` just to reference LoggerProtocol
import LoggingKit

final class PaymentManager {
    // abstraction used, but PaymentManager still depends on LoggingKit module
    let logger: LoggerProtocol
    init(logger: LoggerProtocol) {
        self.logger = logger
    }
}
```

### Go
```go
// Inside a LoggingKit infrastructure module:
type Logger interface {
    Log(message string)
}

type FirebaseLogger struct{}

func (fl *FirebaseLogger) Log(message string) {
    // Firebase log call
}

// PaymentManager (high-level) must import loggingkit to reference Logger
// import "company/loggingkit"
type PaymentManager struct {
    logger Logger
}

func NewPaymentManager(logger Logger) *PaymentManager {
    return &PaymentManager{logger: logger}
}
```

### TypeScript
```typescript
// logging-kit.ts (low-level infrastructure module)
export interface Logger {
    log(message: string): void
}

export class FirebaseLogger implements Logger {
    log(message: string): void {
        // Firebase log call
    }
}

// payment-manager.ts (high-level module)
// Still forced to import from low-level module
import { Logger } from "./logging-kit"

export class PaymentManager {
    constructor(private readonly logger: Logger) {}
}
```

### Python
```python
# logging_kit.py (low-level infrastructure module)
from abc import ABC, abstractmethod

class Logger(ABC):
    @abstractmethod
    def log(self, message: str) -> None:
        pass

class FirebaseLogger(Logger):
    def log(self, message: str) -> None:
        # Firebase log call
        pass

# payment_manager.py (high-level module)
# Still forced to import from low-level module
from logging_kit import Logger

class PaymentManager:
    def __init__(self, logger: Logger) -> None:
        self.logger = logger
```

In this design, the high-level module still imports the low-level module just to access the abstraction, so dependency direction is still wrong.

### Correct Design (Abstraction Owned by Policy or Shared Contracts Module)

When only one high-level module needs an abstraction, the policy module can own it directly. When several high-level modules need the same abstraction, place it in a focused contracts module. The contracts module should contain interfaces only; it should not become a generic `Common` module or depend on infrastructure.

In this example, `PaymentCore` and another business module both use logging, so `BusinessContracts` owns the shared port:

```swift
// BusinessContracts (shared contracts module)
protocol LoggerProtocol {
    func log(_ message: String)
}

// BusinessCore (high-level policy module)
import BusinessContracts

final class PaymentManager {
    private let logger: LoggerProtocol
    init(logger: LoggerProtocol) {
        self.logger = logger
    }
    // PaymentManager's module has ZERO import of any concrete infra module
}

// In a separate Infrastructure module,
// which depends INWARD on the contracts module:
import BusinessContracts
final class FirebaseLogger: LoggerProtocol {
    func log(_ message: String) { /* Firebase call */ }
}
```

### Go
```go
// businesscontracts/logger.go (shared contracts module)
package businesscontracts

type Logger interface {
    Log(message string)
}

// businesscore/payment_manager.go (high-level module)
package businesscore

import "company/businesscontracts"

type PaymentManager struct {
	logger businesscontracts.Logger
}

func NewPaymentManager(logger businesscontracts.Logger) *PaymentManager {
    return &PaymentManager{logger: logger}
}

// infrastructure/firebase_logger.go (low-level module)
package infrastructure

import "company/businesscontracts"

type FirebaseLogger struct{}

func (fl *FirebaseLogger) Log(message string) {
    // Firebase log call
}

var _ businesscontracts.Logger = (*FirebaseLogger)(nil)
```

### TypeScript
```typescript
// business-contracts.ts (shared contracts module)
export interface Logger {
    log(message: string): void
}

// payment-manager.ts (high-level module)
import { Logger } from "./business-contracts"

export class PaymentManager {
    constructor(private readonly logger: Logger) {}
}

// firebase-logger.ts (low-level module)
import { Logger } from "./business-contracts"

export class FirebaseLogger implements Logger {
    log(message: string): void {
        // Firebase log call
    }
}
```

### Python
```python
# business_contracts.py (shared contracts module)
from abc import ABC, abstractmethod

class Logger(ABC):
    @abstractmethod
    def log(self, message: str) -> None:
        pass

# payment_manager.py (high-level module)
from business_contracts import Logger

class PaymentManager:
    def __init__(self, logger: Logger) -> None:
        self.logger = logger

# firebase_logger.py (low-level module)
from business_contracts import Logger

class FirebaseLogger(Logger):
    def log(self, message: str) -> None:
        # Firebase log call
        pass
```

DIP is a dependency direction rule: the interface is owned by the high-level policy layer, either directly or through a dedicated contracts/ports module. Low-level details depend on that abstraction, not vice versa.

Choose the ownership based on the relationship between the modules:

- **One consumer:** keep the abstraction in the high-level policy module.
- **Several consumers:** extract it into a focused contracts or ports module that contains no infrastructure details.
- **Avoid:** placing the abstraction in the low-level module, or creating a broad `Common` module that accumulates unrelated interfaces.
