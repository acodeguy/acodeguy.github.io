---
layout: post
title: Handling errors when parsing JSON in Swift
tags: networking, json
---
## The problem

When parsing JSON the standard way, any errors encountered are very generic and not really that helpful:

```swift
let decoder = try JSONDecoder()
decoder.decode([String.self], from : data)
```

Let's say you have a model that represents some data the API should be returning:

```swift
struct: Person: Codable {
    let name: String
    let age: Int
    let city: String
}
```

But, let's say the backend is returning only `name` and `age`. We've told the parser that there will be a field named `city` in the JSON, and thus parsing will fail with a very generic and unhelpful error like this:

`The data couldn't be read because it is missing.`

## A solution

You can simply put your decoding in a do-try block whilst catching very specific errors, like this:

```swift
do {
            return try decoder.decode(T.self, from: data)
        } catch let DecodingError.keyNotFound(key, context) {
            print("Decoding error (keyNotFound): \(key) not found in \(context.debugDescription)")
            print("Coding path: \(context.codingPath)")
        } catch let DecodingError.dataCorrupted(context) {
            print("Decoding error (dataCorrupted): data corrupted in \(context.debugDescription)")
            print("Coding path: \(context.codingPath)")
        } catch let DecodingError.typeMismatch(type, context) {
            print("Decoding error (typeMismatch): type mismatch of \(type) in \(context.debugDescription)")
            print("Coding path: \(context.codingPath)")
        } catch let DecodingError.valueNotFound(type, context) {
            print("Decoding error (valueNotFound): value not found for \(type) in \(context.debugDescription)")
            print("Coding path: \(context.codingPath)")
        }
```

You'll now get errors that tell you when a type didn't match, a key or value way missing in the JSON, or when the data is corrupted in some way, looking like this:

```
Decoding error (keyNotFound): CodingKeys(stringValue: "city", intValue: nil) not found -  No value associated with key CodingKeys(stringValue: "reportingBusiness", intValue: nil) ("reportingBusiness").

Coding path: [CodingKeys(stringValue: "items", intValue: nil), _JSONKey(stringValue: "Index 6", intValue: 6)]
```

So, at least now you will be armed with the knowledge that the parsing failed because the JSON was missing a key that your struct was expecting, named `reportingBussiness`. 

💡 Remember to delete the print statement before releasing the app!