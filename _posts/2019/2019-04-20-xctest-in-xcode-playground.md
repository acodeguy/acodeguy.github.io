---
layout: post
title: Running XCTests in an Xcode Playground
tags: playgrounds, testing
---
# Running XCTests in an Xcode Playground

Having recently been introduced to the wonders of Test-Driven Development, starting with RSpec and then Jasmine, I was challenged to port over some UI-less Ruby/JavaScript code, test-driven, into Swift, my weapon of choice. After porting it over successfully into test-driven Swift, I wondered why Xcode didn’t have a simpler way to write just simple, UI-less code with tests, as a full Xcode project can be pretty bulky and overkill for just writing a few tiny unit tests.

Today, I switched my Codewars focus to Swift as Ruby has served me well during my eleven weeks at Makers, but I don’t see it in my future; Swift is where I wanna go. After having started a few kata, I really wanted a way to run these tests locally on my own laptop. So I opened-up a new Playground, pasted the kata’s sample tests in for shits-and-giggles, and to my surprise, Xcode did not complain. Too much. I had to change the last line which runs the suite, but that’s all.

Here’s what works for me:

```swift
import Foundation
import XCTest

func addition(_ firstNumber: Int, _ secondNumber: Int)  -> Int {
    return firstNumber + secondNumber
}

class AdditionTests: XCTestCase {
    func testAddition() {
        XCTAssertEqual(addition(2,2), 4)
    }
}

AdditionTests.defaultTestSuite.run()
```

Line 2: import the `XCTest` framework so we can actually use it!

Line 4: begin declaring the method we want to test, in this case a very simple addition method that returns an `Integer`.

Line 8: declare the suite of tests, sub-classing `XCTestCase`.

Line 9: create a test method that when executed will perform some test actions and most crucially, perform an assertion on the outcome.

Line 10: finally, we run the test suite we’ve just defined.

You should then be presented with the results in your console:

![Xcode console with test output](/assets/xcode-console-passing-tests.png)

Of course, in the real world, the tests themselves would be in their own source files, and you would most definitely write a failing test first, watch it fail, then write the bare minimum amount of code to make that test pass, then refactor if need be, as per Test-Driven Development. Learn to love seeing red test failures in your console. It’ll make you a better person. Fact.

I had no idea this was possible before today as I haven’t seen an online bootcamp or decent tutorials that cover testing yet, so I kinda just pieced bits together from here and there. Now I know that the `XCTest` framework is available to Playgrounds, I can go back and morph those full and bulky iOS projects into light-weight Playgrounds where they belong.

Frakkin’ sweet!
