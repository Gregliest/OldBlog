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

In this blog post, I'm going discuss basic view layout, and why I find layouts easier to manage in Android than in iOS.  This post is not meant as a tutorial, I will not go into detail on how to use the view layout systems on either platform.  Instead, I'll focus on comparing their strengths and weaknesses.  

The Basic Layout

Let’s say you have a new app and think, “I want an image, a button, and a text field to show up on screen, one after another.” On Android, enter the Linear Layout.  All you have to do is make a Linear Layout, insert the three items in the order you want them to appear, and voila!  I’ve done so in the following code example:

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

On iOS, the basic view setup is also quite easy.  I’m going to use Interface Builder whenever possible because I think it’s more intuitive  than code layouts (future blog post subject), and because it's Apple's preferred view construction method.  To obtain the same view we built in Android, you simply drag the elements you want onto the screen.  Here’s an example.  

![The Basic Layout](/images/ios/basic-lined-up-layout.png)

Note that I haven't done anything except drag the views out onto the screen.  Auto layout will automatically infer a set of constraints to define how the view should change on rotation or resize.  But, while it is possible to rely solely on this feature, in my experience dragged placements usually don't do what I want them to do, especially if the screen gets rotated.  Also, they don't afford pixel control over the spacing, widths, and heights.  So to define the view, I'm going to add constraints.  I’m not going to go into detail here; there are a number of existing tutorials on how to use Interface Builder.  

![The Basic Layout, with Constraints!](/images/ios/basic-layout-constraints.png)

To center each view, I chose the “Center X Alignment” constraint relative to the parent view, because in my opinion it is the simplest way.  The simplest layouts use constraints that are defined relative to the parent view whenever possible.   However, there are a number of other ways to center the views.  For instance, I could have defined the horizontal position of each view as a distance from the left margin, or I could have chained the Center X Alignments together, so that each view is center aligned with the one above it.  These methods are inferior because they allow less flexibility in changing the view.  But the real point here is that, because there are so many possibilities, a fair amount of thought needs to go into how you constrain a view in iOS.  It's really easy to do sub optimally, especially as the views get more complex and their relationships become less clear.  It can be hard to enter an existing code base and figure out how another developer decided to handle constraints.  Also, notice that I needed to add 8 constraints to do what Android's Linear Layout did for free.  Also, there are a couple of attributes that cannot be set in IB, notably hex colors.  If you want to set a color in the view to be a specific hex value, it must be done in code.  

In Android, I can define my whole view using simple attributes in the xml file.  For instance, I used fill_parent on the width and height attributes of the LinearLayout to fill the screen, and set some margins on the views fix some spacing issues.  I set the source file and the scaling type for the image view, all super easy to understand.  The one quirk I will note about this view is the use of gravity vs. layout_gravity.  “Gravity” dictates the placement of the elements of a view inside its frame.  "Layout_gravity" controls the placement of a view's frame within its parent.  For instance, with a button, `gravity` affects the placement of the title text within the button, while `layout_gravity` affects the placement of the button in the parent linearLayout.  So, to center a button, use `layout_gravity.`

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

Add/Remove Views

The real power of the Linear Layout over iOS's constraint system becomes apparent when as you try to change the view.  Let’s try adding a caption text field between the picture and the button.   In Android in a Linear Layout, simply add the view in the position you want it to appear.  The Linear Layout handles the rest.  

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

On iOS, adding a view is much harder.  First, break the constraint between the image and the button.  Next, drag a new text field onto the screen.   Then, add the new constraints, and make them equal to one another.  I find that the easiest way to do this is to drag the views to the approximate position and then add the constraints.  For some reason, IB does not update the constraints when you drag a view around, so, to complete the process tell IB to update the frames of all the views.  Here is an intermediate stage as I futz with IB to make all the constraints line up.  Life is getting harder in the world of Apple... 

<figure class>
	<img src="{{ site.url }}/images/linear-layout/ios-add-caption.png">
</figure>

To remove a view, simply remove the view from the Linear Layout.  The view's attributes and layout logic are completely encapsulated within the xml tag, or abstracted away by the Linear Layout.  The design is beautiful.  On the other hand, in iOS, you’ll have to go through the whole process of deleting the view and rebuilding the constraints between neighboring views.  See a pattern?  

Visibility

Now let’s say you want a view to appear conditionally.  Android has provided a convenient and simple attribute to make this possible. Simply set visibility:invisible if you want the view to be invisible but not collapsed (if other view placements are measured off this view) or set Visibility:gone if you want the view to be completely gone (collapsed).  The initial visibility of a view can be set in the xml, and further changes can be done in code.  Here's an example of hiding the image.  

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

In iOS, setting the view to be hidden is the equivalent of “invisible” in Android: constraints are still measured off of that hidden view.  But, there is no equivalent to gone, which makes constructing collapsible views a huge pain.  As usual in iOS, there are a number of ways to replicate the behavior of “gone,” and all of them are  awkward in my opinion.  Here's the one I currently favor. 

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

