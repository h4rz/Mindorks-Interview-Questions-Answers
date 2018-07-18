# Core Android Questions

### Tell all the Android application components.[ Android Official](https://developer.android.com/guide/components/fundamentals#Components)
Each component is an entry point through which the system or a user can enter your app. Some components depend on others.
There are four different types of app components:
* Activities - An activity is the entry point for interacting with the user. It represents a single screen with a user interface. 
For example, an email app might have one activity that shows a list of new emails, another activity to compose an email, and another
activity for reading emails.
* Services - A service is a general-purpose entry point for keeping an app running in the background for all kinds of reasons. 
It is a component that runs in the background to perform long-running operations or to perform work for remote processes. 
A service does not provide a user interface. For example, a service might play music in the background while the user is in a
different app, or it might fetch data over the network without blocking user interaction with an activity
* Broadcast receivers - A broadcast receiver is a component that enables the system to deliver events to the app outside of a regular 
user flow, allowing the app to respond to system-wide broadcast announcements. Because broadcast receivers are another well-defined 
entry into the app, the system can deliver broadcasts even to apps that aren't currently running.Although broadcast receivers don't 
display a user interface, they may create a status bar notification to alert the user when a broadcast event occurs.for example, a 
broadcast announcing that the screen has turned off, the battery is low, or a picture was captured. Apps can also initiate 
broadcasts—for example, to let other apps know that some data has been downloaded to the device and is available for them to use.
* Content providers - A content provider manages a shared set of app data that you can store in the file system, in a SQLite database, 
on the web, or on any other persistent storage location that your app can access. Through the content provider, other apps can query or
modify the data if the content provider allows it. For example, the Android system provides a content provider that manages the user's 
contact information. As such, any app with the proper permissions can query the content provider, such as ContactsContract.Data, to read
and write information about a particular person.

