# Proxy Pattern

The Proxy Pattern is a structural design pattern. It provides a stand-in object that has the same interface as another object while controlling access to it.

## Intent

Use a proxy when:

- access to an object must be authorized, logged, cached, or rate-limited
- an expensive object should be created only when it is first needed
- a remote resource should look like a local object to callers
- the real object needs protection from direct use

## Problem

Suppose an application loads private documents from a remote document service. The service is expensive to create and every request must verify that the current user can read the document.

If controllers call the remote service directly, each caller must implement its own authorization check and creates the remote client even when access is denied.

### Swift
```swift
final class RemoteDocumentStore {
    init() {
        print("Connecting to remote document service")
    }

    func load(documentId: String) -> String {
        "Contents of \(documentId)"
    }
}

final class DocumentController {
    private let store = RemoteDocumentStore()

    func showDocument(documentId: String, userId: String) {
        // Each controller must remember to authorize the user.
        let isAllowed = userId == "admin"
        guard isAllowed else { return }

        print(store.load(documentId: documentId))
    }
}
```

### Go
```go
package main

type RemoteDocumentStore struct{}

func NewRemoteDocumentStore() *RemoteDocumentStore {
	println("Connecting to remote document service")
	return &RemoteDocumentStore{}
}

func (s *RemoteDocumentStore) Load(documentID string) string {
	return "Contents of " + documentID
}

type DocumentController struct {
	store *RemoteDocumentStore
}

func (c *DocumentController) ShowDocument(documentID string, userID string) {
	if userID != "admin" {
		return
	}

	println(c.store.Load(documentID))
}
```

### TypeScript
```typescript
class RemoteDocumentStore {
  constructor() {
    console.log("Connecting to remote document service")
  }

  load(documentId: string): string {
    return `Contents of ${documentId}`
  }
}

class DocumentController {
  private readonly store = new RemoteDocumentStore()

  showDocument(documentId: string, userId: string): void {
    if (userId !== "admin") return

    console.log(this.store.load(documentId))
  }
}
```

### Python
```python
class RemoteDocumentStore:
    def __init__(self) -> None:
        print("Connecting to remote document service")

    def load(self, document_id: str) -> str:
        return f"Contents of {document_id}"


class DocumentController:
    def __init__(self) -> None:
        self.store = RemoteDocumentStore()

    def show_document(self, document_id: str, user_id: str) -> None:
        if user_id != "admin":
            return

        print(self.store.load(document_id))
```

This design has a few problems:

- authorization is repeated and can be forgotten by a caller
- the remote service is created even for requests that should be rejected
- changing access rules requires updating every caller
- clients depend directly on remote-service details

## Solution

Define a common interface for the real service and the proxy. The proxy performs the access check, creates the real service only after a valid request, and delegates the work.

The caller uses `DocumentStore` and does not need to know whether it is talking to the proxy or the remote implementation.

### Swift
```swift
protocol DocumentStore {
    func load(documentId: String) -> String?
}

final class RemoteDocumentStore: DocumentStore {
    init() {
        print("Connecting to remote document service")
    }

    func load(documentId: String) -> String? {
        "Contents of \(documentId)"
    }
}

final class ProtectedDocumentStoreProxy: DocumentStore {
    private let userId: String
    private lazy var remoteStore = RemoteDocumentStore()

    init(userId: String) {
        self.userId = userId
    }

    func load(documentId: String) -> String? {
        guard userId == "admin" else {
            print("Access denied")
            return nil
        }

        return remoteStore.load(documentId: documentId)
    }
}

let store: any DocumentStore = ProtectedDocumentStoreProxy(userId: "admin")
print(store.load(documentId: "contract-2026") ?? "Document not found")
```

The `lazy` property makes this a virtual proxy: the real store is not created until an authorized request needs it.