As you can see, I set the height constraint of the caption to be 0.  I then had to set the spacing between the caption and the image to be zero to eliminate the extra space between the image and the button.  Finally, I had to set the text view to be hidden, because views with a height constraint of 0 will still be drawn behind the currently visible view element, which can incur a performance penalty.  Also, if the currently visible view element is transparent,  the collapsed view will show through.  Furthermore, if you want to be able to toggle the view to be visible again, you need to save the spacing and height constraints somewhere.  Since every collapsible view needs at least three wired constraints and a method, they quickly lead to code bloat, especially in a view with multiple collapsible view elements.  Collapsible views are a real pain in iOS.  

There are a couple of other ways to hide a view.  I could have set another constraint that encompassed both the spacing and the text view height constraint.  I would then set the priority of the spacing and height constraints to be lower than the larger constraint.  This method saves having to wire up, change, and save one constraint.  You can also set a constraint between the image and the next item (the button in this case).  Again this saves one constraint in the controller.  However, I find that both of these methods make for confusing views in IB since they add an extra constraint, especially as views get more complex.  

Let’s take this one step further, lets say that instead of one button, we want a row of three equally sized buttons.  In Android, I can embed three buttons in a horizontal LinearLayout.  Android provides a convenient attribute, layout_weight that dictates how elements of a layout expand to fill empty space.  The weight of the current view divided by the total weights in the layout equal the percent of the layout that the view will take up.  To get all of the buttons to be the same width, taking up the whole width of the screen, I simply set all of the layout_weights to be equal, and the layout_width of each button to be "0dp," like so.  

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
    

Now let's say I only want the three buttons to take up part of the screen.  In Android, I simply resize the LinearLayout, and everything else happens for free.  On a whim, I've also centered the buttons with layout_gravity.  It's that easy.  

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

Let's say I want to move these buttons to above the picture.  Because all of the information for this whole layout is nested inside the LinearLayout xml tag, I can simply cut/paste the whole LinearLayout.  Likewise, I can do the same thing if I want to move the layout to another view.  So easy.  

However, in iOS, this flexibility is difficult or impossible to accomplish.  It's enough of a pain to try to recreate on iOS that I'm going to leave it to a future blog post if there's enough interest.  Or if someone has a brilliant, simple way, I'm all ears. In a future blog post I’ll also go into more complex view handling, and how iOS’s quirks and inconsistencies become even more apparent, and even more cumbersome as views get more complicated.  The poor handling of constraints in iOS can make it really painful to change views.  I’ve had situations where all I’ve wanted to do is replace a button or something like that, and have had to tear down and rebuild the majority of the constraints in the view.  Furthermore, the multitude of view controls in iOS can make it extremely difficult to come into an existing code base and figure out where a view attribute is getting set.  It can also make debugging a view a nightmare.  On the other hand, Android provides two other layout classes, RelativeLayout and FrameLayout, that successfully abstract away much of the complexity.  

The fundamental mistake that Apple made is that they have not abstracted away from constraints.  Constraints are a very low level building block for views. Even in a moderately complex view, dozens of constraints are often required to define the layout.  With so much going on, it becomes difficult to figure out what depends layout parameters depend on what, and how a view placements are defined.  It also requires the developer to translate relatively simple thoughts like, I want a couple of buttons on screen to this language of constraints.  The dev has to think about, well, I guess I need to constrain this dimension and this dimension, to the left of X, etc, etc. It's time consuming and brainpower intensive.  Android has managed to abstract away almost all of this incidental complexity, allowing the dev to focus on making decisions like, this view should expand under certain conditions, while this one should shrink, etc.  Also, constraints like margins are contained within a view's xml tag, which clearly organizes view attributes.  

To summarize, the linear layout is a fantastic abstraction for building views.  It allows for easy translation between a design and the layout code.  Also, by encapsulating and abstracting the constraint logic, it allows for much more flexible and malleable views.  By encouraging views to be grouped into logical blocks, Android helps us to design views that can be easily moved, reused, and even subclassed.  Moving a layout from one view to the next is as easy as a copy/paste.  On the other hand, iOS’s view handling is clunky and confusing.  I intentionally hedged in a lot of the iOS examples by saying, “here’s one way to do it,” because often there are multiple ways of accomplishing the same thing, and NONE of them are trivial.  Often different code bases will handle views differently, and even different developers in the same codebase will use different paradigms.  This inconsistency leads to complicated, asymmetric view hierarchies.  As Alan Perlis once said, "Symmetry is a complexity-reducing concept; seek it everywhere."  On the other hand, although there are also multiple ways of doing the same thing in Android, there is usually one easiest way, which encourages consistency from developers, and enhances readability.  I know all of the examples here favored Android, and pointed out deficiencies in iOS.  I honestly couldn’t think of a situation having to do with simple layouts in which iOS was easier than Android.  When I'm building an iOS app, I usually plan to spend the bulk of my time on layout.  In Android, layouts are often less than 25% of my time.  Redesigns in iOS are almost always painful, I once spent 8 hours updating the four screens in my personal app from iOS 6 to iOS 7, AFTER adhering to the best practices outlined for iOS 6.  
