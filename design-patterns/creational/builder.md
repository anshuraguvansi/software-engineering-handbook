# Builder Pattern

The Builder Pattern is a creational design pattern. It constructs a complex object step by step, especially when the object has many optional settings.

## Intent

Use a builder when:

- an object needs many constructor parameters
- several parameters are optional
- a long list of values is hard to read or easy to mix up
- the object must be valid before it is created
- you want different representations built through the same process

## Problem

Suppose we are building a reporting feature. A report always needs a title and data source, but its date range, grouping, format, charts, filters, and recipient are optional.

Passing every possible value to a constructor produces a fragile and difficult-to-read API. As options grow, callers must remember parameter order and supply placeholder values for settings they do not need.

### Swift
```swift
struct Report {
    let title: String
    let dataSource: String
    let startDate: Date?
    let endDate: Date?
    let groupBy: String?
    let includeCharts: Bool
    let format: String
    let recipientEmail: String?
}

let report = Report(
    title: "Monthly sales",
    dataSource: "orders",
    startDate: nil,
    endDate: nil,
    groupBy: "region",
    includeCharts: true,
    format: "pdf",
    recipientEmail: nil
)
```

### Go
```go
package main

import "time"

type Report struct {
	Title          string
	DataSource     string
	StartDate      *time.Time
	EndDate        *time.Time
	GroupBy        string
	IncludeCharts  bool
	Format         string
	RecipientEmail string
}

report := Report{
	Title:         "Monthly sales",
	DataSource:    "orders",
	GroupBy:       "region",
	IncludeCharts: true,
	Format:        "pdf",
}
```

### TypeScript
```typescript
class Report {
  constructor(
    public title: string,
    public dataSource: string,
    public startDate?: Date,
    public endDate?: Date,
    public groupBy?: string,
    public includeCharts = false,
    public format = "csv",
    public recipientEmail?: string,
  ) {}
}

const report = new Report(
  "Monthly sales",
  "orders",
  undefined,
  undefined,
  "region",
  true,
  "pdf",
)
```

### Python
```python
from datetime import date


class Report:
    def __init__(
        self,
        title: str,
        data_source: str,
        start_date: date | None = None,
        end_date: date | None = None,
        group_by: str | None = None,
        include_charts: bool = False,
        report_format: str = "csv",
        recipient_email: str | None = None,
    ) -> None:
        self.title = title
        self.data_source = data_source
        self.start_date = start_date
        self.end_date = end_date
        self.group_by = group_by
        self.include_charts = include_charts
        self.report_format = report_format
        self.recipient_email = recipient_email


report = Report(
    "Monthly sales",
    "orders",
    group_by="region",
    include_charts=True,
    report_format="pdf",
)
```

This design has a few problems:

- constructor calls become long and difficult to understand
- optional values can be supplied in the wrong position
- adding a new option changes the constructor API
- validation rules are scattered between callers and the object
- partially configured objects can be created accidentally

## Solution

Introduce a builder that collects configuration through small, named methods. The builder keeps the construction process separate from the final object and creates the report only after required values and validation rules are satisfied.

The resulting call reads like a description of the object being built.

### Swift
```swift
struct Report {
    let title: String
    let dataSource: String
    let groupBy: String?
    let includeCharts: Bool
    let format: String
    let recipientEmail: String?
}

final class ReportBuilder {
    private let title: String
    private let dataSource: String
    private var groupBy: String?
    private var includeCharts = false
    private var format = "csv"
    private var recipientEmail: String?

    init(title: String, dataSource: String) {
        self.title = title
        self.dataSource = dataSource
    }

    func groupedBy(_ value: String) -> ReportBuilder {
        groupBy = value
        return self
    }

    func withCharts() -> ReportBuilder {
        includeCharts = true
        return self
    }

    func asFormat(_ value: String) -> ReportBuilder {
        format = value
        return self
    }

    func sentTo(_ email: String) -> ReportBuilder {
        recipientEmail = email
        return self
    }

    func build() -> Report {
        Report(
            title: title,
            dataSource: dataSource,
            groupBy: groupBy,
            includeCharts: includeCharts,
            format: format,
            recipientEmail: recipientEmail
        )
    }
}

let report = ReportBuilder(title: "Monthly sales", dataSource: "orders")
    .groupedBy("region")
    .withCharts()
    .asFormat("pdf")
    .build()
```

