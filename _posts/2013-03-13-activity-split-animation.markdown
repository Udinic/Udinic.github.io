---
author: udinic
comments: true
date: 2013-03-13 14:15:32+00:00
layout: post
slug: activity-split-animation
title: Activity Split Animation
wordpress_id: 394
categories:
- Android
- Animation
tags:
- activity
- activity animation
- Android
- animatiion
- split animation
---

This week I had the time to write a small and cool transition animation between Activities.

The regular transition animations between activities animate the Activity as a whole. I wanted to create an animation that split Activity A into 2 parts, animate them in the way out and revealing Activity B. Here’s a gif that illustrate this:

[![split_anim3](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/split_anim3.gif)](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/split_anim3.gif)

The idea is fairly simple:

1. Save Activity A as a bitmap
2. Split bitmap into 2 parts
3. Pass the 2 bitmaps to Activity B
4. Show the bitmaps on top of Activity B’s layout
5. Animate the bitmaps outwards
6. User is now seeing Activity B

The implementation was not as straightforward as I thought. I ran into some difficulties on the way, but I found solutions to all the problems I encountered. Let’s go step by step.

**Note**: This implementation requires holding a snapshot bitmap of the entire screen. For some devices that might be a very expensive operation due to low memory and/or large screen area. If you choose to use this animation - do it with care and don't overuse it. 



## Save as Bitmap


In order to get a bitmap of the Activity, we can use this code:


[sourcecode language="java"]
View root = currActivity.getWindow().getDecorView().findViewById(android.R.id.content);
root.setDrawingCacheEnabled(true);
Bitmap bmp = root.getDrawingCache();
[/sourcecode]




