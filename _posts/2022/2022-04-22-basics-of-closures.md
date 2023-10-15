---
layout: post
title: The Basics of Closures
tags: closures, functions, basics
---
Closures are hard at first. I spent a long time and energy trying avoid them but once I started working professionally it became a matter of time until I embraced them. Now, I use them mostly without much thought, kinda enjoy them actually.

## Contents
1. [What are closures?](#what-are-closures)
2. [Trailing closure syntax](#trailing-closure-syntax)
3. [Escaping closures](#escaping-closures)

<a id="what-are-closures"></a>

## What are closures?
A closure is like a recipe written by one person and then given to another to follow. The recipient could follow it right away if they're in a kitchen, or later, after some asynchronous action, like getting home.

Below, is a *simple* closure. Looks odd, doesn't it?

```swift
let add: (Int, Int) -> Int = { (x: Int, y: Int) in
    return x + y
}
```

Let's break it down...

```swift
let add: (Int, Int) -> Int
```

This declares a closure variable named `add` of type of `(Int, Int) -> Int` (takes in two `Int`s as parameters, and returns an `Int`).

```swift
= { (x: Int, y: Int) in
    return x + y
}
```
This assigns the closure body to the `add` variable. 

This is like a function body. `(x: Int, y: Int)` here is saying that that parameters of type `Int` will be used **locally** in the body of this closure. The body actually begins after `in`; here is where you do stuff.

`x` and `y` are used locally within the closure body and the returned as you would do from a regular function: `return x + y`

So, `add` now has a closure body assigned to it. Big whoop.
Now, you can call it like a regular function:

```swift
add(2, 2) // 5. Not really, this actually returns 4
```

### Bare essentials

It's possible to take the longer form closure we have above and boil it down to its barest of essentials.

We have the following:

```swift
let add: (Int, Int) -> Int = { (x: Int, y: Int) in
    return x + y
}
```

With the magic of Swift's type inference, we can omit the parameter types, giving us:

```swift
let add: (Int, Int) -> Int = { (x, y) in
    return x + y
}
```

To chop-off just a little more, we can remove the brackets:

```swift
let add: (Int, Int) -> Int = { x, y in
    return x + y
}
```

We can go even further. Swift's type inference is smart enough to work out that two `Int` parameters are coming in and one `Int` being returned.  We're able to bin most of the body and return the sum of the two parameters:

```swift
let add: (Int, Int) -> Int = { return $0 + $1 }
```

`$0` is the first parameter (previously `x`), `$1`, the second.

Finally, we can lose the `return` keyword and Swfit supports implicit returns:
```swift
let add: (Int, Int) -> Int = { $0 + $1 }
```
While that looks clever and its brevity, this shortness can make code harder-to-reader for others, so please be careful how you use this.

### Use cases
But you can do that with regular functions, so why bother with closures?

Let's say that depending on some event happening in your code, you're going to either need to add or subtract to values. Starting-off with a closure type defined below (it takes two `Int`s and returns one), it is possible to later assign the brains of this closure, the body, the instruction.

```swift
var operation: (Int, Int) -> Int
```

Right now, the above is just a type declaration. It has no body, so it does nothing.

If we define a type of operation to perform, say, subtraction, we could do this:
```swift
var subtract: (Int, Int) -> Int = { $0 - $1 }

operation = subtract // operation now has the body of the subtract closure expression

operation(5, 4) // 1
```

So what happened there?

We declared a variable of type `(Int, Int) -> Int` but gave it no initial value.

Then, we declared a closure expression that matches the type of the variable, `(Int, Int) -> Int`, named `subtract`. This expression is the actual instruction that does the subtraction, but...

...not until you called it with the arguments: 5 & 4.

<a id="trailing-closure-syntax"></a>
## Trailing closure syntax

A very common pattern you'll see is using trailing closures for completion handlers. A completion handler is basically the block of code to be executed once an asynchronous task completes. It's like saying, **When the cake has cooled, then put on the icing**.

Below, we have a method whose final argument is a closure, named `completion` of type `(String) -> Void`. Assume that this method makes a network call to an API that could take some time to return a response:

```swift
func requestData(from url: URL, completion: @escaping (String) -> Void) {
    // request data from the server

    // O' the asynchronicity! ⏳

    // once the server has returned an "OK" response
    // we pass that back to the caller via the completion handler
    completion("OK")
}
```

Note: The `@escaping` keyword will be explained in the next section, [Escaping closures](#escaping-closures), so please forget you saw it for now.

We could define a closure variable, named `ourCompletion` with the same type as the `completion` argument in `requestData` above, `(String) -> Void` and supply that when calling the method:

```swift
let ourCompletion: (String) -> Void = { serverResponse in
    print(serverResponse) // #1
}

let url = URL(string: "https://example.xyz")!

requestData(from: url, completion: ourCompletion)
```

When the server responds and the `requestData` method processes it, and finally calls the `completion` handler we supplied, you'll see "OK" printed out to the console. Printing "OK" to the console is a slim example. In reality, this would be the data from the server which you could then pass back to the caller of `requestData` to be parsed into `Codable`, for example.

I recommend building this small demo in a full project and setting a break-point at comment **#1** to see it actually hit and stop there.

With trailing closure syntax, we can define the code to be executed on completion in-line, **as  long as the closure is last parameter the method requires**:

The below is a pattern used often when requesting data from a server in Swift. In this example, you supply the `URL` and a trailing closure to be executed once the server has responded. This version has no need to define the `ourCompletion` before-hand as its defined inline at the call-site. This method is more common out in the wild.

Using the same url and method defined slightly earlier we can make the exact same call in this way:

```swift
requestData(from: url) { serverResponse in // #2
    print(serverResponse)
}
```

Notice that we've only used the first argument, `from` at the call-site. Then the curly braces opens with the parameters that the closure will supply, in this case a `String` which we've chosen to call `serverResponse`. You can name this anything but it's always good to be descriptive for other readers and of course for future you.

Both of these examples are functionally equivalent

<a id="escaping-closures"></a>

## Escaping closures

Apple's documetation says
>A closure is said to escape a function when the closure is passed as an argument to the function, but is called after the function returns.

That may not mean a whole bunch on its own so let's go through an example.

Using an updated version of our `requestData` example above, we can begin to understand the Wonderful World of Asynchronous Functions:

```swift
func requestData(using session: URLSession, from url: URL, completion: @escaping (String) -> Void) {
    print("requestData starting") // 1
    
    session.dataTask(with: url) { _, _, _ in // 2
        completion("OK") // 4
    }.resume()

    print("requestData finishing") // 3
}

requestData(using: .shared, from: url) { serverResponse in 
    print(serverResponse)
}
```
If you run the above in a Playground, intuitively, you may be expecting the print statements to run sequentially, generating output like:

```
requestData starting
OK
requestData finishing
```

But the actual output is:

```
requestData starting
requestData finishing
OK
```

If we were dealing with a **synchronous** method, you'd have been correct. But we're in Asynchronous Country now.
### So what's happening?
1. We enter the scope of `requestData(using:, from:, completion)` and the first print statement executes.
2. An **asynchronous** call to `dataTask(with:)` is added to the call stack. This has a trailing closure which will, *at some point* call our `completion` handler. We cannot control when that will happen 🤷🏻‍♂️
3. `requestData(using:, from:, completion)` executes its last print statement before finishing. It will be kicked off of the function call stack, **but** `dataTask(with:)` is still on the call stack executing.
4. `dataTask(with:)` finally gets a response of some kind and then calls the `completion` handler, which now has **escaped** the `requestData(using:, from:, completion)` which supplied it originally.

The Swift compiler is pretty helpful with these and will generate a compile-time error if it thinks you'll need to tag a closure parameter as `@escaping`.