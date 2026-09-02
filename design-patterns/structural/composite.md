# Composite Pattern

The Composite Pattern is a structural design pattern. It lets clients treat individual objects and groups of objects through the same interface.

## Intent

Use a composite when:

- objects naturally form a tree or part-whole hierarchy
- clients should perform the same operation on a single item or a group
- callers should not need type checks to traverse nested structures
- new leaf and container types should fit into the same hierarchy

## Problem

Suppose an application calculates the size of files and folders. A file has one size, while a folder contains files and nested folders. Without a common abstraction, client code must know each concrete type and manually recurse through folders.

As the hierarchy grows, every new operation such as displaying, deleting, or moving nodes must repeat the same type checks and traversal logic.

### Swift
```swift
final class FileItem {
    let name: String
    let bytes: Int

    init(name: String, bytes: Int) {
        self.name = name
        self.bytes = bytes
    }
}

final class Folder {
    let name: String
    var children: [Any] = []

    init(name: String) {
        self.name = name
    }
}

func size(of item: Any) -> Int {
    if let file = item as? FileItem {
        return file.bytes
    }

    if let folder = item as? Folder {
        return folder.children.reduce(0) { total, child in
            total + size(of: child)
        }
    }

    return 0
}
```

### Go
```go
package main

type FileItem struct {
	Name  string
	Bytes int
}

type Folder struct {
	Name     string
	Children []any
}

func Size(item any) int {
	switch value := item.(type) {
	case FileItem:
		return value.Bytes
	case Folder:
		total := 0
		for _, child := range value.Children {
			total += Size(child)
		}
		return total
	default:
		return 0
	}
}
```

### TypeScript
```typescript
class FileItem {
  constructor(
    public name: string,
    public bytes: number,
  ) {}
}

class Folder {
  children: Array<FileItem | Folder> = []

  constructor(public name: string) {}
}

function size(item: FileItem | Folder): number {
  if (item instanceof FileItem) return item.bytes

  return item.children.reduce((total, child) => total + size(child), 0)
}
```

### Python
```python
class FileItem:
    def __init__(self, name: str, bytes: int) -> None:
        self.name = name
        self.bytes = bytes


class Folder:
    def __init__(self, name: str) -> None:
        self.name = name
        self.children: list[FileItem | Folder] = []


def size(item: FileItem | Folder) -> int:
    if isinstance(item, FileItem):
        return item.bytes

    return sum(size(child) for child in item.children)
```

This design has a few problems:

- callers must know the concrete type of every node
- traversal and type-checking logic is repeated for every operation
- adding a new node type requires changing all client-side traversal code
- nested structures become difficult to work with consistently

## Solution

Define one component interface for all nodes. A leaf implements the operation directly. A composite stores child components and implements the operation by delegating to its children.

Clients call the same operation on a file, a folder, or an entire directory tree.

### Swift
```swift
protocol FileSystemNode {
    var name: String { get }
    func size() -> Int
}

final class FileItem: FileSystemNode {
    let name: String
    private let bytes: Int

    init(name: String, bytes: Int) {
        self.name = name
        self.bytes = bytes
    }

    func size() -> Int {
        bytes
    }
}

final class Folder: FileSystemNode {
    let name: String
    private var children: [any FileSystemNode] = []

    init(name: String) {
        self.name = name
    }

    func add(_ child: any FileSystemNode) {
        children.append(child)
    }

    func size() -> Int {
        children.reduce(0) { total, child in
            total + child.size()
        }
    }
}

let documents = Folder(name: "documents")
documents.add(FileItem(name: "resume.pdf", bytes: 120_000))

let invoices = Folder(name: "invoices")
invoices.add(FileItem(name: "january.pdf", bytes: 80_000))
documents.add(invoices)

let node: any FileSystemNode = documents
print(node.size()) // 200000
```

