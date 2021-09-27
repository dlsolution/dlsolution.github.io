---
layout: post
title: "Caching in Swift"
author: "Linh Vo"
tags: "UIKit"
---

Making an app feel fast and responsive isn’t only about tweaking the way its UI is rendered, or improving the sheer execution speed of its operations and algorithms — it’s often just as much about efficiently managing its data and avoiding unnecessary work.

One very common source of such unnecessary work is when we end up reloading the exact same data multiple times. It could be multiple features loading duplicate copies of the same model, or a view’s data being reloaded every time that it reappears on the screen.

This week — let’s take a look at how caching can be an incredibly powerful tool in those kind of situations, how to build an efficient and elegant caching API in Swift, and how strategically caching various values and objects can have a big impact on the overall performance of an app.

# Part of the system

Caching is one of those tasks that at first might seem much simpler than it actually is. Not only do we have to efficiently store and load values, we also need to decide when to evict entries in order to keep our memory footprint low, invalidate stale data, and much more.

Thankfully, Apple has already solved many of those problems for us through the built-in `NSCache` class. However, using it does come with a few caveats, as it’s still an Objective-C class on Apple’s own platforms — which means that it can only store class instances, and it’s only compatible with `NSObject`-based keys:

```swift
// To be able to use strings as caching keys, we have to use
// NSString here, since NSCache is only compatible with keys
// that are subclasses of NSObject:
let cache = NSCache<NSString, MyClass>()
```

However, by writing a thin wrapper around `NSCache`, we can create a much more flexible Swift caching API — that enables us to store structs and other value types, and lets us use any `Hashable` key type — without requiring us to rewrite all of the underlying logic that powers `NSCache`. So, let’s do just that.

# It all starts with a declaration

The first thing we’ll do is to declare our new `cache` type. Let’s call it `Cache`, and make it a generic over any Hashable key type, and any value type. We’ll then give it an `NSCache` property, which will store `Entry` instances keyed by a `WrappedKey` type:

```swift
final class Cache<Key: Hashable, Value> {
    private let wrapped = NSCache<WrappedKey, Entry>()
}
```

Our `WrappedKey` type will, like its name suggests, wrap our public-facing `Key` values in order to make them `NSCache` compatible. To achieve that, let’s subclass `NSObject` and implement `hash` and `isEqual` — since that’s what Objective-C uses to determine whether two instances are equal:

```swift
private extension Cache {
    final class WrappedKey: NSObject {
        let key: Key

        init(_ key: Key) { self.key = key }

        override var hash: Int { return key.hashValue }

        override func isEqual(_ object: Any?) -> Bool {
            guard let value = object as? WrappedKey else {
                return false
            }

            return value.key == key
        }
    }
}
```

When it comes to our `Entry` type, the only requirement is that it needs to be a class (it doesn’t need to subclass `NSObject`), which means that we can simply make it store a `Value` instance:

```swift
private extension Cache {
    final class Entry {
        let value: Value

        init(value: Value) {
            self.value = value
        }
    }
}
```

With the above in place, we’re now ready to give `Cache` an initial set of APIs. Let’s start with three methods — one for inserting a value for a given key, one for retrieving values, and one for removing an existing value:

```swift
final class Cache<Key: Hashable, Value> {
    private let wrapped = NSCache<WrappedKey, Entry>()

    func insert(_ value: Value, forKey key: Key) {
        let entry = Entry(value: value)
        wrapped.setObject(entry, forKey: WrappedKey(key))
    }

    func value(forKey key: Key) -> Value? {
        let entry = wrapped.object(forKey: WrappedKey(key))
        return entry?.value
    }

    func removeValue(forKey key: Key) {
        wrapped.removeObject(forKey: WrappedKey(key))
    }
}
```

