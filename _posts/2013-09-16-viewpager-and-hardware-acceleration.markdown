---
author: udinic
comments: true
date: 2013-09-16 13:30:10+00:00
layout: post
slug: viewpager-and-hardware-acceleration
title: ViewPager and Hardware acceleration
wordpress_id: 587
categories:
- Android
- Animation
- Performance
tags:
- Android
- enableLayers
- gpu
- hardware acceleration
- hardware layer
- HW layers
- OnPageChangeListener
- optimizing hardware layers
- PageTransformer
- performance
- romain guy
- setLayerType
- setScrollState
- Show hardware layers updates
- traceview
- ViewPager
---

Last week, Romain Guy posted about [Optimizing hardware layers](http://www.curious-creature.org/2013/09/13/optimizing-hardware-layers/). He explained how hardware layers are a great way to boost the performance of your animations, but can also degrade it.

When your view is backed by a hardware layer, that layer will get updated every time the view itself will update, and after that the layer will be drawn to the screen. Meaning, if your view is altered during an animation – more work is done per frame and you’ll get a drop in your frame-rate. That’s why it’s not wise to change your animated view while it’s backed by a HW layer.

On his post, Romain suggested an easy way to spot such problems, using the “Show hardware layers updates” under the Developer Options. Every time a HW layer is updated – it’ll be highlighted in green. 

So I tried it.

<!--break-->

I played with one of the apps I’m currently working on, and saw that when scrolling my ViewPager to the sides – its pages are highlighted in green for the **entire **scrolling phase. Since i don’t back any of the pages there with HW layer, I was trying to find who does. For that, I used a small trick – I used DDMS to run TraceView for the time I scroll the ViewPager, sorted the methods calls by name, searched for “android/view/View.setLayerType” and then followed its parents until I found the cause – ViewPager#enableLayers():


{% highlight java %}
private void enableLayers(boolean enable) {
    final int childCount = getChildCount();
    for (int i = 0; i < childCount; i++) {
        final int layerType = enable ?
                ViewCompat.LAYER_TYPE_HARDWARE : ViewCompat.LAYER_TYPE_NONE;
        ViewCompat.setLayerType(getChildAt(i), layerType, null);
    }
}
{% endhighlight %}



This method is in charge of enabling/disabling HW layer for the ViewPager's children. It’s called once from ViewPaper#setScrollState():


{% highlight java %}
private void setScrollState(int newState) {
    if (mScrollState == newState) {
        return;
    }

    mScrollState = newState;
    if (mPageTransformer != null) {
        // PageTransformers can do complex things that benefit from hardware layers.
        enableLayers(newState != SCROLL_STATE_IDLE);
    }
    if (mOnPageChangeListener != null) {
        mOnPageChangeListener.onPageScrollStateChanged(newState);
    }
}
{% endhighlight %}


As we can see, the HW is disabled when the scroll state is IDLE and enabled otherwise, on DRAGGING or SETTLING. This is a good thing do to, especially when the ViewPager uses a PageTransformer. A PageTransformer meant to “apply a custom transformation to the page views using animation properties” ([Source](http://developer.android.com/reference/android/support/v4/view/ViewPager.PageTransformer.html)), and since we're doing animations when we scroll our pages, we can understand why it’s a good thing to use HW layers.

But, what if the PageTransformer **changes **the view as it animates it? 

Maybe our PageTransformer changes the page’s background color to become darker as the page scroll? Or maybe it moves some items inside the page? In that case, the HW layer will get updated for **every frame. **This is the reason my app was showing green when I scrolled the pager – I applied View changes during the page transformation.

So now that we have found the problem – how do we solve it? Well, I thought about overriding one of the ViewPager’s methods I showed above, but since they are private – it’s not possible. But I was able to workaround the problem in a different approach: Notice that on ViewPage#setScrollState(), after calling enableLayers(), we also call OnPageChangeListener#onPageScrollStateChanged(), in case such listener was set. So what I did was to set a listener that reset the layer type of all the ViewPager’s children to NONE when its scroll state is different than IDLE:


{% highlight java %}
@Override
public void onPageScrollStateChanged(int scrollState) {
    // A small hack to remove the HW layer that the viewpager add to each page when scrolling.
    if (scrollState != ViewPager.SCROLL_STATE_IDLE) {
        final int childCount = <your_viewpager>.getChildCount();
        for (int i = 0; i < childCount; i++)
            <your_viewpager>.getChildAt(i).setLayerType(View.LAYER_TYPE_NONE, null);
    }
}
{% endhighlight %}



That way, after ViewPager#setScrollState() set a HW layer for the pages – I set them back to NONE, which disables the use of HW layers. The major difference was shown on the Nexus 7, since the screen is bigger and so was the area my ViewPager was covering. 

Problem solved! No green is now shown when scrolling through my ViewPager. 

After solving this problem, I decided to report it to Google, since there must be a better way. Maybe some kind of a setter in ViewPager to disable the use of HW layers when scrolling through the pages. That’s why I [reported this](https://code.google.com/p/android/issues/detail?id=60094) on Google’s Issue tracker. Hopefully someone will see it and maybe even fix it.
