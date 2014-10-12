---
layout: article
title: "View Controller Lifecycle"
modified:
categories: mobile
excerpt:
tags: []
image:
  feature:
  teaser:
  thumb:
date: 2014-10-10T08:31:09-07:00
---

In this post, I’m going to compare the view controller life cycles of iOS and Android.  I’ll compare the hooks that each platform gives a developer to modify behavior at different stages during the process of creating or destroying a view.  I’ll discuss what happens in different cases like app backgrounding.  I’ll also discuss why I think iOS’s handling of lifecycle is superior to Android’s.  I’m going to be harsh with Android, but to be fair, it’s a hard problem with a lot of complexity.  But, iOS did it gracefully and Android did not.  

The Basics

The basic life cycle of a view controller that is asked to appear on screen is as follows

1. Created
2. View loaded
3. Views laid out
4. View on screen.

And when it is asked to leave the screen

1. Disappear/save state
2. Release memory

Both platforms give the developer a set of hooks to manipulate the controller and its view at certain points in the creation and destruction process.  I’ve outlined the most common methods in the diagram below.

[Insert lifecycle diagrams]

I’ll also note that there are a number of other hooks that can be used on each platform, but their use is discouraged.  One example is Android’s OnGlobalLayoutListener which can be used to manipulate the view after the view has been laid out.  However, since Android views should be responsive, usually manual measurement isn’t necessary and this method should be used sparingly.  Android also has an additional set of lifecycle hooks for Fragments, as outlined below.  

[Insert Fragment diagram]

The point of all of these methods is to handle the complexity surrounding view controllers.  View controllers on both platforms are asked to handle a lot of cases, caching views that go off screen, app backgrounding, app suspend, etc.  Android views are also designed to save themselves if they are on screen, but something else comes to the foreground.  For instance, if another Fragment becomes active.  In general, I find that iOS handles this complexity much more gracefully.  You just need to learn the basics of what method happens when, and you’re off to the races.  Just put the method you want in the right place and forget about it.  

On the other hand, Android exposes a lot of the complexity, especially around views getting backgrounded and memory management.  The optimizations that Android made to conserve memory and CPU leak out into the realm of the developer in the form of additional hooks like onSaveInstanceState, and numerous crashes.  

Here’s an example of an easy way to crash an activity with click listeners.  In the following app, there are two buttons that each update their state and then navigate to the next view on click.  Seems simple right?  But, if I click them both in quick succession, the app will crash!  Here’s what’s happening.  If I manage to click on both of them, both onClickListeners will be queued on the main thread.  Let’s say one of them executes.  It updates it’s view state, and then starts the navigation process to the next activity, which destroys the current activity.  Then, the next onClickListener gets executed.  When that button tries to update it’s view, it finds that its activity, along with the view, has already been destroyed.  Thus, the button doesn’t exist anymore, and the program crashes with a NullPointerException.  The only way that I’ve found to prevent this crash is to wrap every possible asynchronous call in the following conditional.  [Insert conditional] It’s ugly, annoying, and easy to forget.  I believe that this crash reflects some really poor design choices.  There is no equivalent idiocy on iOS.  

In fact, there are many ways to create NullPointerExceptions with Android Activities, and in my mind, it’s inexcusable that the platform makes you worry about them.  Ever had an Android app crash on you when it’s been in the background and you bring it back to the foreground?  Well, the following examples shows why this happens.  
[Insert app backgrounding example]
Furthermore, the handling of the memory is extremely clunky, requiring objects to implement the Parcelable interface to save themselves.  Parcelable is a world of pain, especially with big model classes.  Here’s an example of the type of boilerplate that you’ll have to write to implement Parcelable.  It’s super painful.  I actually had a chat with Dianne Hackborn of the Android framework team about this subject.  She had an interesting angle on the design.  She suggested that, when the data is destroyed, the developer should reinitiate whatever data retrieval is required to bring back the state of the app.  In other words, the data should either be persisted, or the activity should go through whatever path it went through to create its state, such as by fetching data from a server.  To me, this was a poor design choice because Android makes the developer worry about memory management, which is a system level concern.  It encourages the mixing of business logic in your controllers with the bs that the system imposes.  Furthermore, it often is the only reason that a class need to implement Parcelable, which creates code bloat and complexity.  As a side note, iOS handles all of this (relatively) seamlessly, and I never thought about this complexity until I started doing Android..  

As a final note, the Android lifecycle has even more complexity than might be apparent from a first glance at the lifecycle hooks.  There are a number of other quirks and gotchas.  Here are the ones I can think of off the top of my head:

1. The hooks are not all guaranteed to be called.  A fragment or activity can go directly from onCreate to onDestroy.  onDestroy is not guaranteed.  
2. onSaveInstanceState is not always called, such as when the user “intentionally” exits the app
3. Listeners can fire in a random order
4. Google maps objects have their own separate hell of a lifecycle, which I will hopefully discuss in depth in a future blog post.  
5. Supposedly, Fragments require an empty public constructor - ick.  And it’s not well documented.  
6. There is no clear documentation on when other setup methods are called, like onPrepareOptionsMenu.  I ended up having to just experiment to figure out the order, and I’m sure it’s not deterministic.
7. There are quirks with older versions of Android, notably that onStop is not guaranteed to be called in version XXXX

 To sum up, I learned on iOS and I coded for months with only the barest appreciation for what was being handled for me behind the scenes.  On the other hand, Android makes it glaringly obvious what is going on, because it doesn’t handle it for you.  Android leaks system level concerns into the app, necessitating a lot of work and careful attention to avoid crashes and UI bugs.  

