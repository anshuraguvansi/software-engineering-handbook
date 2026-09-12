# Command Pattern

The Command Pattern is a behavioral design pattern. It turns a request into an object that contains everything needed to execute that request.

## Intent

Use Command when:

- request senders should be separated from request handlers
- operations need to be queued, logged, retried, or scheduled
- an operation needs undo support
- the same invoker should trigger different actions

## Problem

Suppose a toolbar directly edits a document. Each button must know which receiver method to call, and adding undo or history spreads state-management logic across the UI.

### Swift
```swift
final class Toolbar {
    let document: Document
    func tapSave() { document.save() }
    func tapClear() { document.replaceText(with: "") }
}
```

### Go
```go
type Toolbar struct { document *Document }
func (t Toolbar) TapSave() { t.document.Save() }
func (t Toolbar) TapClear() { t.document.ReplaceText("") }
```

### TypeScript
```typescript
class Toolbar {
  constructor(private readonly document: TextDocument) {}
  tapSave(): void { this.document.save() }
  tapClear(): void { this.document.replaceText("") }
}
```

### Python
```python
class Toolbar:
    def __init__(self, document: Document) -> None:
        self.document = document

    def tap_clear(self) -> None:
        self.document.replace_text("")
```

This design has a few problems:

- UI controls depend directly on receiver operations
- history and undo logic must be implemented separately for every control
- requests cannot easily be queued, serialized, or replayed
- changing an action requires modifying its invoker

## Solution

Represent each action as a command with `execute` and, when appropriate, `undo`. An invoker runs commands and stores successfully executed commands in a history.

### Swift
```swift
protocol Command {
    func execute()
    func undo()
}

final class Document {
    var text: String
    init(text: String) { self.text = text }
}

final class ReplaceTextCommand: Command {
    private let document: Document
    private let replacement: String
    private var previous = ""

    init(document: Document, replacement: String) {
        self.document = document
        self.replacement = replacement
    }

    func execute() { previous = document.text; document.text = replacement }
    func undo() { document.text = previous }
}

final class CommandHistory {
    private var commands: [any Command] = []

    func run(_ command: any Command) { command.execute(); commands.append(command) }
    func undoLast() { commands.popLast()?.undo() }
}

let document = Document(text: "Draft")
let history = CommandHistory()
history.run(ReplaceTextCommand(document: document, replacement: "Published"))
history.undoLast()
```

### Go
```go
package main

type Command interface { Execute(); Undo() }

type Document struct { Text string }

type ReplaceTextCommand struct {
	document *Document
	replacement string
	previous string
}

func (c *ReplaceTextCommand) Execute() {
	c.previous = c.document.Text
	c.document.Text = c.replacement
}
func (c *ReplaceTextCommand) Undo() { c.document.Text = c.previous }

type CommandHistory struct { commands []Command }

func (h *CommandHistory) Run(command Command) {
	command.Execute()
	h.commands = append(h.commands, command)
}
func (h *CommandHistory) UndoLast() {
	if len(h.commands) == 0 { return }
	last := len(h.commands) - 1
	h.commands[last].Undo()
	h.commands = h.commands[:last]
}

func main() {
	document := &Document{Text: "Draft"}
	history := &CommandHistory{}
	history.Run(&ReplaceTextCommand{document: document, replacement: "Published"})
	history.UndoLast()
}
```

### TypeScript
```typescript
interface Command {
  execute(): void
  undo(): void
}

class TextDocument {
  constructor(public text: string) {}
}

class ReplaceTextCommand implements Command {
  private previous = ""

  constructor(
    private readonly document: TextDocument,
    private readonly replacement: string,
  ) {}

  execute(): void {
    this.previous = this.document.text
    this.document.text = this.replacement
  }

  undo(): void { this.document.text = this.previous }
}

class CommandHistory {
  private readonly commands: Command[] = []

  run(command: Command): void {
    command.execute()
    this.commands.push(command)
  }

  undoLast(): void { this.commands.pop()?.undo() }
}

const document = new TextDocument("Draft")
const history = new CommandHistory()
history.run(new ReplaceTextCommand(document, "Published"))
history.undoLast()
```

### Python
```python
from typing import Protocol


class Command(Protocol):
    def execute(self) -> None:
        pass

    def undo(self) -> None:
        pass


class Document:
    def __init__(self, text: str) -> None:
        self.text = text


class ReplaceTextCommand:
    def __init__(self, document: Document, replacement: str) -> None:
        self.document = document
        self.replacement = replacement
        self.previous = ""

    def execute(self) -> None:
        self.previous = self.document.text
        self.document.text = self.replacement

    def undo(self) -> None:
        self.document.text = self.previous


class CommandHistory:
    def __init__(self) -> None:
        self.commands: list[Command] = []

    def run(self, command: Command) -> None:
        command.execute()
        self.commands.append(command)

    def undo_last(self) -> None:
        if self.commands:
            self.commands.pop().undo()


document = Document("Draft")
history = CommandHistory()
history.run(ReplaceTextCommand(document, "Published"))
history.undo_last()
```

## Why This Is Better

- invokers depend on one command interface instead of concrete receivers
- request parameters and undo state travel with the operation
- commands can be stored, queued, composed, or replayed
- new actions do not require changes to existing invokers

## Relationship With SOLID

- **Single Responsibility Principle:** invokers trigger requests, commands coordinate them, and receivers perform domain work.
- **Open/Closed Principle:** new commands can be introduced without modifying the invoker.
- **Dependency Inversion Principle:** invokers depend on the command abstraction.

## When To Use It

- editors need undo and redo
- jobs must be queued, scheduled, retried, or audited
- UI elements should be configured with actions
- transactions or macros combine several operations

## When Not To Use It

- a direct function call is simple and no request lifecycle is needed
- commands would contain no meaningful data or behavior
- reliable distributed processing is needed but commands cannot be durably serialized

## Notes

- Record a command in history only after successful execution.
- Undo should reverse the command's own effect, not restore unrelated global state.
- A command object is not automatically safe to retry; design idempotency explicitly.
- Command represents an action. Strategy represents one of several interchangeable algorithms used by a context.
