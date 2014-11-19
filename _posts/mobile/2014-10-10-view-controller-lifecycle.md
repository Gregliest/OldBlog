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

 In this post, I’m going to compare the life cycle hooks in iOS and Android, and discuss at a high level why I think iOS’s handling of view and controller life cycle is superior to Android’s.  Life cycle is a huge topic, so in the interest of brevity, I'll leave specifics for future posts.  In general, iOS provides hooks that match the life cycle of the view, which creates an easy and intuitive system that successfully hides system level concerns from the developer. On the other hand, Android tied their hooks to the state of the controller, which conflates the life cycle of the view with the state of the controller, exposing much of the complexity of the controller, and creating a confusing and quirky system.  

MVC refresher
  
To establish terminology, here's a quick description of Model-View-Controller, which is the design paradigm favored by both platforms.   In the Model-View-Controller design paradigm, a controller manages the view's state and life cycle. Controllers are also responsible for coordinating the state of their view with the rest of the app, and encompass significant complexity around memory management, view measurement and layout, and saved app states. Controllers are sometimes referred to as view controllers, but I'm going to leave out the "view" part to avoid confusion. 

The View Life Cycle

When a view is about to appear on screen, the app needs to perform the following tasks:

1. Create the view
2. Load the view into memory
3. Lay out and measure the elements of the view
4. Render the view on screen

And when the view should leave the screen:

1. Persist view's state as needed
2. Release memory

These tasks represent the major stages in a view's life cycle, and a developer might want to take action at some or all of these stages. To support these actions, each platform has provided a number of hooks in their controllers. 

The Life Cycle Hooks

Apple's life cycle hooks closely match the steps of the view life cycle.  The most common methods are:

1. ViewDidLoad
2. ViewWillAppear
3. ViewDidAppear
4. ViewWillDisappear
5. ViewDidDisappear

Here are the equivalent Android docs.  

![Android Lifecycle](/images/lifecycle/basic-lifecycle-android.png)

Here's a quick comparison of the most common lifecycle methods on the two platforms, and where they fit into the life cycle of the view.  

![View Appearing](/images/lifecycle/view-controller-appear.png)

![View Disappearing](/images/lifecycle/view-controller-disappear.png)

A Difference of Philosophy

Side by side, you can see that, while the hooks on the two platforms are roughly equivalent, the method names expose a major difference in the underlying design philosophy between the two systems.  Apple's hooks, with names like "viewDidAppear" and "ViewWillDisappear," are focused on the view and the view's state, while Android's life cycle hooks, like "onPause" and "onStop," are called when the *controller* is going to change to a different state. By abstracting on the view, iOS has created a more intuitive system that better maps to the view life cycle tasks listed above.  Furthermore, the iOS system must handle much of the controller complexity in the background because controller events are not passed to the developer, simplifying app code.  On the other hand,  Android's controller abstraction is confusing and leaks system level concerns into the realm of the developer, exposing massive complexity.  

Apple's system maps directly to the view life cycle tasks listed above, which creates an intuitive and easy to use system.  View life cycle is easy to understand because it relates to a concrete concept: a view that appears on a screen.  Furthermore, the visibility of the view is essentially binary, either the view is on screen or it is not, so there is only one, linear path to calling iOS's hooks.  There are some intermediate steps in the setup and tear down, but those steps are transient, and it's clear where they fit in the larger picture.  So, for instance, it's obvious that `viewWillAppear` will get called when the view is about to appear, there's no ambiguity.  However, since views still encompass a fair amount of complexity, there are some things to be aware of with the iOS hooks.  For instance, in `viewDidLoad`, views are not yet measured so all dimensions are 0, and other view dimensions cannot be inferred.  But, I found these quirks relatively easy to look up and understand.  Overall, the iOS hooks names clearly outline what is happening and when.

In contrast, Android's hook names are ambiguous.  By using the terminology "onEvent," Android fails to communicate whether system level events tied to that event come before or after the hook.  Furthermore, the state names themselves are not informative.  For instance, what does it mean for a controller to be "started," and how is that different than "resumed?"  What is the distinction between "paused" and "stopped?"  While the developer can look up these details, this ambiguity increases learning time and adds cognitive overhead.  Also, notice that the Android docs had to put parentheses explaining how these states match to the view life cycle, which is the more natural way to understand when these hooks will be called.  

