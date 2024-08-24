---
title: 'Unlocking more power from enums with CasePath'
date: 2024-08-11T17:11:25+07:00
draft: false
summary: Adding key path and dynamic lookup support for enum with swift-case-paths
description: Adding key path and dynamic lookup support for enum with swift-case-paths
tags: ["swift-case-paths"]
---

## The problem

Enums, along with classes and structs, play a crucial role in structuring your Swift code, allowing you to define data models and behaviors in a clean, organized, and type-safe manner. However, enums, perhaps Apple’s least favorite child, still haven’t received the attention they deserve. Compared to their siblings, enums lack some great features that structs and classes get for free.

One particular feature that we'll focus on today is [KeyPath](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/expressions/#Key-Path-Expression), a powerful feature that only available for struct and class. Every field on struct or class gets a free key path denoted by a backslash syntax

```swift
struct Employee {
    let name: String
    var age: Int
    var address: Address
}

struct Address {
  var city: String
  var country: String
}

\Employee.name\ // KeyPath<Employee, String>
\Employee.address.city // WritableKeyPath<User, Int>

let employee = Employee(name: "Blob", age: 28, address: Address(city: "Ho Chi Minh", country: "Vietnam"))
employee[keyPath: \.name] // Blob
employee[keyPath: \.address.city] = "Manila"
```

Enums lack the ergonomic ability to drill down and access data. The Swift API provides only two ways to work with enums: `switch` and `case let`, both of which require a lot of boilerplate code just to access the associated data within an enum.

```swift
enum SubscriptionType {
    case pro(renewal: Date)
    case free
}

var subsription: SubscriptionType

...

if case let .pro(renewalDated) = subscription else {
    print(renewalDated)
}

// or

switch subscription {
case let .pro(renewalDated):
    print(renewalDated)
default: 
    break
}
```

The situation becomes even more cumbersome when dealing with nested enums. For example

```swift
enum LoadingStatus {
    case idle
    case loading
    case loaded(SubscriptionType)
    case failure(Error)
}
```

## Introducing CasePath

So, is it possible for enums to have the same features as their siblings? Wouldn't it be nice to do this instead of `switch` or `case let` ?

```swift
subscription[keyPath: \.pro.renewalDate] 
loadingStatus[keyPath: \.loaded.pro.renewalDate]
```

While researching this problem, I came across the [swift-case-paths](https://github.com/pointfreeco/swift-case-paths) package developed by the PointFree team and instantly fell in love with it. I’ve been using it in all my projects ever since. So the PointFree team introduced a new concept called `CasePath`. Simply put, case paths are key paths, but for enums.

All you have to do is to annotate your enum with `@CasePathable` macro. The Swift Syntax will generate the necessary boilerplate that allows you to drill down to an enum’s associated type as if it were a key path.

```swift
@CasePathable
enum SubscriptionType {
    ...
}

@CasePathable
enum LoadingStatus {
    ...
}

var loadingStatus = LoadingStatus.loaded(.pro(renewal: Date(...)))

loadingStatuss[case: \.loaded.pro] // Date?
```

You can also do case matching

```swift
let subscription = SubscriptionType.free
        
subscription.is(\.free) // true
subscription.is(\.pro) // false
```

Or mutate the associated value

```swift
func renewPro() {
    subscription.modify(\.pro) {
        $0.addTimeInterval(365 * 24 * 60 * 60)
    }
}
```

## Struct dot chaining syntax for enum

Recall how easy it is to access the data inside a struct or class using the dot syntax.

```swift
employee.name
employee.address.country
```

CasePath has you covered—you can use `@CasePathable` together with `@dynamicMemberLookup` to enable dot-chaining syntax for enums. How neat! 🙌

```swift
struct User {
    let username: String
    let subscriptionType: SubscriptionType
}
    
@CasePathable
@dynamicMemberLookup
enum SubscriptionType {
    ...
}

let blob = User(username: "Blob", subscriptionType: .free)
let klob = User(username: "Klob", subscriptionType: .pro(renewal: Date(...))

blob.subscriptionType.pro // nil
blob.subscriptionType.pro // Date(...)
```

Moreover, this also enables key path expressions for enum case. Now it can be use in function that accepts key path

```swift
let users: [User] = [...]
let renewalDates = users.compactMap(\.subscriptionType.pro) // [Date]
let renewalDates = users.map(\.subscriptionType.pro) // [Date?]
```

## Final words

CasePath unlocks even more power from enums. The PointFree team has been leveraging CasePath in their other open-source projects, such as [Composable Architecture](http://github.com/pointfreeco/swift-composable-architecture) and [SwiftUI Navigation](http://github.com/pointfreeco/swiftui-navigation). It greatly helps in making our code more readable and ergonomic. This article just scratches the surface. Since I’ve been intensively using CasePath in many of my projects, I’ve created a lot of utilities around it. We’ll discuss this more in a future article.
