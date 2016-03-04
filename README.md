# Swift-hints
It's just a little collection of hints and lessons that I've learned from some talks, articles and so on.

### Embrace Immutability
(https://realm.io/news/slug-keith-smiley-embrace-immutability/)

* In server-driven applications, client models **must** be as immutable as possible.
```swift
struct Ride: Mappable {
    var pickup: Place?
}
```
This code is completely unsafe. Instead, we can create mutable substitutes and receive immutable "dumb models" from the server:
```swift
struct RideRequest {
    var pickup: Place?
}
```
###### Application
In [SwiftyNURE](https://github.com/dreymonde/SwiftyNURE) models which have no sense to be changed (it just won't affect anything), were looking like this:
```swift
public struct Subject {
    public var id: Int
    public var name: String
    public var shortName: String
}
```
Much cleaner now:
```swift
public struct Subject {
    public let id: Int
    public let name: String
    public let shortName: String
}
```
* The problem with most JSON mappers is that they encourage to create mutable models, or models with optional properties (which is not quite good in many cases).
