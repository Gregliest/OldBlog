---
layout: article
title: "Linear Layout"
modified:
categories: mobile
excerpt:
tags: [android, iOS, mobile, layout]
image:
  feature:
  teaser: /linear-layout/android-buttons-weight.png
  thumb: /linear-layout/android-buttons-weight.png
---

In this blog post, I'm going discuss why layouts are easier to manage in Android than in iOS.  In general, iOS's constraint system is too low level, and does not abstract away the complexity around laying out and managing a view.  Developers can quickly find themselves mired in a web of constraints, and the resulting views are difficult and tedious to change.  On the other hand, Android's Linear Layout provides a useful abstraction that hides much of the complexity around simple layouts.  Furthermore, Android's readable xml layout files greatly simplify view hierarchies by allowing developers to create organized, reusable, and flexible layouts.

The Basic Layout

Let’s build a new view with a text field, an image, and a button, one after another.  In Android, all I have to do is make a Linear Layout, insert the three items in the order I want them to appear, and voila!  

{% highlight css %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
                             android:orientation="vertical"
                             android:layout_width="fill_parent"
                             android:layout_height="fill_parent">

<TextView
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:text="Canyon Creek, CA"/>

<ImageView
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:src="@drawable/CanyonCreek"/>

<Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Button 1"/>

</LinearLayout>
{% endhighlight %}

![The Basic Android Layout](/images/linear-layout/android-initial.png)

On iOS, the basic view setup is also quite easy.  I’m going to use Interface Builder whenever possible because it is Apple's preferred view construction tool and because I find the code constraint language to be obtuse and unreadable.  To obtain the same view we built in Android, you simply drag the elements you want onto the screen.  Here’s an example.  

![The Basic Layout](/images/ios/basic-lined-up-layout.png)

Note that auto layout will automatically infer a set of constraints to define how the view should change on rotation or resize.  One can build an app without ever touching constraints, which is handy for prototypes. However, in my experience, the resulting view usually has a few bugs.  Also, auto layout does not afford pixel control over the spacing, widths, and heights.  So, to make a more professional layout, I'm going to add constraints.  I will not go into detail here; there are a number of existing tutorials on how to use Interface Builder.  

![The Basic Layout, with Constraints!](/images/ios/basic-layout-constraints.png)

As you can see, I had to add eight constraints to do what Android did for free, adding significant complexity and code bloat.  As a view gets more complex and the number of constraints multiplies, this system quickly becomes unwieldy.  It can be really difficult to determine how a view element's position is being defined when there are dozens of constraints, and other elements are being dynamically resized.  Also, with so many constraints floating around, there are numerous ways to constrain the view, and the developer is responsible for choosing the best one.  For instance, to center each view, I chose the “Center X Alignment” constraint relative to the parent view. But, I could have defined the horizontal position of each view as a distance from the left margin, or I could have chained the Center X Alignments together, so that each view is center aligned with the one above it.  I chose relative to parent because the simplest layouts use constraints that are defined relative to the parent view whenever possible.  That way, constraints are not chained off of other elements in the view, and it is easier to add/remove views.  Regardless of the choices you make, constraints require a fair amount of thought, and can be very time consuming to get right. 

In contrast to the web of constraints in IB, the readability of the Android xml layout file allows me to use simple attributes to build clean, easy to follow, and encapsulated view elements.  For instance, I used `fill_parent` on the width and height attributes of the LinearLayout to fill the screen, and set some margins on the views to fix some spacing issues.  I set the source file and the scaling type for the image view inside their respective xml tags, where the attributes are easy to find and understand. 

{% highlight css %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
                             android:orientation="vertical"
                             android:layout_width="fill_parent"
                             android:layout_height="fill_parent">

<TextView
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="15dp"
        android:layout_marginBottom="15dp"
        android:gravity="center_horizontal"
        android:text="Canyon Creek, CA"/>

<ImageView
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:gravity="center_horizontal"
        android:scaleType="fitCenter"
        android:src="@drawable/CanyonCreek"/>

<Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="15dp"
        android:layout_gravity="center_horizontal"
        android:text="Button 1"/>

</LinearLayout>
{% endhighlight %}

![The Improved Android Layout](/images/linear-layout/android-improved.png)

The one quirk I will note about this view is the use of `gravity` vs. `layout_gravity`.  `gravity` dictates the placement of the elements of a view inside its frame, while `layout_gravity` controls the placement of a view's frame within its parent.  For instance, with a button, `gravity` affects the placement of the title text within the button, while `layout_gravity` affects the placement of the button in the parent linearLayout.  So, `layout_gravity` centers the button.  

Add/Remove Views

The real power of Android's Linear Layout and readable xml over iOS's constraint system becomes apparent when as you try to change the view.  Let’s try adding a caption text field between the picture and the button.   In Android in a Linear Layout, simply add the new text field in the position you want it to appear.  The Linear Layout handles the rest.  

{% highlight css %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
                             android:orientation="vertical"
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

<TextView
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:gravity="center_horizontal"
        android:text="A really awesome waterfall!"/>
    
<Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="New Button"
        android:layout_gravity="center_horizontal"
        android:id="@+id/button"/>

</LinearLayout>
{% endhighlight %}

On iOS, adding a view is much harder.  First, break the constraint between the image and the button.  Next, drag a new text field onto the screen.   Then, add new constraints, and make them equal to one another.  For some reason, IB does not update the constraints when you drag a view around, or update the frame when you change the constraints.  So, to complete the process, tell IB manually to update the frames of all the views.  Here is an intermediate stage as I futz with IB to make all the constraints line up.  Life is getting harder in the world of Apple... 

<figure class>
	<img src="{{ site.url }}/images/linear-layout/ios-add-caption.png">
</figure>

To remove a view in Android, simply remove the view from the Linear Layout.  The view's attributes and layout logic are completely encapsulated within the xml tag, or abstracted away by the Linear Layout, so the developer doesn't have to do anything else.  This design is simple, clean, and beautiful.  On the other hand, in iOS, you’ll have to go through the whole process of deleting the view and rebuilding the surrounding constraints.  Painful.  

In general, the use of constraints in iOS complicates changes to views.  Even in a moderately complex view, dozens of constraints are required to define the layout.  With so much going on, it becomes difficult to figure out what layout parameters depend on what, and how view placements are defined.  I’ve had situations where all I’ve wanted to do is replace a single view element, and have had to tear down and rebuild dozens of constraints for that seemingly simple change.  Furthermore, the constraints system doesn't scale well, as modifying or debugging a view becomes more and more difficult as more view elements are added.  Redesigns in iOS are almost always painful. I once spent 8 hours updating the four screens in my personal app from iOS 6 to iOS 7, AFTER adhering to the best practices outlined for iOS 6.  Furthermore, since the xml from IB is auto generated and isn't really readable, two developers cannot work on the same part of a view, as the ensuing merge conflicts are usually totally unworkable.  It's also worth noting that some attributes must be set in code, notably hex colors.  So, the system forces the developer to split some layout logic between IB and code.  Please, for the sake of your later self and all the developers coming after you, at least be consistent with what you put in IB and what you put in code.  

Visibility

Now let’s say you want a view to appear conditionally.  Android has provided a convenient and simple attribute to make this possible. Set visibility:invisible if you want the view to be invisible but not collapsed (if other view placements are measured off this view) or set Visibility:gone if you want the view to be completely gone (collapsed).  The initial visibility of a view can be set in the xml, and further changes can be done in code.  Here's an example of hiding the image.  

{% highlight css %}
<ImageView
        android:layout_width="fill_parent"
        android:layout_height="300dp"
        android:gravity="center_horizontal"
        android:scaleType="fitCenter"
        android:src="@drawable/CanyonCreek"
        android:visibility="gone"/>
{% endhighlight %}

<figure class="third">
	<img src="{{ site.url }}/images/linear-layout/android-visible.png">
	<img src="{{ site.url }}/images/linear-layout/android-invisible.png">
	<img src="{{ site.url }}/images/linear-layout/android-gone.png">
	<figcaption>Image view is visible, invisible, and gone</figcaption>
</figure>

Notice that the Linear Layout automatically fixed the spacing around the "gone" view as well, so the remaining elements are evenly spaced.  Awesomesauce.

In iOS, setting the view to be hidden is the equivalent of Android's “invisible”: constraints are still measured off of that hidden view.  But, there is no equivalent to gone, which makes constructing collapsible views a huge pain.  As usual in iOS, there are a number of ways to replicate the behavior of “gone,” and all of them are  awkward in my opinion.  Here's the one I currently favor. 

{% highlight objective-c %}
@interface ViewController ()
@property (weak, nonatomic) IBOutlet NSLayoutConstraint *captionImageSpacingConstraint;
@property (weak, nonatomic) IBOutlet NSLayoutConstraint *captionHeight;
@property (weak, nonatomic) IBOutlet UITextView *captionTextView;
@end

@implementation ViewController

- (void)hideCaption {
    self.captionImageSpacingConstraint.constant = 0;
    self.captionHeight.constant = 0;
    self.captionTextView.hidden = YES;
}
@end
{% endhighlight %}

As you can see, I set the height constraint of the caption to be 0.  I then had to set the spacing between the caption and the image to be zero to eliminate the extra space between the image and the button.  Finally, I had to set the text view to be hidden because views with a height constraint of 0 will still be drawn behind the currently visible view element, which can incur a performance penalty.  Also, if the currently visible view element is transparent,  the collapsed view will show through.  The above code only works if I want to be able to hide the view once. If I want to be able to toggle the view to be visible again, I need to save the spacing and height constraints somewhere, which adds two more variables to the above code.  Thus every collapsible view needs at least two wired constraints, two variables, and a method, which quickly leads to code bloat, especially in a view with multiple collapsible view elements.

There are a couple of other ways to hide a view.  I could have set another constraint that encompassed both the spacing and the text view height constraint.  I would then set the priority of the spacing and height constraints to be lower than the larger constraint, and the height can be toggled using that single constraint.  I also could have set the constraint between the image and the next item (the button in this case).  Again this saves one constraint in the controller, but adds a constraint in IB.  I find that the added constraint and the differing priorities create confusing views in IB, especially as views get more complex and have more constraints on screen.  The added constraint also makes it harder to add or remove views, since the elements are now tied together with one more constraint.  It's a tradeoff between complexity in IB and more code bloat.  Again, all of this can be yours for free in Android.  

The Beauty of layout_weight.  

But Android's power doesn't stop there.  Let’s take this view one step further by replacing the single button with a row of three equally sized buttons.  In Android, I can embed three buttons in a horizontal LinearLayout.  Android provides a convenient attribute, `layout_weight` that dictates how elements of a layout expand to fill empty space.  The weight of the current view divided by the total weights in the layout equal the percent of the layout that the view will take up.  To get all of the buttons to be the same width, taking up the whole width of the screen, I simply set all of the layout_weights to be equal, and the `layout_width` of each button to be "0dp," like so.  

{% highlight css %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
                  android:orientation="horizontal"
                  android:layout_width="fill_parent"
                  android:layout_height="wrap_content">
        <Button
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:text="Button 1"
                android:layout_weight="1"/>
        <Button
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:text="Button 2"
                android:layout_weight="1"/>
        <Button
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:text="Button 3"
                android:layout_weight="1"/>
    </LinearLayout>
{% endhighlight %}
    

Now let's say I only want the three buttons to take up part of the screen.  In Android, I simply resize the LinearLayout, and everything else happens for free.  On a whim, I've also centered the whole button layout with the `layout_gravity` attribute.  It's that easy.  

{% highlight css %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
                  android:orientation="horizontal"
                  android:layout_gravity="center_horizontal"
                  android:layout_width="200dp"
                  android:layout_height="wrap_content">
{% endhighlight %}
                  

<figure class="half">
	<img src="{{ site.url }}/images/linear-layout/android-buttons.png">
	<img src="{{ site.url }}/images/linear-layout/android-buttons-center.png">
	<figcaption>Android button layout, before and after resizing</figcaption>
</figure>

Let's say I want the middle button to be twice as big as the other two buttons.  Simply change the weight of the middle button to 2 instead of 1. It's a one character change.  

{% highlight css %}
        <Button
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:text="Button 2"
                android:layout_weight="2"/>
{% endhighlight %}

<figure class>
	<img src="{{ site.url }}/images/linear-layout/android-buttons-weight.png">
</figure>

Let's say I want to move these buttons to reside above the picture.  Because all of the information for this layout is contained inside the LinearLayout xml tag, I can simply cut/paste the whole LinearLayout.  Likewise, I can cut/paste to move the layout to another view entirely.  As you can see, this abstraction allows for an incredible amount of flexibility, and is extremely powerful.  It successfully hides much of the complexity around layouts, allowing a developer to focus on building beautiful views.  And, for all the cases that Linear Layouts can't handle, Android provides two other awesome layout classes, RelativeLayout and FrameLayout.  I'll discuss them in a future blog post.  

It's a different story in iOS.  A button layout with the type of flexibility I demonstrated on Android is difficult or impossible to reproduce.  There are ways to hard code, or semi-hard code the layout, but the result will not be flexible or robust against certain design changes.  It's enough of a pain to try to recreate on iOS that I'm going to leave it to a future blog post, if there's enough interest.  Or if someone has a brilliant, simple way, I'm all ears. 

To summarize, the fundamental mistake in iOS is a lack of a coherent abstraction.  Constraints are very low level building blocks for views, and they often do not represent the fundamental tasks that a developer is trying to accomplish. So, the developer has to spend a lot of time and energy translating desires like, "I want a couple of buttons on screen that expand on rotation," into this language of constraints.  It's time consuming, brainpower intensive, and is a distraction from the business logic of the app. Usually, there are multiple ways of doing the same thing, none of them simple. Thus, different code bases will often handle views differently, and even different developers in the same codebase will use different paradigms.  This inconsistency leads to complicated, asymmetric view hierarchies.  

On the other hand, by providing layout abstractions, Android has managed to hide much of the complexity surrounding views. Thus, developers can focus on making the important decisions like how a view should behave on rotation and how elements should resize on larger screens. Also, by providing readable xml, Android has successfully encapsulated the relevant attributes of each view element, which keeps the code organized and helps developers design views that can be easily moved, reused, and even subclassed.  When I'm building an iOS app, I usually plan to spend the bulk of my time on layout.  In Android, layouts are often less than 25% of my time.  So, in the future, with Android as an example, I would like to see Apple introduce layout abstractions to reduce complexity in iOS app views.  
