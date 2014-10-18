---
layout: article
title: "View Controller Lifecycle"
modified:
categories: mobile
excerpt:
tags: [android, iOS, mobile, view controllers]
image:
  feature:
  teaser: /lifecycle/basic-lifecycle-android.png
  thumb: /lifecycle/basic-lifecycle-android.png
date: 2014-10-10T08:31:09-07:00
---

In this post, I’m going to compare the view controller life cycles of iOS and Android.  Life cycle is the process of creating and showing a view on screen, and then hiding and destroying it when it is no longer needed.  This process is complex: it touches on memory management, view measurement and layout, and saved app states.  To allow the developer to control the view at each stage, both platforms give a developer a series of hooks in the view controller to modify behavior at certain stages in the lifecycle.  I'll compare these hooks, and discuss why I think iOS’s handling of lifecycle is superior to Android’s.  This article is not meant as a tutorial, and so I'm not going to go in depth into how to use each lifecycle method, except for the purposes of drawing comparisons.  

The Basics

The basic life cycle of a view controller that is asked to appear on screen:

1. Create the view controller
2. Load the view into memory
3. Lay out and measure the views
4. Show the view on screen

And when it is asked to leave the screen:

1. Disappear/save state
2. Release memory

Digging a little deeper, these life cycle states encompass a lot of complexity, and numerous edge cases.  For instance if a view goes off screen, should the view be cached?  Should it's state be persisted?  What happens if a bunch of views are saved in memory, but system resources run low: does the system destroy views in the background to reclaim memory?  These are just some examples of cases that must be handled to show a view in an app successfully.  In general, I find that Android exposes a lot of this complexity, while iOS has managed to abstract most of it away.  In particular, the optimizations that Android made to reclaim memory and CPU from backgrounded views leak out into the realm of the developer, causing numerous crashes and introducing significant complexity.  

Apple's lifecycle hooks closely match the steps outlined above.  The basic life cycle methods are:

1. ViewDidLoad
2. ViewWillAppear
3. ViewDidAppear
4. ViewWillDisappear
5. ViewDidDisappear

The responsibility of the developer is simply to handle whatever view and business logic the app needs at each of these stages.  The app backgrounding, view caching, and system memory management are all handled by the operating system, automatically.  In fact, I coded in iOS for months without appreciating what was being handled for me in the background.  It just worked. 

Here are the equivalent Android docs.  

![Android Lifecycle](/images/lifecycle/basic-lifecycle-android.png)

Here's a quick comparison of the most common lifecycle methods on the two platforms, and where they fit into the life cycle of the view.  

![View Appearing](/images/lifecycle/view-controller-appear.png)

![View Disappearing](/images/lifecycle/view-controller-disappear.png)

Side by side, you can see that, while the methods on the two platforms are roughly equivalent, the naming shows the difference in the underlying design philosophy between the two systems.  Apple's life cycle is focused on the view and the view's state, while Android's life cycle is focused on the state of the view *controller*.  Android envisioned view controllers that are active on screen, paused while something else is in the foreground, stopped, such as if the app is backgrounded, or destroyed such that it has to be recreated from scratch.  To reflect these states, they provided hooks like onPause and onStop.  These hooks define a set of states that a view controller can exist in with multiple entry points, which is fundamentally different than the linear approach favored by Apple.  Once a developer is responsible for taking action according to these different states, it gains responsibility for all of the complexity and all of the edge cases.  For instance, the onPause hook can be called for a number of reasons: the app has been backgrounded, the user has closed the app, another view has appeared, etc.  Sometimes, this method can be followed by a call to onDestroy, but that's not necessarily the case.  Sometimes, the app might have to save itself, signaled by another hook, onSaveInstanceState.  But onSaveInstanceState is not guaranteed to be called, for instance if the user "intentionally exits the view," whatever that means.  The developer must be aware of these different cases when deciding what to put in what hook.  Eight months into the development of my first Android app, I was still encountering issues, edge cases, and crashes pertaining to lifecycle.  I was still having to refer to lifecycle documentation, and it seemed like every week I was discovering a new hook.  

As if Android lifecycle isn't confusing enough, Android has an additional set of lifecycle hooks for Fragments that are interleaved with the hooks of the parent activity, as outlined below.  

![Android Lifecycle](/images/lifecycle/fragment-lifecycle-android.png)

Confusing, no?  

To compound the bad design above, the handling of the memory is extremely clunky.  The seemingly innocuous hook, onSaveInstanceState, opens a world of hurt.  It turns out that, in addition to requiring a developer to manually save the state of a view controller, Android has not provided a reasonable way of doing so.  To be saved, a view controller must serialize itself.  Unfortunately, only primitives can be readily saved.  Higher level objects, including your nicely decomposed model classes, are required to implement the Parcelable interface to be saved in the bundle.  Of all of the poor design choices by Android, I think that requiring a developer to manually save activity state, and requiring the Parcelable interface, represents the worst offense.  Here’s an example of the type of boilerplate that you’ll have to write to implement Parcelable.

