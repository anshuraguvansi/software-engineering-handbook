## Swift fundamentals

Swift is a type-safe language that can be used to build software for mobile, desktop, and server-side platforms.

Let us start with a hello world program:

```swift
print("Hello, world!")
```

If you run this, it prints "Hello, world!" in the console. In scripts and playgrounds, Swift allows top-level code, so you can run this directly.

## Constants and variables

`let` is used for constants, which means immutable once assigned. If a value type is assigned to `let`, you cannot change it. If a reference type is assigned to `let`, you cannot change the reference, but you can still update mutable properties of the object.

```swift
let maxAttempts: Int = 10
// maxAttempts = 15 // error: cannot assign to value: 'maxAttempts' is a 'let' constant

class Point {
    var x: Int
    var y: Int

    init(x: Int, y: Int) {
        self.x = x
        self.y = y
    }
}

let point: Point = Point(x: 10, y: 20)
point.x = 30 // valid: the reference is constant, object properties can still change
```

`var` is used for mutable variables, which means you can change the value.

```swift
var maxAttempts: Int = 10
maxAttempts = 15 // valid: var is mutable, so you can change it.
```

### Quick Notes:
- Use `let` unless you need mutability.
    When you see `let`, you (and the compiler) know that value will not change for the rest of its lifetime. This removes a whole category of bugs where a value gets mutated unexpectedly, especially in large codebases.

- `let` helps the compiler optimize code better.

- Thread safety and data races.
    A `let` constant, once initialized, is safe to read from multiple threads without synchronization.

- `let` means "single-assignment," not "known at compile time." Those are two different things.
    ```swift
    // Compile-time constant — value known when code is compiled
    let x = 5

    // Runtime constant — value NOT known until the program runs
    let currentTime = Date()
    ```

## Lazy Var
`lazy` is used when we want to delay creating a value. The value is created only when it is first accessed, which can save startup work and memory.

- Example:
```swift
class DataLoader {
    lazy var fileData: Data? = {
        try? Data(contentsOf: url)
    }()

    private let url: URL
    init(url: URL) {
        self.url = url
    }
    
    func dumpData() {
        guard let fileData else {
            print("No data loaded")
            return
        }
        print(String(data: fileData, encoding: .utf8) ?? "Invalid UTF-8 data")
    }
}
```

File data is loaded only on the first access of fileData (for example, inside dumpData()), which can reduce initial memory usage and startup work.