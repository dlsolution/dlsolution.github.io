---
layout: post
title: "Getting started with Realm Database"
author: "Linh Vo"
tags: "Databases"
---

# Table of Content

- [Overview](#overview)
- [Installation](#installation)
- [Import Realm](#import-realm)
- [Define Your Object Model](#define-your-object-model)
- [Open a Realm](#open-a-realm)
- [Create, Read, Update, and Delete Objects](#crud)
- [Watch for Changes](#watch-for-changes)
- [Migrations](#migrations)
- [Encrypt a Realm](#encrypt)
- [Core Data vs Realm](#core-data-vs-realm)

# <a id="overview"></a>Overview

**Realm Database** is an offline-first mobile object database in which you can directly access and store live objects without an [ORM](https://en.wikipedia.org/wiki/Object%E2%80%93relational_mapping).

**ORM** convert flat data like SQL database tables into object graphs, so that those can be used from native code. Realm’s custom storage engine doesn’t need to convert data to and from object graphs, so there’s no need for an **ORM**.

> Realm does not use SQLite as its engine, rather it is a database built from scratch to run directly on phones, tablets, and wearables. Realm is an object-oriented database that uses C++ as its core. However, SQLite uses a transactional SQL database engine.

**Realms** are the core data structure used to organize data in **Realm Database**. A realm is a collection of the objects that you use in your application, called **Realm objects**, as well as additional metadata that describe the objects.

**Realm objects** are basically regular Swift or Objective-C classes, but they also bring a few additional features like [live queries](https://docs.mongodb.com/realm/sdk/ios/fundamentals/live-queries/#std-label-ios-live-queries). The iOS SDK memory maps **Realm objects** directly to native Swift or Objective-C objects, which means there's no need to use a special data access library, such as an [ORM](https://en.wikipedia.org/wiki/Object-relational_mapping). Instead, you can work with **Realm objects** as you would any other class instance.

All **Realm objects** are **live objects**, which means they automatically update whenever they're modified. Realm emits a [notification event](https://docs.mongodb.com/realm/sdk/ios/examples/react-to-changes/#std-label-ios-react-to-changes) whenever any property changes.

You can use **live objects** to work with object-oriented data natively without an [ORM](https://en.wikipedia.org/wiki/Object-relational_mapping) tool. **live objects** are direct proxies to the underlying stored data, which means that a **live object** doesn't directly contain data. Instead, a **live object** always references the most up-to-date data on disk and [lazy loads](https://en.wikipedia.org/wiki/Lazy_loading) property values when you access them from a [collection](https://docs.mongodb.com/realm/sdk/ios/data-types/collections/#std-label-ios-client-collections). This means that a realm can contain many objects but only pay the performance cost for data that the application is actually using.

You can read back the data that you have [stored](https://docs.mongodb.com/realm/sdk/ios/fundamentals/write-transactions/#std-label-ios-write-transactions) in [Realm Database](https://docs.mongodb.com/realm/get-started/glossary/#std-term-Realm-Database) by finding, filtering, and sorting objects.

All query, filter, and sort operations return a [results collection](https://docs.mongodb.com/realm/sdk/ios/data-types/collections/#std-label-ios-results-collections). The **results collections** are live, meaning they always contain the latest results of the associated query.

## Optimized for Performance & Low Memory Use

A common pattern in mobile development is displaying long lists of items, be it the user’s contacts, entries in a calendar, or list of tweets.

When using a “traditional” storage, you would often just copy items from disk into an array in memory, effectively duplicating your model — and if you’re not careful with copying arrays, you might just triple it as well.

If you’re dealing with very long lists, you’d like to be more lean than just loading all data into memory. You need to write code to load pages (or batches) of records from disk (preferably in the background to keep UI scrolling smooth) and copy only the current batch into memory. Then you have to keep track of how far the user has scrolled the list and load more batches as they scroll further.

A common solution to this issue would be using **NSFetchedResultsController** (Core Data) or similar. But remember: this is a pre-fetching class that runs on top of an **ORM** that runs on top of an SQLite database. And that feels like a lot of middlemen right there between you and your data:

{% include image.html
img="assets/2019-04-03/image.png"
max-width="300px" %}

In contrast, with the **Realm Database** you use a class called **Results**, which helps you directly query objects on disk and load only the ones you actually are using at a given moment into memory.

**Results** allows you to only load the data you actually need and use. (Within reason of course - you can’t realistically read isolated bytes from disk)

So when you chose to drive a UI table view with the **Results** class, you read from disk only the data to display the currently visible table rows.

The best part? You don’t need to do anything extra to achieve that. While your user scrolls through a table view with hundreds of thousands of rows, **Results** keeps in memory only the data needed to display the 5-6 rows you have on screen at a time.

### Zero-copy

The traditional way of reading data from a database leads to unnecessary copying (raw data -> deserialized representation -> language-level objects). Realm avoids this by mapping the whole data in-memory, using B+ trees and whenever data is queried, Realm simply calculates the offset, reads from the memory mapped region and returns the raw value.

Results to a query are not copies of your data: modifying the results of a query will modify the data on disk directly. This memory mapping also means that results are **live**: that is, they always reflect the current state on disk.

See also: [Collections are Live](https://docs.mongodb.com/realm/sdk/ios/data-types/collections/#std-label-ios-live-collections).

### Lazy loading

Realm Database defers execution of a query until you access the results. You can [chain several filter and sort operations](https://docs.mongodb.com/realm/sdk/ios/examples/read-and-write-data/#std-label-ios-chain-queries) without requiring extra work to process the intermediate state.

See also: [Results are Lazily Evaluated](https://docs.mongodb.com/realm/sdk/ios/data-types/collections/#std-label-ios-lazy-evaluated-results).

### References Are Retained

One benefit of Realm Database's object model is that Realm Database automatically retains all of an object's [relationships](https://docs.mongodb.com/realm/sdk/ios/fundamentals/relationships/#std-label-ios-client-relationships) as direct references, so you can traverse your graph of relationships directly through the results of a query.

A **direct reference**, or pointer, allows you to access a related object's properties directly through the reference.

Other databases typically copy objects from database storage into application memory when you need to work with them directly. Because application objects contain direct references, you are left with a choice: copy the object referred to by each direct reference out of the database in case it's needed, or just copy the foreign key for each object and query for the object with that key if it's accessed. If you choose to copy referenced objects into application memory, you can use up a lot of resources for objects that are never accessed, but if you choose to only copy the foreign key, referenced object lookups can cause your application to slow down.

Realm Database bypasses all of this using [zero-copy](https://docs.mongodb.com/realm/get-started/glossary/#std-term-zero-copy) [live objects](https://docs.mongodb.com/realm/get-started/glossary/#std-term-live-objects). [Realm object](https://docs.mongodb.com/realm/get-started/glossary/#std-term-Realm-object) accessors point directly into database storage using memory mapping, so there is no distinction between the objects in Realm Database and the results of your query in application memory. Because of this, you can traverse direct references across an entire realm from any query result.

# <a id="installation"></a>Installation

Before you begin, ensure you have [Installed the iOS SDK](https://docs.mongodb.com/realm/sdk/ios/install/).

You can use `SwiftPM`, `CocoaPods`, or `Carthage` to add the MongoDB Realm Swift SDK to your project.

# <a id="import-realm"></a>Import Realm

Near the top of any Swift file that uses Realm, add the following import statement:

```swift
import RealmSwift
```

# <a id="define-your-object-model"></a>Define Your Object Model

For a local-only Realm Database you can define your [object model](https://docs.mongodb.com/realm/sdk/ios/fundamentals/object-models-and-schemas/#std-label-ios-realm-objects) directly in code.

```swift
// LocalOnlyQsTask is the Task model for this QuickStart
class LocalOnlyQsTask: Object {
    @Persisted var name: String = ""
    @Persisted var owner: String?
    @Persisted var status: String = ""

    convenience init(name: String) {
        self.init()
        self.name = name
    }
}
```

# <a id="open-a-realm"></a>Open a Realm

In a local-only Realm Database, the simplest option to open a realm is by omitting the configuration parameter, which uses the default realm:

```swift
// Open the local-only default realm
let localRealm = try! Realm()
```

You can also specify a [Realm.Configuration](https://docs.mongodb.com/realm-sdks/swift/latest/Structs/Realm/Configuration.html) parameter to open a realm at a specific file URL, in-memory, or with a subset of classes.

# <a id="crud"></a>Create, Read, Update, and Delete Objects

Once you have opened a realm, you can modify it and its [objects](https://docs.mongodb.com/realm/sdk/ios/fundamentals/object-models-and-schemas/#std-label-ios-realm-objects) in a [write transaction](https://docs.mongodb.com/realm/sdk/ios/fundamentals/write-transactions/#std-label-ios-write-transactions) block.

To create a new Task, instantiate the Task class and add it to the realm in a write block:

```swift
// Add some tasks
let task = LocalOnlyQsTask(name: "Do laundry")
try! localRealm.write {
    localRealm.add(task)
}
```

You can retrieve a live [collection](https://docs.mongodb.com/realm/sdk/ios/data-types/collections/#std-label-ios-client-collections) of all tasks in the realm:

```swift
// Get all tasks in the realm
let tasks = localRealm.objects(LocalOnlyQsTask.self)
```

You can also filter that collection using a [filter](https://docs.mongodb.com/realm/sdk/ios/examples/filter-data/#std-label-ios-client-query-engine):

```swift
// You can also filter a collection
let tasksThatBeginWithA = tasks.filter("name beginsWith 'A'")
print("A list of all tasks that begin with A: \(tasksThatBeginWithA)")
```

To modify a task, update its properties in a write transaction block:

```swift
// All modifications to a realm must happen in a write block.
let taskToUpdate = tasks[0]
try! localRealm.write {
    taskToUpdate.status = "InProgress"
}
```

Finally, you can delete a task:

```swift
// All modifications to a realm must happen in a write block.
let taskToDelete = tasks[0]
try! localRealm.write {
    // Delete the LocalOnlyQsTask.
    localRealm.delete(taskToDelete)
}
```

# <a id="watch-for-changes"></a>Watch for Changes

You can [watch a realm, collection, or object for changes](https://docs.mongodb.com/realm/sdk/ios/examples/react-to-changes/#std-label-ios-react-to-changes) with the `observe` method.

> Be sure to retain the notification token returned by `observe` as long as you want to continue observing. When you are done observing, invalidate the token to free the resources.

```swift
// Observe collection notifications. Keep a strong
// reference to the notification token or the
// observation will stop.
notificationToken = tasks.observe { [weak self] (changes) in
    guard let tableView = self?.tableView else { return }
    switch changes {
    case .initial: break
        // Results are now populated and can be accessed without blocking the UI
    case .update(_, let deletions, let insertions, let modifications):
        // Query results have changed, so apply them to the TableView
        tableView.performBatchUpdates {
            // Always apply updates in the following order: deletions, insertions, then modifications.
            // Handling insertions before deletions may result in unexpected behavior.
            tableView.deleteRows(at: deletions.map({ IndexPath(row: $0, section: 0)}),
                                 with: .automatic)
            tableView.insertRows(at: insertions.map({ IndexPath(row: $0, section: 0) }),
                                 with: .automatic)
            tableView.reloadRows(at: modifications.map({ IndexPath(row: $0, section: 0) }),
                                 with: .automatic)
        } completion: { finished in
            // ...
        }
    case .error(let error):
        // An error occurred while opening the Realm file on the background worker thread
        fatalError("\(error)")
    }
}
```

Later, when done observing:

```swift
// Invalidate notification tokens when done observing
notificationToken.invalidate()
```

# <a id="migrations"></a>Migrations

When you update your object schema, you must increment the schema version and perform a migration.

If your schema update adds or removes properties, Realm Database can perform the migration automatically, and you only need to increment the `schemaVersion`.

For more complex schema updates, such as combining fields, renaming fields, or converting from an object to an embedded object, you must also specify the migration logic in a `migrationBlock`.

Example. A realm using schema version `1` has a `Person` object type that has separate fields for first and last names:

```swift
// In the first version of the app, the Person model
// has separate fields for first and last names,
// and an age property.
class Person: Object {
    @Persisted var firstName = ""
    @Persisted var lastName = ""
    @Persisted var age = 0
}
```

The developer decides that the `Person` class should use a combined `fullName` field instead of the separate `firstName` and `lastName` fields.

To migrate the realm to conform to the updated `Person` schema, the developer sets the realm's schema version to `2` and defines a migration function to set the value of `fullName` based on the existing `firstName` and `lastName` properties.

```swift
// In the second version, the Person model has one
// combined field for the name. A migration will be required
// to convert from version 1 to version 2.
class Person: Object {
    @Persisted var fullName = ""
    @Persisted var age = 0
}
```

```swift
let config = Realm.Configuration(
    schemaVersion: 2, // Set the new schema version.
    migrationBlock: { migration, oldSchemaVersion in
        if oldSchemaVersion < 2 {
            // The enumerateObjects(ofType:_:) method iterates over
            // every Person object stored in the Realm file
            migration.enumerateObjects(ofType: Person.className()) { oldObject, newObject in
                // combine name fields into a single field
                let firstName = oldObject!["firstName"] as? String
                let lastName = oldObject!["lastName"] as? String
                newObject!["fullName"] = "\(firstName!) \(lastName!)"
            }
        }
    }
)

// Now that we've told Realm how to handle the schema change, opening the file
// will automatically perform the migration
let realm = try! Realm(configuration: config)
```

To **rename a property** during a migration, use the `Migration.renameProperty(onType:from:to:)` method.

```swift
let config = Realm.Configuration(
    schemaVersion: 1,
    migrationBlock: { migration, oldSchemaVersion in
        // We haven’t migrated anything yet, so oldSchemaVersion == 0
        if oldSchemaVersion < 1 {
            // The renaming operation should be done outside of calls to `enumerateObjects(ofType: _:)`.
            migration.renameProperty(onType: Person.className(), from: "yearsSinceBirth", to: "age")
        }
    })
```

# <a id="encrypt"></a>Encrypt a Realm

You can encrypt the realm database file on disk with AES-256 + SHA-2 by supplying a 64-byte encryption key when [opening a realm](https://docs.mongodb.com/realm/sdk/ios/examples/configure-and-open-a-realm/#std-label-ios-open-a-local-realm).

Realm transparently encrypts and decrypts data with standard [AES-256 encryption](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) using the first 256 bits of the given 512-bit encryption key. Realm uses the other 256 bits of the 512-bit encryption key to validate integrity using a [hash-based message authentication code (HMAC)](hash-based message authentication code (HMAC)).

> Do not use cryptographically-weak hashes for realm encryption keys. For optimal security, we recommend generating random rather than derived encryption keys.

## Storing & Reusing Keys

You must pass the same encryption key when opening the realm again. Apps should store the encryption key in the Keychain so that other apps cannot read the key.

## Performance Impact

Typically, reads and writes on encrypted realms can be up to 10% slower than unencrypted realms.

## Multiple Processes

If multiple processes need to access a realm simultaneously, use an unencrypted realm. Encrypted realms cannot be accessed by multiple processes at the same time.

As an alternative, you can store data that you want to encrypt as `NSData` properties on realm objects. Then, you can encrypt and decrypt individual fields.

One possible tool to encrypt and decrypt fields is [Apple's CryptoKit framework](https://developer.apple.com/documentation/cryptokit). You can use [Swift Crypto](https://github.com/apple/swift-crypto) to simplify app development with CryptoKit.

## Step by Step

The following code demonstrates how to generate an encryption key and open an encrypted realm:

```swift
// Retrieve the existing encryption key for the app if it exists or create a new one
func getKey() -> Data {
    // Identifier for our keychain entry - should be unique for your application
    let keychainIdentifier = "io.Realm.EncryptionExampleKey"
    let keychainIdentifierData = keychainIdentifier.data(using: String.Encoding.utf8, allowLossyConversion: false)!

    // First check in the keychain for an existing key
    var query: [NSString: AnyObject] = [
        kSecClass: kSecClassKey,
        kSecAttrApplicationTag: keychainIdentifierData as AnyObject,
        kSecAttrKeySizeInBits: 512 as AnyObject,
        kSecReturnData: true as AnyObject
    ]

    // To avoid Swift optimization bug, should use withUnsafeMutablePointer() function to retrieve the keychain item
    // See also: http://stackoverflow.com/questions/24145838/querying-ios-keychain-using-swift/27721328#27721328
    var dataTypeRef: AnyObject?
    var status = withUnsafeMutablePointer(to: &dataTypeRef) { SecItemCopyMatching(query as CFDictionary, UnsafeMutablePointer($0)) }
    if status == errSecSuccess {
        // swiftlint:disable:next force_cast
        return dataTypeRef as! Data
    }

    // No pre-existing key from this application, so generate a new one
    // Generate a random encryption key
    var key = Data(count: 64)
    key.withUnsafeMutableBytes({ (pointer: UnsafeMutableRawBufferPointer) in
        let result = SecRandomCopyBytes(kSecRandomDefault, 64, pointer.baseAddress!)
        assert(result == 0, "Failed to get random bytes")
    })

    // Store the key in the keychain
    query = [
        kSecClass: kSecClassKey,
        kSecAttrApplicationTag: keychainIdentifierData as AnyObject,
        kSecAttrKeySizeInBits: 512 as AnyObject,
        kSecValueData: key as AnyObject
    ]

    status = SecItemAdd(query as CFDictionary, nil)
    assert(status == errSecSuccess, "Failed to insert the new key in the keychain")

    return key
}

// ...
// Use the getKey() function to get the stored encryption key or create a new one
var config = Realm.Configuration(encryptionKey: getKey())

do {
    // Open the realm with the configuration
    let realm = try Realm(configuration: config)

    // Use the realm as normal

} catch let error as NSError {
    // If the encryption key is wrong, `error` will say that it's an invalid database
    fatalError("Error opening realm: \(error)")
}
```

# <a id="core-data-vs-realm"></a>Core Data vs Realm

**Realm** seems to be more convenient solution right now, however I think it’s worth to learn both frameworks. Below there are some advantages of **RealmSwift**.

- has built-in encryption

- faster than Core Data (benchmark)

- uses less space

- a little bit simpler interface

- easy migrations

- easy fetching – no need to use NSFetchRequest

- easy to use in multithreaded environment

- support for auto-updating results

- built-in observable notifications which allow to update UI

- cross-platform library

- support for cloud sync (extra paid)

- Realm Studio which you can use to browse and manage database. It updates live when you open database from simulator’s storage.

- really nice documentation and big community
