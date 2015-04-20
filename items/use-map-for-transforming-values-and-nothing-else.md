### Use `map` for transforming values and nothing else

The documentation for method `map` of type `Array` says:

> Use this method to return a new array containing the results of applying a provided closure to transform each element in the existing array.

And the closure parameter in its declaration is even named `transform`:

```
func map<U>(transform: (T) -> U) -> [U]
```

Given an array of views, we can use `map` to materialize their heights in a new array:

```swift
let viewHeights = views.map { view in
  view.frame.size.height
}
```

Because the `map` method invokes a given closure for each element in an array, it may be tempting to use `map` as a generalized iteration construct. For example:

```swift
views.map { view in
  view.frame.size.height *= 2
}
```

But this code does not fulfill the expectation that `map` will transform the given array. Instead, this use of `map` performs a [side-effect](http://en.wikipedia.org/wiki/Side_effect_%28computer_science%29), namely doubling the height of each view. Any programmer who expects that each use of `map` only transforms arrays will be surprised that the behavior of the program is henceforth changed.

Moreover, if we need the doubled height values, we might be tempted to write:

```
let doubledHeights = views.map { view in
  view.frame.size.height *= 2
}
```

The Swift compiler generates neither a warning nor a error for this code. But the type of the returned array is not `[CGFloat]` as we might expect. It is instead `[Void]`, which is equivalent to `[()]`.

#### Using `each`

To avoid this abuse of `map`, we can extend `Array` with an `each`. Like `map`, it invokes a closure once for each element in the array. Unlike `map`, it does not return any value:

```swift
extension Array {
  func each(applicator: Element -> Void) {
    for element in self {
      applicator(element)
    }
  }
}
```

The `Element` type refers to the `Element` typealias defined inside `Array`. The method itself borrows its name from its corresponding form in Ruby, as well as from the popular [jQuery](http://jquery.com/) and [Underscore](http://underscorejs.org/) libraries for JavaScript.

A client can invoke `each` with a closure exactly like it would with `map`:

```swift
views.each { view in
  view.frame.size.height *= 2
}
```

Because the closure used by the `each` method cannot return a value, we conclude that it must process or consume its parameter -- thereby creating a side-effect. There is no ambiguity. Additionally, if the client attempts to assign the value returned by `each` to `doubledHeights` -- similar to what we did with `map` -- the Swift compiler will helpfully warn that the value is

> inferred to have type '()', which may be unexpected

