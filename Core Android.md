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

### What is the difference between ListView and RecyclerView? [Detail](https://www.truiton.com/2015/03/android-recyclerview-vs-listview-comparison/)
* View Holder Compulsory - Major drawback of not using view holders could lead to a heavy operation of finding views by ids every time. Which resulted in laggy ListViews.This problem is solved in RecylerView by the use of RecyclerView.ViewHolder class. A ViewHolder holds the reference to the id of the view resource and calls to the resource will not be required. Thus performance of the application increases.
* Layouts - ListView only Vertical List can be implemented.RecyclerView supports three types of predefined Layout Managers:
    1. LinearLayoutManager – This is the most commonly used layout manager in case of RecyclerView. Through this, we can create both horizontal and vertical scroll lists.
    2. StaggeredGridLayoutManager – Through this layout manager, we can create staggered lists. Just like the Pinterest screen.
    3. GridLayoutManager– This layout manager can be used to display grids, like any picture gallery.
* Item Animator - In a ListView, as such there are no special provisions through which one can animate, addition or deletion of items. 

### What is the ViewHolder pattern? Why should we use it?
ViewHolder is a design pattern which can be applied when using a custom adapter. 
Every time when the adapter calls getView() method, the findViewById() method is also called. This is a very intensive work for the mobile CPU and so affects the performance of the application and the battery consumption increases.

To avoid this, ViewHolder is used. A ViewHolder holds the reference to the id of the view resource and calls to the resource will not be required. Thus performance of the application increases.

### What is SnapHelper?
SnapHelper is a helper class that helps in snapping any child view of the RecyclerView. For example, you can snap the firstVisibleItem of the RecyclerView as you must have seen in the play store application that the firstVisibleItem will be always completely visible when scrolling comes to the idle position.

### What is Dialog in Android?
A dialog is a small window that prompts the user to make a decision or enter additional information. A dialog does not fill the screen and is normally used for modal events that require users to take an action before they can proceed.

### What is Toast in Android?
Andorid Toast can be used to display information for the short period of time. A toast contains message to be displayed quickly and disappears after sometime. 
`Toast.makeText(getApplicationContext(),"Hello Javatpoint",Toast.LENGTH_SHORT).show();`

### What the difference between Dialog and Dialog Fragment?
Using DialogFragment to manage the dialog ensures that it correctly handles lifecycle events such as when the user presses the Back button or rotates the screen. The DialogFragment class also allows you to reuse the dialog's UI as an embeddable component in a larger UI, just like a traditional Fragment (such as when you want the dialog UI to appear differently on large and small screens).

As a DialogFragment is also a Fragment that displays a dialog window, floating on top of its activity’s window. This fragment contains a Dialog object, which it displays as appropriate based on the fragment’s state.

### What is Intent?
An Intent is an "intention" to perform an action; in other words, a messaging object you can use to request an action from another app component.Intents allow you to interact with components from the same applications as well as with components contributed by other applications. 

### What is an Implicit Intent?

Implicit Intents do not directly specify the Android components which should be called , it only specifies action to be performed.An Uri can be used with the implicit intent to specify data type.

for example

`Intent intent = new Intent(ACTION_VIEW,Uri.parse("http://www.google.com");`

this will cause web browser to open a webpage .Android system searches for all components(displays all browsers installed) which are registered for the specific action and the data type.If many components are found then the user can select which component to use.

### What is an Explicit Intent?
Explicit intents are used in the application itself wherein one activity can switch to other activty.
Example 
`Intent intent = new Intent(this,Target.class);`

This causes switching of activity from current context to the target activity. Explicit Intents can also be used to pass data to other activity using putExtra method and retrieved by target activity by `getIntent().getExtras()` methods.

### What is a BroadcastReceiver?
A broadcast receiver is a component that responds to system-wide broadcast announcements. Many broadcasts originate from the system—for example, a broadcast announcing that the screen has turned off, the battery is low, or a picture was captured. Applications can also initiate broadcasts—for example, to let other applications know that some data has been downloaded to the device and is available for them to use. Although broadcast receivers don't display a user interface, they may create a status bar notification to alert the user when a broadcast event occurs.

### What is a LocalBroadcastManager?
Helper to register for and send broadcasts of Intents to local objects within your process.You know that the data you are broadcasting won't leave your app, so don't need to worry about leaking private data.

### What is the function of an IntentFilter?
An intent filter declares the capabilities of its parent component — what an activity or service can do and what types of broadcasts a receiver can handle.

### What is a Sticky Intent?
Sticky Intent is using for broadcast receiver for unknown time, means it may occur anytime when the situation occur.
Sticks with android, for future broad cast listeners. For example if BATTERY_LOW event occurs then that intent will be stick with android so that if any future user requested for BATTER_LOW, it will be fired;

### What is a PendingIntent?
Pending Intent is using for broadcast receiver for a fixed time.Fixed time means that system knows that when the receiver would fire.

### Describe how broadcasts and intents work to be able to pass messages around your app?
[Link](https://stackoverflow.com/a/7276808/9984145)

### What are the different types of Broadcasts? [Link](https://www.edureka.co/blog/android-tutorials-broadcast-receivers/)
* Ordered Broadcasts: These broadcasts are synchronous, and therefore follow a specific order. The order is defined using android: priority attribute. The receivers with greater priority would receive the broadcast first. In case there are receivers with same priority levels, the broadcast would not follow an order. Each receiver (when it receives the broadcast) can either pass on the notification to the next one, or abort the broadcast completely. On abort, the notification would not be passed on to the receivers next in line.

* Normal Broadcasts: Normal broadcasts are not orderly. Therefore, the registered receivers often run all at the same time. This is very efficient, but the Receivers are unable to utilize the results.

### Difference between Activity Intent and Broadcasting Intent
You must remember that Broadcasting Intents are different from the Intents used to start an Activity or a Service. The intent used to start an Activity makes changes to an operation the user is interacting with, so the user is aware of the process. However, in case of broadcasting intent, the operation runs completely in the background, and is therefore invisible to the user.

### What is Serivce?
A service is a component that runs in the background to perform long-running operations without needing to interact with the user and it works even if application is destroyed. A service can essentially take two states −
* Started - A service is started when an application component, such as an activity, starts it by calling startService(). Once started, a service can run in the background indefinitely, even if the component that started it is destroyed.
* Bound - A service is bound when an application component binds to it by calling bindService(). A bound service offers a client-server interface that allows components to interact with the service, send requests, get results, and even do so across processes with interprocess communication (IPC).

### Service vs IntentService. 
[Answer](https://www.linkedin.com/pulse/service-vs-intentservice-android-anwar-samir/)
Service is a base class of service implementation. *Service class is run in the application’s main thread which may reduce the application performance*. Thus, IntentService, which is a direct subclass of Service is borned to make things easier. The IntentService is used to perform a certain task in the background. Once done, the *instance of IntentService terminate itself automatically*. Examples for its usage would be to download a certain resources from the Internet.

### What is a JobScheduler? [Detail](http://www.vogella.com/tutorials/AndroidTaskScheduling/article.html)
The JobScheduler supports batch scheduling of jobs. The Android system can combine jobs so that battery consumption is reduced.
Here are example when you would use this job scheduler:

* Tasks that should be done once the device is connect to a power supply
* Tasks that require network access or a Wi-Fi connection.
* Task that are not critical or user facing
* Tasks that should be running on a regular basis as batch where the timing is not critical