On the first line, we get the root view for the activity. We find the view with the id of  _android.R.id.content, _which is the FrameLayout that exist on every layout and contains the layout you put on setContentView(). Here’s how it looks on [HierarchyViewer](http://developer.android.com/tools/help/hierarchy-viewer.html):

[![image](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image_thumb14.png)](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image14.png)

In order to get a bitmap of the root view, or any view for that matter, we call the [getDrawingCache()](http://developer.android.com/reference/android/view/View.html#getDrawingCache(boolean)) method. This will return the cached bitmap of this view in case we have enabled the caching for it. That’s why we also call the [setDrawingCacheEnabled](http://developer.android.com/reference/android/view/View.html#setDrawingCacheEnabled(boolean))() before that. If there isn’t a cached bitmap for the view – it’s been created on the spot.


## Split the Bitmap


In order to split the bitmap, I wrote this code:


{% highlight java %}
Bitmap mBmp1 = Bitmap.createBitmap(bmp, 0, 0, bmp.getWidth(), splitYCoord);
Bitmap mBmp2 = Bitmap.createBitmap(bmp, 0, splitYCoord, bmp.getWidth(), bmp.getHeight() - splitYCoord);
{% endhighlight %}




_bmp _is the main Bitmap for the entire Activity.
_splitYCoord _is the Y coordinate of the splitting point.

I create 2 smaller bitmaps. mBmp1 is the upper part of the larger bitmap and mBmp2 is the bottom part. Each part’s height is determined by the splitYCoord.



## Pass the bitmaps to the next Activity


After I have the 2 bitmaps, I want to start the next Activity and put them on top of its layout. That way the user will see Activity A’s layout where in fact we already moved on to Activity B.

At first, I wanted to pass them as [Extras](http://developer.android.com/reference/android/content/Intent.html#putExtra(java.lang.String, android.os.Parcelable)) of the Intent, it’s possible since Bitmap is [Parcelable](http://developer.android.com/reference/android/os/Parcelable.html) . The problem is the bitmaps are too large to transport in an Intent, due to size limitation for IPCs. This is the error I got:


{% highlight java %}
 !!! FAILED BINDER TRANSACTION !!!
{% endhighlight %}




There are few workarounds for this, including writing the bitmaps into a file and then read them back on the other end. I found the easiest and fastest way will be to just keep them as data members in a common area. I created a static class to contain the bitmaps and all the logic to create and animate them. More on that later.


## Show the bitmaps on Activity B


After I start Activity B, disabling any default activity animation using [overridePendingTransition](http://developer.android.com/reference/android/app/Activity.html#overridePendingTransition(int, int))(), I create 2 ImageViews to contain the previously created bitmaps, and presenting them on screen. I do that before calling _setContentView() _to avoid seeing Activity B’s layout ahead of time.

The ImageViews are added directly to the Window of the activity. This allows the ImageViews be on top of the soon-to-be inflated layout and gives more flexibility when deciding the location of each one of them on the screen.


{% highlight java %}
ImageView imageView = new ImageView(destActivity);
imageView.setImageBitmap(bmp);

WindowManager.LayoutParams windowParams = new WindowManager.LayoutParams();
windowParams.gravity = Gravity.TOP;
windowParams.x = loc[0];
windowParams.y = loc[1];
windowParams.height = ViewGroup.LayoutParams.WRAP_CONTENT;
windowParams.width = ViewGroup.LayoutParams.WRAP_CONTENT;
windowParams.flags =
                WindowManager.LayoutParams.FLAG_LAYOUT_IN_SCREEN;
windowParams.format = PixelFormat.TRANSLUCENT;
windowParams.windowAnimations = 0;
destActivity.getWindowManager().addView(imageView, windowParams);
{% endhighlight %}




Pretty straightforward.

The gravity states where we’ll put our layout in the window. Since we calculated the X,Y position of the bitmaps relative to the top of the screen – we’ll set the gravity to TOP.


## Animate the bitmaps


After we created and positioned the bitmaps on Activity B, we can call _setContentView_() to inflate the Activity’s layout. After the layout is inflated we can start the animation that pushes the 2 bitmaps outwards, thus revealing the Activity’s layout.


{% highlight java %}
mSetAnim = new AnimatorSet();
mTopImage.setLayerType(View.LAYER_TYPE_HARDWARE, null);
mBottomImage.setLayerType(View.LAYER_TYPE_HARDWARE, null);
mSetAnim.addListener(new Animator.AnimatorListener() {
                    @Override
                    public void onAnimationEnd(Animator animation) {
                        clean(destActivity);
                    }

                    @Override
                    public void onAnimationCancel(Animator animation) {
                        clean(destActivity);
                    }
						...
                });

// Animating the 2 parts away from each other
Animator anim1 = ObjectAnimator.ofFloat(mTopImage, translationY, mTopImage.getHeight() * -1);
Animator anim2 = ObjectAnimator.ofFloat(mBottomImage, translationY, mBottomImage.getHeight());

mSetAnim.setDuration(duration);
mSetAnim.playTogether(anim1, anim2);
mSetAnim.start();
{% endhighlight %}




The animation is a simple TranslationY animation that moves every ImageView outside the screen, each one to a different directions. I use hardware acceleration (read my [last blog post](http://udinic.wordpress.com/2013/03/04/android-app-to-the-challenge/) to learn more about HW accelerated animations) and I do some cleaning after the animation is finished or canceled (removing hardware layer, removing the ImageViews from the window etc.).


## How to use my Animation


So I was debating with myself what’s the best way to make it easier to use, without limiting the developer too much. The easiest way was to create a base Activity that the developer can extend and the actions will be taken care of without too much effort. The reason I **didn’t** do that is because I **HATE **extending from other base activities just to get some more features. If your application has it’s own base activity it can be very frustrating if every library will demand you to extend from their own branded Activity.

That’s the reason I created a simple class with static methods that do all the work. here’s the API:


{% highlight java %}
/**
 * Utility class to create a split activity animation
 *
 * @author Udi Cohen (@udinic)
 */
public class ActivitySplitAnimationUtil {

    /**
     * Start a new Activity with a Split animation
     *
     * @param currActivity The current Activity
     * @param intent       The Intent needed tot start the new Activity
     * @param splitYCoord  The Y coordinate where we want to split the Activity on the animation. -1 will split the Activity equally
     */
    public static void startActivity(Activity currActivity, Intent intent, int splitYCoord);

    /**
     * Start a new Activity with a Split animation right in the middle of the Activity
     *
     * @param currActivity The current Activity
     * @param intent       The Intent needed tot start the new Activity
     */
    public static void startActivity(Activity currActivity, Intent intent);

    /**
     * Preparing the graphics on the destination Activity.
     * Should be called on the destination activity on Activity#onCreate() BEFORE setContentView()
     *
     * @param destActivity the destination Activity
     */
    public static void prepareAnimation(final Activity destActivity);

    /**
     * Start the animation the reveals the destination Activity
     * Should be called on the destination activity on Activity#onCreate() AFTER setContentView()
     *
     * @param destActivity the destination Activity
     * @param duration The duration of the animation
     * @param interpolator The interpulator to use for the animation. null for no interpulation.
     */
    public static void animate(final Activity destActivity, final int duration, final TimeInterpolator interpolator);

    /**
     * Start the animation that reveals the destination Activity
     * Should be called on the destination activity on Activity#onCreate() AFTER setContentView()
     *
     * @param destActivity the destination Activity
     * @param duration The duration of the animation
     */
    public static void animate(final Activity destActivity, final int duration);

    /**
     * Cancel an in progress animation
     */
    public static void cancel();
}
{% endhighlight %}




In order to use it, just call this from Activity A when you want to animate to Activity B:


{% highlight java %}
ActivitySplitAnimationUtil.startActivity(Activity1.this, new Intent(Activity1.this, Activity2.class));
{% endhighlight %}




And call this on Activity B’s _onCreate_():


{% highlight java %}
// Preparing the 2 images to be split
ActivitySplitAnimationUtil.prepareAnimation(this);

// Setting the Activity's layout
setContentView(R.layout.act_two);

// Animating the items to be open, revealing the new activity.
// Animation duration of 1 second
ActivitySplitAnimationUtil.animate(this, 1000);
{% endhighlight %}




That’s it!

No need to extend anything, just 3 simple calls to static methods.

The code supports API14+ but it can easily be converted to work on older devices using [NineOldAndroid](https://github.com/JakeWharton/NineOldAndroids).

The repository is here:

[https://github.com/Udinic/ActivitySplitAnimation](https://github.com/Udinic/ActivitySplitAnimation)

Use it, Fork it, Extend it.


## What’s next


There are more things that can be done to extends it:

1. Vertical splitting – Let the activity split also to the sides.
2. Split the activity to more than 2 parts
3. You name it!