### Go
```go
package main

type DocumentStore interface {
	Load(documentID string) (string, bool)
}

type RemoteDocumentStore struct{}

func NewRemoteDocumentStore() *RemoteDocumentStore {
	println("Connecting to remote document service")
	return &RemoteDocumentStore{}
}

func (s *RemoteDocumentStore) Load(documentID string) (string, bool) {
	return "Contents of " + documentID, true
}

type ProtectedDocumentStoreProxy struct {
	userID      string
	remoteStore *RemoteDocumentStore
}

func (p *ProtectedDocumentStoreProxy) Load(documentID string) (string, bool) {
	if p.userID != "admin" {
		println("Access denied")
		return "", false
	}

	if p.remoteStore == nil {
		p.remoteStore = NewRemoteDocumentStore()
	}

	return p.remoteStore.Load(documentID)
}

func main() {
	var store DocumentStore = &ProtectedDocumentStoreProxy{userID: "admin"}
	document, found := store.Load("contract-2026")
	if found {
		println(document)
	}
}
```

### TypeScript
```typescript
interface DocumentStore {
  load(documentId: string): string | undefined
}

class RemoteDocumentStore implements DocumentStore {
  constructor() {
    console.log("Connecting to remote document service")
  }

  load(documentId: string): string {
    return `Contents of ${documentId}`
  }
}

class ProtectedDocumentStoreProxy implements DocumentStore {
  private remoteStore?: RemoteDocumentStore

  constructor(private readonly userId: string) {}

  load(documentId: string): string | undefined {
    if (this.userId !== "admin") {
      console.log("Access denied")
      return undefined
    }

    this.remoteStore ??= new RemoteDocumentStore()
    return this.remoteStore.load(documentId)
  }
}

const store: DocumentStore = new ProtectedDocumentStoreProxy("admin")
console.log(store.load("contract-2026"))
```

### Python
```python
from typing import Protocol


class DocumentStore(Protocol):
    def load(self, document_id: str) -> str | None:
        pass


class RemoteDocumentStore:
    def __init__(self) -> None:
        print("Connecting to remote document service")

    def load(self, document_id: str) -> str:
        return f"Contents of {document_id}"


class ProtectedDocumentStoreProxy:
    def __init__(self, user_id: str) -> None:
        self.user_id = user_id
        self.remote_store: RemoteDocumentStore | None = None

    def load(self, document_id: str) -> str | None:
        if self.user_id != "admin":
            print("Access denied")
            return None

        if self.remote_store is None:
            self.remote_store = RemoteDocumentStore()

        return self.remote_store.load(document_id)


store: DocumentStore = ProtectedDocumentStoreProxy("admin")
print(store.load("contract-2026"))
```

## Why This Is Better

- the proxy centralizes access rules around the protected object
- expensive initialization can be delayed until it is actually needed
- callers use the same interface for local, remote, protected, or cached implementations
- the real service remains focused on its primary behavior
- policy changes happen in one place instead of every caller

## Relationship With SOLID

- **Single Responsibility Principle:** the real service loads documents; the proxy controls access and lifecycle concerns.
- **Open/Closed Principle:** add caching, logging, rate limiting, or new access policies through another proxy without changing the real service.
- **Liskov Substitution Principle:** the proxy can be used anywhere the real `DocumentStore` is expected.
- **Dependency Inversion Principle:** clients depend on `DocumentStore`, not the remote implementation.

## When To Use It

- authorization, auditing, caching, rate limiting, or lazy initialization should surround an object
- a remote, database, file, or heavyweight resource should look local to clients
- a resource must be protected from direct access
- you need to enforce a policy consistently before delegating to the real object

## When Not To Use It

- direct access is already simple, safe, and inexpensive
- callers require the real object's concrete API rather than a stable shared interface
- the extra indirection would hide critical failures or performance costs

## Notes

- A protection proxy checks permissions before delegation. A virtual proxy delays expensive object creation. A remote proxy represents an object in another process or service.
- Proxy and Decorator both wrap an object with the same interface. A proxy primarily controls access or lifecycle; a decorator primarily adds optional behavior through composition.
- Keep proxy behavior transparent when possible: callers should receive the same domain-level results and errors they would expect from the real object.
- Do not use a proxy to hide security decisions. Make its authorization policy explicit and test it thoroughly.
