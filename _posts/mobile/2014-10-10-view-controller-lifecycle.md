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

 In this post, I’m going to compare the life cycle hooks in iOS and Android, and discuss why I think iOS’s handling of view controller lifecycle is superior to Android’s.  This article is not meant as a tutorial, so I'm not going to go in depth into how to use each lifecycle method, except to draw comparisons.  

The Basics

When a view is about to appear on screen, the app needs to perform the following tasks:

1. Create the view controller
2. Load the view into memory
3. Lay out and measure the elements of the view
4. Show the view on screen

And when the view should leave the screen:

1. Hide the view/save state
2. Release memory

These tasks represent the major stages in a view controller's life cycle, and a developer might want to take action at some or all of these stages.  To allow such actions, each platform has provided a number of hooks in their view controllers that are called at each stage of the view controllers life cycle.  These life cycle stages encompass a lot of complexity, including complexity around memory management, view measurement and layout, and saved app states. For instance if a view goes off screen, should the view be cached?  Should it's state be persisted?  What happens if a bunch of views are saved in memory, but system resources run low: does the system destroy views in the background to reclaim memory?  The app has to handle all of these cases and more.  In the following discussion, I'll show why Apple's handling of this complexity is superior to Android's.  

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

Side by side, you can see that, while the methods on the two platforms are roughly equivalent, the method names expose a major difference in the underlying design philosophy between the two systems.  Apple's life cycle hooks, with names like "viewDidAppear" and "ViewWillDisappear" are focused on the view and the view's state, while Android's life cycle hooks like "onPause" and "onStop" are called when the view *controller* is going to change state, such as to "paused" or stopped.   There's no mention of the view at all.  In MVC, a view should be just the representation of the app on screen.  While it is stateful, that state and all of its inherent complexity are managed by the view controller.  Furthermore, the view controller is responsible for coordinating with the rest of the app, including app backgrounding and memory management.   Thus, views are a lot simpler than view controllers, and I believe that this philosophical difference is the fundamental reason that iOS handles life cycle more cleanly than Android.  

In fact, we can see the difference between a view abstraction vs. a view controller abstraction in practice.  As you can see, for the most part in iOS, the developer doesn't need to worry about what is happening to the view controller.  The developer is only responsible for performing the necessary actions to show the view on screen, and the system handles the complexity around saved states, foregrounding, backgrounding.  On the other hand, because Android developers must take action according to the state of the view controller, they assume responsibility for the complexity and the edge cases associated with the tasks of the view controller.  This responsibility leaks system level concerns into view controller code, and represents a massive increase in the complexity and responsibilities of the app code.  From personal experience, eight months into the development of my first Android app, I was still encountering issues, edge cases, and crashes pertaining to lifecycle.  I was still having to refer to lifecycle documentation, and it seemed like every week I was discovering a new hook.  On the other hand, I coded in iOS for months without appreciating what was being handled for me in the background.  It just worked. 

The Android Morass

As if Android lifecycle isn't confusing enough, Android has an additional set of lifecycle hooks for Fragments that are interleaved with the hooks of the parent activity, as outlined below.  

![Android Lifecycle](/images/lifecycle/fragment-lifecycle-android.png)

Confusing, no?  

To compound the bad design in Android, the handling of the memory is extremely clunky.  The seemingly innocuous hook, onSaveInstanceState, opens a world of hurt.  It turns out that, in addition to requiring a developer to manually save the state of a view controller, Android has not provided a reasonable way of doing so.  To be saved, a view controller must serialize itself.  Unfortunately, only primitives can be readily saved.  Higher level objects, including your nicely decomposed model classes, are required to implement the Parcelable interface to be saved in the bundle.  Of all of the poor design choices by Android, I think that requiring a developer to manually save activity state, and requiring the Parcelable interface, represents the worst offense.  Here’s an example of the type of boilerplate that you’ll have to write to implement Parcelable, taken from Android's documentation.

{% highlight java %}
 public class MyParcelable implements Parcelable {
     private int mData;

     public int describeContents() {
         return 0;
     }

     public void writeToParcel(Parcel out, int flags) {
         out.writeInt(mData);
     }

     public static final Parcelable.Creator<MyParcelable> CREATOR
             = new Parcelable.Creator<MyParcelable>() {
         public MyParcelable createFromParcel(Parcel in) {
             return new MyParcelable(in);
         }

         public MyParcelable[] newArray(int size) {
             return new MyParcelable[size];
         }
     };
     
     private MyParcelable(Parcel in) {
         mData = in.readInt();
     }
 }
{% endhighlight %}

Yes, all of this code is required to implement Parcelable for a class with a single field.  It’s super painful, error prone, and leads to significant code bloat.  Also, if one of the fields is not a primitive, that class also needs to implement Parcelable.  Furthermore, the parcel works on order only, so if you have two ints in a row, and you mix up the order, the ints will be assigned to the wrong variable, leading to really insidious bugs.  It's heinous that such a low level library is the state of the art in a modern system. There is, however, a slightly safer way to do this.  I don't always implement Parcelable, but when I do, I use a bundle, because it at least provides named parameters.  

{% highlight java %}
 public class MyParcelable implements Parcelable {
     private int mData;
     
    private static final String MDATA = "mData";

     public void writeToParcel(Parcel out, int flags) {
        Bundle bundle = new Bundle();
        bundle.putInt(MDATA, data);
        out.writeBundle(bundle);
     }
     
     private MyParcelable(Parcel in) {
        Bundle bundle = parcel.readBundle();
        mData = bundle.getInt(MDATA);
     }
     
	// Other methods omitted
 }
{% endhighlight %}

