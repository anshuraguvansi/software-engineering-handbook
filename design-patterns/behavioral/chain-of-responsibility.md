# Chain of Responsibility Pattern

The Chain of Responsibility Pattern is a behavioral design pattern. It passes a request through an ordered series of handlers until one handles it or the chain ends.

## Intent

Use Chain of Responsibility when:

- several handlers may process or reject a request
- the sender should not choose a concrete handler
- processing steps should be reordered or composed at runtime
- middleware-like behavior should run in a predictable sequence

## Problem

Suppose an API endpoint authenticates a user, applies rate limits, and validates input. Putting every check in the endpoint couples request handling to a fixed sequence that gets duplicated across endpoints.

### Swift
```swift
func createOrder(_ request: Request) {
    guard request.hasValidToken else { return }
    guard !request.isRateLimited else { return }
    guard request.itemCount > 0 else { return }
    print("Order created")
}
```

### Go
```go
func createOrder(request Request) {
	if !request.HasValidToken || request.IsRateLimited || request.ItemCount == 0 { return }
	println("Order created")
}
```

### TypeScript
```typescript
function createOrder(request: OrderRequest): void {
  if (!request.hasValidToken) return
  if (request.isRateLimited) return
  if (request.itemCount === 0) return
  console.log("Order created")
}
```

### Python
```python
def create_order(request: Request) -> None:
    if not request.has_valid_token or request.is_rate_limited:
        return
    if request.item_count == 0:
        return
    print("Order created")
```

This design has a few problems:

- endpoints repeat infrastructure and validation logic
- adding or reordering a check requires editing the workflow
- checks are difficult to reuse and test independently
- the sender must know the complete processing sequence

## Solution

Give each handler the same interface and a reference to the next handler. A handler either throws or returns an error when it rejects the request, handles it successfully, or delegates it to the next link. Rejections therefore stop the chain and remain visible to the caller.

### Swift
```swift
struct Request {
    let hasValidToken: Bool
    let isRateLimited: Bool
    let itemCount: Int
}

enum RequestError: Error {
    case unauthorized
    case rateLimited
    case emptyOrder
}

protocol Handler: AnyObject {
    func handle(_ request: Request) throws
}

class BaseHandler: Handler {
    private let next: (any Handler)?
    init(next: (any Handler)? = nil) { self.next = next }
    func handle(_ request: Request) throws {
        try next?.handle(request)
    }
}

final class AuthenticationHandler: BaseHandler {
    override func handle(_ request: Request) throws {
        guard request.hasValidToken else { throw RequestError.unauthorized }
        try super.handle(request)
    }
}

final class RateLimitHandler: BaseHandler {
    override func handle(_ request: Request) throws {
        guard !request.isRateLimited else { throw RequestError.rateLimited }
        try super.handle(request)
    }
}

final class CreateOrderHandler: BaseHandler {
    override func handle(_ request: Request) throws {
        guard request.itemCount > 0 else { throw RequestError.emptyOrder }
        print("Order created")
    }
}

let chain = AuthenticationHandler(
    next: RateLimitHandler(next: CreateOrderHandler())
)

do {
    try chain.handle(Request(hasValidToken: true, isRateLimited: false, itemCount: 2))
} catch {
    print("Request rejected: \(error)")
}
```

### Go
```go
package main

import "errors"

type Request struct {
	HasValidToken bool
	IsRateLimited bool
	ItemCount     int
}

type Handler interface { Handle(Request) error }

type AuthenticationHandler struct { next Handler }
func (h AuthenticationHandler) Handle(r Request) error {
	if !r.HasValidToken { return errors.New("unauthorized") }
	return h.next.Handle(r)
}

type RateLimitHandler struct { next Handler }
func (h RateLimitHandler) Handle(r Request) error {
	if r.IsRateLimited { return errors.New("too many requests") }
	return h.next.Handle(r)
}

type CreateOrderHandler struct{}
func (CreateOrderHandler) Handle(r Request) error {
	if r.ItemCount == 0 { return errors.New("empty order") }
	println("Order created")
	return nil
}

func main() {
	chain := AuthenticationHandler{next: RateLimitHandler{next: CreateOrderHandler{}}}
	if err := chain.Handle(Request{HasValidToken: true, ItemCount: 2}); err != nil {
		println("Request rejected: " + err.Error())
	}
}
```