### What is the structure of an Android Application?
![Structure](https://developer.android.com/images/tools/projectview-p1.png)

### What is Context? How is it used?
As the name suggests, it’s the context of the current state of the application/object. It lets newly-created objects understand what has 
been going on.You can get the context by invoking **getApplicationContext(), getContext(), getBaseContext() or this** (when in a class that 
extends from Context, such as the Application, Activity, Service and IntentService classes).
Typical uses of context:
* Creating new objects: Creating new views, adapters, listeners:


`TextView tv = new TextView(getContext());`

`ListAdapter adapter = new SimpleCursorAdapter(getApplicationContext(), ...);`

* Accessing standard common resources: Services like LAYOUT_INFLATER_SERVICE, SharedPreferences:

`context.getSystemService(LAYOUT_INFLATER_SERVICE)`

`getApplicationContext().getSharedPreferences(*name*, *mode*);`

* Accessing components implicitly: Regarding content providers, broadcasts, intent

`getApplicationContext().getContentResolver().query(uri, ...);`

### What is AndroidManifest.xml?
The manifest file describes essential information about your app to the Android build tools, the Android operating system, and Google Play.
* The app's package name
* The components of the app, which include all activities, services, broadcast receivers, and content providers.
* The permissions that the app needs in order to access protected parts of the system or other apps.

### What is Application class? [Video](https://www.youtube.com/watch?v=guzAFDRSpB0)
Base class for maintaining global application state.All activity will be able to use method and vvariables of this class

### Explain Activity Lifecycle.
![Activity Life Cycle](https://developer.android.com/guide/components/images/activity_lifecycle.png)

### Explain Fragment Lifecycle.
A Fragment represents a behavior or a portion of user interface in a FragmentActivity. You can combine multiple fragments
in a single activity to build a multi-pane UI and reuse a fragment in multiple activities. 
![Fragment Life Cycle](https://developer.android.com/images/activity_fragment_lifecycle.png)

### What are "launch modes"? [Link](https://android.jlelse.eu/android-activity-launch-mode-e0df1aa72242)
Launch mode is an instruction for Android OS which specifies how the activity should be launched. It instructs how any new 
activity should be associated with the current task.

Activities are arranged with the order in which each activity is opened. This maintained stack called Back Stack. When you 
start a new activity using startActivity(), it “pushes” a new activity onto your task, and put the previous Activity in the back stack.
![Back Stack](https://cdn-images-1.medium.com/max/1600/1*TpJZEIiD_U6yGc_WmNw0VQ.png)

There are four launch modes for activity. They are:

1. standard -  It creates a new instance of an activity in the task from which it was started.
2. singleTop - In this launch mode if an instance of activity already exists at the top of the
current task, a new instance will not be created and Android system will route the intent information through onNewIntent().
3. singleTask - In this launch mode a new task will always be created and a new instance will be pushed to the task as the root one.
If an instance of activity exists on the separate task, a new instance will not be created and Android system routes the intent 
information through onNewIntent() method.
4. singleInstance -  Any other activity started from here will create in a new task.

### Why is it recommended to use only the default constructor to create a Fragment?
The reason why you should be passing parameters through bundle is because when the system restores a fragment (e.g on config change),
it will automatically restore your bundle.

The callbacks like onCreate or onCreateView should read the parameters from the bundle - this way you are guaranteed to restore the 
state of the fragment correctly to the same state the fragment was initialised with (note this state can be different from the 
onSaveInstanceState bundle that is passed to the onCreate/onCreateView)

The recommendation of using the static newInstance() method is just a recommendation. You can use a non default constructor but make 
sure you populate the initialisation parameters in the bundle inside the body of that constructor. And read those parameters in the 
onCreate() or onCreateView() methods.
So, for example, if we wanted to pass an integer to the fragment we would use something like:

```
public static MyFragment newInstance(int someInt) {
    MyFragment myFragment = new MyFragment();

    Bundle args = new Bundle();
    args.putInt("someInt", someInt);
    myFragment.setArguments(args);

    return myFragment;
}
```
And later in the Fragment onCreate() you can access that integer by using:

`getArguments().getInt("someInt", 0);`

### How would you communicate between two Fragments? 
All Fragment-to-Fragment communication is done either through a shared ViewModel or through the associated Activity. Two Fragments 
should never communicate directly.

[Have a look at the Android developers page](https://developer.android.com/training/basics/fragments/communicating)

Basically, you define an interface in your Fragment A, and let your Activity implement that Interface. Now you can call the interface
method in your Fragment, and your Activity will receive the event. Now in your activity, you can call your second Fragment to update the
textview with the received value

Your Activity implements your interface (See FragmentA below)
```
public class YourActivity implements FragmentA.TextClicked{
    @Override
    public void sendText(String text){
        // Get Fragment B
        FraB frag = (FragB)
            getSupportFragmentManager().findFragmentById(R.id.fragment_b);
        frag.updateText(text);
    }
}
```
Fragment A defines an Interface, and calls the method when needed
```
public class FragA extends Fragment{

    TextClicked mCallback;

    public interface TextClicked{
        public void sendText(String text);
    }

    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);

        // This makes sure that the container activity has implemented
        // the callback interface. If not, it throws an exception
        try {
            mCallback = (TextClicked) activity;
        } catch (ClassCastException e) {
            throw new ClassCastException(activity.toString()
                + " must implement TextClicked");
        }
    }

    public void someMethod(){
        mCallback.sendText("YOUR TEXT");
    }

    @Override
    public void onDetach() {
        mCallback = null; // => avoid leaking, thanks @Deepscorn
        super.onDetach();
    }
}
```
Fragment B has a public method to do something with the text
```
public class FragB extends Fragment{

    public void updateText(String text){
        // Here you have it
    }
}
```
### What is retained Fragment? [Answer](https://www.androiddesignpatterns.com/2013/04/retaining-objects-across-config-changes.html)

### What is View in Android?
The View class is a superclass for all GUI components in Android. For instance, the TextView class which is used to display text labels in Android apps is a subclass of View. Android contains the following commonly used View subclasses:

* TextView
* EditText
* ImageView
* ProgressBar
* Button
* ImageButton
* CheckBox
* DatePicker
These are only some of the many, many subclasses of the View class.

### Difference between View.GONE and View.INVISIBLE?
* INVISIBLE - This view is invisible, but it still takes up space for layout purposes.
* GONE - This view is invisible, and it doesn’t take any space for layout purposes.

### Can you create custom views? How?
If none of the prebuilt widgets or layouts meets your needs, you can create your own View subclass. If you only need to make small adjustments to an existing widget or layout, you can simply subclass the widget or layout and override its methods.

Creating your own View subclasses gives you precise control over the appearance and function of a screen element.

You could combine a group of View components into a new single component, perhaps to make something like a ComboBox (a combination of popup list and free entry text field), a dual-pane selector control (a left and right pane with a list in each where you can reassign which item is in which list), and so on.

*Creating your own View components:*

> 1. Extend an existing View class or subclass with your own class.
> 2. Override some of the methods from the superclass. The superclass methods to override start with 'on', for example, onDraw(), onMeasure(), and onKeyDown(). This is similar to the on... events in Activity or ListActivity that you override for lifecycle and other functionality hooks.
> 3. Use your new extension class. Once completed, your new extension class can be used in place of the view upon which it was based.

### What are ViewGroups and how they are different from the Views?
The ViewGroup class is a subclass of the View class. ViewGroup instances work as containers for View instances to group View instances together. Android contains the following commonly used ViewGroup subclasses:

* LinearLayout
* RelativeLayout
* ListView
* GridView

### What is a canvas? [Example](https://android.jlelse.eu/canvas-the-real-play-ground-android-c0faa4b79943)
When it comes to create something which is not available on the layout.xml as native component, developers face real challenge. When drawables are not enough and styles are not sufficient, what we left with is the option of Drawing it!

Canvas — the real play ground wherein a developer can create any type of view or animation.

The Canvas class holds the "draw" calls. To draw something, you need 4 basic components: 
* A Bitmap to hold the pixels, 
* a Canvas to host the draw calls (writing into the bitmap), 
* a drawing primitive (e.g. Rect, Path, text, Bitmap), 
* a paint (to describe the colors and styles for the drawing).

### What is a SurfaceView? [Link](https://google-developer-training.gitbooks.io/android-developer-advanced-course-practicals/unit-5-advanced-graphics-and-views/lesson-11-canvas/11-2-p-create-a-surfaceview/11-2-p-create-a-surfaceview.html)
In Android, all simple layout views are all drawn on the same GUI thread which is also used for all user interaction. So if we need to update GUI rapidly or if the rendering takes too much time and affects user experience then we should use SurfaceView.

The Android SurfaceView provides a dedicated drawing surface embedded inside of a view hierarchy. You can control the format of this surface, however, the SurfaceView takes care of placing the surface at the correct location on the screen.

When you create a custom view and override its onDraw() method, all drawing happens on the UI thread. Drawing on the UI thread puts an upper limit on how long or complex your drawing operations can be, because your app has to complete all its work for every screen refresh.One option is to move some of the drawing work to a different thread using a SurfaceView.

* All the views in your view hierarchy are rendered onto one Surface in the UI thread.
* In the context of the Android framework, Surface refers to a lower-level drawing surface whose contents are eventually displayed on the user's screen.
* To draw, start a thread, lock the SurfaceView's canvas, do your drawing, and post it to the Surface.

![Image](https://google-developer-training.gitbooks.io/android-developer-advanced-course-practicals/images/11-2-p-create-a-surfaceview/dg_Surfaceview_surface.png)

### Relative Layout vs Linear Layout.
* Linear Layout  -  Arranges elements either vertically or horizontally. i.e. in a row or column.
* Relative Layout  - Arranges elements relative to parent or other elements. When you have scenarios like - place this TextView in right of ImageView and below particular EditText - Relative Layout can help in that scenarios.

### Tell about Constraint Layout
A ConstraintLayout is a ViewGroup which allows you to position and size widgets in a flexible way.

The aim of the ConstraintLayout is to help reduce the number of nested views, which will improve the performance of our layout files.

Relative positioning is one of the basic building block of creating layouts in ConstraintLayout. Those constraints allow you to position a given widget relative to another one. You can constrain a widget on the horizontal and vertical axis:

Horizontal Axis: left, right, start and end sides
Vertical Axis: top, bottom sides and text baseline
The general concept is to constrain a given side of a widget to another side of any other widget.

For example, in order to position button B to the right of button A (Fig. 1): 
![image](https://developer.android.com/reference/android/support/constraint/resources/images/relative-positioning.png)
```
<Button android:id="@+id/buttonA" ... />
         <Button android:id="@+id/buttonB" ...
                 app:layout_constraintLeft_toRightOf="@+id/buttonA" />
```  
This tells the system that we want the left side of button B to be constrained to the right side of button A. Such a position constraint means that the system will try to have both sides share the same location. 

### Do you know what is the view tree? How can you optimize its depth? [Link1](https://developer.android.com/training/improving-layouts/optimizing-layout) [Link2](https://developer.android.com/studio/debug/layout-inspector)
View Tree: The hierarchy of views in the layout.
It is always good practice to run the lint tool on your layout files to search for possible view hierarchy optimizations. Lint has replaced the Layoutopt tool and has much greater functionality.
Another benefit of Lint is that it is integrated into Android Studio. Lint automatically runs whenever you compile your program. With Android Studio, you can also run lint inspections for a specific build variant, or for all build variants.
