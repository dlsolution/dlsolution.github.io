---
layout: post
title: "Getting started with Core Data"
author: "Linh Vo"
tags: "Database"
---

# Table of Content

- [Overview](#overview)
- [Data Model](#data-model)
- [Core Data Stack](#core-data-stack)
- [Operations (CRUD)](#operations)
  - [Create (Insert)](#create)
  - [Read (Fetch)](#read)
  - [Update](#update)
  - [Delete](#delete)
  - [Batch Delete](#batch-delete)
  - [Delete Everything (Delete All Objects, Reset Core Data)](#delete-everything)
  - [Delete Rule](#delete-rule)
  - [Transactions](#transactions)
- [Core Data Concurrency](#core-data-concurrency)
  - [Is Core Data Thread Safe?](#is-core-data-thread-safe)
  - [.mainQueueConcurrencyType](#main-queue-concurrency-type)
  - [.privateQueueConcurrencyType](#private-queue-concurrency-type)
  - [NSManagedObjectContext.perform](#context-perform)
  - [NSManagedObjectContext.performAndWait](#context-performAndWait)
  - [Core Data Background Context](#core-data-background-context)
  - [Core Data Background Fetch Request](#core-data-background-fetch-request)
  - [Save Core Data On A Background Thread](#save-core-data-on-a-background-thread)
  - [Create A New Core Data Object On A Background Thread](#create-a-new-core-data-object-on-a-background-thread)
- [Parent/Child Managed Object Context](#parent-child-moc)
- [Fetch Requests](#fetch-requests)
  - [Get Object By ID](#get-object-by-id)
  - [Fetch A Single Object](#fetch-a-single-object)
  - [Filter Fetch Request With Predicate](#filter-fetch-request-with-predicate)
  - [Filter Predicate With Multiple Conditions](#filter-predicate-with-multiple-conditions)
  - [Fetch All Objects Of One Entity](#fetch-all-objects-of-one-entity)
- [Queries Using NSPredicate](#queries-using-nspredicate)
  - [Exact String Match With A NSPredicate](#exact-string-match-with-a-nspredicate)
  - [Contains String With A NSPredicate](#contains-string-with-a-nspredicate)
  - [NSPredicate Regex Search](#nspredicate-regex-search)
  - [Search For Matches That Begin With String](#search-for-matches-that-begin-with-string)
  - [Search For Matches That End With String](#search-for-matches-that-end-with-string)
- [Migrations](#migrations)
  - [Lightweight migration (automatic)](#lightweight)
  - [Heavyweight migration](#heavyweight)
- [Core Data vs Realm](#core-data-vs-realm)

# <a id="overview"></a>Overview

Core Data is an object graph and persistence framework that you use to manage the model layer objects in your application. Core Data provides **on-disk persistence**, which means your data will be accessible even after terminating your app or shutting down your device. This is different from in-memory persistence, which will only save your data as long as your app is in memory, either in the foreground or in the background.

Use Core Data to save your application’s permanent data for offline use, to cache temporary data, and to add undo functionality to your app on a single device. To sync data across multiple devices in a single iCloud account, Core Data automatically mirrors your schema to a CloudKit container.

Through Core Data’s Data Model editor, you define your data’s types and relationships, and generate respective class definitions. Core Data can then manage object instances at runtime to provide the following features.

> What is object graph? In an object-oriented program, groups of objects form a network through their relationships with each other — either through a direct reference to another object or through a chain of intermediate references. These groups of objects are referred to as object graphs. Object graphs may be small or large, simple or complex.

### Core Data vs SQLite

![#](https://docs-assets.developer.apple.com/published/7d6b410668/95d28eec-4652-4824-bac8-a413ede649ea.png)

- Basically, Core Data is a framework used to save, track, modify and filter the data within iOS apps, however, Core Data is not a Database.

- SQLite is a database itself like we have MS SQL Server.

- Core Data is using SQLite as its persistent store but the framework itself is not the database.

- But Core Data is an ORM (Object Relational Model) which creates a layer between the database and the UI.

- It speeds-up the process of interaction as we don’t have to write queries, just work with the ORM and let ORM handles the backend.

### Undo and Redo of Individual or Batched Changes

Core Data’s undo manager tracks changes and can roll them back individually, in groups, or all at once, making it easy to add undo and redo support to your app.

![#](https://docs-assets.developer.apple.com/published/6e6259c804/9ec0237d-0f5c-4f01-ac35-b6b55de236a9.png)

### Background Data Tasks

Perform potentially UI-blocking data tasks, like parsing JSON into objects, in the background. You can then cache or store the results to reduce server roundtrips.

![#](https://docs-assets.developer.apple.com/published/d090c20938/df1bcb18-0917-4fa0-a269-62e23d0ac3c9.png)

### View Synchronization

Core Data also helps keep your views and data synchronized by providing data sources for table and collection views.

### Versioning and Migration

Core Data includes mechanisms for versioning your data model and migrating user data as your app evolves.

# <a id="data-model"></a>Data Model

To start working with Core Data we need to create a Data Model – a file containing definition of our entities. In order to do that, just press `CMD+N` and select `Data Model` template.

{% include image.html
img="assets/2019-03-07/image.png"
max-width="700px" %}

- Xcode comes with a powerful **Data Model editor**, which you can use to create your **managed object model**.

- A managed object model is made up of **entities**, **attributes** and **relationships**

- An **entity** is a class definition in Core Data.

- An **attribute** is a piece of information attached to an entity.

- A **relationship** is a link between multiple entities.

# <a id="core-data-stack"></a>Core Data Stack

![#](https://docs-assets.developer.apple.com/published/fb7e996966/6484bcb2-50dd-4f04-b9f3-b9282ce55153.png)

- An instance of [NSManagedObjectModel](https://developer.apple.com/documentation/coredata/nsmanagedobjectmodel) represents your app’s model file describing your app’s types, properties, and relationships.

- An instance of [NSManagedObjectContext](https://developer.apple.com/documentation/coredata/nsmanagedobjectcontext) An object space to manipulate and track changes to managed objects.

- An instance of [NSPersistentStoreCoordinator](https://developer.apple.com/documentation/coredata/nspersistentstorecoordinator) saves and fetches instances of your app’s types from stores. A coordinator that uses the model to help contexts and persistent stores communicate.

- An instance of [NSPersistentContainer](https://developer.apple.com/documentation/coredata/nspersistentcontainer) sets up the model, context, and store coordinator all at once.

{% include image.html
img="assets/2019-03-07/image3.jpg" %}

### Persistent store

A persistent store is the interface between the coordinator and the permanent state of the object graph for both reading and writing. When a change is pushed onto the store, the change becomes a permanent part of the object state, meaning the position of the storage in memory or on disk is not relevant.

You can think of a persistent store as a database file (Sqlite) where individual records each hold the last-saved values of a managed object (Entity).

- **NSInMemoryStoreType**: Stores the entire object graph in memory. Persistent, although not in the case that you turn off the machine or shut down the App.

- **NSBinaryStoreType**: Of the atomic storetypes this occupies the least disk space, and loads the fastest. Not compressed. As an atomic store type every Code Data object is loaded at once.

- **NSSQLiteStoreType**: Non-atomic. Occupies binary-like disk space (small), loads quickly but the file needs to be available with exclusive access. Supports atomic updates. Core Data uses this type by default, and many Apps seem to use this as their go-to backing store. Non-atomic, so has a smaller memory footprint than the atomic types.

- **NSXMLStoreType**: Not supported in iOS, but XML is human readable rather than going for outright speed. An atomic type, so can have a large memory footprint.

```swift
import CoreData

/*
 This is the CoreDataStack used by the app. It saves changes to disk.

 Managers can do operations via the:
 - `mainContext` with interacts on the main UI thread, or
 - `backgroundContext` with has a separate queue for background processing

 */
class CoreDataStack {

    static let shared = CoreDataStack()

    let persistentContainer: NSPersistentContainer
    let backgroundContext: NSManagedObjectContext
    let mainContext: NSManagedObjectContext

    private init() {
        let container = NSPersistentContainer(name: "CoreDataUnitTest")

        let description = container.persistentStoreDescriptions.first
        description?.type = NSSQLiteStoreType
        description?.shouldMigrateStoreAutomatically = true
        description?.shouldInferMappingModelAutomatically = true

        container.loadPersistentStores { description, error in
            guard error == nil else {
                fatalError("was unable to load store \(error!)")
            }
        }

        mainContext = container.viewContext
        mainContext.automaticallyMergesChangesFromParent = true

        backgroundContext = container.newBackgroundContext()
        backgroundContext.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy

        persistentContainer = container
    }
}
```

# <a id="operations"></a>Operations (CRUD)

## <a id="create"></a>Create (Insert)

```swift
@discardableResult
func createEmployee(firstName: String) -> Employee? {
    let context = persistentContainer.viewContext

    let employee = Employee(context: context)

    employee.firstName = firstName

    do {
        try context.save()
        return employee
    } catch let error {
        print("Failed to create: \(error)")
    }

    return nil
}
```

## <a id="read"></a>Read (Fetch)

By default Xcode inside each entity will generate `fetchRequest` class function which can be used to retrieve all objects. However you can also create your own `NSFetchRequest` with custom `NSPredicate`.

```swift
func fetchEmployees() -> [Employee]? {
    let context = persistentContainer.viewContext

    let fetchRequest = NSFetchRequest<Employee>(entityName: "Employee")
    fetchRequest.sortDescriptors = [NSSortDescriptor(key: "firstName", ascending: true)]

    do {
        let employees = try context.fetch(fetchRequest)
        return employees
    } catch let error {
        print("Failed to fetch companies: \(error)")
    }

    return nil
}

func fetchEmployee(withName name: String) -> Employee? {
    let context = persistentContainer.viewContext

    let fetchRequest = NSFetchRequest<Employee>(entityName: "Employee")
    fetchRequest.fetchLimit = 1
    fetchRequest.predicate = NSPredicate(format: "firstName == %@", name)

    do {
        let employees = try context.fetch(fetchRequest)
        return employees.first
    } catch let error {
        print("Failed to fetch: \(error)")
    }

    return nil
}
```

## <a id="update"></a>Update

```swift
func updateEmployee(employee: Employee) {
    // Update entity properties as needed
    employee.firstName = "Linh"

    let context = persistentContainer.viewContext

    do {
        try context.save()
    } catch let error {
        print("Failed to update: \(error)")
    }
}
```

## <a id="delete"></a>Delete

```swift
func deleteEmployee(employee: Employee) {
    let context = persistentContainer.viewContext
    context.delete(employee)

    do {
        try context.save()
    } catch let error {
        print("Failed to delete: \(error)")
    }
}
```

## <a id="batch-delete"></a>Batch Delete

Calling `delete(_:)` for each individual `NSManagedObject` may take time if there are many objects to delete. A faster approach to batch delete objects in Core Data is `NSBatchDeleteRequest`. Use batch processes to manage large data changes.

Some important considerations when using `NSBatchDeleteRequest` are:

1. `NSBatchDeleteRequest` directly modifies the `NSPersistentStore` without loading any data into memory.

2. An `NSBatchDeleteRequest` does not modify an `NSManagedObjectContext`. Use `mergeChanges(fromRemoteContextSave:, into:)` as shown in the example if needed to inform a `NSManagedObjectContext` of the batch delete.

```swift
// Specify a batch to delete with a fetch request
let fetchRequest: NSFetchRequest<NSFetchRequestResult>
fetchRequest = NSFetchRequest(entityName: "Business")

// Create a batch delete request for the
// fetch request
let deleteRequest = NSBatchDeleteRequest(
    fetchRequest: fetchRequest
)

// Specify the result of the NSBatchDeleteRequest
// should be the NSManagedObject IDs for the
// deleted objects
deleteRequest.resultType = .resultTypeObjectIDs

// Get a reference to a managed object context
let context = persistentContainer.viewContext

// Perform the batch delete
let batchDelete = try context.execute(deleteRequest)
    as? NSBatchDeleteResult

guard let deleteResult = batchDelete?.result
    as? [NSManagedObjectID]
    else { return }

let deletedObjects: [AnyHashable: Any] = [
    NSDeletedObjectsKey: deleteResult
]

// Merge the delete changes into the managed
// object context
NSManagedObjectContext.mergeChanges(
    fromRemoteContextSave: deletedObjects,
    into: [context]
)
```

## <a id="delete-everything"></a>Delete Everything (Delete All Objects, Reset Core Data)

One approach to delete everything and reset Core Data is to destroy the persistent store. Deleting and re-creating the persistent store will delete all objects in Core Data.

An important consideration when deleting everything in Core Data, make sure to release and re-initialize existing `NSManagedObjectContext` references. Using a `NSManagedObjectContext` backed by a deleted persistent store may have unexpected behavior.

```swift
// Get a reference to a NSPersistentStoreCoordinator
let storeContainer =
    persistentContainer.persistentStoreCoordinator

// Delete each existing persistent store
for store in storeContainer.persistentStores {
    try storeContainer.destroyPersistentStore(
        at: store.url!,
        ofType: store.type,
        options: nil
    )
}

// Re-create the persistent container
persistentContainer = NSPersistentContainer(
    name: "CoreDataModelFile" // the name of
    // a .xcdatamodeld file
)

// Calling loadPersistentStores will re-create the
// persistent stores
persistentContainer.loadPersistentStores {
    (store, error) in
    // Handle errors
}
```

## <a id="delete-rule"></a>Delete Rule

A delete rule defines what happens when the record that **owns** the relationship is deleted.

{% include image.html
img="assets/2019-03-07/image1.png"
max-width="700px" %}

Core Data supports 4 delete rules: `Cascade`, `Deny`, `Nullify`, `No Action`. By default, the delete rule of a relationship is set to `nullify`.

{% include image.html
img="assets/2019-03-07/image2.png"
max-width="700px" %}

### No Action

If the delete rule is set to No Action to the relationship then nothing happens. Let's take an example, If department has many employees, If the department is deleted, Nothing happens to an employee. Employee assumes that its still asscoiated with deleted deparment.

### Nullify

If the delete rule is set to Nullify to the relationship then the destination of the relationship gets nullify. In our case, If the department has many employees and department is deleted, the relationship between department and employee gets nullify.

This is default delete rule and we often use this rule in our project.

### Cascade

This delete rule is important and makes sure for all possible scenarios while setting this rule to the realtionship. This rule is mostly used when data model has more dependency. Means in our case, If department has many employees and if department is deleted, automatically all employee belonging to that department are deleted.

Before using this rule please make sure your project requirement. Because this rule will delete all your records without any intimation if your relationship object is deleted.

### Deny

This rule is powerful and simply opposite to the cascade rule. Instead of deletion of all your records in cascade rule, it prevents the deletion of records.

In our case, If department has many employees, department is only deleted if all employee belonging to same department are deleted or no employee is tied with the same department.

## <a id="transactions"></a>Transactions

Using SQL databases you have transactions. Using Core Data you can rely on `NSManagedObjectContext` and built-in `UndoManager`. There is a set of simple methods to manage current state:

- `save()` – similar to `commit` – persists all changes and resets undo stack.

- `rollback()` – discards all changes since last `save()`.

- `undo()` – reverts last change.

- `redo()` – reverts last undo action.

# <a id="core-data-concurrency"></a>Core Data Concurrency

Core Data has built-in interfaces for managing operations across different threads. The most important things to keep in mind when supporting Core Data concurrency are:

1. A `NSManagedObjectContext` must perform operations on the thread it was created

2. The `NSManagedObjectContext` `perform(_:)` and `performAndWait(_:)` functions are thread safe and can be called from other threads

3. A `NSManagedObject` cannot be shared across threads

4. A `NSManagedObjectID` can be shared across threads

5. Changes from one `NSManagedObjectContext` can be merged into another `NSManagedObjectContext`, even on another thread

**In general, avoid doing data processing on the main queue that is not user-related.** Data processing can be CPU-intensive, and if it is performed on the main queue, it can result in unresponsiveness in the user interface. If your application will be processing data, like importing data into Core Data from JSON, create a private queue context and perform the import on the private context.

**Do not pass NSManagedObject instances between queues.** Doing so can result in corruption of the data and termination of the app. When it is necessary to hand off a managed object reference from one queue to another, use `NSManagedObjectID` instances.

You retrieve the managed object ID of a managed object by calling the objectID accessor on the `NSManagedObject` instance.

## <a id="is-core-data-thread-safe"></a>Is Core Data Thread Safe?

Only some objects and functions in Core Data are thread safe. For example, `NSManagedObject` is not thread safe. However, `NSManagedObjectID` is thread safe. One way to get access to the same `NSManagedObject` across threads is to pass the `NSManagedObjectID` across threads and read the object:

```swift
let objectID = // a NSManagedObjectID

// Read the object using a managed object context
try context.existingObject(with: objectID)
```

Note: newly created Core Data objects have temporary object ids until the object is saved for the first time. Temporary object ids cannot be used to read objects on other threads.

Another example, `NSManagedObjectContext` contains both thread safe and not thread safe functions. For example, `save()` and `fetch(_:)` are not thread safe. However, `perform(_:)` and `performAndWait(_:)` are thread safe.

## <a id="main-queue-concurrency-type"></a>.mainQueueConcurrencyType

The `.mainQueueConcurrencyType` concurrency type is used for a `NSManagedObjectContext` on the main thread. A `NSPersistentContainer` has a property, `viewContext`, that is a reference to a `NSManagedObjectContext` with a `.mainQueueConcurrencyType` concurrency type:

```swift
// Get the main thread NSManagedObjectContext
// from a NSPersistentContainer
let context = persistentContainer.viewContext
```

If needed, a `NSManagedObjectContext` with a `.mainQueueConcurrencyType` concurrency type can be created directly:

```swift
// Use NSManagedObjectContext to directly create
// a main thread context
let context = NSManagedObjectContext(
    concurrencyType: .mainQueueConcurrencyType
)

// Set the persistent store coordinator so the
// managed object context knows where to save
// to and read from
context.persistentStoreCoordinator = // NSPersistentStoreCoordinator
```

## <a id="private-queue-concurrency-type"></a>.privateQueueConcurrencyType

A background `NSManagedObjectContext` has a concurrency type of `.privateQueueConcurrencyType`. Two common approaches to create a background `NSManagedObjectContext` with a `.privateQueueConcurrencyType are using a `NSPersistentContainer`and directly with`NSManagedObjectContext`:

```swift
// Use newBackgroundContext() with a
// NSPersistentContainer to create a
// background NSManagedObjectContext
let context = persistentContainer.newBackgroundContext()

// If needed, ensure the background context
// stays up to date with changes from
// the parent
context.automaticallyMergesChangesFromParent = true
```

```swift
// Use NSManagedObjectContext to directly
// create a background context
let context = NSManagedObjectContext(
    concurrencyType: .privateQueueConcurrencyType
)

// Set the parent context of the background context
context.parent = // NSManagedObjectContext

// If needed, ensure the background context
// stays up to date with changes from
// the parent
context.automaticallyMergesChangesFromParent = true
```

## <a id="context-perform"></a>NSManagedObjectContext.perform

The `perform(_:)` function is thread safe and will run the provided code block asynchronously on the thread the `NSManagedObjectContext` was created on.

```swift
// Run the block asynchronously on the thread
// the context was created on
context.perform {
    // Perform background Core Data operations
    // ...
}

// Code after perform(_:) may run before the
// provided code block
```

## <a id="context-performAndWait"></a>NSManagedObjectContext.performAndWait

Like `perform(_:)`, `performAndWait(_:)` is thread safe and will run the provided code block on the thread the `NSManagedObjectContext` was created on. Unlike `perform(_:)`, `performAndWait(_:)` will run the provided code block synchronously and wait for the block to complete:

```swift
// Run the block synchronously on the thread
// the context was created on
context.performAndWait {
    // Perform background Core Data operations
    // ...
}

// Code after performAndWait(_:) will run
// after the provided code block
```

## <a id="core-data-background-context"></a>Core Data Background Context

There are multiple ways to use Core Data and a `NSManagedObjectContext` on a background thread. One way is the `performBackgroundTask(_:)` method on an `NSPersistentContainer`:

```swift
// Spawns a new thread and creates a background
// managed object context to perform operations
// in the background
persistentContainer.performBackgroundTask {
    (context) in
    // Perform background Core Data operations
    // ...
}
```

A drawback of `performBackgroundTask(_:)` is that `performBackgroundTask(_:)` creates a new thread. Calling `performBackgroundTask(_:)` repeatedly may negatively impact performance. Instead, `newBackgroundContext()` on a `NSPersistentContainer` can be used to create a background `NSManagedObjectContext`:

```swift
// Create a new background managed object context
let context = persistentContainer.newBackgroundContext()

// If needed, ensure the background context stays
// up to date with changes from the parent
context.automaticallyMergesChangesFromParent = true
```

In some cases a `NSPersistentContainer may not be available. A background `NSManagedObjectContext` can also be created explicitly:

```swift
// Create a new background context
let context = NSManagedObjectContext(
    concurrencyType: .privateQueueConcurrencyType
)

// Set the parent context
context.parent = // NSManagedObjectContext

// If needed, ensure the background context stays
// up to date with changes from the parent
context.automaticallyMergesChangesFromParent = true
```

## <a id="core-data-background-fetch-request"></a>Core Data Background Fetch Request

To get objects from a fetch request on a background thread, use `perform(_:)` or `performAndWait(_:)` on a background `NSManagedObjectContext`:

```swift
// Create a new background managed object context
let context = persistentContainer.newBackgroundContext()

// If needed, ensure the background context stays
// up to date with changes from the parent
context.automaticallyMergesChangesFromParent = true

// Perform operations on the background context
// asynchronously
context.perform {
    do {
        // Create a fetch request
        let fetchRequest: NSFetchRequest<CustomEntity>

        fetchRequest = CustomEntity.fetchRequest()
        fetchRequest.fetchLimit = 1

        let objects = try context.fetch(fetchRequest)

        // Handle fetched objects
    }
    catch let error {
        // Handle error
    }
}
```

## <a id="save-core-data-on-a-background-thread"></a>Save Core Data On A Background Thread

Use `perform(_:)` or `performAndWait(_:)` to ensure `save()` is called on the background thread a managed object context was created on:

```swift
// Create a new background managed object context
let context = persistentContainer.newBackgroundContext()

// If needed, ensure the background context stays
// up to date with changes from the parent
context.automaticallyMergesChangesFromParent = true

// Perform operations on the background context
// asynchronously
context.perform {
    do {
        // Modify the background context
        // ...

        // Save the background context
        try context.save()
    }
    catch let error {
        // Handle error
    }
}
```

## <a id="create-a-new-core-data-object-on-a-background-thread"></a>Create A New Core Data Object On A Background Thread

To create a new core data object in the background, use `perform(_:)` or `performAndWait(_:)` on a background `NSManagedObjectContext`:

```swift
// Create a new background managed object context
let context = persistentContainer.newBackgroundContext()

// If needed, ensure the background context stays
// up to date with changes from the parent
context.automaticallyMergesChangesFromParent = true

// Perform operations on the background context
// asynchronously
context.perform {
    do {
        // Create A New Object
        let object = CustomEntity(context: context)

        // Configure object properties
        // ...

        // Save the background context
        try context.save()
    }
    catch let error {
        // Handle errors
    }
}
```

# <a id="parent-child-moc"></a>Parent/Child Managed Object Context

{% include image.html
img="assets/2019-03-07/image4.png" %}

## Performance Using Main Context

{% include image.html
img="assets/2019-03-07/image5.png" %}

## Improve Performance Using Concurrency

{% include image.html
img="assets/2019-03-07/image6.png" %}

# <a id="fetch-requests"></a>Fetch Requests

## <a id="get-object-by-id"></a>Get Object By ID

```swift
// Get the managed object ID of the object
let managedObject = // ... an NSManagedObject
let managedObjectID = managedObject.objectID

// Get a reference to a NSManagedObjectContext
let context = persistentContainer.viewContext

// Get the object by ID from the NSManagedObjectContext
let object = try context.existingObject(
    with: managedObjectID
)
```

## <a id="fetch-a-single-object"></a>Fetch A Single Object

```swift
// Configure a fetch request to fetch at most 1 result
let fetchRequest = // An NSFetchRequest
fetchRequest.fetchLimit = 1

// Get a reference to a NSManagedObjectContext
let context = persistentContainer.viewContext

// Fetch a single object. If the object does not exist,
// nil is returned
let object = try context.fetch(fetchRequest).first
```

## <a id="filter-fetch-request-with-predicate"></a>Filter Fetch Request With Predicate

```swift
// Create a fetch request with a string filter
// for an entity’s name
let fetchRequest: NSFetchRequest<Entity>
fetchRequest = Entity.fetchRequest()

fetchRequest.predicate = NSPredicate(
    format: "name LIKE %@", "Robert"
)

// Get a reference to a NSManagedObjectContext
let context = persistentContainer.viewContext

// Perform the fetch request to get the objects
// matching the predicate
let objects = try context.fetch(fetchRequest)
```

## <a id="filter-predicate-with-multiple-conditions"></a>Filter Predicate With Multiple Conditions

```swift
// Create a fetch request with a compound predicate
let fetchRequest: NSFetchRequest<Entity>
fetchRequest = Entity.fetchRequest()

// Create the component predicates
let namePredicate = NSPredicate(
    format: "name LIKE %@", "Robert"
)

let planetPredicate = NSPredicate(
    format: "country = %@", "Earth"
)

// Create an "and" compound predicate, meaning the
// query requires all the predicates to be satisfied.
// In other words, for an object to be returned by
// an "and" compound predicate, all the component
// predicates must be true for the object.
fetchRequest.predicate = NSCompoundPredicate(
    andPredicateWithSubpredicates: [
        namePredicate,
        planetPredicate
    ]
)

// Get a reference to a NSManagedObjectContext
let context = persistentContainer.viewContext

// Perform the fetch request to get the objects
// matching the compound predicate
let objects = try context.fetch(fetchRequest)
```

## <a id="fetch-all-objects-of-one-entity"></a>Fetch All Objects Of One Entity

```swift
// Create a fetch request for a specific Entity type
let fetchRequest: NSFetchRequest<Entity>
fetchRequest = Entity.fetchRequest()

// Get a reference to a NSManagedObjectContext
let context = persistentContainer.viewContext

// Fetch all objects of one Entity type
let objects = try context.fetch(fetchRequest)
```

# <a id="queries-using-nspredicate"></a>Queries Using NSPredicate

With `LIKE`, `CONTAINS`, `MATCHES`, `BEGINSWITH`, and `ENDSWITH` you can perform a wide array of queries in Core Data with String arguments.

The examples below will query a Core Data database with a single entity description, `Person`. `Person` has a single attribute `name`, a `String`.

## <a id="exact-string-match-with-a-nspredicate"></a>Exact String Match With A NSPredicate

The `LIKE` keyword in an `NSPredicate` will perform an exact string match.

```swift
let query = "Rob"
let request: NSFetchRequest<Person> = Person.fetchRequest()
request.predicate = NSPredicate(format: "name LIKE %@", query)

// The == syntax may also be used to search for an exact match
request.predicate = NSPredicate(format: "name == %@", query)
```

Additionally, the `c` and `d` modifiers can be used to perform a case-insensitive and diacritic insensitive (c would match ç as well) search:

```swift
let query = "Rob"
let request: NSFetchRequest<Person> = Person.fetchRequest()
request.predicate = NSPredicate(format: "name LIKE[cd] %@", query)
```

## <a id="contains-string-with-a-nspredicate"></a>Contains String With A NSPredicate

The `CONTAINS` keyword in an `NSPredicate` will perform a contains string match.

```swift
let query = "Rob"
let request: NSFetchRequest<Person> = Person.fetchRequest()
request.predicate = NSPredicate(format: "name CONTAINS %@", query)
```

The `c` and `d` modifiers can also be used with `CONTAINS` to perform a case-insensitive. and diacritic insensitive (c would match ç as well) search.

## <a id="nspredicate-regex-search"></a>NSPredicate Regex Search

The `MATCHES` keyword in an `NSPredicate` is used to perform a regular expression search.

```swift
let query = "??bert*"
let request: NSFetchRequest<Person> = Person.fetchRequest()
request.predicate = NSPredicate(format: "name MATCHES %@", query)
```

## <a id="search-for-matches-that-begin-with-string"></a>Search For Matches That Begin With String

The `BEGINSWITH` keyboard in an `NSPredicate` is used to perform a search for strings that begin with a specific substring.

```swift
let query = "R"
let request: NSFetchRequest<Person> = Person.fetchRequest()
request.predicate = NSPredicate(format: "name BEGINSWITH %@", query)
```

## <a id="search-for-matches-that-end-with-string"></a>Search For Matches That End With String

The `ENDSWITH` keyboard in an `NSPredicate` is used to perform a search for strings that end with a specific substring.

```swift
let query = "R"
let request: NSFetchRequest<Person> = Person.fetchRequest()
request.predicate = NSPredicate(format: "name ENDSWITH %@", query)
```

# <a id="migrations"></a>Migrations

When is a migration necessary? The easiest answer to this common question is “when you need to make changes to the data model.”

To start the migration process, Core Data needs the original data model and the destination model. It uses these two versions to load or create a mapping model for the migration, which it uses to convert data in the original store to data that it can store in the new store. Once Core Data determines the mapping model, the migration process can start in earnest.

Migrations happen in three steps:

1. First, Core Data copies over all the objects from one data store to the next.

2. Next, Core Data connects and relates all the objects according to the relationship mapping.

3. Finally, enforce any data validations in the destination model. Core Data disables destination model validations during the data copy.

Migrations can be handled using one of two techniques:

1. **Lightweight Migration** - when Core Data can automatically infer how the migration should happen and creates the mapping model on the fly.

2. **Heavyweight Migration** - when Core Data cannot infer how the migration should happen and so we must write a custom migration by providing a mapping model (`xcmappingmodel`) and/or a migration policy (`NSEntityMigrationPolicy`).

## <a id="lightweight"></a>Lightweight migration (automatic)

If changes in the new schema are not very complex then you can simply use lightweight migration. Core data will automatically infer the mapping model and perform the data migration.

You can use lightweight migration, when

- New `entity/attribute/relationship` is **added**

- Any `entity/attribute/relationship` is **removed**

- The name or type of any `entity/attribute/relationship` is **changed**

### Managing Changes to Entities and Properties

If you rename an entity or property, you can set the renaming identifier in the destination model to the name of the corresponding property or entity in the source model. Use the Xcode Data Modeling tool’s property inspector (for either an entity or a property) to set the renaming identifier in the managed object model. For example, you can:

- Rename a Car entity to Automobile

- Rename a Car’s color attribute to paintColor

The renaming identifier creates a canonical name, so set the renaming identifier to the name of the property in the source model (unless that property already has a renaming identifier). This means you can rename a property in version 2 of a model, then rename it again in version 3. The renaming will work correctly going from version 2 to version 3, or from version 1 to version 3.

### Managing Changes to Relationships

Lightweight migration can also manage changes to relationships and to the type of relationship. You can add a new relationship or delete an existing relationship. You can also rename a relationship by using a renaming identifier, just like an attribute.

In addition, you can change a relationship from a to-one to a to-many, or a nonordered to-many to an ordered (and vice versa).

### Managing Changes to Hierarchies

You can add, remove, and rename entities in the hierarchy. You can also create a new parent or child entity and move properties up and down the entity hierarchy. You can move entities out of a hierarchy. You cannot, however, merge entity hierarchies; if two existing entities do not share a common parent in the source, they cannot share a common parent in the destination.

### Requesting Lightweight Migration

To perform lightweight migration, all you have to do is, set property `shouldMigrateStoreAutomatically` and `shouldInferMappingModelAutomatically` **true**.

```swift
let description = container.persistentStoreDescriptions.first
description?.type = NSSQLiteStoreType
description?.shouldMigrateStoreAutomatically = true
description?.shouldInferMappingModelAutomatically = true
```

### Step by Step

1. Select your Data Model.

2. Open `Editor` menu.

3. Select `Add Model version`...

4. Select in `Navigator` (left panel) your new version.

5. Perform desired changes – add/remove attributes/entities.

Once the new version is ready, you can migrate your Data Model to the latest version:

{% include image.html
img="assets/2019-03-07/image7.png" %}

1. Select any version of Data Model.

2. Select any Entity (just to get a focus).

3. Select `File inspector`.

4. Set desired Model version.

## <a id="heavyweight"></a>Heavyweight migration

This migration is required when you need to perform custom mapping which can’t be handled automatically by Core Data. For example you may want to change attribute type from `String` to `Integer 16` or divide value by 2.0.

> Heavyweight migration is tricky and it’s very easy to make a mistake, so pay attention to details in all steps.

### Step 1 - Add Model Version

{% include image.html
img="assets/2019-03-07/image8.png" %}

Example, we decide to expand our posting functionality by allowing the user to add multiple `sections` to a `post`.

Each `section` consists of:

- A title.
- A body.
- An index.

which in turn reduces a `post` to:

- A unique ID.
- A random associated colour represented as a hex string.
- The date the post was created on.
- A collection of sections.

Such that:

{% include image.html
img="assets/2019-03-07/image9.png" %}

And:

{% include image.html
img="assets/2019-03-07/image10.png" %}

### Step 2 - Add Mapping Model

Create `Mapping Model`. This will open a wizard where you can select your source and destination model versions so in this case: `CoreDataMigration_Example 2` and `CoreDataMigration_Example 3`.

{% include image.html
img="assets/2019-03-07/image11.png" %}

### Step 3 - Custom NSEntityMigrationPolicy

Create a new class `CustomMigration` which inherits from `NSEntityMigrationPolicy`.

```swift
import CoreData

final class Post2ToPost3MigrationPolicy: NSEntityMigrationPolicy {

    override func createDestinationInstances(forSource sInstance: NSManagedObject, in mapping: NSEntityMapping, manager: NSMigrationManager) throws {
        try super.createDestinationInstances(forSource: sInstance, in: mapping, manager: manager)

        guard let destinationPost = manager.destinationInstances(forEntityMappingName: mapping.name, sourceInstances: [sInstance]).first else {
            fatalError("was expected a post")
        }

        let sourceBody = sInstance.value(forKey: "content") as? String
        let sourceTitle = sourceBody?.prefix(4).appending("...")

        let section = NSEntityDescription.insertNewObject(forEntityName: "Section", into: destinationPost.managedObjectContext!)
        section.setValue(sourceTitle, forKey: "title")
        section.setValue(sourceBody, forKey: "body")
        section.setValue(destinationPost, forKey: "post")
        section.setValue(0, forKey: "index")

        var sections = Set<NSManagedObject>()
        sections.insert(section)

        destinationPost.setValue(sections, forKey: "sections")
    }
}
```

In the right panel of `Mapping Model`, set `Custom Policy` on the `PostToPost` to `CoreDataMigration_Example.Post2ToPost3MigrationPolicy`, notice that it’s important to specify module here.

So now we are all done with the heavyweight migration thing. let’s run the app and perform data migration.

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
