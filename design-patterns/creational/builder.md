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

Suppose we are building a SQL query. A query always needs a table, but it may also include selected fields, filtering conditions, sorting, and a row limit.

Passing every possible value to a constructor makes the call difficult to read. Callers must remember parameter order, provide empty collections for options they do not need, and assemble the SQL string themselves.

### Swift
```swift
struct SQLQuery {
    let statement: String
}

let query = SQLQuery(
    statement: "SELECT id, name FROM users WHERE active = true AND role = 'admin' ORDER BY name LIMIT 20"
)
```

### Go
```go
type SQLQuery struct {
	Statement string
}

query := SQLQuery{
	Statement: "SELECT id, name FROM users WHERE active = true AND role = 'admin' ORDER BY name LIMIT 20",
}
```

### TypeScript
```typescript
class SQLQuery {
  constructor(public readonly statement: string) {}
}

const query = new SQLQuery(
  "SELECT id, name FROM users WHERE active = true AND role = 'admin' ORDER BY name LIMIT 20",
)
```

### Python
```python
from dataclasses import dataclass


@dataclass(frozen=True)
class SQLQuery:
    statement: str


query = SQLQuery(
    "SELECT id, name FROM users WHERE active = true AND role = 'admin' ORDER BY name LIMIT 20"
)
```

This design has a few problems:

- query construction is one long, hard-to-read expression
- callers must format SQL correctly and in the right order
- optional values require empty arrays, `nil`, or placeholder arguments
- validation rules are scattered between callers and the query object
- adding a new clause changes the constructor API

## Solution

Introduce a builder that collects query configuration through small, named methods. The builder owns the order in which SQL clauses are assembled and creates the final query only after required values and validation rules are satisfied.

The resulting call reads like a description of the query being built.

### Swift
```swift
struct SQLQuery {
    let statement: String
}

final class SQLQueryBuilder {
    private var selectFields: [String] = ["*"]
    private var table: String = ""
    private var conditions: [String] = []
    private var orderBy: String?
    private var limitCount: Int?

    func select(_ fields: String...) -> SQLQueryBuilder {
        selectFields = fields.isEmpty ? ["*"] : fields
        return self
    }

    func from(_ table: String) -> SQLQueryBuilder {
        self.table = table
        return self
    }

    func whereCondition(_ condition: String) -> SQLQueryBuilder {
        conditions.append(condition)
        return self
    }

    func orderBy(_ field: String) -> SQLQueryBuilder {
        self.orderBy = field
        return self
    }

    func limit(_ count: Int) -> SQLQueryBuilder {
        limitCount = count
        return self
    }

    func build() -> SQLQuery {
        precondition(!table.isEmpty, "A table is required")
        precondition(limitCount == nil || limitCount! > 0, "Limit must be positive")

        var statement = "SELECT \(selectFields.joined(separator: ", ")) FROM \(table)"
        if !conditions.isEmpty {
            statement += " WHERE " + conditions.joined(separator: " AND ")
        }
        if let orderBy {
            statement += " ORDER BY \(orderBy)"
        }
        if let limitCount {
            statement += " LIMIT \(limitCount)"
        }
        return SQLQuery(statement: statement)
    }
}

let query = SQLQueryBuilder()
    .select("id", "name")
    .from("users")
    .whereCondition("active = true")
    .whereCondition("role = 'admin'")
    .orderBy("name")
    .limit(20)
    .build()
```

### Go
```go
import (
	"fmt"
	"strings"
)

type SQLQuery struct {
	Statement string
}

type SQLQueryBuilder struct {
	selectFields []string
	table        string
	conditions   []string
	orderBy      string
	limitCount   *int
}

func NewSQLQueryBuilder() *SQLQueryBuilder {
	return &SQLQueryBuilder{selectFields: []string{"*"}}
}

func (b *SQLQueryBuilder) Select(fields ...string) *SQLQueryBuilder {
	if len(fields) > 0 { b.selectFields = fields }
	return b
}

func (b *SQLQueryBuilder) From(table string) *SQLQueryBuilder {
	b.table = table
	return b
}

func (b *SQLQueryBuilder) Where(condition string) *SQLQueryBuilder {
	b.conditions = append(b.conditions, condition)
	return b
}

func (b *SQLQueryBuilder) OrderBy(field string) *SQLQueryBuilder {
	b.orderBy = field
	return b
}

func (b *SQLQueryBuilder) Limit(count int) *SQLQueryBuilder {
	b.limitCount = &count
	return b
}

func (b *SQLQueryBuilder) Build() SQLQuery {
	if b.table == "" { panic("a table is required") }
	statement := fmt.Sprintf("SELECT %s FROM %s", strings.Join(b.selectFields, ", "), b.table)
	if len(b.conditions) > 0 { statement += " WHERE " + strings.Join(b.conditions, " AND ") }
	if b.orderBy != "" { statement += " ORDER BY " + b.orderBy }
	if b.limitCount != nil {
		if *b.limitCount <= 0 { panic("limit must be positive") }
		statement += fmt.Sprintf(" LIMIT %d", *b.limitCount)
	}
	return SQLQuery{Statement: statement}
}

query := NewSQLQueryBuilder().
	Select("id", "name").
	From("users").
	Where("active = true").
	Where("role = 'admin'").
	OrderBy("name").
	Limit(20).
	Build()
```

