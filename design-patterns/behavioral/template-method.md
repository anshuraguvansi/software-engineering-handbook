# Template Method Pattern

The Template Method Pattern is a behavioral design pattern. It defines the fixed skeleton of an algorithm while allowing subclasses or conforming types to customize selected steps.

## Intent

Use Template Method when:

- several workflows share the same sequence
- only a few steps vary between implementations
- ordering and invariant steps must remain controlled
- duplicated workflow code should be centralized

## Problem

Suppose CSV and JSON imports both read, validate, save, and report records. Implementing the full workflow in each importer duplicates sequencing and cleanup rules.

### Swift
```swift
func importCSV() { readCSV(); validateCSV(); saveCSV(); reportCSV() }
func importJSON() { readJSON(); validateJSON(); saveJSON(); reportJSON() }
```

### Go
```go
func importCSV() { readCSV(); validateCSV(); saveCSV(); reportCSV() }
func importJSON() { readJSON(); validateJSON(); saveJSON(); reportJSON() }
```

### TypeScript
```typescript
function importCsv(): void { readCsv(); validateCsv(); saveCsv(); reportCsv() }
function importJson(): void { readJson(); validateJson(); saveJson(); reportJson() }
```

### Python
```python
def import_csv() -> None:
    read_csv(); validate_csv(); save_csv(); report_csv()

def import_json() -> None:
    read_json(); validate_json(); save_json(); report_json()
```

This design has a few problems:

- the shared algorithm is duplicated
- implementations can accidentally perform steps in different orders
- adding common setup, cleanup, or reporting requires several edits
- it is unclear which steps are fixed and which are customizable

## Solution

Put the algorithm skeleton in one template method. Keep invariant steps in the base abstraction and expose primitive operations or hooks for the steps that vary.

### Swift
```swift
protocol DataImporter {
    func read() -> [String]
    func validate(_ records: [String]) -> [String]
    func save(_ records: [String])
    func didImport(count: Int)
}

extension DataImporter {
    func run() {
        let validRecords = validate(read())
        save(validRecords)
        didImport(count: validRecords.count)
    }

    func save(_ records: [String]) { print("Saved \(records.count) records") }
    func didImport(count: Int) {}
}

struct CSVImporter: DataImporter {
    func read() -> [String] { ["csv-record"] }
    func validate(_ records: [String]) -> [String] { records }
}

struct JSONImporter: DataImporter {
    func read() -> [String] { ["json-record"] }
    func validate(_ records: [String]) -> [String] { records.filter { !$0.isEmpty } }
    func didImport(count: Int) { print("JSON import complete: \(count)") }
}

CSVImporter().run()
JSONImporter().run()
```

### Go
```go
package main

type ImportSteps interface {
	Read() []string
	Validate([]string) []string
	DidImport(int)
}

func RunImport(steps ImportSteps) {
	records := steps.Validate(steps.Read())
	println("Saved", len(records), "records")
	steps.DidImport(len(records))
}

type CSVImporter struct{}
func (CSVImporter) Read() []string { return []string{"csv-record"} }
func (CSVImporter) Validate(records []string) []string { return records }
func (CSVImporter) DidImport(int) {}

type JSONImporter struct{}
func (JSONImporter) Read() []string { return []string{"json-record"} }
func (JSONImporter) Validate(records []string) []string { return records }
func (JSONImporter) DidImport(count int) { println("JSON import complete:", count) }

func main() {
	RunImport(CSVImporter{})
	RunImport(JSONImporter{})
}
```

Go favors composition over inheritance, so the template is a function that accepts the customizable steps through an interface.

### TypeScript
```typescript
abstract class DataImporter {
  run(): void {
    const validRecords = this.validate(this.read())
    this.save(validRecords)
    this.didImport(validRecords.length)
  }

  protected abstract read(): string[]
  protected abstract validate(records: string[]): string[]

  protected save(records: string[]): void {
    console.log(`Saved ${records.length} records`)
  }

  protected didImport(_count: number): void {}
}

class CsvImporter extends DataImporter {
  protected read(): string[] { return ["csv-record"] }
  protected validate(records: string[]): string[] { return records }
}

class JsonImporter extends DataImporter {
  protected read(): string[] { return ["json-record"] }
  protected validate(records: string[]): string[] {
    return records.filter((record) => record.length > 0)
  }
  protected didImport(count: number): void {
    console.log(`JSON import complete: ${count}`)
  }
}

new CsvImporter().run()
new JsonImporter().run()
```

### Python
```python
from abc import ABC, abstractmethod


class DataImporter(ABC):
    def run(self) -> None:
        valid_records = self.validate(self.read())
        self.save(valid_records)
        self.did_import(len(valid_records))

    @abstractmethod
    def read(self) -> list[str]:
        pass

    @abstractmethod
    def validate(self, records: list[str]) -> list[str]:
        pass

    def save(self, records: list[str]) -> None:
        print(f"Saved {len(records)} records")

    def did_import(self, count: int) -> None:
        pass


class CsvImporter(DataImporter):
    def read(self) -> list[str]:
        return ["csv-record"]

    def validate(self, records: list[str]) -> list[str]:
        return records


class JsonImporter(DataImporter):
    def read(self) -> list[str]:
        return ["json-record"]

    def validate(self, records: list[str]) -> list[str]:
        return [record for record in records if record]

    def did_import(self, count: int) -> None:
        print(f"JSON import complete: {count}")


CsvImporter().run()
JsonImporter().run()
```

## Why This Is Better

- the algorithm order is defined once
- variable steps are explicit extension points
- common behavior and fixes apply to every implementation
- subclasses focus only on meaningful differences

## Relationship With SOLID

- **Single Responsibility Principle:** the base abstraction owns sequencing while implementations own format-specific steps.
- **Open/Closed Principle:** new workflow variants can reuse the template without modifying it.
- **Liskov Substitution Principle:** implementations must preserve the template's expectations for each step.

## When To Use It

- importers, exporters, test fixtures, or build processes share a lifecycle
- the algorithm order is an invariant
- subclasses differ in a few well-defined operations
- hooks are needed before or after fixed steps

## When Not To Use It

- workflows differ substantially in structure or ordering
- runtime composition is more important than inheritance-based reuse
- subclasses would need to override most of the template

## Notes

- Keep the template method non-overridable where the language permits so subclasses cannot break the sequence.
- Required primitive operations should be abstract; optional hooks can have no-op defaults.
- Too many hooks create implicit coupling between the base class and subclasses.
- Strategy composes an entire interchangeable algorithm. Template Method reuses a fixed algorithm while varying selected steps.
