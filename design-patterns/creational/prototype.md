# Prototype Pattern

The Prototype Pattern is a creational design pattern. It creates new objects by cloning an existing, preconfigured object instead of constructing one from scratch.

## Intent

Use a prototype when:

- creating an object requires expensive setup
- several objects share a common starting configuration
- the concrete type is only known at runtime
- callers should create copies without depending on concrete classes

## Problem

Suppose a dashboard supports reusable widgets. A sales widget needs a title, data source, filters, refresh interval, and visualization settings. Creating a similar widget for another region repeats the same setup every time.

As widgets gain more configuration, recreating them manually becomes verbose and risks missing or inconsistently applying settings.

### Swift
```swift
struct DashboardWidget {
    let title: String
    let dataSource: String
    let filters: [String: String]
    let refreshInterval: Int
    let chartType: String
}

let northAmericaSales = DashboardWidget(
    title: "Sales by region",
    dataSource: "orders",
    filters: ["region": "north-america", "status": "paid"],
    refreshInterval: 60,
    chartType: "bar"
)

let europeSales = DashboardWidget(
    title: "Sales by region",
    dataSource: "orders",
    filters: ["region": "europe", "status": "paid"],
    refreshInterval: 60,
    chartType: "bar"
)
```

### Go
```go
package main

type DashboardWidget struct {
	Title           string
	DataSource      string
	Filters         map[string]string
	RefreshInterval int
	ChartType       string
}

func main() {
	northAmericaSales := DashboardWidget{
		Title:           "Sales by region",
		DataSource:      "orders",
		Filters:         map[string]string{"region": "north-america", "status": "paid"},
		RefreshInterval: 60,
		ChartType:       "bar",
	}

	europeSales := DashboardWidget{
		Title:           "Sales by region",
		DataSource:      "orders",
		Filters:         map[string]string{"region": "europe", "status": "paid"},
		RefreshInterval: 60,
		ChartType:       "bar",
	}

	_, _ = northAmericaSales, europeSales
}
```

### TypeScript
```typescript
class DashboardWidget {
  constructor(
    public title: string,
    public dataSource: string,
    public filters: Map<string, string>,
    public refreshInterval: number,
    public chartType: string,
  ) {}
}

const northAmericaSales = new DashboardWidget(
  "Sales by region",
  "orders",
  new Map([["region", "north-america"], ["status", "paid"]]),
  60,
  "bar",
)

const europeSales = new DashboardWidget(
  "Sales by region",
  "orders",
  new Map([["region", "europe"], ["status", "paid"]]),
  60,
  "bar",
)
```

### Python
```python
class DashboardWidget:
    def __init__(
        self,
        title: str,
        data_source: str,
        filters: dict[str, str],
        refresh_interval: int,
        chart_type: str,
    ) -> None:
        self.title = title
        self.data_source = data_source
        self.filters = filters
        self.refresh_interval = refresh_interval
        self.chart_type = chart_type


north_america_sales = DashboardWidget(
    "Sales by region",
    "orders",
    {"region": "north-america", "status": "paid"},
    60,
    "bar",
)

europe_sales = DashboardWidget(
    "Sales by region",
    "orders",
    {"region": "europe", "status": "paid"},
    60,
    "bar",
)
```

This design has a few problems:

- repeated setup obscures the small differences between similar objects
- changes to default configuration must be updated in many places
- callers can forget a required setting or choose inconsistent defaults
- construction can be slow when it includes parsing, loading, or computation

## Solution

Give the object a `clone` operation. A caller starts with a configured prototype, creates a copy, and changes only the fields that differ.

The clone must copy mutable nested state when each copy needs to change it independently. This is called a deep copy. A shallow copy shares nested references and can cause one clone's changes to affect another.

### Swift
```swift
final class DashboardWidget {
    var title: String
    let dataSource: String
    var filters: [String: String]
    let refreshInterval: Int
    let chartType: String

    init(
        title: String,
        dataSource: String,
        filters: [String: String],
        refreshInterval: Int,
        chartType: String
    ) {
        self.title = title
        self.dataSource = dataSource
        self.filters = filters
        self.refreshInterval = refreshInterval
        self.chartType = chartType
    }

    func clone() -> DashboardWidget {
        DashboardWidget(
            title: title,
            dataSource: dataSource,
            filters: filters,
            refreshInterval: refreshInterval,
            chartType: chartType
        )
    }
}

let salesPrototype = DashboardWidget(
    title: "Sales by region",
    dataSource: "orders",
    filters: ["status": "paid"],
    refreshInterval: 60,
    chartType: "bar"
)

let europeSales = salesPrototype.clone()
europeSales.filters["region"] = "europe"
```

