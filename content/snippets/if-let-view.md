---
title: 'IfLet View'
date: 2024-08-24T17:11:25+07:00
draft: false
summary: Creating a view that unwraps an optional value intended to be consumed by its child
description: Creating a view that unwraps an optional value intended to be consumed by its child
params:
  ShowReadingTime: false
  ShowBreadCrumbs: false
  ShowWordCount: false
---

## The Scenario

Tired of using `if let` to unwrap optional value inside a SwiftUI View that doesn’t feel natural?

```swift
struct ContentView: View {
    @Binding var error: String?

    var body: some View {
      ...
      if let text {
        Text(text)
      }
    }
}
```

## The Solution

Create a generic view that wraps around `if let`

```swift
struct IfLet<Data, Content>: View where Content: View {
    let data: Data?
    let content: (Data) -> Content

    init(
        data: Data?,
        @ViewBuilder content: @escaping (Data) -> Content
    ) {
        self.data = data
        self.content = content
    }

    var body: some View {
        if let data {
            content(data)
        }
    }
}
```

## The Usage

```swift
struct ContentView: View {
    @Binding var error: String?

    var body: some View {
      ...
      IfLet(data: error) {
        Text($0)
      }
    }
}
```
