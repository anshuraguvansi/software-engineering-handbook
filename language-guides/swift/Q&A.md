## Swift Interview Questions and Answers

### 1. What is the difference between let and var?
- Answer:
`let` creates a constant, so you cannot assign a new value after initialization. `var` creates a variable, so you can change the value later.
- Example:
```swift
let name = "Anshu"
// name = "Kumar" // not allowed

var count = 1
count = 2 // allowed
```

### 2. Why is let preferred over var wherever possible?
- Answer:
It avoids unexpected value changes, helps compiler optimization, and improves safety when reading code across threads.

### 3. Why does lazy require var and not let?
- Answer:
Because lazy properties are initialized after instance initialization, on first access. That delayed assignment is a mutation, so the property must be `var`.
- Example:
```swift
class Counter {
    let id = 10
    lazy var value: Int = {
        return self.id * 2
    }()
}
```

Conceptually, you can think of it like this:

```swift
private var _value: Int?
var value: Int {
    if _value == nil {
        _value = self.id * 2
    }
    return _value!
}
```

So a lazy var is mutable state. It starts as "not yet computed" and gets assigned the first time it is accessed. That assignment from "uninitialized" to "computed value" is a mutation that happens after `init()` completes.

`let` requires the value to be set exactly once during initialization, before the object is fully initialized. A lazy let would be a contradiction: `lazy` delays assignment, but `let` requires assignment by the end of `init()`.