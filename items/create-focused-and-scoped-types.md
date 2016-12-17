### Create focused and scoped types

Classes in Objective-C are heavyweight and so creating them is laborious. Typically you must specify:

* the public `@interface` in its `.h` file
* a class extension in its `.m` file defining any private properties or instance variables
* the `@implementation` in its `.m` file

Additionally, you may speicfy in a separate header file an additional class extension that declares methods to be called or overridden only by subclasses of your class. (An example of this rare practice is the `UIGestureRecognizerSubclass.h` header file for `UIGestureRecognizer`.)

Swift follows the modern convention of allowing you to define a single file containing a single class, where we can specify an access level for the properties and methods of the class. The Swift blog explains [why there is no `protected` access level](https://developer.apple.com/swift/blog/?id=11); note that [favoring composition over inheritance](http://en.wikipedia.org/wiki/Composition_over_inheritance) renders this concern moot anyway.

Consequently, types in Swift are lightweight and easy to create. Leverage this to create a suite of focused and scoped types.

#### Focused

For example, most problems on Khan Academy offer a sequence of hints if the user needs help. Tapping the hint button reveals the next hint in the sequence until all hints are exhausted.

As for the visual treatment of the hint button, it is enabled as long as hints remain. Additionally, it only displays how many hints remain after the user has revealed the first hint.

If we track the number of hints total and the number of hints remaining as `numberHintsTotal` and `numberHintsRemaining`, respectively, then we could write:

```swift
hintsButton.enabled = (numberHintsRemaining > 0)

if numberHintsTotal != numberHintsRemaining {
  hintsButton.numberHintsRemaining = numberHintsRemaining
  hintsButton.numberHintsLabelVisible = true
} else {
  hintsButton.numberHintsLabelVisible = false
}
```

To help ensure that we pass these two values together, we can easily create a `HintsCounter` structure that composes them. Moreover, we can add computed properties to `HintsCounter` that assign meaningful names to the conditional expressions above:

```swift
struct HintsCounter {
  let numberTotal: UInt
  let numberRemaining: UInt

  var hasUsedHints: Bool {
    return (numberTotal != numberRemaining)
  }
  var hasRemainingHints: Bool {
    return (numberRemaining > 0)
  }
}
```

Rewriting the code to use the `HintsCounter` instance:

```swift
hintsButton.enabled = hintsCounter.hasRemainingHints

if hintsCounter.hasUsedHints {
  hintsButton.numberHintsRemaining = hintsCounter.numberRemaining
  hintsButton.numberHintsLabelVisible = true
} else {
  hintsButton.numberHintsLabelVisible = false
}
```

The code now is now simpler and more readable. And although `HintsCounter` is very simple, it is now very easy to test its computed properties `hasUsedHints` and `hasRemainingHints`. Moreover, such an abstraction is ripe for reuse. For example, the following method to compute the accessibility label of the hint button uses both `hasUsedHints` and `hasRemainingHints` again:

```swift
static func accessibilityLabelForHintButtonWithCounter(hintsCounter: HintsCounter) -> String {
  if hintsCounter.hasUsedHints {
    return hintsCounter.hasRemainingHints ?
        "Show the next hint" : "No hints remain"
  } else {
    return (hintsCounter.numberTotal > 0) ?
        "Show the first hint" : "No hints available"
  }
}
```

Even this small `struct` pays big dividends.

#### Scoped

If one type is only meaningful within the context of another type, put the former within the scope of the latter, and set its access level appropriately.

For example, after the user demonstrates proficiency in a skill, we display how many points the user earned, any badges the user earned, and the user's mastery level for the skill. A `CompletedSkillData` structure composes all of this data. Its inner structure `PointsBreakdown` specifies the multiple sources from which the user earned points:

```swift
struct CompletedSkillData {
  struct PointsBreakdown {
    let pointsFromQuestions: Int
    let pointsFromBadges: Int
    let otherPoints: Int

    var totalPoints: Int {
      return pointsFromQuestions + pointsFromBadges + otherPoints
    }
  }

  let pointsBreakdown: PointsBreakdown
  let badges: [Badge]
  let masteryLevel: MasteryLevel
}
```

The properties of `PointsBreakdown` represent only the user's points for a completed skill, and so a `PointsBreakdown` instance disassociated from the completed skill has no meaning. Therefore `PointsBreakdown` is an inner type of `CompletedSkillData`.

Also consider the extension from the item _[prefer enumerations with raw values over constants](prefer-enums-with-raw-values-over-constants.md)_ that provides JSON encoding for the `UserExerciseFeedback` struct:

```swift
extension UserExerciseFeedback {
  private enum JSONKey: String {
    case exerciseID = "exercise_id"
    case feedback = "feedback"
  }

  func toJSON() -> [String: AnyObject] {
    let json = [
      JSONKeys.exerciseID.rawValue: exerciseId,
      JSONKeys.feedback.rawValue: feedback
    ]
    return json
  }
}
```

The `UserExerciseFeedback` extension should encapsulate encoding an instance to JSON, and the keys of the returned dictionary are an implementation detail. Therefore the `JSONKey` enumeration is a private inner type of `UserExerciseFeedback`.

