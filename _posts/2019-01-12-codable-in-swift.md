---
layout: post
title: "Codable in Swift"
author: "Linh Vo"
tags: "Networking"
---

In this article I want to provide quick code samples to help you solve answer common questions and solve common problems, all using `Codable`.

# Table of contents

- [Nested types](#nested-types)
- [Snake case vs camel case](#snake-case-vs-camel-case)
- [Custom coding keys](#custom-coding-keys)
- [Keyed containers](#keyed-containers)
- [Nested keyed containers](#nested-keyed-containers)
- [Dates](#dates)
- [Subclasses](#subclasses)
- [Polymorphic types](#polymorphic-types)
- [Unkeyed containers](#unkeyed-containers)
- [Nested unkeyed containers](#nested-unkeyed-containers)
- [Dynamic Keys](#dynamic-keys)
- [Enum](#enum)
- [Nested structure](#nested-structure)
- [quicktype](#quicktype)
- [Codable cheat sheet](#codable-cheat-sheet)

# Nested types

```swift
let jsonData = """
{
  "name" : "John Appleseed",
  "id" : 7,
  "favoriteToy" : {
    "name" : "Teddy Bear"
  }
}
""".data(using: .utf8)!

struct Toy: Codable {
  var name: String
}

struct Employee: Codable {
  var name: String
  var id: Int
  var favoriteToy: Toy
}

let toy = Toy(name: "Teddy Bear")
let employee = Employee(name: "John Appleseed", id: 7, favoriteToy: toy)

let encoder = JSONEncoder()
let decoder = JSONDecoder()

let dataEncoder = try encoder.encode(employee)
let string = String(data: dataEncoder, encoding: .utf8)!
print(string)

let sameEmployee = try decoder.decode(Employee.self, from: jsonData)
print(sameEmployee)
print(sameEmployee.favoriteToy)
```

# Snake case vs camel case

```swift
let jsonData = """
{
  "name" : "John Appleseed",
  "id" : 7,
  "favorite_toy" : {
    "name" : "Teddy Bear"
  }
}
""".data(using: .utf8)!

struct Toy: Codable {
  var name: String
}

struct Employee: Codable {
  var name: String
  var id: Int
  var favoriteToy: Toy
}

let toy = Toy(name: "Teddy Bear")
let employee = Employee(name: "John Appleseed", id: 7, favoriteToy: toy)

let encoder = JSONEncoder()
let decoder = JSONDecoder()

encoder.keyEncodingStrategy = .convertToSnakeCase
let snakeData = try encoder.encode(employee)
let snakeString = String(data: snakeData, encoding: .utf8)!
print(snakeString)

decoder.keyDecodingStrategy = .convertFromSnakeCase
let camelEmployee = try decoder.decode(Employee.self, from: jsonData)
print(camelEmployee)
print(camelEmployee.favoriteToy)
```

# Custom coding keys

```swift
let jsonData = """
{
  "name" : "John Appleseed",
  "id" : 7,
  "gift" : {
    "name" : "Teddy Bear"
  }
}
""".data(using: .utf8)!

struct Toy: Codable {
  var name: String
}

struct Employee: Codable {
  var name: String
  var id: Int
  var favoriteToy: Toy

  enum CodingKeys: String, CodingKey {
    case name, id, favoriteToy = "gift"
  }
}

let toy = Toy(name: "Teddy Bear")
let employee = Employee(name: "John Appleseed", id: 7, favoriteToy: toy)

let encoder = JSONEncoder()
let decoder = JSONDecoder()

let data = try encoder.encode(employee)
let string = String(data: data, encoding: .utf8)!
print(string)

let sameEmployee = try decoder.decode(Employee.self, from: jsonData)
print(sameEmployee)
```

# Keyed containers

```swift
let jsonData = """
{
  "name" : "John Appleseed",
  "id" : 7,
  "gift" : "Teddy Bear"
}
""".data(using: .utf8)!

struct Toy: Codable {
  var name: String
}

struct Employee: Encodable {
  var name: String
  var id: Int
  var favoriteToy: Toy

  enum CodingKeys: CodingKey {
    case name, id, gift
  }

  func encode(to encoder: Encoder) throws {
    var container = encoder.container(keyedBy: CodingKeys.self)
    try container.encode(name, forKey: .name)
    try container.encode(id, forKey: .id)
    try container.encode(favoriteToy.name, forKey: .gift)
  }
}

extension Employee: Decodable {
  init(from decoder: Decoder) throws {
    let container = try decoder.container(keyedBy: CodingKeys.self)
    name = try container.decode(String.self, forKey: .name)
    id = try container.decode(Int.self, forKey: .id)
    let gift = try container.decode(String.self, forKey: .gift)
    favoriteToy = Toy(name: gift)
  }
}

let toy = Toy(name: "Teddy Bear")
let employee = Employee(name: "John Appleseed", id: 7, favoriteToy: toy)

let encoder = JSONEncoder()
let decoder = JSONDecoder()

let data = try encoder.encode(employee)
let string = String(data: data, encoding: .utf8)!
print(string)

let sameEmployee = try decoder.decode(Employee.self, from: jsonData)
print(sameEmployee)
```

# Nested keyed containers

```swift
let jsonData = """
{
  "name" : "John Appleseed",
  "id" : 7,
  "gift" : {
    "toy" : {
      "name" : "Teddy Bear"
    }
  }
}
""".data(using: .utf8)!

struct Toy: Codable {
  var name: String
}

struct Employee: Encodable {
  var name: String
  var id: Int
  var favoriteToy: Toy

  enum CodingKeys: CodingKey {
    case name, id, gift
  }

  enum GiftKeys: CodingKey {
    case toy
  }

  func encode(to encoder: Encoder) throws {
    var container = encoder.container(keyedBy: CodingKeys.self)
    try container.encode(name, forKey: .name)
    try container.encode(id, forKey: .id)
    var giftContainer = container.nestedContainer(keyedBy: GiftKeys.self, forKey: .gift)
    try giftContainer.encode(favoriteToy, forKey: .toy)
  }
}

extension Employee: Decodable {
  init(from decoder: Decoder) throws {
    let container = try decoder.container(keyedBy: CodingKeys.self)
    name = try container.decode(String.self, forKey: .name)
    id = try container.decode(Int.self, forKey: .id)
    let giftContainer =  try container.nestedContainer(keyedBy: GiftKeys.self, forKey: .gift)
    favoriteToy = try giftContainer.decode(Toy.self, forKey: .toy)
  }
}

let toy = Toy(name: "Teddy Bear")
let employee = Employee(name: "John Appleseed", id: 7, favoriteToy: toy)

let encoder = JSONEncoder()
let decoder = JSONDecoder()

let nestedData = try encoder.encode(employee)
let nestedString = String(data: nestedData, encoding: .utf8)!
print(nestedString)

let sameEmployee = try decoder.decode(Employee.self, from: jsonData)
print(sameEmployee)
```

# Dates

```swift
let jsonData = """
{
  "id" : 7,
  "name" : "John Appleseed",
  "birthday" : "29-05-2019",
  "toy" : {
    "name" : "Teddy Bear"
  }
}
""".data(using: .utf8)!

struct Toy: Codable {
  var name: String
}

let encoder = JSONEncoder()
let decoder = JSONDecoder()

struct Employee: Codable {
  var name: String
  var id: Int
  var birthday: Date
  var toy: Toy
}

let toy = Toy(name: "Teddy Bear")
let employee = Employee(name: "John Appleseed", id: 7, birthday: Date(), toy: toy)

extension DateFormatter {
  static let dateFormatter: DateFormatter = {
    let formatter = DateFormatter()
    formatter.dateFormat = "dd-MM-yyyy"
    return formatter
  }()
}

encoder.dateEncodingStrategy = .formatted(.dateFormatter)
decoder.dateDecodingStrategy = .formatted(.dateFormatter)

let dateData = try encoder.encode(employee)
let dateString = String(data: dateData, encoding: .utf8)!
print(dateString)

let sameEmployee = try decoder.decode(Employee.self, from: jsonData)
print(sameEmployee)
```

# Subclasses

```swift
let jsonData = """
{
  "toy" : {
    "name" : "Teddy Bear"
  },
  "employee" : {
    "name" : "John Appleseed",
    "id" : 7
  },
  "birthday" : 580794178.33482599
}
""".data(using: .utf8)!

struct Toy: Codable {
  var name: String
}

class BasicEmployee: Codable {
  var name: String
  var id: Int

  init(name: String, id: Int) {
    self.name = name
    self.id = id
  }
}

class GiftEmployee: BasicEmployee {
  var birthday: Date
  var toy: Toy

  enum CodingKeys: CodingKey {
    case employee, birthday, toy
  }

  init(name: String, id: Int, birthday: Date, toy: Toy) {
    self.birthday = birthday
    self.toy = toy
    super.init(name: name, id: id)
  }

  required init(from decoder: Decoder) throws {
    let container = try decoder.container(keyedBy: CodingKeys.self)
    birthday = try container.decode(Date.self, forKey: .birthday)
    toy = try container.decode(Toy.self, forKey: .toy)
    let baseDecoder = try container.superDecoder(forKey: .employee)
    try super.init(from: baseDecoder)
  }

  override func encode(to encoder: Encoder) throws {
    var container = encoder.container(keyedBy: CodingKeys.self)
    try container.encode(birthday, forKey: .birthday)
    try container.encode(toy, forKey: .toy)
    let baseEncoder = container.superEncoder(forKey: .employee)
    try super.encode(to: baseEncoder)
  }
}

let toy = Toy(name: "Teddy Bear")
let encoder = JSONEncoder()
let decoder = JSONDecoder()

let giftEmployee = GiftEmployee(name: "John Appleseed", id: 7, birthday: Date(), toy: toy)
let giftData = try encoder.encode(giftEmployee)
let giftString = String(data: giftData, encoding: .utf8)!

let sameGiftEmployee = try decoder.decode(GiftEmployee.self, from: jsonData)
print(sameGiftEmployee.birthday)
print(sameGiftEmployee.toy)
print(sameGiftEmployee.id)
print(sameGiftEmployee.name)
```

# Polymorphic types

```swift
let jsonData = """
[
  {
    "name" : "John Appleseed",
    "id" : 7
  },
  {
    "id" : 7,
    "name" : "John Appleseed",
    "birthday" : 580797832.94787002,
    "toy" : {
      "name" : "Teddy Bear"
    }
  }
]
""".data(using: .utf8)!

struct Toy: Codable {
  var name: String
}
let toy = Toy(name: "Teddy Bear")
let encoder = JSONEncoder()
let decoder = JSONDecoder()

enum AnyEmployee: Encodable {
  case defaultEmployee(String, Int)
  case customEmployee(String, Int, Date, Toy)
  case noEmployee

  enum CodingKeys: CodingKey {
    case name, id, birthday, toy
  }

  func encode(to encoder: Encoder) throws {
    var container = encoder.container(keyedBy: CodingKeys.self)

    switch self {
      case .defaultEmployee(let name, let id):
        try container.encode(name, forKey: .name)
        try container.encode(id, forKey: .id)
      case .customEmployee(let name, let id, let birthday, let toy):
        try container.encode(name, forKey: .name)
        try container.encode(id, forKey: .id)
        try container.encode(birthday, forKey: .birthday)
        try container.encode(toy, forKey: .toy)
      case .noEmployee:
        let context = EncodingError.Context(codingPath: encoder.codingPath, debugDescription: "Invalid employee!")
        throw EncodingError.invalidValue(self, context)
    }
  }
}

extension AnyEmployee: Decodable {
  init(from decoder: Decoder) throws {
    let container = try decoder.container(keyedBy: CodingKeys.self)
    let containerKeys = Set(container.allKeys)
    let defaultKeys = Set<CodingKeys>([.name, .id])
    let customKeys = Set<CodingKeys>([.name, .id, .birthday, .toy])

    guard containerKeys == defaultKeys || containerKeys == customKeys else {
      let context = DecodingError.Context(codingPath: decoder.codingPath, debugDescription: "Not enough keys!")
      throw DecodingError.dataCorrupted(context)
    }

    switch containerKeys {
      case defaultKeys:
        let name = try container.decode(String.self, forKey: .name)
        let id = try container.decode(Int.self, forKey: .id)
        self = .defaultEmployee(name, id)
      case customKeys:
        let name = try container.decode(String.self, forKey: .name)
        let id = try container.decode(Int.self, forKey: .id)
        let birthday = try container.decode(Date.self, forKey: .birthday)
        let toy = try container.decode(Toy.self, forKey: .toy)
        self = .customEmployee(name, id, birthday, toy)
      default:
        self = .noEmployee
    }
  }
}

let employees = [AnyEmployee.defaultEmployee("John Appleseed", 7), AnyEmployee.customEmployee("John Appleseed", 7, Date(), toy)]
let employeesData = try encoder.encode(employees)
let employeesString = String(data: employeesData, encoding: .utf8)!
print(employeesString)

let sameEmployees = try decoder.decode([AnyEmployee].self, from: jsonData)
print(sameEmployees.count)
sameEmployees.forEach {
    print($0)
}
```

# Unkeyed containers

```swift
let jsonData = """
[
  "teddy bear",
  "TEDDY BEAR",
  "Teddy Bear"
]
""".data(using: .utf8)!

struct Toy: Codable {
  var name: String
}

let toy = Toy(name: "Teddy Bear")

let encoder = JSONEncoder()
let decoder = JSONDecoder()

struct Label: Encodable {
  var toy: Toy

  func encode(to encoder: Encoder) throws {
    var container = encoder.unkeyedContainer()
    try container.encode(toy.name.lowercased())
    try container.encode(toy.name.uppercased())
    try container.encode(toy.name)
  }
}

extension Label: Decodable {
  init(from decoder: Decoder) throws {
    var container = try decoder.unkeyedContainer()
    var name = ""
    while !container.isAtEnd {
      name = try container.decode(String.self)
    }
    toy = Toy(name: name)
  }
}

let label = Label(toy: toy)
let labelData = try encoder.encode(label)
let labelString = String(data: labelData, encoding: .utf8)!
print(labelString)

let sameLabel = try decoder.decode(Label.self, from: jsonData)
print(sameLabel.toy)
```

# Nested unkeyed containers

```swift
let jsonData = """
{
  "name" : "Teddy Bear",
  "label" : [
    "teddy bear",
    "TEDDY BEAR",
    "Teddy Bear"
  ]
}
""".data(using: .utf8)!

let encoder = JSONEncoder()
let decoder = JSONDecoder()

struct Toy: Encodable {
  var name: String
  var label: String

  enum CodingKeys: CodingKey {
    case name, label
  }

  func encode(to encoder: Encoder) throws {
    var container = encoder.container(keyedBy: CodingKeys.self)
    try container.encode(name, forKey: .name)
    var labelContainer = container.nestedUnkeyedContainer(forKey: .label)
    try labelContainer.encode(label.lowercased())
    try labelContainer.encode(label.uppercased())
    try labelContainer.encode(label)
  }
}

extension Toy: Decodable {
  init(from decoder: Decoder) throws {
    let container = try decoder.container(keyedBy: CodingKeys.self)
    name = try container.decode(String.self, forKey: .name)
    var labelContainer = try container.nestedUnkeyedContainer(forKey: .label)
    var labelName = ""
    while !labelContainer.isAtEnd {
        labelName = try labelContainer.decode(String.self)
    }
    label = labelName
  }
}

let toy = Toy(name: "Teddy Bear", label: "Teddy Bear")
let data = try encoder.encode(toy)
let string = String(data: data, encoding: .utf8)!
print(string)

let sameToy = try decoder.decode(Toy.self, from: jsonData)
print(sameToy)
```

# Dynamic Keys

```swift
let jsonString = """
{
  "S001": {
    "firstName": "Tony",
    "lastName": "Stark"
  },
  "S002": {
    "firstName": "Peter",
    "lastName": "Parker"
  },
  "S003": {
    "firstName": "Bruce",
    "lastName": "Wayne"
  }
}
"""

struct Student: Decodable {
    let firstName: String
    let lastName: String
    let studentId: String

    enum CodingKeys: CodingKey {
        case firstName
        case lastName
    }

    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        firstName = try container.decode(String.self, forKey: CodingKeys.firstName)
        lastName = try container.decode(String.self, forKey: CodingKeys.lastName)
        studentId = container.codingPath.first!.stringValue
    }
}

struct DecodedArray: Decodable {

    var array: [Student]

    // Define DynamicCodingKeys type needed for creating
    // decoding container from JSONDecoder
    private struct DynamicCodingKeys: CodingKey {

        // Use for string-keyed dictionary
        var stringValue: String
        init?(stringValue: String) {
            self.stringValue = stringValue
        }

        // Use for integer-keyed dictionary
        var intValue: Int?
        init?(intValue: Int) {
            // We are not using this, thus just return nil
            return nil
        }
    }

    init(from decoder: Decoder) throws {

        // Create a decoding container using DynamicCodingKeys
        // The container will contain all the JSON first level key
        let container = try decoder.container(keyedBy: DynamicCodingKeys.self)

        var tempArray = [Student]()

        // Loop through each key (student ID) in container
        for key in container.allKeys {

            // Decode Student using key & keep decoded Student object in tempArray
            let decodedObject = try container.decode(Student.self, forKey: DynamicCodingKeys(stringValue: key.stringValue)!)
            tempArray.append(decodedObject)
        }

        // Finish decoding all Student objects. Thus assign tempArray to array.
        array = tempArray
    }
}

let jsonData = Data(jsonString.utf8)
let decodedResult = try! JSONDecoder().decode(DecodedArray.self, from: jsonData)
dump(decodedResult.array)
```

# Enum

Let’s say you have json object like this:

```swift
{
    "name": "iOS developer",
    "status": "open"
}
```

You can create Swift struct like this:

```swift
struct Job: Codable {

  enum Status: String, Codable {
    case open
    case close
  }

  let name: String
  let status: Status
}
```

If you have enum which isn't `String` or `Int` representable, you can still make it conform to `Codable` as long as that `raw value` is `Codable`.

# Nested structure

```swift
{
    "name": "John",
    "district": "District",
    "subDistrict": "Sub District",
    "country": "Country",
    "postalCode": "Postal Code"
}
```

And want this Swift structure:

```swift
struct User: Codable {
    var name: String
    var billingAddress: BillingAddress
}

struct BillingAddress: Codable {
    var district: String
    var subDistrict: String
    var country: String
    var postalCode: String
}
```

Following are what we need to implement:

```swift
struct User: Codable {
    ....
    enum CodingKeys: String, CodingKey {
        case name
        case billingAddress
        case district
        case subDistrict
        case country
        case postalCode
    }
}

init(from decoder: Decoder) throws {
    let values = try decoder.container(keyedBy: CodingKeys.self)
    name = try values.decode(String.self, forKey: .name)
    billingAddress = try BillingAddress(from: decoder)
}

func encode(to encoder: Encoder) throws {
    var container = encoder.container(keyedBy: CodingKeys.self)
    try container.encode(name, forKey: .name)
    try container.encode(billingAddress.district, forKey: .district)
    try container.encode(billingAddress.subDistrict, forKey: .subDistrict)
    try container.encode(billingAddress.country, forKey: .country)
    try container.encode(billingAddress.postalCode, forKey: .postalCode)
}
```

# quicktype

[quicktype](https://app.quicktype.io/#l=swift) produces nice types and JSON (de)serializers for many programming languages. It can infer types from JSON but also takes types from JSON Schema, TypeScript, and GraphQL.

# Codable cheat sheet

[codable-cheat-sheet](https://github.com/dlsolution/ios-tips-tricks/blob/master/codable-cheat-sheet/codable-cheat-sheet.md)