[Insert Parcelable code]

It’s super painful, and its error prone.  There is no type checking on the arguments in the bundle, it's order only.  It's really heinous.  
  
I actually had a quick chat with Dianne Hackborn of the Android framework team about this subject at Google I/O this year.  She had an interesting perspective on the design.  She suggested that, if a view controller is destroyed in the background, the developer should reinitiate whatever data retrieval is required to bring back the state of the app.  In other words, the data should either be persisted, or the activity should go through whatever path it went through to create its state, such as by fetching data from a server.  To me, this was a poor design choice because Android requires the developer to be aware of the different states that a view controller can be in, and handle them accordingly, adding incidental complexity.  Furthermore, the developer has to worry about memory management, which should be a system level concern.  This recommended recreation process cannot account for state changes due to user interaction or other one-time events.  It encourages the mixing of business logic in your controllers with code that handles all of these cases.  Furthermore, serialization is often the only reason that a class need to implement Parcelable, which creates significant code bloat.  Again, iOS handles all of this (relatively) seamlessly, and I never thought about this complexity until I started doing Android. There is no equivalent of onSaveInstanceState in iOS, it is simply not needed.  

The complexity doesn't end there, a number of other quirks and gotchas await the unwary developer.  A lot of these reflect, or are consequences of Android's core poor design choices pertaining to lifecycle, to leak system concerns to the developer.  Here are the ones I can think of off the top of my head, some of which were mentioned above:

1. The hooks are not all guaranteed to be called.  A fragment or activity can go directly from onCreate to onDestroy.  onDestroy is not guaranteed.  
2. onSaveInstanceState is not always called, such as when the user “intentionally” exits the app
3. Listeners can fire in a random order
4. Google maps objects have their own separate hell of a lifecycle, which I will hopefully discuss in depth in a future blog post.  
5. Supposedly, Fragments require an empty public constructor, which is annoying and not well documented.  
6. There is no clear documentation on when other setup methods are called, like onPrepareOptionsMenu.  I ended up having to just experiment to figure out the order, and I’m not convinced that it's even deterministic.
7. There are quirks with older versions of Android, notably that onStop is not guaranteed to be called in version XXXX
8. OnClickListeners and other listeners will cause crashes if called in quick succession or after the activity is destroyed (future post)
9. Backgrounded apps brought to the foreground will often crash due to state not being persisted.  

 To sum up, when I learned iOS, I coded for months with only the barest appreciation for what was being handled for me behind the scenes.  I had no idea what the system was doing to my views when an app was backgrounded, or when the view was rotated.  And I didn't care, it just worked.  On the other hand, Android leaks these system level concerns into the app, necessitating a lot of work and careful attention to avoid crashes and UI bugs.  In the defense of the framework engineers, I will say that at least some of this complexity on Android is due to limitations imposed by Java.  Also, I'm not familiar with all of the design tradeoffs and constraints that I'm sure pushed the design in this direction.  However, with iOS as an example, in future versions of Android I would like to see reductions in the complexity of view controller life cycle.  
 
**Future posts**
  
Here’s an example of an easy way to crash an activity with click listeners.  In the following app, there are two buttons that each update their state and then navigate to the next screen.  Seems simple right?  But, if I click them both in quick succession, the app will crash!  Here’s what’s happening.  If I click on both of them, both onClickListeners will be queued on the main thread.  When the first one executes, it updates it’s view state, and then starts the navigation process to the next activity, which destroys the current activity.  Then, the next onClickListener gets executed.  When that button tries to update it’s view, it finds that the activity, along with the view, have already been destroyed.  Thus, the button doesn’t exist anymore, and the program crashes with a NullPointerException.  The only way that I’ve found to prevent this crash is to wrap every possible asynchronous call in the following conditional.  [Insert conditional] It’s ugly, annoying, and easy to forget.  I believe that this crash reflects some really poor design choices, and reflects some shortcomings of Java.  There is no equivalent idiocy on iOS.  

In fact, there are many ways to create NullPointerExceptions with Android Activities, and in my mind, it’s inexcusable that the platform makes you worry about them.  Ever had an Android app crash on you when you bring it to the foreground?  Well, the following examples shows why this happens.  

[Insert app backgrounding example]


I’ll also note that there are a number of other hooks that can be used on each platform, but their use is discouraged.  One example is Android’s OnGlobalLayoutListener which can be used to manipulate the view after the view has been laid out.  However, since Android views should be responsive, manual measurement usually isn’t necessary and this method should be used sparingly.  Also, on iOS, there is often a strong temptation to use viewDidAppear.  If you manipulate the view after the view is on screen, you'll often see weird flashes and resizing views in your app.  Unless you know what you are doing, modify views in viewWillAppear.  It's fine to do background operations that don't affect the view in viewDidAppear.