### TypeScript
```typescript
type OrderRequest = {
  hasValidToken: boolean
  isRateLimited: boolean
  itemCount: number
}

interface Handler { handle(request: OrderRequest): void }

abstract class ChainedHandler implements Handler {
  constructor(private readonly next?: Handler) {}
  handle(request: OrderRequest): void { this.next?.handle(request) }
}

class AuthenticationHandler extends ChainedHandler {
  handle(request: OrderRequest): void {
    if (!request.hasValidToken) throw new Error("Unauthorized")
    super.handle(request)
  }
}

class RateLimitHandler extends ChainedHandler {
  handle(request: OrderRequest): void {
    if (request.isRateLimited) throw new Error("Too many requests")
    super.handle(request)
  }
}

class CreateOrderHandler implements Handler {
  handle(request: OrderRequest): void {
    if (request.itemCount === 0) throw new Error("Empty order")
    console.log("Order created")
  }
}

const chain = new AuthenticationHandler(
  new RateLimitHandler(new CreateOrderHandler()),
)

try {
  chain.handle({ hasValidToken: true, isRateLimited: false, itemCount: 2 })
} catch (error) {
  console.log(`Request rejected: ${String(error)}`)
}
```

### Python
```python
from dataclasses import dataclass
from typing import Protocol


@dataclass(frozen=True)
class Request:
    has_valid_token: bool
    is_rate_limited: bool
    item_count: int


class Handler(Protocol):
    def handle(self, request: Request) -> None:
        pass


class RequestRejected(Exception):
    pass


class AuthenticationHandler:
    def __init__(self, next_handler: Handler) -> None:
        self.next = next_handler

    def handle(self, request: Request) -> None:
        if not request.has_valid_token:
            raise RequestRejected("Unauthorized")
        self.next.handle(request)


class RateLimitHandler:
    def __init__(self, next_handler: Handler) -> None:
        self.next = next_handler

    def handle(self, request: Request) -> None:
        if request.is_rate_limited:
            raise RequestRejected("Too many requests")
        self.next.handle(request)


class CreateOrderHandler:
    def handle(self, request: Request) -> None:
        if request.item_count == 0:
            raise RequestRejected("Empty order")
        print("Order created")


chain = AuthenticationHandler(RateLimitHandler(CreateOrderHandler()))
try:
    chain.handle(Request(True, False, 2))
except RequestRejected as error:
    print(f"Request rejected: {error}")
```

## Why This Is Better

- senders depend on one entry point instead of every processing step
- handlers are reusable, independently testable, and reorderable
- a rejection stops processing and propagates an explicit failure to the caller
- new handlers can be inserted without modifying existing ones

## Relationship With SOLID

- **Single Responsibility Principle:** each handler performs one check or action.
- **Open/Closed Principle:** the chain can be extended with new handlers.
- **Dependency Inversion Principle:** handlers depend on the shared handler contract.

## When To Use It

- HTTP middleware or request pipelines
- validation, authorization, approval, or escalation workflows
- events should be offered to several potential handlers
- processing order varies by configuration

## When Not To Use It

- exactly one handler is known and a direct call is clearer
- every step must always run; an explicit pipeline may communicate that better
- the chain makes it difficult to guarantee that a request is handled

## Notes

- Decide whether one handler stops the chain or every handler may process the request.
- Use exceptions or errors when rejection follows the application's error-handling flow. For expected business outcomes, a result type such as `handled` or `rejected(reason)` may be clearer.
- Provide an explicit terminal handler or result when an unhandled request would be an error.
- Ordering is part of behavior: authentication should usually precede work that assumes an identity.
- Middleware often implements this pattern with a `next` function instead of handler objects.
