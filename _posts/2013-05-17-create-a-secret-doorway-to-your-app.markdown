---
author: udinic
comments: true
date: 2013-05-17 00:07:09+00:00
layout: post
slug: create-a-secret-doorway-to-your-app
title: Create a secret doorway to your app
wordpress_id: 493
categories:
- Android
tags:
- ACRA
- Android
- android.provider.Telephony.SECRET_CODE
- android:host
- android_secret_code
- diagnose
- diagnose screen
- diagnostics
- Error Report
- secret
- secret code
- SECRET_CODE
---

This short post will be about creating a back-door to your app, giving you the option to provide more information or actions than you want end users to have. This will be using the Secret Code feature.

I came across some articles with list of codes in order to access some phone's data (i.e. Camera's firmware spec.), run some tests (i.e. Vibration test) or even perform actions such as reset to factory settings. Some examples can be found [here](http://www.redmondpie.com/hidden-android-secret-codes-for-samsung-htc-motorola-sony-lg-and-other-devices/) and [here](http://www.itworld.com/consumerization-it/351248/debug-your-phone-these-hidden-android-secret-codes) and even as [an app](https://play.google.com/store/apps/details?id=cx.makaveli.androidsecretcodes&hl=en) that indexes all that information.


## Create your own


Creating your own secret doorway for your app can help you get more information from users of your app in a time of need. Getting crash reports from end users can be done by several services such as ACRA or BugSense, but how about errors that are not crashes? How many times did your Mom/Spouse/Friend/Mailman showed you their phone, running your app, and said something like: "Why do I see duplicates on the list here?! Why can't I sync??". You probably can't do anything without debugging the app or at least take a look at the app's SharedPrefs/SQLiteDB. Using a backdoor can help you get more internal information that will help you find the cause for the problem, no computer is necessary!

Another cool thing is providing more actions to do with your app. You want to keep the app clean for your users, but you might want more power to yourself. You can tweak your internal settings, whether for testing purposes or just to try out new features on your everyday use. Not needing to compile a new version after each change of internal setting, can help a lot in the task of identifying tricky problems.


## Examples


Before learning how you can create your own secret code to do some cool stuff, let's review some use cases that you can use it for:



	
  * Diagnose information - Presenting technical details of you app, such as generated UUID that can be used to find relevant server data.

	
  * Send internal data outside - You can dump app information, such as the SharedPrefs/DB data, and send it to you via email. Since this code runs on the same process as your app - you have access to everything!

	
  * Change settings - You can change the internal settings of your app. Showing more debug logs, communicating with a different server or even change the sorting order of the list you present the user. Anything that can help you understand more about the current situation of the app.

	
  * Demo mode - Need to show your app to investors? Showoff to your Friends? You can create shortcuts to some app features that are relatively difficult to simulate. For example: Notification that triggers every morning or when the phone is idle for more than 30mins. Maybe a SMS/phone-call triggered action, QR code scan or a sensor's incoming data. You can create a demo screen to start everything you want to show on the app, and with sample data to make a neat presentation.




## How?


Let's see how to create one of our own secret code:

{% highlight xml %}
<receiver android:name=".receiver.DiagnoserReceiver">
    <intent-filter>
        <action android:name="android.provider.Telephony.SECRET_CODE"/>
        <data android:scheme="android_secret_code" android:host="111222"/>
    </intent-filter>
</receiver>
{% endhighlight %}

That's what you put on your manifest. Looks simple right? All we do is provide a listener to a secret code, defined in a \<data\> tag as the "android:host" attribute, and declare a receiver for it. On the receiver, we can just run some code, open an Activity, run a Service etc. Here's an example of a receiver that starts a new Activity:

{% highlight java %}
public class DiagnoserReceiver extends BroadcastReceiver {
	public void onReceive(Context context, Intent intent) {
        if ("android.provider.Telephony.SECRET_CODE".equals(intent.getAction())) {
            Intent i = new Intent(Intent.ACTION_MAIN);
            i.setClass(context, Diagnoser.class);
            i.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            context.startActivity(i);
        }
	 }
}
{% endhighlight %}

We create an intent with the Activity class, Diagnoser, and start it. That activity will run on the same process as our app, allowing us to access all the data that our app has, using all the permissions that our app has been granted with.


## Use it


To start our diagnose screen, all you need to do is go to the phone's **stock** dialer (third party dialers not always work with secret codes) and dial the secret code you defined in the "android:host" attribute on the manifest, in the following pattern: "*#*#<secret-code>#*#*. In our example, you'll need to dial *#*#111222#*#*, the DiagnoserReceiver will receive the intent and start the Diagnoser activity.

That's it. I hope this post helped you get more from your app. If you have more ideas for use cases for this feature - write them on the comments to help us all exploit this nice feature.
