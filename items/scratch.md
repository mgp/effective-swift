### item: prefer struct to class

Whenever appropriate, TODO.

### item: compose constant structs for immutability

TODO: Tie in with immutability

This bestows on the value the property of *immutability*, which has the following advantages:

* Our principal goal as programmers is to reduce complexity. At any point in a program, we can reason about the state of a constant trivially -- its state is the state upon assignment. By contrast, the state of a variable can change endlessly.
* With judicious use, immutable objects can lead to quicker execution speed and lower memory usage. The `hash` value of a `String`, the query parameters of an immutable URL, and the distance traced by an immutable sequence of points are all immutable as well. We can cache their values for later use instead of recompute them on every access. We can also share an immutable object with clients without requiring those clients to defensively copy it.
* An immutable value is *safe* in many ways. For example, `String` is immutable and so its `hash` value is immutable. Consequently, it is safe to use as a `String` as a key in a `Dictionary`: The lower order bits of that `hash` will never change, and so an inserted key will never reside in the wrong bucket. An immutable value is also thread-safe, becuase a client must acquire a lock only to read data that another client can concurrently modify. An immutable value renders that moot.

### other items:

* implement printable for value types

