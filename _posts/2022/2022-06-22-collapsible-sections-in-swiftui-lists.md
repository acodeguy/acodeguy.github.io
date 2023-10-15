---
layout: post
title: Create a fully-collapsible Section for iOS using SwiftUI
tags: swiftui, lists
---
Recently, I wanted to add in collapsible sections into one of my personal projects build in SwiftUI, something like this

![](/assets/images/collapsible-section-example.gif)

Knowing that SwiftUI has a [`Section`](https://developer.apple.com/documentation/swiftui/section) component, I immediately looked into this, to find out how I'd make a `Section` collapsible.

I found [collapsible](https://developer.apple.com/documentation/swiftui/section/collapsible(_:)) in the official docs and got excited until I saw:

> macOS 10.15+ 🤦🏻‍♂️

Then I decided to think about how I could make this myself for iOS, as lean as possible.

The first version that I wrote required arrays of typed data to be fed-in to populate list rows of text and images which made the whole thing far too rigid, so I went back to the drawing board. I wanted it to be flexible enough to accept any content. But I didn't know how this was achieved, but I know it could be done, as I've done this so many times:

```swift
VStack {
    Text("I am child content")
    Text("So am I")
}
```
Knowing the above very familiar pattern led me to the SwiftUI source code for `VStack`:

```swift
@inlinable public init(alignment: HorizontalAlignment = .center, spacing: CGFloat? = nil, @ViewBuilder content: () -> Content)
```

What's this `@ViewBuilder` thingy here? That's the key that unlocks the above way of providing content to the `VStack`s **trailing closure** in its init.


## The collapsible Section code

```swift
struct CollapsibleSection<Content: View>: View {

    private let title: String
    private let content: Content
    private let alignment: HorizontalAlignment
    private let spacing: CGFloat
    @State private var isExpanded = false

    init(title: String, alignment: HorizontalAlignment = .leading, spacing: CGFloat = 16, @ViewBuilder content: () -> Content) {
        self.title = title
        self.alignment = alignment
        self.spacing = spacing
        self.content = content()
    }

    var body: some View {
        Section {
            if isExpanded {
                VStack(alignment: alignment, spacing: spacing) {
                    content
                }
            }
        } header: {
            HStack {
                Text(title)
                Spacer()
                Image(systemName: "chevron.up")
                    .rotationEffect(.degrees(isExpanded ? 180 : 0))
            }
            .onTapGesture {
                withAnimation { isExpanded.toggle() }
            }
        }
    }
}
```

## Usage

Once you've copied the `CollapsibleSection` into your codebase, use it like this:

```swift
Form { // Form will give us a lot of free, consistent styling!
    CollapsibleSection(title: "Vehicles") { // @ViewBuilder at work here
        Text("Millennium Falcon")
        Text("AT-AT")
        Text("Landspeeder")
        Text("TIE Fighter")
        Text("Slave I")
        Text("AT-ST")
        Text("X-Wing")
        Text("Dreadnought")
    }
}
```

And here is what the above code yields:

![expanding and collapsing the section](/assets/images/collapsible-section-demo.gif)

You've got a very lean collapsible section to use perhaps on a `Form`. I chose to keep this demo as lean as possible, else I'd have got carried away animating the content fading in or even better, Star Wars movies transitions 😁

## In summary

1. Create a struct that will represent your section. It should have a generic type for the content conforming to `View (<Content: View>)`, as well as conforming to `View` itself
2. Have a trailing closure in the `init`, tagged as `@ViewBuilder` and of type `() -> Content`.