### Go
```go
package main

type FileSystemNode interface {
	Size() int
}

type FileItem struct {
	name  string
	bytes int
}

func (f *FileItem) Size() int {
	return f.bytes
}

type Folder struct {
	name     string
	children []FileSystemNode
}

func (f *Folder) Add(child FileSystemNode) {
	f.children = append(f.children, child)
}

func (f *Folder) Size() int {
	total := 0
	for _, child := range f.children {
		total += child.Size()
	}
	return total
}

func main() {
	documents := &Folder{name: "documents"}
	documents.Add(&FileItem{name: "resume.pdf", bytes: 120_000})

	invoices := &Folder{name: "invoices"}
	invoices.Add(&FileItem{name: "january.pdf", bytes: 80_000})
	documents.Add(invoices)

	var node FileSystemNode = documents
	println(node.Size()) // 200000
}
```

### TypeScript
```typescript
interface FileSystemNode {
  size(): number
}

class FileItem implements FileSystemNode {
  constructor(
    readonly name: string,
    private readonly bytes: number,
  ) {}

  size(): number {
    return this.bytes
  }
}

class Folder implements FileSystemNode {
  private children: FileSystemNode[] = []

  constructor(readonly name: string) {}

  add(child: FileSystemNode): void {
    this.children.push(child)
  }

  size(): number {
    return this.children.reduce((total, child) => total + child.size(), 0)
  }
}

const documents = new Folder("documents")
documents.add(new FileItem("resume.pdf", 120_000))

const invoices = new Folder("invoices")
invoices.add(new FileItem("january.pdf", 80_000))
documents.add(invoices)

const node: FileSystemNode = documents
console.log(node.size()) // 200000
```

### Python
```python
from typing import Protocol


class FileSystemNode(Protocol):
    def size(self) -> int:
        pass


class FileItem:
    def __init__(self, name: str, bytes: int) -> None:
        self.name = name
        self._bytes = bytes

    def size(self) -> int:
        return self._bytes


class Folder:
    def __init__(self, name: str) -> None:
        self.name = name
        self._children: list[FileSystemNode] = []

    def add(self, child: FileSystemNode) -> None:
        self._children.append(child)

    def size(self) -> int:
        return sum(child.size() for child in self._children)


documents = Folder("documents")
documents.add(FileItem("resume.pdf", 120_000))

invoices = Folder("invoices")
invoices.add(FileItem("january.pdf", 80_000))
documents.add(invoices)

node: FileSystemNode = documents
print(node.size())  # 200000
```

## Why This Is Better

- clients use one interface for leaves and nested groups
- recursive behavior lives in the composite instead of every caller
- new node types can participate without changing traversal code
- whole hierarchies can be processed with the same operation used for one item
- tree structure is represented directly in the object model

## Relationship With SOLID

- **Open/Closed Principle:** add new leaf or composite types that implement `FileSystemNode` without changing clients.
- **Liskov Substitution Principle:** a folder can be used anywhere a file-system node is expected.
- **Dependency Inversion Principle:** clients depend on the common component interface, not on `FileItem` or `Folder`.
- **Single Responsibility Principle:** leaves manage their own values; composites manage child aggregation.

## When To Use It

- data naturally forms trees, such as file systems, menus, organization charts, UI components, or product bundles
- clients should apply an operation to one item or an entire hierarchy in the same way
- recursive traversal logic is appearing in several places
- new leaf and group types need to be added independently

## When Not To Use It

- the structure is flat and grouping does not provide meaningful behavior
- leaves and containers do not share a useful common operation
- forcing every node to support child-management methods would make the interface misleading

## Notes

- A transparent composite puts child-management methods such as `add` on the shared interface. This treats leaves and composites uniformly but lets callers attempt invalid operations on leaves.
- A safe composite keeps child-management methods only on container types, as in this example. Clients use the shared interface only for common operations such as `size`.
- Prevent cycles when the domain requires a true tree. Adding a folder as its own descendant causes infinite recursion.
- Composite pairs naturally with Visitor when many unrelated operations must be applied across the same hierarchy.
