---
layout: post
title: The Basics of Property Wrappers
tags: basics, property wrappers
---
## What is a property wrapper?
A property wrapper is a type that wraps a value, adding some extra logic to it in the process. They're really useful for instance if you wanted to store a `String` value but only after having processed it in some way, like truncating or capitalizing it. The value, the `String` value itself would be wrapped/hidden in a type that also handles the logic for truncation automatically.

Let's say that we want to uppercase the first character of a name and first string when they're assigned. This could be done with a simple property observer that changes the new value every time one is assigned:

```swift
var firstName: String = "" {
    didSet {
        firstName = firstName.capitalized
    }
}

var lastName: String = "" {
    didSet {
        lastName = lastName.capitalized
    }
}

firstName = "asohka" // firstName = Asohka
lastName = "tano" // lastName = Tano
```

This approach will quickly litter your code with lots of boilerplate code when you begin adding more and more observers that change values as well as more properties that want validating. What would your code look like if you had to do this 15 times? 30? This is where property wrappers can be useful.

Using a property wrapper in this example will allow us to define the common funcionality, capitalizing the `String` on assignment once, and re-use it for any number of `String`s:

```swift
@propertyWrapper // must have this
struct Capitalized {
    private var value: String = ""
    var wrappedValue: String { // also, must have one of these
        get { value }

        set { value = newValue.capitalized }
    }
}
```

The above code defines a property wrapper `struct` named `Capitalized`. The compiler enforces the presence of computed property named `wrappedValue` that either returns the current `value` or sets it to whatever its new value should be, capitalized. `value` is where the name string is actually stored and is maked as `private` to prevent access from the outside, by-passing the `wrappedValue` front-end for it which does all of its processing. You don't have to mark is at `private` but this is my preference to keep the under-lying value safe from accidentaly mutation.

So now, we can quickly create variables that take advantage of all of this lovely new, re-usable functionality by simply defining them using the `@Capitalized` annotation. Notice how below two are defined but the functionality is only defined once rather than each time per variable as with the property observer approach before:

```swift
struct Person {
    @Capitalized var firstName: String
    @Capitalized var lastName: String

    init(firstName: String, lastName: String) {
        self.firstName = firstName
        self.lastName = lastName
    }
}


var person = Person(firstName: "asohka", lastName: "tano")
// person.firstName has been capitalized to Asohka
// person.lastName been capitalized to Tano
```

In the above small example you'll notice that both the `firstName` and `lastName` properties were delcared as `Capitalized`, matching the `@propertyWrapper` above it.

Property wrappers can be used on properties of both classes and structs, for **local** (not global) variables and as function arguments.