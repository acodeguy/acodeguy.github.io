---
layout: post
title:  "IsNotEmpty: A Simple Extension to Improve Readability when Checking the Count of Collections in Swift"
date:   2020-11-19 16:37:42 +0000
categories: swift
---

Long title, I know.
I don’t know about you, but my mind always jolts a little when I read something like below (line 9):

{% highlight swift %}
let bands = [
    "Megadeth",
    "Metallica",
    "Anthrax",
    "Slayer"
]

// if the bands array is not empty
if !bands.isEmpty {
    // do some stuff here
}
{% endhighlight %}

Okay, this is a small example, but it does illustrate the point: who speaks or writes like that! “if not bands is empty…”
For readability, I just had a brainstorm: extend the Collection protocol to provide something more readable:

{% highlight swift %}
extension Collection {
    var isNotEmpty: Bool {
        return !self.isEmpty
    }
}

let bands = [
    "Megadeth",
    "Metallica",
    "Anthrax",
    "Slayer"
]

// if the bands array is not empty
if bands.isNotEmpty {
    // do stuff here
}
{% endhighlight %}

Notice that it reads way more like English now: “if bands is not empty…”
The ugliness with the notting is done once, hidden away in the extension on line 3, leaving your code that uses it, très readable.
And because it is declared as an extension to Collection, you get this simple tweak for Array, Set and Dictionary at the same time. Noice!
Comments and improvements always welcome 😊