### Go
```go
package main

type Report struct {
	Title          string
	DataSource     string
	GroupBy        string
	IncludeCharts  bool
	Format         string
	RecipientEmail string
}

type ReportBuilder struct {
	report Report
}

func NewReportBuilder(title string, dataSource string) *ReportBuilder {
	return &ReportBuilder{
		report: Report{
			Title:      title,
			DataSource: dataSource,
			Format:     "csv",
		},
	}
}

func (b *ReportBuilder) GroupedBy(value string) *ReportBuilder {
	b.report.GroupBy = value
	return b
}

func (b *ReportBuilder) WithCharts() *ReportBuilder {
	b.report.IncludeCharts = true
	return b
}

func (b *ReportBuilder) AsFormat(value string) *ReportBuilder {
	b.report.Format = value
	return b
}

func (b *ReportBuilder) SentTo(email string) *ReportBuilder {
	b.report.RecipientEmail = email
	return b
}

func (b *ReportBuilder) Build() Report {
	return b.report
}

report := NewReportBuilder("Monthly sales", "orders").
	GroupedBy("region").
	WithCharts().
	AsFormat("pdf").
	Build()
```

### TypeScript
```typescript
class Report {
  constructor(
    public readonly title: string,
    public readonly dataSource: string,
    public readonly groupBy?: string,
    public readonly includeCharts = false,
    public readonly format = "csv",
    public readonly recipientEmail?: string,
  ) {}
}

class ReportBuilder {
  private groupBy?: string
  private includeCharts = false
  private format = "csv"
  private recipientEmail?: string

  constructor(
    private readonly title: string,
    private readonly dataSource: string,
  ) {}

  groupedBy(value: string): this {
    this.groupBy = value
    return this
  }

  withCharts(): this {
    this.includeCharts = true
    return this
  }

  asFormat(value: string): this {
    this.format = value
    return this
  }

  sentTo(email: string): this {
    this.recipientEmail = email
    return this
  }

  build(): Report {
    return new Report(
      this.title,
      this.dataSource,
      this.groupBy,
      this.includeCharts,
      this.format,
      this.recipientEmail,
    )
  }
}

const report = new ReportBuilder("Monthly sales", "orders")
  .groupedBy("region")
  .withCharts()
  .asFormat("pdf")
  .build()
```

### Python
```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Report:
    title: str
    data_source: str
    group_by: str | None
    include_charts: bool
    report_format: str
    recipient_email: str | None


class ReportBuilder:
    def __init__(self, title: str, data_source: str) -> None:
        self._title = title
        self._data_source = data_source
        self._group_by: str | None = None
        self._include_charts = False
        self._report_format = "csv"
        self._recipient_email: str | None = None

    def grouped_by(self, value: str) -> "ReportBuilder":
        self._group_by = value
        return self

    def with_charts(self) -> "ReportBuilder":
        self._include_charts = True
        return self

    def as_format(self, value: str) -> "ReportBuilder":
        self._report_format = value
        return self

    def sent_to(self, email: str) -> "ReportBuilder":
        self._recipient_email = email
        return self

    def build(self) -> Report:
        return Report(
            title=self._title,
            data_source=self._data_source,
            group_by=self._group_by,
            include_charts=self._include_charts,
            report_format=self._report_format,
            recipient_email=self._recipient_email,
        )


report = (
    ReportBuilder("Monthly sales", "orders")
    .grouped_by("region")
    .with_charts()
    .as_format("pdf")
    .build()
)
```

## Why This Is Better

- named builder methods make optional configuration clear at the call site
- callers provide only the values they need
- default values are centralized in one place
- validation can happen before the final object is returned
- the final object can be immutable because configuration is handled by the builder

## Relationship With SOLID

- **Single Responsibility Principle:** the builder handles construction while the product represents the completed object.
- **Open/Closed Principle:** new optional configuration can usually be added as a builder method without changing existing calls.
- **Dependency Inversion Principle:** a director or service can depend on a builder abstraction when it must support different product representations.

## When To Use It

- objects have many optional fields or sensible defaults
- constructor parameters are difficult to read or remember
- the construction process has multiple steps or validation rules
- you need immutable objects that are configured before creation
- the same construction steps should produce different representations

## When Not To Use It

- the object has only a few required values
- a simple constructor or language-native named arguments are already clear
- the builder would only duplicate a small, stable constructor

## Notes

- A builder is especially useful when optional parameters keep growing over time.
- Fluent methods should return the builder so calls can be chained clearly.
- Builders are often combined with immutable value objects: mutate the builder during setup, then return an immutable product from `build()`.
- In languages with named parameters or option structs, those features can be a lighter alternative for simple cases.
