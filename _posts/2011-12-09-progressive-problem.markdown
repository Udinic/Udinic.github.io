---
author: udinic
comments: true
date: 2011-12-09 01:37:40+00:00
layout: post
slug: progressive-problem
title: Progressive Problem
wordpress_id: 176
categories:
- Android
tags:
- ACRA
- Any.DO
- dialog
- exception
---

On my last post I was talking about the integration of ACRA, error reporting library, which sends us crash reports from the users of [Any.DO](https://market.android.com/details?id=com.anydo). We've got some very interesting errors from the users, some exceptions were a matter of minutes to resolve, and others took more time, as the one I'm about to share here.

One of the most repeatable exceptions was an IllegalArgumentException. The stack trace was something like this:

{% highlight java %}

java.lang.IllegalArgumentException: View not attached to window manager
 at android.view.WindowManagerImpl.findViewLocked(WindowManagerImpl.java:355)
 at android.view.WindowManagerImpl.removeView(WindowManagerImpl.java:200)
 at android.view.Window$LocalWindowManager.removeView(Window.java:432)
 at android.app.Dialog.dismissDialog(Dialog.java:278)
 at android.app.Dialog.access$000(Dialog.java:71)
 at android.app.Dialog$1.run(Dialog.java:111)
 at android.os.Handler.handleCallback(Handler.java:587)
 at android.os.Handler.dispatchMessage(Handler.java:92)
 at android.os.Looper.loop(Looper.java:123)
 at android.app.ActivityThread.main(ActivityThread.java:4627)
 at java.lang.reflect.Method.invokeNative(Native Method)
 at java.lang.reflect.Method.invoke(Method.java:521)
 at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:876)
 at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:634)
 at dalvik.system.NativeStart.main(Native Method)

{% endhighlight %}


It's easy to see that the cause of it is a dismissed dialog, which its window doesn't belong to the current Window Manager. In a nutshell, the Window Manager is a service, designated to create all the drawing surfaces on which your application (and all other applications) draw their components. I won't elaborate on this subject on this post, maybe on a future one; in the mean time, you can Google it for more information. But why is that happening? The **common** reason for that is the notorious [orientation problem](http://stackoverflow.com/questions/1111980/how-to-handle-screen-orientation-change-when-progress-dialog-and-background-thre). We couldn't come across that issue, because we simply applied the magic attribute of __android:configChanges="keyboardHidden|orientation"__ to almost all of our Activities. Without this configuration, the default behavior for all the activities is to be destroyed and recreated whenever the user bend his arm by 90 degrees.  This configuration is doing what I believe should be the default behavior - leaving the activity be. Actually it means that you, the developer, takes responsibility to handle those orientation changes, so it's actually suppose to generate more work by declaring that. In fact, for inexperienced developers it can cause more harm than good, as on that demonic orientation problem. But, again, this is not the issue here.