Furthermore, controllers are inherently more difficult to understand than views, which makes Android's system fundamentally more complex than iOS's.  As shown by Android's docs, controllers can have a number of intermediate states, and there are multiple ways to enter and exit those states.  In fact, Android's documentation has a number of bullet points outlining different situations where `onStart` can be called, because there are so many possibilities.  For instance, `onStart` can be called when creating a new activity, after a phone call has interrupted the app, or when the user navigates back to the app from another app.  In iOS, the developer doesn't have to worry about any of these scenarios, only whether a view is appearing or disappearing.  Also, there are a number of situations where certain hooks are not guaranteed to be called in Android.  For instance, an activity can go directly from `onCreate` to `onDestroy`.  Furthermore, `onDestroy` is not guaranteed to be called, so it seems kind of pointless to put code there.  When deciding what code to put in which hook, the developer needs to be aware of these different situations to make an informed decision.  It's extremely complicated, and requires a significant amount of study and attention to get right. 

On the other hand, by tying hooks to the view, iOS has required that their controllers handle much of the controller complexity in the background.  The hooks only assert that the view did appear, or the view will disappear.  There is no mention of, and really no room for, a concept of why these events might have happened.  Thus, all of the complexity about whether a view should be cached, whether the system needs to reclaim memory from the controller, etc, MUST be handled by the system.  So, Apple has found a good abstraction that hides complexity.  

In contrast, by abstracting on the controller events, Android has tightly coupled the view's life cycle with the controller's life cycle.  For instance, on screen rotation, Android envisioned a scenario where a developer would want to substitute different layouts for landscape and portrait modes.  This is a noble goal, and I applaud the flexibility that it grants.  However, due to the controller abstraction, in order to change the view, the **controller** also needs to change.  In other words, to swap out the view and call all of the life cycle hooks again, the entire controller needs to be destroyed and recreated, along with all of its data and state.  In essence, by abstracting on the controller, Android has complected the state of the view and the controller, which breaks the basic abstraction of MVC.   Furthermore, if the controller is not recreated properly, the resulting view could have bugs, lose data, or even crash, even though nothing in the controller has changed.  It is stunningly bad design.   As if that's not bad enough, saving and recreating a view can be heinously difficult, which I will cover in a separate blog post.  

Unfortunately, the complexity on Android doesn't end there, a number of other life cycle quirks and gotchas await the unwary Android developer.  A lot of these reflect, or are consequences of Android's core poor design choice to abstract on the controller level.  Here are the ones I can think of off the top of my head, some of which were mentioned above:

1. onSaveInstanceState is not always called, such as when the user “intentionally” exits the app.
2. Listeners can fire in a random order.
3. Google maps objects have their own separate hell of a life cycle.  
4. Supposedly, Fragments require an empty public constructor, which is annoying and not well documented.  
5. There is no clear documentation on when other setup methods are called, like onPrepareOptionsMenu.  I ended up having to experiment to figure out the order, and I’m not convinced that it's deterministic.
6. There are quirks with older versions of Android, notably that onStop is not guaranteed to be called in early versions of Android.
7. OnClickListeners and other listeners will cause crashes if called in quick succession or after the activity is destroyed.
8. Backgrounded apps brought to the foreground will often crash due to state not being persisted.  
9. There are often multiple ways that a hook can be called, and the developer needs to be aware of the differences in certain situations. 
10. The developer has to be aware that the system can destroy activities in the background.  
11. Fragments have their own set of life cycle methods that are interleaved with the hooks of the parent activity.  

Some of these points represent major failures of design, while others merely add complexity.  There's a lot more I could say about the complexity of Android's view life cycle, but I'll leave those discussions for future blog posts.  If there's anything in particular you would like to read more about, shoot me an email and I'll do my best to put together a post on that subject.  

To sum up, when I learned iOS, I coded for months with only the barest appreciation for what was being handled for me behind the scenes.  I had no idea what the system was doing to my views and controllers when an app was backgrounded, or when the app was rotated.  And I didn't care, I put my code in the logical life cycle hook, and it just worked.  On the other hand, by requiring the developer to be aware of the controller's state, Android adds a massive amount of incidental complexity to their apps, resulting in a confusing and quirky system.  From personal experience, eight months into the development of my first Android app, I was still encountering issues, edge cases, and crashes pertaining to lifecycle. I was still referring to documentation, and discovering new, important details about how the system worked.  I find that in general, Android developers must devote a lot of work and careful attention to handle all of the cases properly, to avoid crashes and UI bugs. So, with iOS as an example, I would like to see reductions in the complexity of view and controller life cycle in future versions of Android.  


