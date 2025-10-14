---
author: udinic
comments: true
date: 2012-01-04 23:49:08+00:00
layout: post
slug: problem-solving-using-the-android-source-code
title: Problem solving using the Android source code
wordpress_id: 217
categories:
- Android
- AOSP
tags:
- Android
- AOSP
- baseline
- baselineAligned
- debug
- gap
- layout code
- LinearLayout
- source-code
---

On one of my latest posts, I've showed how to download the Android source code, even if you work on Windows (The official instructions are for Linux to Mac). You can check that post [here](http://udinic.wordpress.com/category/android/aosp/). Having the Android source code can be useful not just to find bugs in the operation system, which I bet you do on your spare time, but can also help understand problems in your own code. To demonstrate, here's a nice story about a problem I had while developing some layout, and found the problem using the Android source code:

I didn't intend to make any complex layout, just a simple LinearLayout with 3 buttons. I wanted the buttons to be even by size, take all the space they can have and center the text inside of them. Sounds simple, right? I wrote the following layout:

{% highlight xml %}

<LinearLayout android:layout_height="70dip"
 android:layout_width="fill_parent"
 android:background="#ffffff">

<Button
 android:layout_width="0dip"
 android:layout_height="70dip"
 android:gravity="center"
 android:layout_weight="1"
 android:textSize="15sp"
 android:text="Udinic"/>

<Button
 android:layout_width="0dip"
 android:layout_height="70dip"
 android:gravity="center"
 android:layout_weight="1"
 android:textSize="15sp"
 android:text="Udinic with a long text"/>

<Button
 android:layout_width="0dip"
 android:layout_height="70dip"
 android:gravity="center"
 android:layout_weight="1"
 android:textSize="15sp"
 android:text="Chopi"/>
 </LinearLayout>
{% endhighlight %}

When I ran the code, that's what I got:

[![](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/1.png)](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/1.png)

That's odd...I gave the buttons the same height as the layout, why the hell the middle one got this gap above it?? Changing the layout_height to "fill_parent" to each of them will solve the problem on this example, but in my case it didn't (I had a little more complex layout :)). Since it seems like it should be as it is in my head, I've decided to get to the bottom of this.

My first step was to check what arguments the [onLayout()](http://developer.android.com/reference/android/view/View.html#onLayout(boolean, int, int, int, int)) gets. This method is basically in charge of assigning the views' positions on the layout. Since we have a position problem, it'll be square one for us. I've created a subclass to Button and override the onLayout() method.

{% highlight java %}

@Override
 protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
 super.onLayout(changed, left, top, right, bottom);
 }

{% endhighlight %}

I started debugging, not before adding a breakpoint on that method. A peek on the arguments showed me that the "top" argument is the bad boy here. It's got a 9 instead of a plain 0, which causing this gap on top of the button. Using IntelliJ and the Android source code, I could get up the Call Stack for this method, and found the following code in the LinearLayout's source:

{% highlight java %}
....
switch (gravity & Gravity.VERTICAL_GRAVITY_MASK) {
    case Gravity.TOP:
      childTop = paddingTop + lp.topMargin;
      if (childBaseline != -1) {
         childTop += maxAscent[INDEX_TOP] - childBaseline;
      }
 ....

{% endhighlight %}


The "childTop", later to be know as just "top", is getting its value from the childBaseline, which is calculated on the TextView class, and "Button" is extending it:



{% highlight java %}

@Override
 public int getBaseline() {
    if (mLayout == null) {
       return super.getBaseline();
    }

   int voffset = 0;
    if ((mGravity & Gravity.VERTICAL_GRAVITY_MASK) != Gravity.TOP) {
       voffset = getVerticalOffset(true);
    }

   return getExtendedPaddingTop() + voffset + mLayout.getLineBaseline(0);
 }

{% endhighlight %}


Only the mLayout.getLineBaseline(0) was held responsible for the evil gap, and after exploring it a bit got me to realize that this is the baseline for the layout, in charge of making all the text in all the TextView/Buttons under it to be aligned to the same horizontal line! After looking closely at my phone, I could see that the text is really aligned to the same baseline (and in the real case, it wasn't as obvious as on this example). This might be great for other applications of buttons inside a layout, but not for my case. Apparently the layout's "baselineAligned" flag is always "true" unless told otherwise, which is kind of confusing if you'll ask me. Adding the android:baselineAligned="false" to the layout got rid of the problem:




[![](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/2.png)](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/2.png)




This experience can show you how you can, pretty easily, find problems in your code just by having the operation system's code. Maybe a clearer behavior of the LinearLayout could be better, but sadly every framework, no matter how great it is, got its dark side.




Download the source code, attach it to your IDE, it'll probably come in handy in the future.