After lots of playing around, I've managed to reproduce the problem on my device. That's how I did it:






	
  * Go to some activity other than the main

	
  * Created [AsyncTask ](http://developer.android.com/reference/android/os/AsyncTask.html)that **starts** and **dismisses** a progress dialog, and **sleeps** for 5 seconds in between

	
  * While the AsyncTask was taking a nap, I've pressed on the** HOME** **button**

	
  * When the home screen appeared, I quickly pressed on the** app's shortcut**

	
  * BOOM!




Why is that? Well, The AsyncTask was created on one of the sub-activities of the app, let's call it Activity B, and the main activity will be A. When it was time to dismiss the dialog, I've already pressed the HOME key and Activity B was no longer visible (HOME key -> Home screen). This is **not** suppose to be an issue, because the activity remains on the memory until killed by an over-excited Task Killer app or by the operation system, if it needs to clear some memory. The **problem** is that I've also pressed on the app's home screen shortcut, and it has the flag "Intent.FLAG_ACTIVITY_CLEAR_TOP", which practically saying that it'll kill all the other activities on the task's activities stack. You can read more about that stack, and how it operates, here: [http://developer.android.com/guide/topics/fundamentals/tasks-and-back-stack.html](http://developer.android.com/guide/topics/fundamentals/tasks-and-back-stack.html). Since I've killed Activity B (with all its dialog), and the app's shortcut is starting activity A, the AsyncTask was trying to dismiss a a dialog that's no longer exists! We get an IllegalArgumentException. Hooray! Now what?




On a this [very comprehensive post](http://blog.doityourselfandroid.com/2010/11/14/handling-progress-dialogs-and-screen-orientation-changes/) about the orientation problem, there's a nice solution for that issue, but not entirely for my problem. That solution transfers the AsyncTask between the destroyed activity and the created one when screen orientation changes. Since I'm just killing the activity, I want the activity to ignore any dismissing command for dead dialogs! I've decided to build a solution on top of my progress dialog and activities, allowing me to start a progress dialog on any activity, and let this whole thing be managed.




When started, I've tried using the AsyncTask solution, presented on that blog, and just extend it to write my own doInBackground() logic. Unfortunately, there was some AsyncTask internal problem with that approach; It's too long to start explaining here, so I'll pass. My solution includes this interface:



{% highlight java %}
public interface ProgressDlgHandlingActivity {
 public void startProgressDialog(String title);
 public void dismissProgressDialog();
 public void stopProgressDialog();
 }
{% endhighlight %}

Every activity that runs a progress bar should implement it, and it's fairly easy:

{% highlight java %}
/**
 * Starts the progress dialog
 * @param title
 */
 @Override
 public void startProgressDialog() {
    showDialog(AnydoProgressDialog.DIALOG_ID, args);
 }
/**
 * Stopping a currently active progress dialog
 */
 @Override
 public void stopProgressDialog() {
    // Detaching the currently showing dialog from the activity.
    if (mCurrProgressDlg != null)
       mCurrProgressDlg.detachFromActivity();
    }
/**
 * Dismissing the progress dialog. In use by the dialog itself!
 * In order to dismiss a progress dialog - use the #stopProgressDialog
 */
 @Override
 public void dismissProgressDialog() {
    if (isDialogShown) {
       dismissDialog(AnydoProgressDialog.DIALOG_ID);
    }
 }
{% endhighlight %}


As you can see, I keep the current displayed dialog (mCurrProgressDlg), and dismiss it only if it's shown. On the activity, all I need to know about is the startProgressDialog() and the stopProgressDialog() functions. How do I know that the dialog is shown? And what is that "detach" thing? On the progress dialog itself I save the calling activity as a field from type ProgressDlgHandlingActivity (as our interface), and accessing it with those 2 methods:



{% highlight java %}
/**
 * Attaches the dialog to the argumented activity.
 * @param activity
 */
 public void attachToActivity(ProgressDlgHandlingActivity activity) {
    mActivity = activity;
 }
/**
 * Removing the dialog from the activity
 */
 public void detachFromActivity() {
   if (mActivity != null) {
       mActivity.dismissProgressDialog();
    }
   mActivity = null;
 }
{% endhighlight %}


Notice that the detach function also dismisses the dialog if there's an activity currently attached to. Back to our activity, we need to add the code to create the dialog when calling the showDialog() function, I've implemented it like this:



{% highlight java %}
@Override
 protected Dialog onCreateDialog(int id, Bundle args) {
    switch (id) {
       case (AnydoProgressDialog.DIALOG_ID):
          if (mCurrProgressDlg == null)
             mCurrProgressDlg = new AnydoProgressDialog(this);
             return mCurrProgressDlg;
       default:
    return super.onCreateDialog(id, args);
    }
 }
@Override
 protected void onPrepareDialog(int id, Dialog dialog, Bundle args) {
    super.onPrepareDialog(id, dialog, args);
    if (id == AnydoProgressDialog.DIALOG_ID) {
       AnydoProgressDialog dlg = ((AnydoProgressDialog)dialog);
       dlg.attachToActivity(this);
       isDialogShown = true;
    }
 }
{% endhighlight %}


The create function is pretty straightforward and just create the dialog if not exist. On the prepare dialog, we attach the current activity to the dialog and set the isDialogShown flag, which we saw earlier. The most important part, is to close the dialog when the activity closes (or killed); For that, I've added the stopProgressDialog() to the onDestroy() function of the activity.




That's it! When I want to performs a long operation with a progress dialog running during it, I can do it like this:



{% highlight java %}

new AsyncTask() {
@Override
 protected void onPreExecute() {
    super.onPreExecute();
    startProgressDialog();
 }
@Override
 protected Void doInBackground(Void... voids) {
   calculateSomething("udinic");
   return null;
 }
@Override
 protected void onPostExecute(Void aVoid) {
    super.onPostExecute(aVoid);
    stopProgressDialog();
 }

}.execute();
{% endhighlight %}


When the activity will finish, the dialog will do so too. Now I know what you're probably thinking - why did I go all this trouble just to dismiss a dialog in the end of the activity? Well, this solution also combines a solution to the orientation problem. In order to support that as well, you need to override this method on the activity:



{% highlight java %}

@Override
 public Object onRetainNonConfigurationInstance() {
    if (mCurrProgressDlg != null) {
       return mCurrProgressDlg;
    }

    return super.onRetainNonConfigurationInstance();
 }

{% endhighlight %}

And add this to the activity's onCreate() method:

{% highlight java %}

// In case we created after a rotation has occurred, we'll reload any running progress dialog
 Object retained = getLastNonConfigurationInstance();
 if (retained instanceof AnydoProgressDialog)
    mCurrProgressDlg = (AnydoProgressDialog)retained;

{% endhighlight %}


Congrats! Now you have safe to use Progress dialog, with no need to extend AsyncTask with particular results types. Hope this will help someone, I know It did the job for me. If I'll get any requests for it - I'll also upload an example project to my GitHub account.
