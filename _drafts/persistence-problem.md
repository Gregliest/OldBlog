
**The Persistence Problem**

Persisting the state of a controller is horrendously difficult in Android.  The seemingly innocuous hook, onSaveInstanceState, opens a world of hurt around memory management.  First, its use is complex, and it is not guaranteed to be called.  Second, it uses a bundle, which requires that objects be serializable.  To be serializable, higher level objects need to implement the Parcelable interface, which is a low level library with minimal error checking, and that introduces massive code bloat to implement.  The result is a system that is unwieldy and error prone, and one that discourages developers from implementing best practices.  I know that there were many times that I asked myself if I really needed to implement Parcelable, or if I could get away with some dirty hack.  This pain is made worse by the fact that this whole serialization process should be a system level concern, and there is no reason that a developer should be responsible for persisting the transient state of an activity.  

OnSaveInstanceState is intended to be used to persist any relevant state in the controller.  However, because onSaveInstanceState is not guaranteed to be called, it is meant for state that should persist in a single session, but should not persist if the user exits the app.  State that should persist between sessions must be saved elsewhere.  This distinction is confusing, and often it is difficult to distinguish between the two types of state. Furthermore, this design requires the developer to split the relevant data between two places, the bundle and a permanent store, which adds significant complexity when reconstructing the view and controller. Persisting in the wrong place can lead to some insidious bugs, because there are so many cases to account for.  For instance to test persistence properly, for every controller in your app, you would need to go to that view, create some state that should be persisted, background the app (and verify that the app didn't get killed in the background), bring it to the foreground, and make sure nothing changed.  Then you would have to exit the app, restart it, and make sure that it saved what it should have.  It really sucks.  In iOS, the state of the controller is cached automatically, and the developer can choose to persist the user data in viewWillDisappear as desired.

To compound this bad design, Android has not provided a reasonable way of persisting the state of the view controller.  To be saved, a view controller must serialize itself.  Unfortunately, only primitives can be readily serialized.  Higher level objects, including your nicely decomposed model classes, must implement the Parcelable interface to be saved in the bundle, which introduces complexity and an enormous amount of code bloat.  Of all of the poor design choices by Android, I think that requiring a developer to manually save activity state, and requiring the Parcelable interface, represents the worst offense.  Here’s an example of the type of boilerplate that you’ll have to write to implement Parcelable, taken from Android's own documentation.

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

Yes, all of this code is required to implement Parcelable for a model class with a single field. Also, if the model contains a field that is not a primitive, that class also needs to implement Parcelable.  Furthermore, the parcel works on order only, so if you have two ints in a row, and you mix up the order, the ints will be assigned to the wrong variable, leading to really insidious bugs.   Parcelable is super painful, error prone, and leads to significant code bloat.  In my opinion, it's unacceptable that such a low level library is the state of the art in a modern system. I haven't found a good alternative, but here is a slightly safer way to implement Parcelable using a bundle.

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

As you can see, the bundle introduces even more code bloat, but at least it's readable, and you're not going to accidentally introduce a bug by changing the order.  It's worth noting that iOS has a rough equivalent to onSaveInstanceState, didReceiveMemoryWarning, but for the most part it isn't necessary in ARC systems.  Apple has done a good job in recent versions of handling memory issues in the background.  
  
I actually had a quick chat with Dianne Hackborn of the Android framework team about saving app state at Google I/O this year.  She had an interesting perspective on view controller persistence.  She suggested that, if a view controller is destroyed in the background, the developer should reinitiate whatever data retrieval is required to recreate the state of the app.  In other words, the data should either be persisted, or the activity should go through whatever path it went through to create its state in the first place, such as by fetching data from a server.  However, this design requires the developer to be aware of the different states that a view controller can be in and handle them accordingly, which adds significant incidental complexity.  Furthermore, the developer has to worry about memory management, which should be a system level concern.  Finally, this recommended recreation process cannot account for state changes due to user interaction or other one-time events.  This added complexity encourages the mixing of business logic in controllers with code that handles all of these cases, which obfuscates the true function of the app.  Serialization is often the only reason that a class need to implement Parcelable, which creates significant code bloat.  In the defense of the framework engineers, I'm sure that at least some of this complexity on Android is due to limitations imposed by Java.  Also, I'm not familiar with all of the design tradeoffs and constraints that I'm sure pushed the design in this direction.  But, in future versions of Android, I would like to see this complexity hidden from the developer.  
