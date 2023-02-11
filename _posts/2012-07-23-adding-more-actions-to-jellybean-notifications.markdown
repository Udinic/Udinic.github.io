---
author: udinic
comments: true
date: 2012-07-23 22:40:55+00:00
layout: post
slug: adding-more-actions-to-jellybean-notifications
title: Adding more actions to JellyBean Notifications
wordpress_id: 275
categories:
- Android
tags:
- actions
- Android
- jelly-bean
- Jellybean
- Notification bar
- NotificationCompat2
- notifications
- RemoteViews
---

One of most useful features on Android are the Notifications. While iOS just got them – Android keeps improving them. JellyBean has presented some sweet notifications’ features. Larger area, multiline text entry, list of different items and pictures. There’s also an option to create your own custom layout to present there (though I think it potentially can ruin the uniformity of the notifications design). All those “tall” notifications can be also collapsed/expended to save some space. One more thing, and this could be the coolest of all, is the option to add **actions **to the notifications.

![image](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image_thumb5.png)   ![image](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image5_thumb.png)

(Sample notifications on JellyBean. Taken using [NotificationCompat2](https://github.com/JakeWharton/NotificationCompat2) sample app, by Jake Wharton)

Three actions don’t (always) make it right.

Each notification can suggest **up to 3 actions**, but what if I want **more**? Maybe I want that a click on one action will give some other actions to pick from? I know it’s not a common case, but I can think of several cases where it can be useful. Since all the notifications are built on the same way AppWidgets are built, using RemoteViews, there are not much place for run-time manipulation.

Seems like the **old switch-a-roo** will do the trick here.

Changing the actions is possible by replacing the whole notification with a new one. The replacing notification can have a different title and different actions from the previous one, but it must have the same notification id. This is how it’ll look like:

[![image](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image_thumb6.png)](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image6.png)

How to do that?

First, let’s clarify something – notification’s actions don't just have an OnClickListener like regular buttons, they fire an Intent using a PendingIntent. A receiver needs to receive that Intent and “notify” the replacing notification to the Android’s Notification Service. The receiver can be a BroadcastReceiver, Service or even an Activity, but we want something that doesn’t take much resources/loading-time (as an Activity) or doesn’t have the potential to be called from other applications (as a BroadcastReveiver). A Service, or even simpler – IntentService, will fit perfectly here.


{% highlight java %}
public class NotificationReplaceService extends IntentService {

    public static String ACTION_SWITCH_NOTIFICATIONS = "com.jakewharton.notificationcompat2.SWITCH_NOTIFICATIONS";
    public static String SWITCH_NOTIFICATION_ARG_ID = "NOTIF_ID";
    public static String SWITCH_NOTIFICATION_ARG_NOTIFICATION = "NOTIF_NOTIFICATION";

    public NotificationReplaceService() {
        super("NotificationReplaceService");
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        if (ACTION_SWITCH_NOTIFICATIONS.equals(intent.getAction())) {

            int notifiId = intent.getIntExtra(SWITCH_NOTIFICATION_ARG_ID, -1);
            Notification notification = intent.getParcelableExtra(SWITCH_NOTIFICATION_ARG_NOTIFICATION);

            // Creating the new notification based on the data came from the intent
            NotificationManager mgr = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
            mgr.notify(notifiId, notification);
        }
    }
}
{% endhighlight %}




This service supports the “SWITCH_NOTIFICATION” action, which has 2 extras:



	
  * Notification Id: Must be the same as the notification we’re replacing

	
  * Notification object: The second notification we replace the current notification with


Luckily for us – the Notification class is [Parcelable](http://developer.android.com/reference/android/os/Parcelable.html), meaning I can pass it as one of the Intent’s extras with no problem.

To illustrate this idea, I used Jake Wharton’s [NotificationCompat2](https://github.com/JakeWharton/NotificationCompat2) project, which gives a simple API to build notifications for all Android versions from 1.6 to 4.1. My service was added to the manifest


{% highlight xml %}
        <!-- Declaring the service -->
        <service android:name="com.jakewharton.notificationcompat2.NotificationReplaceService"/>
{% endhighlight %}




I created the first notification, the one to be replaced:


{% highlight java %}
Notification notiMain = getSimple("Action with extension")
        .addAction(android.R.drawable.sym_def_app_icon, "Action", getPendingIntent())
        .addAction(android.R.drawable.sym_def_app_icon, "More..", intentReplaceNotification)
        .build();
{% endhighlight %}




The _intentReplaceNotification _is the PendingIntent fired when pressing the “More..” action on the notification. This pending intent is the one in charge of calling the _NotificationReplaceService_ with the replacing notification.


{% highlight java %}
// First - Building the extended notification.
// This will replace the first one when the "More.." button will be pressed
Notification notiReplacement = getSimple("Actions extended")
        .addAction(R.drawable.no_icon, "udinic1", getPendingIntent())
        .addAction(R.drawable.no_icon, "udinic2", getPendingIntent())
        .addAction(R.drawable.no_icon, "udinic3", getPendingIntent())
        .build();

// Creating the switch notification intent.
// The receiver will get this and send notiExtended to the NotificationManager, replacing the current one with the same Id
Intent switchNotificationIntent = new Intent();
switchNotificationIntent.setClass(SampleActivity.this, NotificationReplaceService.class);
switchNotificationIntent.setAction(ACTION_SWITCH_NOTIFICATIONS);
switchNotificationIntent.putExtra(SWITCH_NOTIFICATION_ARG_ID, notifiId);
switchNotificationIntent.putExtra(SWITCH_NOTIFICATION_ARG_NOTIFICATION, notiReplacement);
PendingIntent intentReplaceNotification = PendingIntent.getService(SampleActivity.this, 0, switchNotificationIntent, 0);
{% endhighlight %}




Here we created the replacing notification, and passing it to the replacing action intent. This intent will start my _NotificationReplaceService, _which will use the intent’s extras to replace the main notification.

All that is left now to do is to notify the main notification to the Android’s Notification Service


{% highlight java %}
NotificationManager mgr = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
mgr.notify(notifiId, notiMain);
{% endhighlight %}




And that’s it!

You can check the full code on my [GitHub account](https://github.com/Udinic/NotificationCompat2). Just please do me one favor – don’t abuse this method to create a complex menu in the users’ notification bar, think about your users!
