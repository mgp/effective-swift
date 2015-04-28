### Avoid implicitly unwrapped optionals

A program written using implicitly unwrapped optionals is no more safe than Objective-C code. You lose all the benefits of optionals.

The unwrapping of an optional is obvious:

```swift
let possibleString: String? = "An optional string."
let forcedString: String = possibleString! // requires an exclamation mark
```

The unwrapping of an implicitly unwrapped optional is not obvious:

```swift
let assumedString: String! = "An implicitly unwrapped optional string."
let implicitString: String = assumedString // no need for an exclamation mark
```

Implicitly unwrapped optionals can creep into your Swift code if you are interfacing with Objective-C code.

```swift
@interface KHABadge : NSObject

/** Returns the KHABadge with the given name, or nil if missing. */
+ (instancetype)badgeWithName:(NSString *)badgeName;

@end
```

Swift imports this as:

```swift
init!(name: String!)
```

The return type of this method is `KHABadge!`. A Swift client can do:

```swift
let badge: KHABadge = KHABadge(name: nil)
displayBadge(badge)
```

Swift 1.2 adds the `nonnull` and `nullable` annotations:

```swift
+ (nullable instancetype)badgeWithName:(nonnull NSString *)badgeName;
```

Swift now imports the initializer as a standard failable initializer:

```swift
init?(name: String)
```

It is now a compilation error to pass `nil` as the `name` parameter of the initializer. Moreover, we must now explicitly unwrap the returned optional, such as by using an `if let` statement:

```swift
let badgeName = "persistence"
if let badge = KHABadge(name: badgeName) {
  displayBadge(badge)
} else {
  println("Unrecognized badge name: \(badgeName)")
}
```

#### Acceptable use cases

An initializer must initialize its constant properties, defined using `let`, before it returns or calls its superclass initializer. If the initializer of the property requires a reference to the initializing instance -- typically by the latter passing `self` to the former` -- then TODO.

TODO: Example using signals?

Implicitly unwrapped optionals are also suitable for the properties of a test class, where the `setUp` method creates a new instance before every test. For example:

```swift
class QueueTestCase: XCTestCase {
  private var queue: Queue<Int>!

  override func setUp() {
    super.setUp()
    queue = Queue<Int>()
  }

  /* Test methods follow here. */
}
```

Each test method assumes that the `queue` property has been set up accordingly.

```swift
func testEmptyQueue() {
  XCTAssertEqual(0, queue.length)
  XCTAssertEqual([], queue.values)

  XCTAssertNil(queue.dequeue())
}
```

The test will fail if `setUp` does not assign a value to `queue`, because `queue` will be `nil`. This is ideal, since the only purpose of the test is to assert the correctness of `queue`!

