---
layout: page
title: The Basics Combine
tags: basics, combine
---
# A brain dump on Combine

My learnings on Combine. Shared with you.

It's recommended that you have a basic understanding of these topics
1. Functional programming in Swift

## Contents
1. [Publishers](#publishers)
2. [Subscribers and Subscriptions](#subscribers-and-subscriptions)
3. [Operators](#operators)
    1. [filter](#filter)
    2. [compactMap](#compactMap)
    3. [replaceNil](#replaceNil)
    3. [map](#map)

<a id="publishers"></a>
## Publishers

A Publisher publishes values to interested parties known as Subscribers. The publisher alerts its subscribers that new values are available but it's upto the subscriber to actually grab those new values. This serves to alleviate backpressure.

The following creates a Publisher that publishes `Integer` values and never sends any errors:

```swift
let publisher = PassthroughSubject<Int, Never>()
```

If nobody is subscribed then no values are published.

<a id="subscribers-and-subscriptions"></a>
## Subscribers & Subscriptions

A Subscriber subscribes to a Publisher to receive values from it over time.

`.sink` can be called on a Publisher to create a closure-based Subscriber:

```swift
let publisher = PassthroughSubject<Int, Never>()

publisher
    .sink { value in
        print("Recieved:", value) // receives: 1
    }

publisher.send(1) // send the integer value 1 to any Subscribers
```

`send()` can be called infitely on this type of Publisher as we have no means to cancel it. Yet.

`.sink` returns a token of type `AnyCancellable` which should be stored to keep the connection beween the Publisher and Subscriber alive. We can then call `cancel()` to remove the subscription:

```swift
var subscription: AnyCancellable?
let publisher = PassthroughSubject<Int, Never>()

subscription = publisher
    .sink { value in
        print("Recieved:", value) // receives: 1
    }

publisher.send(1) // send the integer value 1 to any Subscribers. Received in the .sink closure above

subscription?.cancel() // subscription dies

publisher.send(2) // 2 is not recieved in the .sink closure as the subscription has just been cancelled
```

<a id="operators"></a>
## Operators

<a id="filter"></a>
### filter

Filters-out any published values that do not meet the filter's requirements.

The below adds the filter operator to filter any values lower than 0 so they are not passed on to the Subscribers:

```swift
var subscription: Cancellable?

let publisher = PassthroughSubject<Int, Never>()

subscription = publisher
    .filter { $0 > 0}
    .sink { value in
        print("Recieved:", value)
    }

publisher.send(2) // prints
publisher.send(1) // prints
publisher.send(0) // does not print; less than 0, filtered-out
```

<a id="compactMap"></a>
### compactMap

Removes any values of `nil` and passes them along the stream.

Take the following publisher of integers. It's optional because it contains one optional value. Using `compactMap`, only non-nil values are allowed to be passed on down to be dealt with by `filter`, etc.

```swift
var subscription: Cancellable?

let publisher = [1, 2, Optional<Int>(nil), 3].publisher

subscription = publisher
    .compactMap { $0 }
    .filter { $0 > 0}
    .sink { value in
        print("Recieved:", value)
    }

subscription?.cancel()
```

The above example will print the following to your console:

```
Recieved: 1
Recieved: 2
Recieved: 3
```

<a id="replaceNil"></a>
### replaceNil

`replaceNil` will let you remove any `nil` values and instead return a more useful value.

```swift
var subscription: Cancellable?

let publisher = [1, 2, Optional<Int>(nil), 3].publisher

subscription = publisher
    .replaceNil(with: 999)
    .filter { $0 > 0}
    .sink { value in
        print("Recieved:", value)
    }

subscription?.cancel()
```

This will replace the `nil` value in the array with `999` which also passes the `.filter()`. The console will show:

```
Recieved: 1
Recieved: 2
Recieved: 999
Recieved: 3
```

<a id="map"></a>

### map

`map` will allow you to transform a value in some way. 

In the below example, the integer values that pass the filter are transformed into an interpolated string using the `map` operator, adding an `x` on either side of the number:

```swift
var subscription: Cancellable?

let publisher = [1, 2, Optional<Int>(nil), 3].publisher

subscription = publisher
    .replaceNil(with: 999)
    .filter { $0 > 0}
    .map { "x\($0)x" }
    .sink { value in
        print("Recieved:", value)
    }

subscription?.cancel()
```

The `sink` above is changed from a publisher of `Integer`s to one of `String`s now:

![Xcode pop-up showing summary of our publisher](/assets/images/map-publisher-string.png)