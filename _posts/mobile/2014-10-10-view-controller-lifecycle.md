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

 In this post, I’m going to compare the life cycle hooks in iOS and Android, and discuss why I think iOS’s handling of view and controller lifecycle is superior to Android’s.  This article is not meant as a tutorial, so I'm not going to go in depth into how to use each lifecycle method, except to draw comparisons.  

The Basics

When a view is about to appear on screen, the app needs to perform the following tasks:

1. Create the view
2. Load the view into memory
3. Lay out and measure the elements of the view
4. Render the view on screen

And when the view should leave the screen:

1. Persist view's state as needed
2. Release memory

These tasks represent the major stages in a view's life cycle, and a developer might want to take action at some or all of these stages.  In the Model-View-Controller design paradigm, a controller manages the view's state and life cycle. So, to allow developers to control the view, each platform has provided a number of hooks in their controllers that are called at each stage in the view's life cycle. Controllers are also responsible for coordinating the state of their view with the rest of the app, and encompass significant complexity around memory management, view measurement and layout, and saved app states. 

The Life Cycle Hooks

Apple's life cycle hooks closely match the steps outlined above.  The most common methods are:

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

Side by side, you can see that, while the hooks on the two platforms are roughly equivalent, the method names expose a major difference in the underlying design philosophy between the two systems.  Apple's hooks, with names like "viewDidAppear" and "ViewWillDisappear," are focused on the view and the view's state, while Android's life cycle hooks, like "onPause" and "onStop," are called when the *controller* is going to change to a different state. There's no mention of the view at all. This difference is not just linguistic, it turns out that iOS has in fact matched their hooks to the state of the view, while Android has abstracted on the state of the controller.  Thus, iOS has created a more intuitive system that better maps to the view life cycle tasks listed above.  Furthermore, by abstracting on views, Apple's framework engineers have required that their system to handle much of the controller complexity in the background.  On the other hand,  Android's controller abstraction is confusing and leaks system level concerns into the realm of the developer, exposing massive complexity.  

By abstracting on the view, iOS has created an intuitive and easy to use system.   At risk of stating the obvious, by abstracting on the view, iOS has created a system that maps directly to the view life cycle.  For instance, it's obvious that `viewWillAppear` will get called when the view is about to appear.  Views still encompass a fair amount of complexity, so there are some things to be aware of with the iOS hooks.  For instance, views are not yet measured in `viewDidLoad`, so if you ask a view for a dimension, it will always return 0.  But, I found these quirks relatively easy to look up and understand.  

On the other hand, I find that many of the basic lifecycle hooks in Android are tied to confusing and poorly named events.   What does it mean for a controller to be "started," and how is that different than "resumed?"  What's the distinction between "paused" and "stopped?"  Notice that the Android docs had to put parentheses explaining how these states match to the view life cycle in their documentation.  I find resumed to be a terrible name, it implies that a view is always coming back to the foreground, even if it has just been created.  Also, by using the terminology "onEvent," Android does not make it clear from the hook name whether the hook is called before or after the event.  One has to refer to the docs to realize that they are always called before the state change.  This also does not inform us as to whether system level events tied to that event come before or after the hook.  This naming is in contrast to iOS's names like `viewDidAppear`, which make it very clear that the event already happened and the system took care of its shit already.  

By tying hooks to the view, iOS has required that their controllers handle much of the controller complexity in the background.  The visibility of the view is essentially binary, either the view is on screen or it is not, so there is only one conceptual path to calling iOS's hooks.  The hooks only assert that the view did appear, or the view will disappear.  There is no mention of, and really no room for, a concept of why these events might have happened.  This is the mark of a good abstraction.  All of the complexity about whether a view should be cached, whether the system needs to reclaim memory from the controller, etc, MUST be handled by the system, because it has boiled down the hooks to be called on view events only.  

On the other hand, by abstracting on the controller events, Android has tightly coupled the view's life cycle with the controller's life cycle.  For instance, on screen rotation, Android envisioned a scenario where a developer would want to substitute a different layouts for landscape and portrait modes.  This is a noble goal, and I applaud the flexibility that it grants.  However, due to the controller abstraction, in order to change the view, the **controller** also needs to change.  In other words, to swap out the view and call all of the life cycle hooks again, the entire controller needs to be destroyed and recreated, with all of its data and state.  If it is not recreated properly, the resulting view will have bugs, lose data, or even crash, even though conceptually nothing in the controller has changed.  As if that's not bad enough, saving and recreating a view can be heinously difficult, which I will cover in a separate blog post.  In essence, by abstracting on the controller, Android has tied together the state of the view and the controller, which breaks the basic abstraction of MVC.  

Unfortunately, the complexity doesn't end there, a number of other life cycle quirks and gotchas await the unwary Android developer.  A lot of these reflect, or are consequences of Android's core poor design choice to abstract on the controller level.  Here are the ones I can think of off the top of my head, some of which were mentioned above:

1. The hooks are not all guaranteed to be called.  A fragment or activity can go directly from onCreate to onDestroy.  onDestroy is not guaranteed.  
2. onSaveInstanceState is not always called, such as when the user “intentionally” exits the app.
3. Listeners can fire in a random order.
4. Google maps objects have their own separate hell of a life cycle.  
5. Supposedly, Fragments require an empty public constructor, which is annoying and not well documented.  
6. There is no clear documentation on when other setup methods are called, like onPrepareOptionsMenu.  I ended up having to experiment to figure out the order, and I’m not convinced that it's deterministic.
7. There are quirks with older versions of Android, notably that onStop is not guaranteed to be called in early versions of Android.
8. OnClickListeners and other listeners will cause crashes if called in quick succession or after the activity is destroyed.
9. Backgrounded apps brought to the foreground will often crash due to state not being persisted.  
10. There are often multiple ways that a hook can be called, and the developer needs to be aware of the differences in certain situations. 
11. The developer has to be aware that the system can destroy activities in the background.  
12. Fragments have their own set of life cycle methods that are interleaved with the hooks of the parent activity.  

Some of these points represent major failures of design, while others merely add complexity.  There's a lot more I could say about the complexity of Android's view life cycle, but I'll leave those discussions for future blog posts.  I considered doing an analysis of Android's life cycle docs to point out the complexity that they explicitly call out, and also the complexity that they gloss over.  I also considered centering this post around the system level concerns that Android's abstraction leaks into their apps, but I couldn't put together a coherent enough argument.  If there's anything in particular you would like to read more about, shoot me an email and I'll do my best to put together a post on that subject.  

 To sum up, when I learned iOS, I coded for months with only the barest appreciation for what was being handled for me behind the scenes.  I had no idea what the system was doing to my views and controllers when an app was backgrounded, or when the app was rotated.  And I didn't care, I put my code in the logical life cycle hook, and it just worked.  On the other hand, by requiring the developer to be aware of the controller's state, Android adds a massive amount of incidental complexity to their apps, resulting in a confusing and quirky system.  From personal experience, eight months into the development of my first Android app, I was still encountering issues, edge cases, and crashes pertaining to lifecycle. In general, Android developers must devote a lot of work and careful attention to handle all of the cases properly, and avoid crashes and UI bugs. So, with iOS as an example, I would like to see reductions in the complexity of view controller life cycle in future versions of Android.  


