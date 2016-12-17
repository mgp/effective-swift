### Prefer enumerations with raw values over constants

In Objective-C, members of an enumeration have a unique integer value. Swift extends this idea by allowing you to define enumerations having a unique string, character, integer, or floating point type. These are called the *raw values* for the enumeration.

Consider a `struct` named `UserExerciseFeedback` that pairs the unique identifier of an exercise with the user's feedback for that exercise:

```swift
struct UserExerciseFeedback {
  let exerciseID: String
  let feedback: String
}
```

The client may want to serialize an instance of this struct as a JSON object for delivery to the server. The keys of this object must be unique. We can enforce this uniqueness at compile-time by specifying the keys as raw values in an enumeration. This enumeration can belong to an extension that provides a `toJSON` method like so:

```swift
extension UserExerciseFeedback {
  private enum JSONKey: String {
    case exerciseID = "exercise_id"
    case feedback = "feedback"
  }

  func toJSON() -> [String: AnyObject] {
    let json = [
      JSONKey.exerciseID.rawValue: exerciseId,
      JSONKey.feedback.rawValue: feedback
    ]
    return json
  }
}
```

If we chose not to use an enumeration to define the keys, but instead used a pair of as static constants, then no compilation error will catch the following careless copy-and-paste mistake:

```swift
private static let exerciseIDJSONKey = "exercise_id"
private static let feedbackJSONKey = "exercise_id"  // error -- should be "feedback"
```

Therefore, prefer enumerations with raw values, and have the compiler catch such errors for you.

#### Promote to enumeration values

The `toJSON` method of the preceding example "demoted" the `JSONKey` abstraction to `String` because it was constructing a JSON object, and a JSON object must have keys that are `String` values. But whenever possible, "promote" a value of the raw type to its associated enumeration value by using the enumeration's `rawValue:` initializer.

For example, consider `UIAlertView` and the `UIAlertViewDelegate` method `alertView(_:clickedButtonAtIndex:)`. Say the `UIAlertView` prompts the user to retry or cancel a request for data from the server. We can specify the indexes of these buttons as the raw values of an enumeration:

```swift
enum RequestDataErrorButton: Int /* button index */ {
  case cancel = 0
  case retry = 1
}
```

When `alertView(_:clickedButtonAtIndex:)` is invoked on the delegate, the `buttonIndex` parameter is an `Int`. We can promote it to its corresponding `RequestDataErrorButton` value by using the enumeration's `rawValue:` initializer, and then invoke `switch` on the enumeration value like so:

```swift
func alertView(alertView: UIAlertView, clickedButtonAtIndex buttonIndex: Int) {
  if let tappedButton = RequestDataErrorButton(rawValue: buttonIndex) {
    switch tappedButton {
    case .cancel:
      cancelRequest()
    case .retry:
      retryRequest()
  } else {
    println("Unrecognized button index: \(buttonIndex)")
  }
}
```

Again, `tappedButton` is a higher level of abstraction than `buttonIndex`, because `tappedButton` represents the button itself.

As in the preceding JSON example, using an enumeration allows the compiler to catch errors for us: If we add a new value to the `RequestDataErrorButton` enumeration, then the compiler will generate an error if we do not add a corresponding `case` to the `switch` statement.

