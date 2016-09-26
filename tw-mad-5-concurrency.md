% MAD - Android 5: Concurrency
% Patrick Sturm
% 21.09.2016

## Information

* Any issues with this presentation? Write a ticket or send me a pull request ;).
* Repo: [https://github.com/siyb/tw-mad-1-activities-and-fragments](https://github.com/siyb/tw-mad-1-activities-and-fragments)

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



# Any Questions?