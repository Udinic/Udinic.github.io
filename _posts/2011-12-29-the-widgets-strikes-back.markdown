---
author: udinic
comments: true
date: 2011-12-29 13:13:17+00:00
layout: post
slug: the-widgets-strikes-back
title: The widgets strikes back
wordpress_id: 203
categories:
- Android
- Widget
tags:
- ADW Launcher
- Android
- Any.DO
- app widget
- launcher
- open source
- widget
---

We're getting LOTS of feedback about Any.DO. The users write us some kind words about the app, maybe report about a problem they were having and mostly suggesting very useful features we could add for future versions. Of course, some feature requests aren't that reasonable (Klingon translation? Really??). The most requested features were widgets related: black theme, more sizes, option to scroll over the tasks' list etc. I gathered all those requests and got me a 3 day trip to Wigeteria-Lane.


One of the widgets is having a ticker to present today's tasks. The ticker uses a recurring alert to update the current showing task. The reason I'm using alert and not the onUpdate() method. is that the onUpdate can be called in intervals of minimum 30 mins. which is way too long to wait between tasks! I start the alert on the onEnabled() and cancel it on the onDisabled(). They are called when the **first instance** of that widget was added to the screen, and when the** last instance** was removed from the screen, respectively. But one day I've noticed the log messages from the widget are running, even though it's not currently placed on my home screen!




I couldn't find the cause for the problem and I didn't see anything in my code that could lead to such behavior. Luckily, I always work with my Logcat window open (you'll never know when a hidden exception will pop up), and then it happened again!




I'm using CyanogenMod on my device, which has the ADW Launcher built in. When I was about to add the widget to my home screen, I got the "Widget span config" dialog, allowing me to choose myself the size I want to grant the widget (it's a feature belongs to the launcher itself) and then I've clicked the **back button**. On my log I could see the onEnabled() **do** gets called, and when I clicked on the back button I saw that the onDisabled() was **not called** at all! The alert was running and gave log messages for each cycle, though the widget was not added to the home screen. Trying to add an instance of the widget and removing it didn't cause the onDisabled() to be called, and even restarting the device didn't make the widget disappear from the background. I've decided to take a side-trip to see what's wrong with the ADW Launcher.




Have I mentioned I love open-source projects? Well, I do. The ADW Launcher is [open-sourced](http://code.google.com/p/adw-launcher-android/) - few minutes later I found myself looking at its code on my IntelliJ. The shortest way to find the code for that problematic dialog, is to search its title. That's the code for this dialog's OK button:



{% highlight java %}

alertDialog.setButton(DialogInterface.BUTTON_POSITIVE, getResources().getString(android.R.string.ok),
 new DialogInterface.OnClickListener() {
     public void onClick(DialogInterface dialog, int which) {
         spans[0]=ncols.getCurrent();
         spans[1]=nrows.getCurrent();
         realAddWidget(appWidgetInfo,cInfo,spans,appWidgetId,insertAtFirst);
     }
 });

{% endhighlight %}


The realAddWidget() is in charge of the actual inflation of the widget, but if there's not enough room at the home screen, the onDisabled() **do** gets called. We can see it on the code:



{% highlight java %}
if (!findSlot(cellInfo, xy, spans[0], spans[1])) {
    if (appWidgetId != -1) mAppWidgetHost.deleteAppWidgetId(appWidgetId);
        return;
}
{% endhighlight %}


The deleteAppWidgetId() is calling the onDisabled() method eventually. Using the mAppWidgetHost.deleteAppWidgetId() I was able to track down the function in charge of adding the widget - mAppWidgetHost.allocateAppWidgetId() and after some looking around, I found out that it gets called right after selecting the widget from the list, before the span config dialog shows. Which means, if anything goes wrong after that - we need to call the mAppWidgetHost.deleteAppWidgetId() to remove the added widget. The case where there's no space left on the screen is already handled, but what about canceling the "Widget span config" dialog? It's called **after** the allocation of the widget. I got back to the code creating that dialog, and saw there's no handling for the case where the dialog is canceled. That's the cause of the problem! Mission accomplished.




In order to fix that, all needed to be done is to add an OnCancelListener to the dialog, with a call to the   mAppWidgetHost.deleteAppWidgetId() and that's it! I've submitted an [Issue ](http://code.google.com/p/adw-launcher-android/issues/detail?id=368)on the project's page, hopefully this will be fixed for future versions.




After that enlightening adventure, I got back to work on the widgets. The results can be seen on the version we published yesterday on the Android Market, go and [get it](https://market.android.com/details?id=com.anydo)! And don't forget to send us feedback, we're open-minded (even about the Klingon thing).