Swift dictionaries are value types, so the cloned widget receives an independent dictionary.

### Go
```go
package main

type DashboardWidget struct {
	Title           string
	DataSource      string
	Filters         map[string]string
	RefreshInterval int
	ChartType       string
}

func (w *DashboardWidget) Clone() *DashboardWidget {
	filters := make(map[string]string, len(w.Filters))
	for key, value := range w.Filters {
		filters[key] = value
	}

	return &DashboardWidget{
		Title:           w.Title,
		DataSource:      w.DataSource,
		Filters:         filters,
		RefreshInterval: w.RefreshInterval,
		ChartType:       w.ChartType,
	}
}

func main() {
	salesPrototype := &DashboardWidget{
		Title:           "Sales by region",
		DataSource:      "orders",
		Filters:         map[string]string{"status": "paid"},
		RefreshInterval: 60,
		ChartType:       "bar",
	}

	europeSales := salesPrototype.Clone()
	europeSales.Filters["region"] = "europe"
}
```

Maps are reference types in Go, so cloning the map is necessary to keep each widget's filters independent.

### TypeScript
```typescript
class DashboardWidget {
  constructor(
    public title: string,
    public dataSource: string,
    public filters: Map<string, string>,
    public refreshInterval: number,
    public chartType: string,
  ) {}

  clone(): DashboardWidget {
    return new DashboardWidget(
      this.title,
      this.dataSource,
      new Map(this.filters),
      this.refreshInterval,
      this.chartType,
    )
  }
}

const salesPrototype = new DashboardWidget(
  "Sales by region",
  "orders",
  new Map([["status", "paid"]]),
  60,
  "bar",
)

const europeSales = salesPrototype.clone()
europeSales.filters.set("region", "europe")
```

### Python
```python
from copy import deepcopy


class DashboardWidget:
    def __init__(
        self,
        title: str,
        data_source: str,
        filters: dict[str, str],
        refresh_interval: int,
        chart_type: str,
    ) -> None:
        self.title = title
        self.data_source = data_source
        self.filters = filters
        self.refresh_interval = refresh_interval
        self.chart_type = chart_type

    def clone(self) -> "DashboardWidget":
        return deepcopy(self)


sales_prototype = DashboardWidget(
    "Sales by region",
    "orders",
    {"status": "paid"},
    60,
    "bar",
)

europe_sales = sales_prototype.clone()
europe_sales.filters["region"] = "europe"
```

## Why This Is Better

- callers express only the differences from a known-good configuration
- shared defaults live in the prototype instead of being repeated
- cloning can avoid repeating expensive object setup
- callers can create objects without knowing their concrete classes
- a prototype registry can provide named templates selected at runtime

## Relationship With SOLID

- **Open/Closed Principle:** new prototype types can be added without changing the code that requests a clone through a common abstraction.
- **Single Responsibility Principle:** each prototype owns the knowledge required to copy its own state correctly.
- **Dependency Inversion Principle:** services can depend on a cloning interface or prototype registry instead of concrete product classes.

## When To Use It

- objects start from a common template and differ in a few fields
- initialization requires expensive database, network, parsing, or computation work
- object types are selected dynamically from a registry or configuration
- clients should create copies without knowing the concrete type

## When Not To Use It

- a simple constructor or builder is clearer than maintaining clone logic
- each new object has substantially different configuration
- correctly copying nested mutable state is difficult or unsafe
- the object has external resources that should not be duplicated, such as open file handles or active network connections

## Notes

- Decide explicitly whether cloning is shallow or deep. Deep copying is needed when nested mutable state must not be shared.
- Immutable objects are easier and safer to clone because their internal state cannot be changed after copying.
- Copy constructors and language-provided copy operations can implement Prototype even without a method literally named `clone`.
- Do not blindly clone external resources. Usually create a new resource or share it through a separately managed abstraction.
