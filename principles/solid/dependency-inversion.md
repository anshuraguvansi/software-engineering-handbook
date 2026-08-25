## Dependency Inversion Principle

High-level modules should not depend on low-level modules — both should depend on abstractions. Abstractions should not depend on details; details should depend on abstractions.

"Depend on protocols, not concrete types" is necessary but incomplete. The key idea is that both layers depend on the same abstraction, and that abstraction is usually owned by the high-level (policy) module.

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

### Correct Design (Abstraction Owned by High-Level Module)

```swift
// BusinessCore (high-level policy module)
protocol LoggerProtocol {
    func log(_ message: String)
}

final class PaymentManager {
    private let logger: LoggerProtocol
    init(logger: LoggerProtocol) {
        self.logger = logger
    }
    // PaymentManager's module has ZERO import of any concrete infra module
}

// In a separate Infrastructure module,
// which depends INWARD on the business module's protocols:
import BusinessCore
final class FirebaseLogger: LoggerProtocol {
    func log(_ message: String) { /* Firebase call */ }
}
```

### Go
```go
// businesscore/logger.go (high-level module)
package businesscore

type Logger interface {
    Log(message string)
}

type PaymentManager struct {
    logger Logger
}

func NewPaymentManager(logger Logger) *PaymentManager {
    return &PaymentManager{logger: logger}
}

// infrastructure/firebase_logger.go (low-level module)
package infrastructure

import "company/businesscore"

type FirebaseLogger struct{}

func (fl *FirebaseLogger) Log(message string) {
    // Firebase log call
}

var _ businesscore.Logger = (*FirebaseLogger)(nil)
```

### TypeScript
```typescript
// business-core.ts (high-level module)
export interface Logger {
    log(message: string): void
}

export class PaymentManager {
    constructor(private readonly logger: Logger) {}
}

// firebase-logger.ts (low-level module)
import { Logger } from "./business-core"

export class FirebaseLogger implements Logger {
    log(message: string): void {
        // Firebase log call
    }
}
```

### Python
```python
# business_core.py (high-level module)
from abc import ABC, abstractmethod

class Logger(ABC):
    @abstractmethod
    def log(self, message: str) -> None:
        pass

class PaymentManager:
    def __init__(self, logger: Logger) -> None:
        self.logger = logger

# firebase_logger.py (low-level module)
from business_core import Logger

class FirebaseLogger(Logger):
    def log(self, message: str) -> None:
        # Firebase log call
        pass
```

DIP is a dependency direction rule: the interface is owned by the high-level policy layer, and low-level details depend on it, not vice versa.