---
layout: post
title: Building an iOS app without Storyboards
tags: uikit, views in code
---
Written with Xcode 12.1 targeting iOS 14.1.

## Why should I?
Interface Builder is great whilst you’re learning on your own, but as your app and team grows, you’re likely to have regular fights with merge conflicts because of storyboards. They just don’t play well with version control.

## Okay, I'm sold. How do I do this?
Glad you asked.

1. Start a new project (single view template is great). Once it’s ready, delete Main.storyboard from the Project navigator (Right-click -> Delete -> 👋🏼
2. Next, click on your project’s Info.plist file and delete the two highlighted keys:

![Xcode build settings](/assets/images/xcode-build-settings.png)

3. Now head on over to SceneDelegate.swift and make it like so:

```swift
import UIKit

final class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        guard let windowScene = (scene as? UIWindowScene) else { return }

        // this is a great place to build and inject any dependencies into your root view controller
        let navigationController = UINavigationController(rootViewController: ViewController()) // a
        let window = UIWindow(windowScene: windowScene) // b
        self.window = window // c
        window.rootViewController = navigationController // d
        window.makeKeyAndVisible() // e
    }
}
```

a. Firstly, we build a new navigation controller to house our root view controller, which in this case is the standard one that Apple’s template gave us.

b. Then we build a new window using the scene object.

c. Then we set the app’s window to our newly-built one.

d. Then we assign the previously-built navigation controller as window’s root view controller.

e. Finally, we position the window on top of all others.

6. And then in ViewController.swift, like so:

```swift
import UIKit

final class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        title = "No Boards"

        view.backgroundColor = .systemBackground // 1
    }
}
```

Now, you’ll be in exactly the same place before we started, but without a pesky storyboard to get in the way, and the ability to take better advantage of dependency injection. Nice 🙌🏼

You may have noticed that my code snippets above are very lean. That’s how I code, as least clutter as possible. The first thing I do when I start a new projects is all of the above, as well as gutting Apple’s boilerplate & comments. Less is more, to me 🤘🏼