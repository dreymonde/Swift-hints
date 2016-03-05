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

## Swift Scripting
(https://realm.io/news/swift-scripting/ 
http://www.bensnider.com/using-swift-to-make-command-line-scripts-part-2.html
http://cerebration.typed.com/blog/swift-2-as-a-scripting-language)

#### Using third-party frameworks
There are two common ways to use third-party frameworks. First is to place them in /Library/Frameworks/ and then place following shebang in the beginning of .swift file:
```
#!/usr/bin/env xcrun swift -F /Library/Frameworks
```
It's kinda ugly because you need to manually build all frameworks and place them in just one place. Much better is to use Carthage just as with any other Swift project:
```
#!/usr/bin/env xcrun swift -F Carthage/Build/Mac
```

#### Executing scripts
The easier way is to make .swift file executable with `chmod +x` command, but this not giving you the actual *executable*, it's just running the script line-by-line. Instead, you can use `swiftc` or `xcrun`:
```
$ swiftc hello.swift
// or
$ xcrun -sdk macos swiftc hello.swift
```
If you wan't to use third-party frameworks (via Carthage), it becomes a less uglier:
```
$ xcrun -sdk macosx swiftc -F Carthage/Build/Mac/ btc.swift -Xlinker -v -Xlinker -rpath -Xlinker @executable_path/Carthage/Build/Mac/
```
And you can't actually pack the executable with the frameworks, you're just *linking* to them. Sadly, but that's how dynamic frameworks actually work.

## Functional View Controllers
(https://www.youtube.com/watch?v=uQFI9rDrl8s)

#### Lighter view controllers
It's a good practice to put some work *out* of view controllers which are insanely complex. A lot of "model stuff" needs to be implemented by model - for example, `UITableViewDataSource`. Same thing with the views.

## Swift-ly Secure
(https://realm.io/news/seth-law-swift-security/)

- Your product does *not* trust it's users.
- Most of Apple's storaging APIs does not provide an encryption.
- Starting from iOS 8.3, you can't access file system, but still can access it with the backups.
- Don't store sensitive data in the property lists.
- Store it in the Keychain.
- Be aware of App Sharing.
- There is no good reason not to encrypt anymore.
- Third-party frameworks can't be 100% trusted.
- Security is hard. *Try harder.*
