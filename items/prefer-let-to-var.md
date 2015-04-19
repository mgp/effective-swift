## Prefer `let` to `var`

Whenever possible, declare a value using `let` instead of using `var`, thereby declaring it as a constant instead of a variable. This makes its state easier to reason about: If the value is a `struct` -- including primitive types like `Int`, `String`, and contianers like `Array` and `Dictionary` -- then the value is immutable throughout its lifetime. If the value is an instance of a `class`, then the value will refer to the same instance throughout its lifetime, but you cannot ensure that its state is immutable. (The use of `struct` vs `class` and their relation to immutability is explored more in the next item, TODO.)

In Swift 1.0, you do not need to sacrifice `let` for `var` to cope with conditionals. For example, consider a method that TODO.

```swift
var progress = UserProgress.NotStarted
if (secondsWatched >= videoLength) {
  progress = .Finished
} else if (secondsWatched > 0) {
  progress = .Started
}
```

You can do:

```swift
let progress = progressForSecondsWatched(secondsWatched, ofVideoWithLength: videoLength)
```

```swift
static func progressForSecondsWatched(
    secondsWatched: NSTimeInterval,
    ofVideoWithLength videoLength: NSTimeInterval) -> UserProgress {
  if (secondsWatched >= videoLength) {
    return .Finished
  } else if (secondsWatched > 0) {
    return .Started
  } else {
    return .NotStarted
  }
}
```

In Swift 1.2, TODO.

```swift
let progress: UserProgress
if (secondsWatched >= videoLength) {
  progress = .Finished
} else if (secondsWatched > 0) {
  progress = .Started
} else {
  progress = .NotStarted
}
```

TODO: Swift 1.2

item: prefer struct to class

Whenever appropriate, TODO.


item: compose constant structs for immutability


### TODO: Tie in with immutability

This bestows on the value the property of *immutability*, which has the following advantages:


* Our principal goal as programmers is to reduce complexity. At any point in a program, we can reason about the state of a constant trivially -- its state is the state upon assignment. By contrast, the state of a variable can change endlessly.
* With judicious use, immutable objects can lead to quicker execution speed and lower memory usage. The `hash` value of a `String`, the query parameters of an immutable URL, and the distance traced by an immutable sequence of points are all immutable as well. We can cache their values for later use instead of recompute them on every access. We can also share an immutable object with clients without requiring those clients to defensively copy it.
* An immutable value is *safe* in many ways. For example, `String` is immutable and so its `hash` value is immutable. Consequently, it is safe to use as a `String` as a key in a `Dictionary`: The lower order bits of that `hash` will never change, and so an inserted key will never reside in the wrong bucket. An immutable value is also thread-safe, becuase a client must acquire a lock only to read data that another client can concurrently modify. An immutable value renders that moot.

