---
layout: post
title: Testing Core Data Entities in Swift with XCTest
tags: xctest, testing
---
# Testing Core Data Entities in Swift with XCTest

Currently, I’m teaching myself to build iOS apps in Swift as my main focus, whilst up-skilling in other areas too, such as Ruby on Rails. I’ve recently graduated from Makers Academy, where I learned the black art of test-driven development and am keen to use it everywhere I can. The course loosely teaches TDD with Ruby & RSpec/Capybara then JavaScript & Jasmine, then they cut you loose and tell you to learn to test-drive code in two new languages of your choosing. Go!

Since graduation last week, I’m keeping my skills sharp by continuing with development daily whilst I job-hunt that elusive junior Swift developer role. (Did someone say “unicorn”?) I’ve chosen to create an iOS todo list app as it seemed like a good place to begin with test-driving Swift development as well as getting used to components like table views and Core Data.

I found that testing custom models is a piece of piss if they are only kept in memory using arrays, for example. Easy peasy. Shit gets really tricky after you plumb-in Core Data and start putting all that into a database. Suddenly, you can no longer just spin-up a new instance of your model and query its properties. What is a boy to do? Or girl. I met some awesome girl coders on my course at Makers…

After sleeping on the problem, I did some searching and found that Core Data has a wonderful feature called in-memory stores that allows you to essentially spin-up a copy of your database in RAM, at will. Here’s what I managed to put together to allow me to test Core Data entities with XCTest, explained as best I can:

```swift
import XCTest
import CoreData

class MyAppTests: XCTestCase {
  
  private var appContext: NSManagedObjectContext?
  
  override func setUp() {
     super.setUp()
        
    appContext = createInMemoryManagedObjectContext()
        
  }
  
  func testUserIsNotAnAdminOnCreation() {
      let user = User(context: appContext!)
      XCTAssertFalse(user.isAdmin)
   }
  
  // MARK: my test methods
  
  func createInMemoryManagedObjectContext() -> NSManagedObjectContext? {
        
        guard let managedObjectModel = NSManagedObjectModel.mergedModel(from: [Bundle.main]) else { return nil }
        
        let persistentStoreCoordinator = NSPersistentStoreCoordinator(managedObjectModel: managedObjectModel)
        do {
            try persistentStoreCoordinator.addPersistentStore(ofType: NSInMemoryStoreType, configurationName: nil, at: nil, options: nil)
        } catch {
            print("Error creating test core data store: \(error)")
            return nil
        }
        
        let managedObjectContext = NSManagedObjectContext(concurrencyType: .mainQueueConcurrencyType)
        managedObjectContext.persistentStoreCoordinator = persistentStoreCoordinator
        
        return managedObjectContext
    }
}
```

Line 6: a variable is declared of an optional `NSManagedObjectContext`. This will hold our context in which the app is running. This like like getting the context from the AppDelegate using `UIApplication` for when you want to use Core Data normally.

Line 11: the `appContext` variable is assigned a newly created context in RAM, rather than an actual database file

Line 16: we create a User object within this new in-memory app context, using the method I’ve defined below from line 22.

Line 22: declare method that returns an optional `NSManagedObjectContext`.

Line 24: get all the the models available in our app and merge them into one model (programatic representations of our `.xcdatamodeld` files)

Line 26–32: attempt to assign the `persistentStoreCoordinator` variable which allows the communication between the context and the data store.

Line 34: this variable needed to track changes to our models.

Line 35: assign the co-ordinator to the managed context so it knows where to save its changes.

With the above, my tests were broken but now they are fixed. Green, green, green. Back to adding new features ✊🏽

If you’d like a peek at the todo list app to see the above code in action, see my Dunnit! app on GitHub.