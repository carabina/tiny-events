# TinyEvents
A tiny event system for Swift (1 source file, 59 lines of code)

## Usage
```Swift
import TinyEvents
```

Create an event and add a closure to call when the event is fired:
```Swift
event = TinyEvent()
observer = event.add { print("Event fired") }
```

And then later fire it:
```Swift
event.fire()
```

To remove an observer, just destroy it. Either explicitly:
```Swift
observer = nil
```

or if the observer is stored as a property on some object of yours, it will be removed automatically when your object is destroyed:
```Swift
class SomeViewController: UIViewController {
    // […]
    override func viewDidLoad() {
        // […]
        observer = someSystem.importantEvent.add { [unowned self] in
            self.importantEventLabel.isHidden = false
        }
    }
    
    var observer: TinyEventObserver? // I'll be destroyed automatically with the view controller.
}
```

You can pass data to the observers by using `TinyEventWithData<T>` instead of `TinyEvent`:
```Swift
enum Option { case good, better, best }
userSelectedOption = TinyEventWithData<Option>()
// The observers will be of type `TinyEventWithDataObserver<Option>`
optionObserver = userSelectedOption.add { selectedOption in
    print(selectedOption)
}
userSelectedOption.fire(Option.best)
```

Pass multiple pieces of data using tuples (or structs or any other kind of structure):
```Swift
titleRatedByUser = TinyEventWithData<(String, Int)>()
observer = titleRatedByUser.add { title, rating in
    print("\(title) rated: \(rating)")
}
titleRatedByUser.fire(("Foo", 10))
```

## Avoiding Strong Reference Cycles (Important)

In short, whenever you reference `self` in your closure and store the observer on that same object you should use `[unowned self]` in the closure's capture list to avoid creating a reference cycle (see the `SomeViewController` listing above for an example of this).

This is important, because if you didn't add the `[unowned self]` then you'd have created a reference cycle — in this example `view controller -> observer -> closure -> view controller` — and then even if the view controller would normally have been destroyed and deallocated, it would be kept alive by the reference in your closure and you'd leak memory.

Please see the [Automatic Reference Counting chapter of The Swift Programming Language](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/AutomaticReferenceCounting.html) if you'd like to know more about this subject.

## Installation

You can add TinyEvents to your project using:
* [Carthage](https://github.com/Carthage/Carthage): Add `github "JJC1138/tiny-events"` to your Cartfile
* [Swift Package Manager](https://swift.org/package-manager/): Add `.Package(url: "https://github.com/JJC1138/tiny-events.git", majorVersion: 1)` to your dependencies
* [CocoaPods](https://cocoapods.org/): Add `pod 'TinyEvents'` to your Podfile

or just drop the [`TinyEvents.swift`](https://raw.githubusercontent.com/JJC1138/tiny-events/master/Sources/TinyEvents.swift) file into your project.

## Stability and Support

I'm using TinyEvents in production in my app [Day Planner](https://itunes.apple.com/us/app/day-planner-organize-your-time/id1232385157?ls=1&mt=8) and I intend to keep it updated for any breaking changes in new Swift versions, but I don't have any plans for adding new features. If you'd like a slightly heavier event system with more GitHub stars I'd suggest having a look at [emitter-kit](https://github.com/aleclarson/emitter-kit) or [Observable-Swift](https://github.com/slazyk/Observable-Swift).

Please use GitHub issues for questions, bug reports, suggestions, and patches.

## Frequently Asked Questions

#### Why am I getting warnings about `Immutable value 'someObserver' was never used; consider replacing with '_' or removing it` or `Variable 'someObserver' was written to, but never read`?
Are you storing an observer in a local variable or constant instead of in a property? That's fine, but those warnings are hinting at something important, which is that [the automatic reference counting system will consider unused local objects as being unnecessary and can destroy them immediately](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20151207/001501.html), and your closure won't get called if the observer is destroyed. The solution to this is to use the Swift standard library function [`withExtendedLifetime(_:_:)`](https://developer.apple.com/reference/swift/1541033-withextendedlifetime) to tell the system explicitly that you want the observer to stay alive:
```Swift
let event = TinyEvent()
let observer = event.add {
    // […]
}
withExtendedLifetime(observer) {
    event.fire()
}
```
In my experience this doesn't come up frequently (if at all) in normal usage because it's unusual to have an event observer that is as short-lived as a function invocation, but this idiom is used extensively in the TinyEvent tests.
