### Prefer `let` over `var`

Whenever possible, declare a value using `let` instead of using `var`, thereby declaring it as a constant instead of a variable. This makes its state easier to reason about: If the value is a `struct` -- including primitive types like `Int`, `String`, and containers like `Array` and `Dictionary` -- then the value is immutable throughout its lifetime. If the value is an instance of a `class`, then the value will refer to the same instance throughout its lifetime, albeit it may be mutable. (The use of `struct` vs `class` and their relation to immutability is explored more in future items.)

#### Local values

In Swift 1.0, you do not need to sacrifice `let` for `var` to cope with conditionals. For example, consider a `UserProgress` enum that specifies whether the user has not started, has started, or has finished watching a video. Given the length of a video in seconds and how many seconds the user has watched, we can compute the `UserProgress` value as:

```swift
var progress = UserProgress.NotStarted
if secondsWatched >= videoLength {
  progress = .Finished
} else if secondsWatched > 0 {
  progress = .Started
}
```

But if `progress` is not modified later, we should prefer to specify it as a constant using `let`. We can do so by factoring out the computation of `UserProgress` into a method:

```swift
func progressForSecondsWatched(
    secondsWatched: NSTimeInterval,
    ofVideoWithLength videoLength: NSTimeInterval) -> UserProgress {
  if secondsWatched >= videoLength {
    return .Finished
  } else if secondsWatched > 0 {
    return .Started
  } else {
    return .NotStarted
  }
}
```

We then invoke this method and assign its return value to `progress`:

```swift
let progress = progressForSecondsWatched(secondsWatched, ofVideoWithLength: videoLength)
```

In Swift 1.2, the new rule is that a `let` constant must be initialized before it is used, but we do not need to initialize it upon declaration. We can forego factoring out the computation of `UserProgress` into a method, and instead write:

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

Note that factoring out a method anyway has other benefits, such as improved readability, the opportunity for reuse, and the ability to be tested in isolation.

#### Static values

Named constants are always preferable over [magic numbers](http://en.wikipedia.org/wiki/Magic_number_%28programming%29). In Swift 1.0, class constants are not yet supported, but a computed class property that returns a constant value can serve the same function:

```swift
private class var fadeInAnimationsDuration: NSTimeInterval {
  return 0.5
}
```

But Swift 1.2 supports class constants, so we can simply write:

```swift
private static let fadeInAnimationsDuration: NSTimeInterval = 0.5
```

