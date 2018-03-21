% MAD - Android 5: Concurrency
% Patrick Sturm
% 21.03.2018

## Information

* Any issues with this presentation? Write a ticket or send me a pull request ;).
* Repo: [https://github.com/siyb/tw-mad-5-concurrency](https://github.com/siyb/tw-mad-5-concurrency)

# Agenda

## Agenda

* Introduction
* AsyncTask
* Handler

# Introduction

## Introduction - 1 - Resources

* Lesson: 
    * [http://developer.android.com/guide/components/processes-and-threads.html](http://developer.android.com/guide/components/processes-and-threads.html)
    * [http://developer.android.com/guide/practices/responsiveness.html](http://developer.android.com/guide/practices/responsiveness.html)
* Javadoc:
    * [https://developer.android.com/reference/android/os/AsyncTask.html](https://developer.android.com/reference/android/os/AsyncTask.html)
    * [https://developer.android.com/reference/android/os/Handler.html](https://developer.android.com/reference/android/os/Handler.html)

## Introduction - 2 - Processes

* When an application component is started and no other component of that application is running, Android will create a new process
* All Android components (Activities, Services, BroadcastReceiver and ContentProvider) run in the same thread
    * Potentially a bad thing – ANR (Application Not Responding)
    * You may use android:process to specify that another process for components,  usually, there is no reason to do so!
* For more info on processes check out the lesson link. It’s more important to talk about threading ;)

## Introduction - 3 - Threads

* When a process is created, Android also creates the Main thread, which is sometimes called the UI thread
    * Dispatching events (touch)
    * Rendering UI
* Two rules:
    * Do not block the Main thread (ANR)
        * No response to event within 5 seconds (e.g. touch)
        * BroadcastReceiver hasn’t finished executing within 10 seconds
    * Do not access the Android UI toolkit from outside the Main thread
* These two rules might sound contradictive, what if I want to display the result of a long lasting operation to the user?
* Android offers multiple ways of executing a task asynchronously and posting the result to the Main (UI) thread. We will learn about two of them today:
    * Handler
    * AsyncTask

# AsyncTask

## AsyncTask - 1 - Basics

* AsyncTask takes three generic parameters
    * *Params:* The type of parameters that are provided to the AsyncTask on start
    * *Progress:* The units of progress used to represent the tasks progress
    * *Result:* The type of result of the task
* AsyncTask knows 4 callback methods that we are going to discuss in detail:
    * onPreExecute, doInBackground, onProgressUpdate and onPostExecute

## AsyncTask - 2 - Methods

* onPreExecute
    * This method is called after the execution of the task has been triggered. It is invoked on the UI thread, so don't do any heavy lifting here. Usually, this method is used to setup the task on an UI level, creating a progress dialog for instance.
* doInBackground
    * This method is executed in a worker thread and should be used to do the heavy lifting. It will be executed right after onPreExecute has finished. The parameters provided to execute(...) will be used to call doInBackground. The result returned by this method will be published on the UI thread in onPostExecute. In addition, you are able to publish the progress of the operation using publishProgress(...)
* onPostExecute
    * Invoked on the UI thread when doInBackground finished. The parameter passed to this method is the one returned by doInBackground

## AsyncTask - 3 - Example Activity

```java
public class MyAsyncTaskActivity 
  extends Activity 
  implements OnClickListener { 
  private ProgressDialog progressDialog; 
  private int iterations = 100000; 
  private TextView resultText; 
  @Override 
  protected void onCreate(Bundle savedInstanceState) { 
    super.onCreate(savedInstanceState); 
    setContentView(R.layout.myasynctaskactivity); 
    resultText = (TextView) 
      findViewById(R.id.myasynctaskactivity_result); 
    findViewById(R.id.myasynctaskactivity_start)
      .setOnClickListener(this); 
  }
  ...
}
```

## AsyncTask - 4 - Example Activity cont.

```java
  @Override public void onClick(View v) { 
    MyAsyncTask at = new MyAsyncTask(); 
    at.execute(iterations); 
  }
```

## AsyncTask - 4 - Example AsyncTask

```java
public class MyAsyncTask 
  extends AsyncTask<Integer, Integer, String> { 
  @Override protected void onPreExecute() { 
    progressDialog = ProgressDialog
      .show(MyAsyncTaskActivity.this, 
      "AsyncTask", "AsyncTaskTest",true); 
    progressDialog.setMax(iterations); 
  }
  @Override protected String 
    doInBackground(Integer... params) { 
    Integer result = 0; 
    for (int i = 0; i < params[0]; i++) { 
      result = +i; 
      publishProgress(i); // calls onProgressUpdate
    } 
    // returning a string to prove the generic behaviour
    return "The result is: " + result; 
  }
```

## AsyncTask - 5 - Example AsyncTask

```java
  @Override 
  protected void onProgressUpdate(Integer... values) { 
    progressDialog.setProgress(values[0]); 
  } 
  @Override 
  protected void onPostExecute(String result) { 
    progressDialog.dismiss();
    resultText.setText(result); 
  } 
}
```

# Handler

## Handler - 1 - Introduction

* A Handler allows you to process Message and Runnable objects in a thread
* Two main reasons to use a Handler:
    * Schedule something for later execution
    * Execute something on a thread that you are not on (do not own)
* The second bit is pretty important, since most of the time, Handlers are used to execute something on the UI thread.
* Example: You run a web request in a separate thread and want to modify an UI element on completion.
* Important methods: sendMessage and post (for sending messages and posting Runnables respectivly)

## Handler - 2 - Looper

* The Handler helps us post messages to the Looper
* The Looper creates a message queue within a Thread and processes the queue until stopped
* The Main (also: UI Thread) is a Looper
    * You can use Looper.prepare() / Looper.loop() to create a message queue within your Thread
* The Handler needs to be created on a Looper Thread, to be more precise, it needs to be created on the Looper Thread you wish to post messages to
    * If you mess up Handler initialization (i.e. create the Handler in a non Looper Thread), you will get a notification informing you about the issue

## Handler - 3 - Example

```java
public class MyHandlerActivity extends Activity { 
  private TextView display; 
  private boolean causeException = false; 
  private MyHandler handler; 
  @Override 
  protected void onCreate(Bundle savedInstanceState) { 
    super.onCreate(savedInstanceState); 
    setContentView(R.layout.myhandleractivity); 
    display = (TextView) 
      findViewById(R.id.myhandleractivity_display); 
    handler = new MyHandler();
    MyThread myThread = new MyThread(); 
    myThread.start(); 
  }
```

## Handler - 3 - Example cont.

```java
  // why isn't this a good idea? (in general!)
  public class MyHandler extends Handler { 
   @Override 
   public void handleMessage(Message msg) { 
     display.setText(String.valueOf(msg.arg1)); 
  }
  private class MyThread extends Thread { 
    @Override 
    public void run() { 
      for (int i = 0; i < 60; i++) { 
        try {Thread.sleep(1000);} 
        catch (Throwable tr) {} 
          Message m = new Message(); 
          m.arg1 = i; 
          handler.sendMessage(m); 
      } 
    } 
  } 
}
```

# Conclusion

## Conclusion - 1 - AsyncTask vs. Handler

* AsyncTask provides the possibility to execute a task in an own thread and report back to whatever thread started the task
* Handlers do not provide their own thread to execute a task in, they can be used to communicate with other threads and append Runnables / Messages to their queue
* Both facilities allow communication with another thread:
    * AsyncTask may be used whenever something needs to be threaded (web request, database access, calculations, etc.)
    * Handlers can be used to provide feedback and manipulate UI elements, created by another thread. Sometimes it makes sense to use Handlers in combination with native threads (e.g. threads running in a Service).


# Any Questions?
