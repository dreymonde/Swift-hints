# Swift-hints

### Embrace Immutability
(https://realm.io/news/slug-keith-smiley-embrace-immutability/)

1. In server-driven applications, client models **must** be as immutable as possible.
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

2. The problem with most JSON mappers is that they encourage to create mutable models, or models with optional properties (which is not quite good in many cases).
