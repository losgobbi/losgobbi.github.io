---
title: "Old source code from my first iPhone app"
date: 2026-09-03T14:06:00-03:00
categories: ["misc"]
description: "My first swift source-code"
---

#### Nothing else to do...

It's curious to read our own old source code if we have a chance to do so. One case that I was remembering recently, was our first attempt of developing an iPhone mobile app targeting the restaurant/delivery scene in our city. More info about it is available in the README file of the [repository](https://github.com/losgobbi/foodApp) - internally we called this "*generic food app or food app*". It's worth noting that this was my first iPhone source code after buying the Mac Book Air for our mission to create a "software company". Imagine that at that time we were coding a lot of stuff, during business hours, using `C programming language`, and we were familiar with `OOP` (Object-Oriented Programming) but here we were dealing with a whole different target system and development `framework`/`SDK`, different `IDE` (`Xcode`) as well. It was a nice challenge.

#### Asked Claude for inspecting a curious class

After you work with Android, you know that the system/framework gives you a lot of useful "building blocks" for building UI elements, meaning that you don't need to build anything like that "by hand". The system will give you that for free, you just need to extend one or more classes and feed extra information back to the system/framework so that the particular building block can work properly.

Going back to the food app, I remember that we did something very strange considering how things are today (and at that time). We created a custom "*ScrollView*" class for dealing with a huge memory consumption from our app when the user was "swiping" the cards viewing the food description, picture, price and other details. I guess the system was killing the app due to the amount of memory used (not completely sure), and every scroll view element was "attached" to an `ImageView` as well, which was definitely increasing the usage.

```swift
//
//  ScrollRecycler.swift
//  FoodApp
//
//  Created by Rodrigo Celso Gobbi on 10/28/15.
//  Copyright © 2015 Hagen. All rights reserved.
//
//  Scrollview recycler
//

import UIKit
import Foundation

func dbgRec(message: String, function: String = #function) {
    #if DEBUG_RECYCLER
    print("\(function)> " + message)
    #endif
}

class ScrollRecycler {
    
    /* Scrolling pointers */
    private var triggerAsc: Int = 0
    private var triggerDesc: Int = 0
    
    /* How many pages will be recycled */
    private var recycleNum: Int = recycleNumber
    
    /* Pages for recycler */
    private var currentPage: Int = 0
    private var recycledPages = [ProductView]()
    private var visiblePages = [ProductView]()

    /* Controller for this recycler, we need to access its subviews */
    private var vc: ViewController?
    
    /* The scrollview and its content size */
    private var scrollMenu: UIScrollView?
    private var scrollMenuSize: Int = 0
    ...
```

The full source file is [here](https://github.com/losgobbi/foodApp/blob/master/FoodApp/ScrollRecycler.swift), for the record. But... yes, we started to use something "native" and faced a lot of memory instabilities, then, we switched to a custom version of ScrollView. Very weird, but it was something like that. Either way, I asked Claude to inspect and reason about that source code in order to refresh my memory and see what we were trying to do.

#### What Claude found

**What the class is:** *"`ScrollRecycler` is a hand-built version of the view-recycling that `UITableView`/`UICollectionView` give you for free — bolted onto a plain `UIScrollView` instead. It keeps a small, fixed pool of `ProductView` instances, and as you page left/right it detects when you've crossed a "trigger" page (triggerAsc/triggerDesc), pulls the view that just went off-screen, repositions it (frame.origin.x) to the new edge, and refills it with the next product's data (flushProductView) — the same idea as `prepareForReuse`, just written by hand."*

**Where it's used:** *`prepareScrollMenu()`, in `ViewController+Iphone.swift`. On iPhone, the entire menu is browsed one product per full screen, swiping through a paging `UIScrollView`. The constants in `AppConfig.swift` reveal the intent: `scrollInitialNumber = 10` (only 10 ProductViews ever exist, no matter how big the menu is) and `recycleNumber = 3` (recycled in batches of 3). Since each `ProductView` is heavy — image, line image, name, price, bookmark/add buttons — instantiating one per menu item would've been unworkable on a 2015 iPhone. Capping it at 10 live views and recycling them was a legitimate, deliberate memory optimization.*

**Why by hand** *"instead of `UICollectionView`/`UIPageViewController`: the repo's own timeline tells the story. `ScrollRecycler.swift` is dated *Oct 28, 2015*, and the README says this was the first native iOS project for you and your partners. The app's actual `UICollectionView` usage (the iPad grid, `ProductCollectionViewCell`/`prepareCollection()`) only shows up in *April–May 2016* — roughly six months later, partly written by a teammate. So when this iPhone screen was built, the paging UIScrollView already sitting in the storyboard was the most direct tool at hand, and since it needed effectively unbounded content with only a few live views, the reuse-queue concept got reinvented from scratch — because `UICollectionView` hadn't entered the picture yet. By the time it did (the iPad grid, months later), the iPhone screen already worked and was never refactored to match. That's exactly why `recDidScroll`/`recUpdateTrigger` read as so gnarly today: it's `dequeueReusableCell` + `prepareForReuse` being reimplemented from the ground up, before you knew the platform already had a name for this problem."*

#### Top issues found by Claude

So, we shouldn't have been doing things that way. I asked him a few more things to wrap up the discussion:

- Retain cycle — `ScrollRecycler.vc` holds a strong reference back to the ViewController, which itself owns the recycler as a property (should be `weak` var vc). It likely never surfaced as a visible leak because this VC lives for the app's whole lifetime.
- "Side" with no real type, represented two different ways — sometimes a String ("ascending"/"descending"), sometimes a Bool, with no enum and no compiler safety. Worse, `-1` is used as a sentinel error value (`recGetScrollPointer` returns `-1` on failure) that no caller ever checks — it just flows into index/array math downstream. A silent failure mode instead of a crash — the worse kind of bug.
- The recycling trigger (`recUpdateTrigger`) decides state transitions using exact `CGFloat` equality, which is fragile to rounding; and `recDequeueRecyclePages` reassigns `scrollMenu.contentSize` inside a loop, mid-gesture (it's called synchronously from scrollViewDidScroll while the user is still dragging), which is a known way to cause a visible jump/stutter in an actively-panned `UIScrollView`. These are exactly the details `UICollectionView` already handles internally — the kind you only discover the hard way when you reinvent reuse by hand.

...overall, we were not respecting the DRY principle...:sweat_smile:, even so, it was good to revisit this.


