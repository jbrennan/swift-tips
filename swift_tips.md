Swift Tips
==========

Here are some tips for programming in Swift for Hopscotch, which has lots of Objective C code too. Please update this as you learn new things! These are in no particular order...

Handy Links
-----------

- [Swift Programming Language](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/index.html#//apple_ref/doc/uid/TP40014097)
- [Using Swift with Objective C](https://developer.apple.com/library/prerelease/mac/documentation/Swift/Conceptual/BuildingCocoaApps/index.html#//apple_ref/doc/uid/TP40014216-CH2-XID_0)
- [FAQ on Swift](http://www.raywenderlich.com/74138/swift-language-faq)

Using Objective C code in Swift
-------------------------------

In order to use an Objective C class or file in Swift you must expose the class in the "Swift Bridging header" file, which is called `Hopscotch-Bridging-Header.h`. Anything you import in there will be exposed to Swift code.

### Casting in for-loops

If you have an `NSArray` and you want to loop over elements in Swift, you can use the following pattern:

```
for aView in foundationArray as [UIView] {
    // aView is of type UIView
}
```

### Objective C enums in Swift

If you have an Objective C enum like the following:

```
typedef NS_ENUM(NSUInteger, JBMyEnumType) {
	JBMyEnumTypeFirst,
	JBMyEnumTypeSecond
};
```

When you use this enum in Swift, the values will lose their two letter prefix (e.g., "JB"), so the first value becomes `JBNyEnumType.MyEnumTypeFirst`.

Using Swift code in Objective C
-------------------------------

### Importing Swift code

To import Swift symbols you must import the Swift header. In our project this is called `Hopscotch-Swift.h`. Import that wherever you need to use it.

At first glance it looks like we can't just import this in our `pch` file because then our `Constants.h` file can't find things like CGFloat. I'm sure there's a way to fix this, we should look into it. We should also consider breaking the constants file up more.

### Protocols

#### NSObject

Swift protocols you create that conform to `<NSObject>` must instead conform to `NSObjectProtocol` because in Swift protocol names must be different than class names (I think). So your protocol would look like:


```
protocol MyProtocol: NSObjectProtocol {
	...
}
```

#### Adopting Swift protocols in Objective C

To make your Objective C class conform to a Swift protocol, you must first make sure you import the Swift header (mentioned above) and you must also make sure your protocol has the `@objc` keyword in front of it.

```
/** MyProtocol.swift */
@objc protocol MyProtocol: NSObjectProtocol {
	...
}

/** MyConformingClass.h */
#import "Hopscotch-Swift.h"
@interface MyConformingClass: NSObject <MyProtocol>
	...
@end
```

Pragma mark
-----------

To do a pragma mark you use the following syntax

```
// MARK: - Awesome bit of methods
```


NSLocalizedStrings
------------------

Apparently the localized string macros have been replaced with one function:

```
NSLocalizedString(key: String, tableName: String?, bundle: NSBundle, value: String, comment: String)
```

Where you can omit table, bundle, and value (they have default values) and just use something like:

```
NSLocalizedString("Hello", comment: "standard greeting")
```


Initializers
------------

### "With" initializers

Swift seems to automatically convert "with" initializers to its `init()` syntax. This is expected for things like `initWithFrame:` which becomes `init(frame:)` but it *also* seems to happen for things like `+ buttonWithSuperview:` which becomes `init(superview:)`.


Closures
--------

### Retain cycles

Swift didn't solve the problem of strongly capturing `self` in closures. That sucks. What you can do is provide a "capture list" of values in the closure and explicitly declare self as a weak or unowned reference. You do the following:

```
var closure = {
	[weak self] in
	self?.doSomething()
	self?.doSomethingElse()
}
```

When you use `weak` that makes `self` an Optional, which means it could go away and be nil by the time the closure executes. So, you have to either use the unwrapped optional (i.e., `ronBurgundy?`) or "let it out" by using the if-let syntax (`if let strongSelf = self { ... }`)

You can use `unowned` instead of `weak` to avoid the "optional" issue, but only do so if you know for sure that the value will still exist. If you try to reference it and it's nil, the app will crash. This seems less safe, and we should probably generally use `weak` until we understand the memory issues better.

### Return values

[From the docs](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/GuidedTour.html):

>Single statement closures implicitly return the value of their only statement.

Which means something like

```
button.tapAction = {
	[weak self] in
	self?.animateOutWithCompletion(nil)
}
```

Won't compile. I think this is because `self` is `weak`, which means it's an optional type. So, it's possible that `self?.whatever()` won't execute because `self` could be nil. So the return type can't be verified.


Protocols
---------

### Delegates

To make a Swift protocol that represents a delegate, you should declare the protocol as a "class" type:

```
protocol MyDelegate : class {
	// methods/properties
}
```

This means the protocol can only be conformed to by reference types (e.g., a `struct` can't conform to it). The reason why you should do this is so you can make the delegate property be `weak` so retain cycles can be avoided.

```
class MyClass {
	weak var delegate : MyDelegate? // weak properties must be Optionals
}
```

Enums
-----

### Naming enum cases

It appears you can't name an enum case to be called `Type`. My guess is this is some kind of existing known member already in Swift objects (I'd love to have a docs link for this!). Come up with a better name.

```
enum MyEnum {
    case Type // Will compile but not really usable.
    case MyType // Just peachy.
}
```