Using a bundle introduces even more code bloat, but at least it's readable, and you're not going to accidentally introduce a bug by changing the order.  It's worth noting that iOS has a rough equivalent to onSaveInstanceState, didReceiveMemoryWarning, but for the most part it isn't used in ARC systems.  Apple has done a good job handling memory issues in the background.  
  
I actually had a quick chat with Dianne Hackborn of the Android framework team about saving app state at Google I/O this year.  She had an interesting perspective on view controller persistence.  She suggested that, if a view controller is destroyed in the background, the developer should reinitiate whatever data retrieval is required to recreate the state of the app.  In other words, the data should either be persisted, or the activity should go through whatever path it went through to create its state in the first place, such as by fetching data from a server.  However, this design requires the developer to be aware of the different states that a view controller can be in and handle them accordingly, which adds significant incidental complexity.  Furthermore, the developer has to worry about memory management, which should be a system level concern.  Finally, this recommended recreation process cannot account for state changes due to user interaction or other one-time events.  In essence, this added complexity encourages the mixing of business logic in your controllers with code that handles all of these cases.  Furthermore, serialization is often the only reason that a class need to implement Parcelable, which creates significant code bloat.  In the defense of the framework engineers, I'm sure that at least some of this complexity on Android is due to limitations imposed by Java.  Also, I'm not familiar with all of the design tradeoffs and constraints that I'm sure pushed the design in this direction.  

However, the complexity doesn't end there, a number of other quirks and gotchas await the unwary developer.  A lot of these reflect, or are consequences of Android's core poor design choices pertaining to lifecycle, to leak system concerns to the developer.  Here are the ones I can think of off the top of my head, some of which were mentioned above:

1. The hooks are not all guaranteed to be called.  A fragment or activity can go directly from onCreate to onDestroy.  onDestroy is not guaranteed.  
2. onSaveInstanceState is not always called, such as when the user “intentionally” exits the app
3. Listeners can fire in a random order
4. Google maps objects have their own separate hell of a lifecycle, which I will hopefully discuss in depth in a future blog post.  
5. Supposedly, Fragments require an empty public constructor, which is annoying and not well documented.  
6. There is no clear documentation on when other setup methods are called, like onPrepareOptionsMenu.  I ended up having to experiment to figure out the order, and I’m not convinced that it's deterministic.
7. There are quirks with older versions of Android, notably that onStop is not guaranteed to be called in version XXXX
8. OnClickListeners and other listeners will cause crashes if called in quick succession or after the activity is destroyed (future post)
9. Backgrounded apps brought to the foreground will often crash due to state not being persisted.  

 To sum up, when I learned iOS, I coded for months with only the barest appreciation for what was being handled for me behind the scenes.  I had no idea what the system was doing to my views when an app was backgrounded, or when the view was rotated.  And I didn't care, it just worked.  On the other hand, by requiring the developer to be aware of the view controller's state, Android leaks these system level concerns into the app, necessitating a lot of work and careful attention to avoid crashes and UI bugs.  With iOS as an example, I would like to see reductions in the complexity of view controller life cycle in future versions of Android .  
  
**Future posts**
  
Here’s an example of an easy way to crash an activity with click listeners.  In the following app, there are two buttons that each update their state and then navigate to the next screen.  Seems simple right?  But, if I click them both in quick succession, the app will crash!  Here’s what’s happening.  If I click on both of them, both onClickListeners will be queued on the main thread.  When the first one executes, it updates it’s view state, and then starts the navigation process to the next activity, which destroys the current activity.  Then, the next onClickListener gets executed.  When that button tries to update it’s view, it finds that the activity, along with the view, have already been destroyed.  Thus, the button doesn’t exist anymore, and the program crashes with a NullPointerException.  The only way that I’ve found to prevent this crash is to wrap every possible asynchronous call in the following conditional.  [Insert conditional] It’s ugly, annoying, and easy to forget.  I believe that this crash reflects some really poor design choices, and reflects some shortcomings of Java.  There is no equivalent idiocy on iOS.  

In fact, there are many ways to create NullPointerExceptions with Android Activities, and in my mind, it’s inexcusable that the platform makes you worry about them.  Ever had an Android app crash on you when you bring it to the foreground?  Well, the following examples shows why this happens.  

[Insert app backgrounding example]


For instance, the onPause hook can be called for a number of reasons: the app has been backgrounded, the user has closed the app, another view has appeared, etc.  Sometimes, this method can be followed by a call to onDestroy, but that's not necessarily the case.  Sometimes, the app might have to save itself, signaled by another hook, onSaveInstanceState.  But onSaveInstanceState is not guaranteed to be called, for instance if the user "intentionally exits the view," whatever that means.  The developer must be aware of these different cases when deciding what to put in what hook.  

The optimizations that Android made to reclaim memory and CPU from backgrounded views leak out into the realm of the developer, causing numerous crashes and introducing significant complexity. 
 Android envisioned view controllers that are active on screen, paused while something else is in the foreground, stopped, such as if the app is backgrounded, or destroyed such that it has to be recreated from scratch.  To reflect these states, they provided hooks like onPause and onStop.  These hooks define a set of states that a view controller can exist in with multiple entry points, which is fundamentally different than the linear approach favored by Apple.  

I’ll also note that there are a number of other hooks that can be used on each platform, but their use is discouraged.  One example is Android’s OnGlobalLayoutListener which can be used to manipulate the view after the view has been laid out.  However, since Android views should be responsive, manual measurement usually isn’t necessary and this method should be used sparingly.  Also, on iOS, there is often a strong temptation to use viewDidAppear.  If you manipulate the view after the view is on screen, you'll often see weird flashes and resizing views in your app.  Unless you know what you are doing, modify views in viewWillAppear.  It's fine to do background operations that don't affect the view in viewDidAppear.