Since a cache is essentially just a specialized key-value store, it’s an ideal use case for [subscripting](https://www.swiftbysundell.com/articles/the-power-of-subscripts-in-swift/) — so let’s also make it possible to both retrieve and insert values that way:

```swift
extension Cache {
    subscript(key: Key) -> Value? {
        get { return value(forKey: key) }
        set {
            guard let value = newValue else {
                // If nil was assigned using our subscript,
                // then we remove any value for that key:
                removeValue(forKey: key)
                return
            }

            insert(value, forKey: key)
        }
    }
}
```

With that initial set of features implemented — let’s take our new `Cache` for a spin! Let’s say that we’re working on an app for reading articles, and that we’re using an `ArticleLoader` to load `Article` models. By using our new cache to both store the articles that we load, as well as check for any previously cached article before loading a new one — we can make sure that we’ll only load each article once, like this:

```swift
class ArticleLoader {
    typealias Handler = (Result<Article, Error>) -> Void

    private let cache = Cache<Article.ID, Article>()

    func loadArticle(withID id: Article.ID,
                     then handler: @escaping Handler) {
        if let cached = cache[id] {
            return handler(.success(cached))
        }

        performLoading { [weak self] result in
            let article = try? result.get()
            article.map { self?.cache[id] = $0 }
            handler(result)
        }
    }
}
```

> Another way to optimize the above loading code would be to avoid making duplicate requests if the article that we’re looking to load is already being loaded. To learn more about a technique for doing that, check out [“Avoiding race conditions in Swift”](https://www.swiftbysundell.com/articles/avoiding-race-conditions-in-swift).

The above might not seem like it’ll have a big impact on our app’s performance, but it could really make our app seem faster, as when the user will navigate back to an article that was already loaded — it’ll now instantly be there. If we also combine the above with prefetching articles that the user is likely to open (for example the latest article in the user’s favorite category), then we could really make our app much more delightful to use.

# Avoiding stale data

What makes `NSCache` a better fit for caching values compared to the collections found in the Swift standard library (such as `Dictionary`) is that it’ll automatically evict objects when the system is running low on memory — which in turn enables our app itself to remain in memory for longer.

However, we might want to add some cache invalidation conditions of our own, as otherwise we might end up keeping stale data around. While being able to reuse data that we’ve already loaded is certainly a good thing, displaying outdated data to our users is definitely not.

One way to mitigate that problem is to limit the lifetime of our cache entries by removing them after a certain time interval. To do that, we’ll start by adding an `expirationDate` property to our `Entry` class, to be able to keep track of the remaining lifetime for each entry:

```swift
final class Entry {
    let value: Value
    let expirationDate: Date

    init(value: Value, expirationDate: Date) {
        self.value = value
        self.expirationDate = expirationDate
    }
}
```

Next, we’ll need a way for `Cache` to get a hold of the current date, in order to determine whether a given entry is still valid. While we could just call `Date()` inline whenever needed, that’d make [unit testing](https://www.swiftbysundell.com/basics/unit-testing) really difficult — so let’s instead inject a `Date`-producing function as part of our initializer. We’ll also add an `entryLifetime` property, with a default value of 12 hours:

```swift
final class Cache<Key: Hashable, Value> {
    private let wrapped = NSCache<WrappedKey, Entry>()
    private let dateProvider: () -> Date
    private let entryLifetime: TimeInterval

    init(dateProvider: @escaping () -> Date = Date.init,
         entryLifetime: TimeInterval = 12 * 60 * 60) {
        self.dateProvider = dateProvider
        self.entryLifetime = entryLifetime
    }

    ...
}
```

> To learn more about the above kind of dependency injection, check out [“Simple Swift dependency injection with functions”](https://www.swiftbysundell.com/articles/simple-swift-dependency-injection-with-functions).

With the above in place, let’s now update our methods for inserting and retrieving values to take the current date and the specified `entryLifetime` into account:

```swift
func insert(_ value: Value, forKey key: Key) {
    let date = dateProvider().addingTimeInterval(entryLifetime)
    let entry = Entry(value: value, expirationDate: date)
    wrapped.setObject(entry, forKey: WrappedKey(key))
}

func value(forKey key: Key) -> Value? {
    guard let entry = wrapped.object(forKey: WrappedKey(key)) else {
        return nil
    }

    guard dateProvider() < entry.expirationDate else {
        // Discard values that have expired
        removeValue(forKey: key)
        return nil
    }

    return entry.value
}
```

While accurately invalidating stale entries is arguably the most difficult part of implementing any kind of caching — by combining the above kind of expiration dates with model-specific logic for removing values based on certain events (for example if the user deletes an article), we can most often avoid both duplicate work and invalid data.

# Persistent caching

So far we’ve only been caching values in memory, which means that as soon as our app is terminated, that data will be gone. While that might be what we actually want, sometimes also enabling cached values to be persisted on disk can be really valuable, and might also unlock new ways of using our app — such as still having access to data downloaded over the network when launching the app while offline.

Since we might only want to selectively persist specific caches on disk — let’s make it a completely optional feature. To get started, we’ll update `Entry` to also store the `Key` that it’s associated with, so that we’ll both be able to persist each entry directly, and to be able to remove unused keys:

```swift
final class Entry {
    let key: Key
    let value: Value
    let expirationDate: Date

    init(key: Key, value: Value, expirationDate: Date) {
        self.key = key
        self.value = value
        self.expirationDate = expirationDate
    }
}
```

Next, we’ll need a way to keep track of what keys that our cache contains entries for, since `NSCache` doesn’t expose that information. For that we’ll add a dedicated `KeyTracker` type, which will become the delegate of our underlying `NSCache`, in order to get notified whenever an entry was removed:

```swift
private extension Cache {
    final class KeyTracker: NSObject, NSCacheDelegate {
        var keys = Set<Key>()

        func cache(_ cache: NSCache<AnyObject, AnyObject>,
                   willEvictObject object: Any) {
            guard let entry = object as? Entry else {
                return
            }

            keys.remove(entry.key)
        }
    }
}
```

We’ll set up our `KeyTracker` while initializing `Cache` — and we’ll also set a maximum number of entries, which will help us avoid writing too much data onto disk — like this:

```swift
final class Cache<Key: Hashable, Value> {
    private let wrapped = NSCache<WrappedKey, Entry>()
    private let dateProvider: () -> Date
    private let entryLifetime: TimeInterval
    private let keyTracker = KeyTracker()

    init(dateProvider: @escaping () -> Date = Date.init,
         entryLifetime: TimeInterval = 12 * 60 * 60,
         maximumEntryCount: Int = 50) {
        self.dateProvider = dateProvider
        self.entryLifetime = entryLifetime
        wrapped.countLimit = maximumEntryCount
        wrapped.delegate = keyTracker
    }

    ...
}
```

Since our `KeyTracker` already gets notified whenever an entry was removed from our cache, all that we need to do to finish its integration is to notify it whenever a key was added, which we’ll do as part of our `insert` method:

```swift
func insert(_ value: Value, forKey key: Key) {
    ...
    keyTracker.keys.insert(key)
}
```

To be able to actually persist the contents of our cache, we’ll first need to serialize it. Just like how we leveraged `NSCache` to build our own caching API right on top of the system, let’s use `Codable` to enable our cache to be encoded and decoded using any compatible format (such as JSON).

We’ll start by making our `Entry` type conform to `Codable` — but we wouldn’t want to require all cache entries to be codable — so let’s use a [conditional conformance](https://www.swiftbysundell.com/articles/conditional-conformances-in-swift) to only adopt `Codable` for entries that have codable keys and values, like this:

```swift
extension Cache.Entry: Codable where Key: Codable, Value: Codable {}
```

During the encoding and decoding processes, we’ll both retrieve and insert entries, so in order to avoid duplicating code from our previous `insert` and `value` methods — let’s also move all of our logic that deals with `Entry` instances into two new private utility methods:

```swift
private extension Cache {
    func entry(forKey key: Key) -> Entry? {
        guard let entry = wrapped.object(forKey: WrappedKey(key)) else {
            return nil
        }

        guard dateProvider() < entry.expirationDate else {
            removeValue(forKey: key)
            return nil
        }

        return entry
    }

    func insert(_ entry: Entry) {
        wrapped.setObject(entry, forKey: WrappedKey(entry.key))
        keyTracker.keys.insert(entry.key)
    }
}
```

The last piece of the puzzle is to make `Cache` itself `Codable` under the same conditions that we used before — and by using the above two utility methods we can now both encode and decode all of our entries really easily:

```swift
extension Cache: Codable where Key: Codable, Value: Codable {
    convenience init(from decoder: Decoder) throws {
        self.init()

        let container = try decoder.singleValueContainer()
        let entries = try container.decode([Entry].self)
        entries.forEach(insert)
    }

    func encode(to encoder: Encoder) throws {
        var container = encoder.singleValueContainer()
        try container.encode(keyTracker.keys.compactMap(entry))
    }
}
```

With the above in place we can now save any `Cache` that contains `Codable` keys and values to disk — by simply encoding it into `Data`, and then writing that data to a file within our app’s dedicated caching directory, like this:

```swift
extension Cache where Key: Codable, Value: Codable {
    func saveToDisk(
        withName name: String,
        using fileManager: FileManager = .default
    ) throws {
        let folderURLs = fileManager.urls(
            for: .cachesDirectory,
            in: .userDomainMask
        )

        let fileURL = folderURLs[0].appendingPathComponent(name + ".cache")
        let data = try JSONEncoder().encode(self)
        try data.write(to: fileURL)
    }
}
```

Just like that, we’ve built a highly dynamic cache that’s fully Swift-compatible — with support for time-based invalidation, on-disk persistence, and a limit on the number of entries it contains — all by leveraging system APIs like `NSCache` and `Codable` to avoid having to re-invent the wheel.

# Conclusion

Strategically deploying caching to avoid having to reload the same data multiple times can have a big positive impact on an app’s performance. After all, even though we might optimize the way we load data within an app, not having to load that data at all will always be faster — and caching can be a great way to achieve just that.

However, there are multiple things to keep in mind when adding caching to a data loading pipeline — such as not keeping stale data around for too long, invalidating cache entries when the app’s environment changes (such as when the user changes their preferred locale), and making sure that deleted items are correctly purged.

Another thing to consider when deploying caching is what data to cache, and where to do it. While we’ve taken a look at an `NSCache`-based approach in this article, there are multiple other routes that can be explored as well, such as using another system API — `URLCache` — to perform our caching within the networking layer. We’ll take a closer look at that, and other types of caching, in upcoming articles.

What do you think? How do you usually use caching in Swift, and do you prefer NSCache, URLCache, or a completely custom solution?

Thanks for reading! 🚀

Original post: [https://www.swiftbysundell.com/articles/caching-in-swift/](https://www.swiftbysundell.com/articles/caching-in-swift/)
