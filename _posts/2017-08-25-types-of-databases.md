---
layout: post
title: "Types of Databases"
author: "Linh Vo"
tags: "UIKit"
---

# Types of Databases

When it comes to designing a database for your app, you can choose from 3 types of databases:

- A **relational database** is structured to support relationships between pieces of information. Most relational databases use an SQL-like query language that allows you to filter and combine datasets in one result. Relational databases typically use a spreadsheet-like row-column format to save data. Examples include [MySQL](https://www.mysql.com/), [SQLite](https://www.sqlite.org/), [Realm](https://realm.io/)...

- A **document database** is similar to a relational database, except that it stores loosely formatted documents. This way you don’t have to strictly define your database structure before working with it. They often support relationships between objects, but can’t combine results in a so-called join query. Document-based databases are sometimes called No-SQL databases. Examples include [MongoDB](https://www.mongodb.com/), [Firebase Cloud Firestore](https://firebase.google.com/docs/firestore/)...

- A **graph database** uses graph structures with nodes, edges and properties to represent data. It typically structures objects by the relationships they have with other objects. Think of nodes as people in a group of friends, and relationships as the friendships between them. You can traverse the graph of friendships to quickly find out who knows who. Structuring this kind of data in a graph is more efficient than using a relational database. Examples include [Neo4j](https://neo4j.com/) and [OrientDB]http://orientdb.com/orientdb/...

In this tutorial, we’ll only work with **relational databases**. Much of what you learn about **relational databases** can be applied to other types of app databases, but that doesn’t work the other way around.

# The Schema: Defining Database Objects & Properties

Let’s say we are creating a mobile app database for a social media app like Facebook or Instagram.

We’ll design 3 objects:

1. A `Post` that stores information for a social media post, like some text, a photo, and an array of likes

2. A `User` that stores information for a user, like username, password, and posts the user has created

3. A `Like` that stores information related to a “like” given to a post by a user

When you’re designing your database, simply start by looking through its mockups or UI designs. Every single user interface or app screen typically corresponds to a data object. Which types of data do you see? Start there.

In a social media app we probably have a timeline. The timeline shows posts from users, so that’s how we know we need a `Post` object. Users need to identify themselves, for logging in, and to organise posts. Now we know we need a `User` object. (More about likes, later.)

Some more examples:

- An accounting app needs `Invoice`, `InvoiceItem` and `Customer` objects.

- A sketching app needs `Sketch`, `Collection` and `User` objects.

- A book review app needs `Book`, `Review` and `User` objects.

Once you know the types of objects you want to save in your app’s database, you can define the properties they have. The objects and properties together make up the so-called schema of your app database.

You’re most likely already familiar with the concept of “properties” – they are no different from Swift classes and Object-Oriented Programming.

We can define the properties of a Post object like this:

```swift
class Post {
    var text: String
    var image: URL
    var user: User
    var likes: [Like]
}
```

In the example above, we’ve defined a text property of type `String` and an image property of type `URL`. See how it’s just an ordinary Swift class with properties? Relational database objects are no different.

We’ll get to the `user` and `likes` properties next…

# Defining Relationships Between Objects

In almost any app, database objects interact with each other. Posts are liked, users create posts, and comments are added to a post. When you design your app database these interactions are defined as relationships between objects.

There are 3 types of relationships:

- A **one-to-one** relationship links one object with another object, such as `User` and `AvatarImage`. No user has the same avatar image, so they are linked one-on-one.

- A **one-to-many** relationship links one object to many other objects, such as `Post` and `Like`. A post can have many likes, but one like can only connect to one post.

- A **many-to-many** relationship links many objects to many other objects, such as `Author` and `Book`. An author can write many books, and many books can have the same authors.

Most relationships you define in your apps will be `one-to-many`. Let’s look at an example.

Here is the `Post` object again:

```swift
class Post {
    var likes:[Like]
}
```

And this is the `Like` object:

```swift
class Like {
    var post:Post
}
```

The `Post` object can have multiple `likes`, that’s why the `likes` property has the type “array of likes” or `[Like]`. Conversely, a single like can only be given to one post, evidenced by the property post on the `Like` object.

Interestingly, some many-to-many relationships are in fact just multiple one-to-many relationships! In a social media app, for example, many users can “follow” many other users. This is a many-to-many relationship, but it’s defined as two directional one-to-many relationships in your database. You commonly use a helper table/object to accomplish a many-to-many relation.

When User A follows User B, that User A is added to the followers property of User B. When User B follows back, it’s added to the followers property of User A. Conversely, when User A follows User B, the User A adds User B to its own following property (and vice versa).

By making these two one-to-many relationships, both users know who they are following and who follows them! You could say that a follower relationship is directional, i.e. when I follow you, that doesn’t mean you’re also following me back.

However, when you think of befriending someone it is different: when we’re friends, I am your friend, and you are my friend. This implies a many-to-many relationship.

Exciting, isn’t it? Make sure to keep the difference between one-to-many and many-to-many in mind, as it will save you a lot of headache in the future.

Reference: [https://learnappmaking.com/app-database-design/](https://learnappmaking.com/app-database-design/)