### TypeScript
```typescript
class SQLQuery {
  constructor(public readonly statement: string) {}
}

class SQLQueryBuilder {
  private selectFields = ["*"]
  private table = ""
  private conditions: string[] = []
  private orderByField?: string
  private limitCount?: number

  select(...fields: string[]): this {
    if (fields.length > 0) this.selectFields = fields
    return this
  }
  from(table: string): this { this.table = table; return this }
  where(condition: string): this { this.conditions.push(condition); return this }
  orderBy(field: string): this { this.orderByField = field; return this }
  limit(count: number): this { this.limitCount = count; return this }

  build(): SQLQuery {
    if (!this.table) throw new Error("A table is required")
    if (this.limitCount !== undefined && this.limitCount <= 0) {
      throw new Error("Limit must be positive")
    }
    let statement = `SELECT ${this.selectFields.join(", ")} FROM ${this.table}`
    if (this.conditions.length > 0) statement += ` WHERE ${this.conditions.join(" AND ")}`
    if (this.orderByField) statement += ` ORDER BY ${this.orderByField}`
    if (this.limitCount !== undefined) statement += ` LIMIT ${this.limitCount}`
    return new SQLQuery(statement)
  }
}

const query = new SQLQueryBuilder()
  .select("id", "name")
  .from("users")
  .where("active = true")
  .where("role = 'admin'")
  .orderBy("name")
  .limit(20)
  .build()
```

### Python
```python
from dataclasses import dataclass


@dataclass(frozen=True)
class SQLQuery:
    statement: str


class SQLQueryBuilder:
    def __init__(self) -> None:
        self._select_fields = ["*"]
        self._table = ""
        self._conditions: list[str] = []
        self._order_by: str | None = None
        self._limit_count: int | None = None

    def select(self, *fields: str) -> "SQLQueryBuilder":
        if fields: self._select_fields = list(fields)
        return self
    def from_table(self, table: str) -> "SQLQueryBuilder":
        self._table = table
        return self
    def where(self, condition: str) -> "SQLQueryBuilder":
        self._conditions.append(condition)
        return self
    def order_by(self, field: str) -> "SQLQueryBuilder":
        self._order_by = field
        return self
    def limit(self, count: int) -> "SQLQueryBuilder":
        self._limit_count = count
        return self

    def build(self) -> SQLQuery:
        if not self._table: raise ValueError("A table is required")
        if self._limit_count is not None and self._limit_count <= 0:
            raise ValueError("Limit must be positive")
        statement = f"SELECT {', '.join(self._select_fields)} FROM {self._table}"
        if self._conditions: statement += " WHERE " + " AND ".join(self._conditions)
        if self._order_by: statement += f" ORDER BY {self._order_by}"
        if self._limit_count is not None: statement += f" LIMIT {self._limit_count}"
        return SQLQuery(statement)


query = (SQLQueryBuilder()
    .select("id", "name")
    .from_table("users")
    .where("active = true")
    .where("role = 'admin'")
    .order_by("name")
    .limit(20)
    .build())
```

## Why This Is Better

- named builder methods make each SQL clause clear at the call site
- the builder assembles clauses in the correct SQL order
- callers provide only the options they need
- default values, such as `SELECT *`, are centralized in one place
- validation happens before the final query is returned
- the final query can be immutable because configuration is handled by the builder

## Relationship With SOLID

- **Single Responsibility Principle:** the builder handles query construction while the product represents the completed SQL query.
- **Open/Closed Principle:** new optional clauses can usually be added as builder methods without changing existing calls.
- **Dependency Inversion Principle:** a service can depend on a builder abstraction when it must support different query representations or database dialects.

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
- In production SQL builders, values should be parameterized rather than interpolated directly into the query string.
- In languages with named parameters or option structs, those features can be a lighter alternative for simple cases.
