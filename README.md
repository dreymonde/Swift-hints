# Swift-hints
It's just a little collection of hints and lessons that I've learned from some talks, articles and so on.

## Embrace Immutability
(https://realm.io/news/slug-keith-smiley-embrace-immutability/)

#### Dumb models
In server-driven applications, client models **must** be as immutable as possible.
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
#### Parsing heterogeneous data
The problem with most JSON mappers is that they encourage to create mutable models, or models with optional properties (which is not quite good in many cases). For example, [Gloss](https://github.com/hkellaway/Gloss):
```swift
init?(json: JSON) {
        guard let ownerId: Int = "id" <~~ json
            else { return nil }

        self.ownerId = ownerId
        self.username = "login" <~~ json
    }
```
This `guard let` statement looks like you're doing something wrong when you're using it, instead of "much simpler" possibilities written below. The secret is that all other properties are *optionals*, which is awful when you're dealing with ID's, names, passwords and other pretty important stuff. Instead, "throwable" initializers in Lyft's [Mapper](https://github.com/lyft/mapper) looks much cleaner and they're actually ***embracing immutability***:
```swift
init(map: Mapper) throws {
    try id = map.from("id")
    photoURL = map.optionalFrom("avatar_url")
  }
```
You can still use `optionalFrom` when you really need that optional, in other cases it's just safer and simplier to have non-optional immutable properties.
