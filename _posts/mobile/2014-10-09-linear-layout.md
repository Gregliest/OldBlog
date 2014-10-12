---
layout: article
title: "Linear Layout"
modified:
categories: mobile
excerpt:
tags: [android]
---


Building a view is one of the core tasks of a mobile developer.  I find that Android makes building views much easier than iOS does.   For instance let’s say you have a view and think, “I want a bunch of stuff to show up on screen, one after another.”  On Android, enter the Linear Layout.  It maps directly to this thought.  On the other hand, iOS   

Here’s how a Linear Layout works.  Let’s say you have the following three items you want to put on screen, an image, a button, and a text field.  All you have to do is make a linear layout, and plunk the three items in it in the order you want them to appear.  I’ve done so in the following code example:

{% highlight css %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
                             android:layout_width="fill_parent"
                             android:layout_height="fill_parent">

 <TextView
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:text="Canyon Creek, CA"/>

<ImageView
        android:layout_width="fill_parent"
        android:layout_height="300dp"
        android:src="@drawable/CanyonCreek"/>
<Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="New Button"/>

</LinearLayout>
{% endhighlight %}

On iOS, Interface Builder makes setting up the basic view quite easy.  I’m going to use Interface Builder whenever possible because I think it’s easier (future blog post subject) and it’s also the way the Apple is encouraging their views to be built.  To build the same view we built in Android, you simply drag the elements you want onto the screen.  Here’s an example.  

![The Basic Layout](/images/ios/basic-lined-up-layout.png)


Now, we have the elements showing on both apps, but there are a number of things that I can do to make them look better. I don’t like the way the text is cut off in the text field in iOS.  So far, I’ve just relied on auto layout to infer what I want from where and how I dragged the elements onto the screen.  In my experience, this doesn’t usually work too well, especially if the screen gets rotated.  Also, I don’t have pixel control over the spacing, widths, and heights.  So, I’m going to define the view using constraints, to gain some level of control, as follows.

![The Basic Layout, with Constraints!](/images/ios/basic-layout-constraints.png)

I’m not going to go into detail about what I did here; there are a number of existing tutorials on how to use Interface Builder.  However, I will point out the number of constraints required to gain control of the view.  I needed to add 8 constraints to do what Android's Linear Layout did for free.  By using the “Center X Alignment” constraint relative to the left margin, I picked one of the simpler ways to constrain this view, but there are many others.  For instance, I could have defined the horizontal position of each view as a distance from the left margin, or I could have chained the Center X Alignments together, so that each view is center aligned with the one above it.  Each of these has its disadvantages which will become apparent shortly, but the point here is that some thought needs to go into how you constrain your view in iOS.  Also, I find it difficult to determine which constraint in the left bar corresponds to which constraint on the screen.  Go ahead and try to build this view on your own, chances are you’ll have to do some reordering and futzing to get the constraints right.  

In Android, I gained control over all of the dimensions above using simple properties in the xml.  Width and height are built in.  I used fill_parent on the LinearLayout to make it fill the screen.  For the ImageView, I manually set the height.  I also set the source file and the scaling type for the image view, all super easy to understand.  I center aligned everything using “android:gravity.” The one quirk I will note about this view pertains to gravity vs. layout_gravity.  “gravity” dictates the placement of the … 

{% highlight css %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
                             android:layout_width="fill_parent"
                             android:layout_height="fill_parent">

 <TextView
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:gravity="center_horizontal"
        android:text="Canyon Creek, CA"/>

<ImageView
        android:layout_width="fill_parent"
        android:layout_height="300dp"
        android:gravity="center_horizontal"
        android:scaleType="fitCenter"
        android:src="@drawable/CanyonCreek"
        android:id="@+id/imageView"/>
<Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="New Button"
        android:layout_gravity="center_horizontal"
        android:id="@+id/button"/>

</LinearLayout>
{% endhighlight %}

Android’s Linear Layout abstraction is already starting to show its value.  

However, views are not static, and chances are that you won’t get a view right the first time.  You’ll want to add or remove views, or resize them.  The real power of the Linear Layout becomes apparent when as you try to change the view.  Let’s try adding a caption text field between the picture and the button.   In Android in a Linear Layout, simply add the view in the position you want it to appear.  On iOS, things are a bit harder.  You need to break a constraint, add the view, add new constraints, and make them equal to one another.  You’ll also need to tell IB to update its layout.  Getting harder…  To remove a view, simply remove the view from the Linear Layout.  In iOS, you’ll have to delete it, make a new constraint, and make the constraint be equal to the others.  See a pattern?  

Now let’s say you want a view to appear conditionally.  In Android, simply set Visibility:gone if you want the view to be completely gone, or Visibility:invisible if you want the view to be invisible but retain its dimensions.  In iOS, setting the view to be hidden, is the equivalent of “invisible” in Android: constraints are still measured off of that hidden view.  But, the comparison stops there.  there are a number of ways to make views behave equivalently to “gone,” and all of them are  awkward in my opinion.  Here are a couple.  You can set the view to hidden, but constraints are still measure off that view.  You need to change both of the constraints of the hidden view to make the rest of the view line up correctly.  You can also set the height constraint of the view to be zero.  However, you also need to change one of the constraints so that there isn’t an extra space.  Also, be aware that the view will still be drawn behind the view below it, which can incur a performance penalty, and also means that if that view is transparent, the hidden view will be visible, like so.  

Let’s take this one step further, lets say that you want this section of the view to remain the same height, and the spacing between the elements to increase to compensate.  In Android, simply peg the height of the Linear Layout, and you get the rest for free.  However, in iOS, this is downright difficult to accomplish.  Here’s an attempt.  

Let’s say you want to resize one of the views in your layout, for instance, the text field has a varying number of lines of text.  Both systems handle this gracefully, everything will just move to accommodate.  However, let’s say that you want the height of the whole layout to remain the same.  Android handles this for free as follows.  The controls for how the views will resize are intuitive (talk about match_parent, center, etc).   On the other hand, this is difficult in iOS.  Here’s one way to do it.  

Conditions
1. Add/Remove a view permanently
2. Remove a view conditionally
3. Resize a view 
4. Resize the whole view

There are two conditions under which I evaluate an abstraction, is it easy to learn, and is it easy to use once you’ve learned it.  The Linear Layout is superior to iOS constraints in both regards.  I intentionally hedged in a lot of the iOS examples by saying, “here’s one way to do it,” because often there are multiple ways of accomplishing the same thing, and NONE of them are easy.  Often different code bases will handle views differently, and even different developers in the same codebase will use different paradigms.  This inconsistency leads to difficult to understand view hierarchies.  On the other hand, although there are also multiple ways of doing the same thing in Android, there is usually one easiest way, which encourages consistency from developers, and enhances readability.  

To summarize, the linear layout is a fantastic abstraction for building views.  It allows for easy translation between a design and the layout code.  Also, by encapsulating and abstracting the constraint logic, it allows for much more flexible and malleable views.  by encouraging views to be grouped into logical blocks, Android helps us to design views that can be easily moved, reused, and even subclassed.  On the other hand, iOS’s view handling is clunky and confusing.  I know all of the examples here favored Android, and pointed out deficiencies in iOS.  I honestly couldn’t think of a situation having to do with simple layouts that iOS was easier than Android.  I also didn’t go into common bugs and mistakes, but this is a good time to mention a few.  Interface Builder has a number of quirks.  One is its insistence on being over constrained when you’re trying to define how a view will change.  It can be non intuitive to have to change the priority on constraints, and in a mess of constraints, it can be extremely difficult to determine what elements are dictating an attribute of the layout, like width or height.  Furthermore, views without a defined height (read, a single omitted constraint) will often just not show up, or will show up behind other views.  

In a future blog post I’ll go into more complex view handling, and how iOS’s quirks and inconsistencies become more apparent, and more cumbersome.  




Now, this is the simplest possible example.  As you build more and more complex views, the quirks and inconsistencies in iOS’s view handling become more and more apparent, and more and more frustrating.  (example with a bunch of ways of doing the same thing, each overwriting the last).  The multitude of view controls in iOS can make it extremely difficult to come into an existing code base and figure out where something in the view is getting set.  It can also make debugging a view a nightmare.  Finally, the poor handling of constraints in iOS can make it really painful to change views.  I’ve had situations where all I’ve wanted to do is replace a button or something like that, and have had to tear down and rebuild the majority of the constraints in the view.  

 