---
author: udinic
date: 2015-09-15 09:30:10-05:00
layout: post
slug: speed-up-your-app
title: Speed up your app
categories:
- android
- performance
tags:
- android
- androiddev
- perfmatters
- droidcon
- droidconnyc 2015
- trade-offs
- systrace
- UI thread
- RenderThread
- Long View#draw()
- Slice
- Inflation during ListView recycling
- Scheduling delay
- 16ms
- 60 fps
- cpu duration
- wall duration
- traceview
- inclusive cpu time
- exclusive cpu time
- inclusive real time
- exclusive real time
- real time / call
- cpu cycles
- cpu time slice
- Memory Profiling
- GC event
- Garbage Collection
- Garbage Collector
- Heap dump
- hprof
- histogram
- reference tree
- memory footprint
- retained size
- shallow size
- heap count
- memory leak
- OutOfMemoryException
- OOM Exception
- Eclipse MAT
- Eclipse Memory Analyzer Tool
- Merge shortest paths to GC roots
- LeakCanary
- Allocation Tracker
- object pool
- enums
- autoboxing
- auto-box
- hashmap
- arraymap
- SparseIntArray
- SparseLongArray
- non-static inner class
- GPU profiling
- Android Studio 1.4
- Draw
- Prepare
- Process
- Execute
- OpenGL
- DisplayList
- display lists
- Measure pass
- Layout pass
- VSync delay
- Input Handling
- Command Issue
- Swap Buffers
- Profile GPU rendering
- adb shell dumpsys gfxinfo
- Marshmallow
- Janky frames
- 90th percentile 
- 95th percentile 
- 99th percentile 
- Number of missed Vsync
- Number high input latency
- Number slow UI thread
- Number slow bitmap uploads
- Number slow draw
- framestats
- On screen as bars
- complex view
- Hierarchy Viewer
- Android Monitor
- View Hierarchy
- onMeasure
- onLayout
- RelativeLayout
- LinearLayout
- positioning
- Overdraw
- Debug GPU overdraw
- setBackgroundDrawable
- Photoshop
- PSD
- Alpha
- Transparency
- setAlpha
- off-screen buffer
- setTextColor
- setImageAlpha
- hasOverlappingRendering
- onSetAlpha
- Hardware Layer
- Drawing model
- view layer
- hardware acceleration
- texture
- animation
- setLayerType
- LAYER_TYPE_HARDWARE
- LAYER_TYPE_NONE
- withLayer
- rotation
- scale
- translation
- translationx
- translationy
- pivot
- Show hardware layers updates
- ViewPager
- Page scroll
- onPageScrollStateChanged
- Android Performance Patterns
- Graphics Architecture
- Devoxx
- Google IO
- Preventative Optimizations
- Premature Optimizations
- case study
- Romain Guy
yt_talk: v3DlGOQAIbw
---


A few weeks ago, I gave a talk about Android Performance Optimization, at Droidcon NYC. 

I invested a lot of time in this presentation, since I wanted to show real examples for performance issues, and how to identify them with the available tools. I had to cut down about a half of my slides because I didn't have enough time to show everything. In this post, I'll summarize everything I was talking about, and show examples I didn't have time to go over.

The talk was recorded and you can watch it here:
{% include youtubePlayer_h.html id=page.yt_talk %}

The slides are also available:

<div style="width: 70%">
<script async class="speakerdeck-embed" data-id="596740dd42254419935aa604d098f2d3" data-ratio="1.77777777777778" src="//speakerdeck.com/assets/embed.js"></script>
</div>

Now, Let's go over some of the important things I was talking about, hopefully I can give a deeper explanation for everything, starting with the basic rules I follow when I work on optimizations:

<!--break-->

## My Rules

Everytime I approach a performance problem, or looking for performance problems, I follow these rules:

- __Always Measure__ - Optimizing with your eyes is never a good idea. After looking at the same animation for a few times, you'll start imagining it's running faster. Numbers don't lie. Use the tools we'll go over soon, and measure how your app performs a few times before and after you make your change.

- __Use slow devices__ - If you really want to expose all the weak spots, slower devices will help you more. Newer and stronger devices might not get too excited about performance issues you may have, but not all your users use the latest and greatest.

- __Trade-offs__ - Performance optimization is all about trade-offs. You optimize one thing - it comes on the expense of another. In many cases, that other thing can be your time spent finding and fixing it, but it can also be the quality of your bitmaps or the amount of data you should store in a specific data structure. Prepare yourself to make sacrifices.


## Systrace

Systrace is one of the greatest tools that you probably don't use. That's because developers weren't sure what to do with the information it provides. 

Systrace shows us an overview of what's currently running on the phone. This tool reminds us that the phone we hold in our hands, is actually is a powerful computer that can do many things at the same time. In one of the latest SDK tools updates, this tool was improved with generated insights from the data, helping us to find problems. Let's see how a trace file looks like:

[![img](/assets/images/{{ page.slug }}/systrace-overview.png)](/assets/images/{{ page.slug }}/systrace-overview.png)

You can generate a trace file using the Android Device Monitor tool, or using command line. You can find more info [here](http://developer.android.com/tools/help/systrace.html).

On the video, I explained the different sections. The interesting parts are the Alerts and the Frames, showing you insights the tool has generated based on the data it collected. Let's look at a trace I took, and select one of the alerts on the top:

[![img](/assets/images/{{ page.slug }}/systrace-alert.png)](/assets/images/{{ page.slug }}/systrace-alert.png)

The alert states that there was a long View#draw() call. We get a description, links to the documentation and even video links for relevant talks on that subject. Looking at the Frames row below, we see an indication for every frame that was rendered, and it's colored green, yellow and red to indicate a performance issue while rendering that frame. Let's select one of these red frames:

[![img](/assets/images/{{ page.slug }}/systrace-frame.png)](/assets/images/{{ page.slug }}/systrace-frame.png)

On the bottom, we'll see all the relevant alerts for that frame. We see there were 3 of those, one of them is the one we saw earlier. Let's zoom-in that frame and expand the "Inflation during ListView recycling" alert on the bottom:

[![img](/assets/images/{{ page.slug }}/systrace-frame-zoomin.png)](/assets/images/{{ page.slug }}/systrace-frame-zoomin.png)

We see the amount of time it took for this part, 32ms, which puts the frame's rendering time way over the 16ms boundary, a requirement to achieve 60fps. There's more timing information for each of the items in the ListView for that frame - about 6ms was spent per item, and we have 5 of these. The description helps us understand the problem, and even provides a solution. On the graph at the top, we see a visualization of everything, we can even zoom-in on the "inflate" slice to see what views took longer to inflate in the layout.

Another example of a slow rendered frame:

[![img](/assets/images/{{ page.slug }}/systrace-2-frame.png)](/assets/images/{{ page.slug }}/systrace-2-frame.png)

After selecting a frame, we can press the 'm' key to highlight it and see how long that part took. Looking at the top, we see it took over 19ms to render that frame. Expanding the only alert for that frame, shows us there was a "Scheduling delay".

A Scheduling delay means that the thread, processing that specific slice, was not scheduled on the CPU for a long time. Therefore, it took longer for this thread to complete. Selecting the longest slice in the frame shows more specific information:

[![img](/assets/images/{{ page.slug }}/systrace-2-slice.png)](/assets/images/{{ page.slug }}/systrace-2-slice.png)

The __Wall duration__ is the time passed from the moment that slice started until finished. It's called "Wall duration", because it's like looking at a wall clock since the thread has started.

The __CPU duration__ is the actual time the CPU spent processing that slice.

It's noticeable that there's a wide difference between these durations. While it took 18ms to complete this slice, the CPU spent only 4ms working on it. That's a little strange, so now will be a good time to look up and see what the CPU was doing this entire time:

[![img](/assets/images/{{ page.slug }}/systrace-2-cpu.png)](/assets/images/{{ page.slug }}/systrace-2-cpu.png)

All 4 cores were pretty busy.

Selecting one the threads shows us where it was originated from, an app called __com.udinic.keepbusyapp__. In this case, a different app was causing the CPU to work harder, denying it from dedicating some energy to our app. 

While this specific scenario is usually temporary, since other apps don't often hog the CPU in the background (..right?), these threads could come from a different process on your app, or even from the main process. Since Systrace is an overview tool, there's a limit to how deep we can get. To find what's keeping our CPU busy in our app, we'll use another tool called Traceview.


## Traceview

Traceview is a profiling tool, showing how long it took for each method to run. Let's see how a trace file looks like:

[![img](/assets/images/{{ page.slug }}/traceview-overview.png)](/assets/images/{{ page.slug }}/traceview-overview.png)

The tool can be started from the Android Device Monitor and from code. More information is available [here](http://developer.android.com/tools/debugging/debugging-tracing.html). 

Let's go over the different columns:

- __Name__ - The name of the method, along with a color to identify it on the graph above.
- __Inclusive CPU Time__ - The time it took the CPU to process that method and its children (i.e. all the methods it called).
- __Exclusive CPU Time__ - The time it took the CPU to process that method alone.
- __Inclusive / Exclusive Real Time__ - The time that has passed from the moment the method started until completed. Same as "Wall duration" on Systrace.
- __Calls+Recursion__ - How many times this method was called, and the amount of recursive calls too.
- __CPU/Real time per Call__ - The CPU/Real time it took per call to this method, on average. The other time fields are showing the aggregated time of all calls to a method.

I opened an app that had hard time scrolling smoothly. I started the trace, scrolled a little and stopped the trace. I found the getView() method and expanded it, here's what I saw:

[![img](/assets/images/{{ page.slug }}/traceview-getview.png)](/assets/images/{{ page.slug }}/traceview-getview.png)

This method was called 12 times, the CPU spent ~3ms for each call, but the real time it took for each call to finish is 162ms! Definitely a problem..

Looking at the children of this method, we can see how the overall time splits between the different methods. The Thread.join() took ~98% of the inclusive real time. This method is used when we want to wait for another thread to finish. One of the other children is Thread.start(), which gets me to assume that the getView() method is starting a thread and waiting for it to finish. 

But where is that thread?

We can't see what that thread was doing as a child of getView(), since getView() is not doing that work directly . To look for it, I searched for a Thread.run() method, which is the method being invoked when spawning a new thread. I followed it until I reached the culprit:

[![img](/assets/images/{{ page.slug }}/traceview-thread.png)](/assets/images/{{ page.slug }}/traceview-thread.png)

I found that the BgService.doWork() method took ~14ms per call, and we have about 40 of these! There's a chance each getView() calls it more than once, and explains why each getView() call takes such a long time. This method was keeping the CPU busy for a long time. Looking at the Exclusive CPU time, we see that it used 80% of the CPU time in the entire trace! Sorting by the Exclusive CPU time is also a great way to find the busiest methods in the trace, it's possible they contribute to the performance issue you're experiencing.

Following time critical methods, such as getView(), View#onDraw() and others, will help us find the reasons why our app is acting slow. But sometimes, there's something else keeping the CPU busy, taking away precious CPU cycles, that could be spent on rendering our UI more smoothly. The Garbage Collector is running occasionally, clearing out unused objects, and usually don't have a strong impact on the app running in the foreground. If the GC is running too often, it could slow down our app, and it's possible that we are to blame..

## Memory Profiling

Android Studio was improved a lot lately, with more and more tools to help us find and analyze performace issues. The Memory tab on the Android window, will show us the amount of data being allocated on the heap over time. That's how it looks like:

[![img](/assets/images/{{ page.slug }}/mem-graph.png)](/assets/images/{{ page.slug }}/mem-graph.png)

Wherever we see little drops in the graph, a GC event has occurred, removing unused objects and freeing space on the heap.

There are 2 tools available on the left side of the graph: Heap dump and Allocation Tracker.

### Heap dump

In order to investigate what's currently allocated in our heap, we can use the heap dump button on the left. This will take a snapshot of what's currently allocated in the heap, and will show it in a special report screen inside Android Studio:

[![img](/assets/images/{{ page.slug }}/heap-overview.png)](/assets/images/{{ page.slug }}/heap-overview.png)

On the left, we see a histogram of the instances in the heap, grouped by their class name. For each one, there's the amount of objects allocated, the size of these instances (Shallow size) and the size these objects are retaining in memory. The latter tells us how much memory could be free if these instances will get freed. This view gives us an important glimpse to our app's memory footprint, helping us identify large data structures and objects relations. This information could help us build more efficient data structures, untying object connections to reduce the retained memory and ultimately - reducing the memory footprint as much as possible.

Looking at the histogram, we see that the MemoryActivity has 39 instances, which seems odd for an activity. Picking one of its instances on the right, will reveal all the references for this instance in the Reference Tree at the bottom.

[![img](/assets/images/{{ page.slug }}/heap-reftree.png)](/assets/images/{{ page.slug }}/heap-reftree.png)

One of them is part of an array inside an on object of ListenersManager. Looking at the other instances of the activity will reveal that all of them are retained by this object. This explains why the only object of this class is retaining so much memory:

[![img](/assets/images/{{ page.slug }}/heap-retained.png)](/assets/images/{{ page.slug }}/heap-retained.png)

This situations is notoriously called "Memory Leak", since the activities were clearly destroyed and that unused memory cannot be garbage collected due to that reference. We can avoid situations like this, by making sure our objects are not being referenced by other objects that outlive it. In this situation, the ListenersManager does not need to keep that reference after the activity was destroyed. A solution will be to remove that reference when the activity is about to be destroyed, in the onDestory() callback method. 

Memory leaks and other large objects are occupying large space in the heap, reducing the available memory and causing many GC events to try and free more space. These GC events will keep the CPU busy, causing performance degradation of our app. If the amount of available memory isn't sufficient for the app, and the heap cannot grow any bigger, a more dramatic result will happen - OutOfMemoryException, leading to an app crash.

A more advanced tool is the Eclipse Memory Analyzer Tool (Eclipse MAT):

[![img](/assets/images/{{ page.slug }}/eclipse-mat.png)](/assets/images/{{ page.slug }}/eclipse-mat.png)

This tool can do everything Android Studio can, and also identify potential memory leaks and provide more advanced instances searching, such as searching for all the Bitmap instances that are larger than 2 MB, or all the [empty Rect objects](http://kohlerm.blogspot.com/2009/04/analyzing-memory-usage-off-your-android.html).

Another great tool is a library called [LeakCanary](https://corner.squareup.com/2015/05/leak-canary.html), which tracks your objects and make sure they aren't leaked. If so - you'll get a notification to let you know what happened and where.

[![img](/assets/images/{{ page.slug }}/leakcanary.png)](/assets/images/{{ page.slug }}/leakcanary.png)

### Allocation Tracker 

The Allocation Tracker is started/stopped using one of the other buttons to the left of the memory graph. It will generate a report of all the instances being allocated at that period of time, grouped by class:

[![img](/assets/images/{{ page.slug }}/alloc-class.png)](/assets/images/{{ page.slug }}/alloc-class.png)

or by method:

[![img](/assets/images/{{ page.slug }}/alloc-method.png)](/assets/images/{{ page.slug }}/alloc-method.png)

There's also a nice visualization, showing us the biggest allocated instances.

Using this information, we can find time-critical methods that allocate too much memory and could trigger many GC events. We can also find many short-living instances for the same class, where we can consider using an [Object pool](https://en.wikipedia.org/wiki/Object_pool_pattern) to reduce the amount of allocations.

### General memory tips

Here are some quick tips/guidelines I use when I'm writing code:

- __Enums__ are already a hot subject when discussing performance. There's a [Video](https://youtu.be/Hzs6OBcvNQE) about it, showing the size enums spend, and a [discussion](https://plus.google.com/+JakeWharton/posts/bTtjuFia5wm) about that video and some potential misleading information it has. Do enums take more space than regular constants? Definitely. Is that bad? not necessarily. If you're writing a library and need a strong type safety, that could justify using it over other solutions, such as [@IntDef](https://developer.android.com/reference/android/support/annotation/IntDef.html). If you just have a bunch of constants that can be grouped together - it may not be wise to use an Enum for that purpose. As always, there's a trade-off that you need to consider when making this decision.

- __Auto-boxing__ - auto-boxing is the automatic conversion from primitive types to their object representation (e.g. int -> Integer). Every time a primitive type is being "boxed" to an object representation, a new object is created (shocking, I know). If we have many of these - the GC will run more frequently. It's easy not to notice the amount of auto-boxing that happens, because it's automatically done for us when assigning a primitive to an object. As a solution, try to be consistent with these types. If you use primitives throughout your app, try to avoid getting them auto-boxed for no real reason. You can use the memory profiling tools to find many objects representing a primitive type. You can also use Traceview and look for Integer.valueOf(), Long.valueOf() etc.

- __HashMap vs ArrayMap / Sparse*Array__ - In a related manner to the auto-boxing issue, using HashMaps requires us to use objects as keys. If we use the primitive "int" type in the app, and it's getting auto-boxed to Integer when interacting with a HashMap, we could probably just use SparseIntArray. In case we still need keys that are objects, we can use the ArrayMap class. It's very similar to HashMap, but it [works differently under the hood](https://www.youtube.com/watch?v=ORgucLTtTDI), making it more memory efficient, with a cost of being slower. Both alternatives have a smaller memory footprint than HashMap, but the time it takes to retrieve the items or allocate more space is a little higher than HashMap. Unless you have 1000+ or entries, there's almost no difference in the runtime, making them a viable option for your mapping purposes.

- __Context Awareness__ - As seen earlier, it's relatively easy to leak your activities' memory. You may wouldn't be surprised to learn that activities are the most common memory leak in Android(!). They are also very expensive to leak, since they hold all the view hierarchy of their UI, which could take a lot of space on its own. Many operations in the platform require a Context object, and an Activity is usually what you send. Make sure to understand what happens to that Activity. If a reference to it is cached, and that object lives longer than your Activity, without clearing that reference, you got yourself a memory leak.

- __Avoid non-static inner classes__ - When creating a non-static inner class, and instantiating it, you create an implicit reference to the outer class. If the inner class's instance is needed for longer period of time than the outer class, the outer class will still be retained in memory, even though it's not needed anymore. For example, creating a non-static class that extends AsyncTask inside an Activity class, then proceeding to start that async task and while it's running - killing the activity. That async task will keep that activity alive for as long as it runs. The solution - just don't do it, declare a __static__ inner class if needed. 

## GPU Profiling

A new addition to Android Studio 1.4, is a tool to profile GPU rendering. 

Under the Android window, go to the GPU tab, and you'll see a graph showing the time it took to render each frame on your screen:

[![img](/assets/images/{{ page.slug }}/gpu-overview.png)](/assets/images/{{ page.slug }}/gpu-overview.png)

Each bar on the graph represents one frame being rendered, and the colors represents the different phases in the process:

- __Draw (blue)__ - Represents the View#onDraw() method. That part builds/updates the DisplayList objects, being converted later to OpenGL commands the GPU can understand. High values can be due to complex views, requiring more time to build their display lists, or many views being invalidated at a short period of time.

- __Prepare (purple)__ - In Lollipop, another thread was added to help the UI Thread render the UI faster. That thread is called RenderThread. It's responsible for converting the display lists to OpenGL commands and sending them to the GPU. While that is happening, the UI thread can move on to start processing the next frame. The time it takes for the UI thread to pass all the resources to the RenderThread, is being in this step. If we have a lot of resources to pass on, such as many/heavy display lists, this step could take longer.

- __Process (red)__ - Executing the display lists to create OpenGL commands. This step could take longer if there are many/complex display lists to execute, because many views needs to be redrawn. A view can be redrawn due to an invalidation or if it's revealed by moving an overlapping view.

- __Execute (yellow)__ - Sending the OpenGL commands to the GPU. That part is a blocking call, since the CPU is sending a buffer with these commands to the GPU, expecting to get back a clean buffer for the next frame. The amount of buffers is limited, and if the GPU is too busy - the CPU will find itself waiting for one to get freed first. Therefore, if we see high values for this step, it probably means the GPU was busy drawing our UI, which could be too complex to be drawn at a shorter time.

In Marshmallow, more colors were added to indicate more steps, such as Measure/Layout, input handling and others:

[![img](/assets/images/{{ page.slug }}/gpu-colors-marsh.png)](/assets/images/{{ page.slug }}/gpu-colors-marsh.png)

EDIT 09/29/2015: John Reck, a framework engineer in Google, [has added](https://plus.google.com/+AlexLockwood/posts/Yj23fFY7Eon) this information about some of the new colors:

> The exact definition of "animation" is everything that's registered with Choreographer as CALLBACK_ANIMATION. This includes Choreographer#postFrameCallback and View#postOnAnimation which are what's used by view.animate(), ObjectAnimator, Transitions, etc... And yup, it's the same thing systrace labels as "animation".
>
> "misc" is the delay between the vsync's timestamp and the current timestamp when it was received. If you've ever seen logs from Choreographer about "Missed vsync by *blabla* ms skipping *blabla* frames", that now shows up as "misc". This is the difference between INTENDED_VSYNC and VSYNC in the framestats dump (https://developer.android.com/preview/testing/performance.html#timing-info)

But before using this feature, you need to enable GPU rendering first, from the developer options:

[![img](/assets/images/{{ page.slug }}/gpu-settings2.png)](/assets/images/{{ page.slug }}/gpu-settings2.png)

This will allow the tool to use ADB commands to get all the information it needs, and so are we (!), using:

{% highlight bash %}
adb shell dumpsys gfxinfo <PACKAGE_NAME>
{% endhighlight %}

we can receive all that data and create the graph ourselves. The command will print other useful information, such as the number of views in the hierarchy, size of all the display lists and more. In Marshmallow, we'll get even more stats.

[![img](/assets/images/{{ page.slug }}/gpu-adb.png)](/assets/images/{{ page.slug }}/gpu-adb.png)

If we have automated UI testing for our app, we can make our build server run this command after certain interactions (list scroll, heavy animation etc.) and see if there's a change in the values, such as "Janky Frames", over time. This could help identify a performance degrade after some commits were pushed, allowing us time to address the problem before the app hits production. We can get even more precise rendering information, when using the "framestats" keyword, as explained [here](https://developer.android.com/preview/testing/performance.html).

But that's not the only way to see this graph!

As seen in the "Profile GPU Rendering" developer option, there's also an option to see the graph "On screen as bars". Enabling that, will show the graph for each window on our screen, along with a green line to indicate the 16ms threshold.

[![img](/assets/images/{{ page.slug }}/gpu-onscreen.png)](/assets/images/{{ page.slug }}/gpu-onscreen.png)

On the right example, we can see that some frames crossed the green line, which means it took longer than 16ms to render them. Since the blue color seems to be dominating these bars, we understand there were many and/or complex views to draw. In that scenario, I scrolled over the newsfeed list, which supports different types of views. Some of the views are being invalidated and some are also more complex to render than others. It's possible that the reason some frames cross that threshold, is because there was a complex view to render at the time.

## Hierarchy Viewer

I love this tool, and it saddens me that many aren't using it at all!

Using the Hierarchy Viewer, we can get performance stats, see the complete view hierarchy on the screen and have access to all the views' properties. You can also dump the theme's data, see all the values used for each style attribute, but that's available only when running Hierarchy Viewer as a standalone, not from Android Monitor. I use this tool when I design my layouts and when I want to optimize them. 

[![img](/assets/images/{{ page.slug }}/hierview-overview.png)](/assets/images/{{ page.slug }}/hierview-overview.png)

At the center, we see a tree representing the view hierarchy. The view hierarchy can be wide, but if it's too deep (~10 levels), this may cost us with expensive layout/measurement phases. Every time a view is being measured, in View#onMeasure(), or when it's positioning all its child views, in View#onLayout(), these commands propagate to the child views, which do the same. Some layouts will do each step twice, such as RelativeLayout and some LinearLayout configurations, and if they are nested - the number of passes increases exponentially. 

At the bottom-right, we see a "blueprint" of our layout, marking where each view is positioned. We can select a view here, or in the tree, and see all its properties on the left. When designing a layout, I sometimes not sure why a certain view ended up where it is. Using this tool, I can track it on the tree, select it and see where it is in the preview window. I can design interesting animations by looking the final measurements of the views on the screen, and use that information to move things around accurately. I can find lost views that were overlapped by other views unintentionally, and much more.

[![img](/assets/images/{{ page.slug }}/hierview-colors.png)](/assets/images/{{ page.slug }}/hierview-colors.png)

For each view we have the time it took to measure/layout/draw it and all its child views. Colors indicate how this view performed in compare to the other views in the tree, a great way to find the weakest link. Since we also see a preview of that view, we can go over the tree and follow the steps created it, finding redundant steps that we could remove. One of those things, that has a great impact on performance, is called Overdraw.

## Overdraw

As seen in the GPU Profiling section - the Execute phase, represented by the yellow color on the graph, could take longer to complete if the GPU has many things to draw on the screen, increasing the time it takes to draw each frame. Overdraw occurs when we draw something on top of something else, say a yellow button on a red background. The GPU needs to draw the red background first and later the yellow button on top of that, making overdraws inevitable. If we have too many layers of overdraw, it will cause the GPU to work harder and be farther from the 16ms goal.

[![img](/assets/images/{{ page.slug }}/overdraw-gif.gif)](/assets/images/{{ page.slug }}/overdraw-gif.gif)

Using the "Debug GPU Overdraw" setting in the Developer Options, all overdraws will be colored to indicate the severity of the overdraw at that area. Having 1x/2x overdraw is fine, even some small light-red areas are not bad, but if we see too much red on the screen - we might have a problem. Let's see couple of examples:

[![img](/assets/images/{{ page.slug }}/overdraw-examples.png)](/assets/images/{{ page.slug }}/overdraw-examples.png)

On the left example, there's a list drawn in green, which usually is fine, but there's an overlay on the top that makes it red, and that's starting to become a problem. On the right example, the entire list is light red. In both cases, there's an opaque list that has 2x/3x overdraw. These overdraws could happen if there's a full screen background color to the window holding your Activity/Fragment, the list view and each list view item. We can solve such problem by setting a background color for only one of them. 

Note: the default theme declares a full screen background color to your window. If you have an activity with an opaque layout that covers the entire screen, you could remove the window background to remove one layer of overdraw. This can be done in the theme or in code, by calling getWindow().setBackgroundDrawable(null) inside onCreate().

Using Hierarchy Viewer, you can export all the layers of your hierarchy to a PSD file, to open in Photoshop. Investigating the different layers in Photoshop, will reveal all the overdraws in the layout. Use this information to remove redundant overdraws, and don't settle on the green, go for the blue!

## Alpha

Using transparency could have performance implications, and to understand why - let's see what happens when setting an alpha value to a view. Consider the following layout:

[![img](/assets/images/{{ page.slug }}/alpha-before.png)](/assets/images/{{ page.slug }}/alpha-before.png)

We see a layout that holds 3 ImageViews that overlap each other. In the direct/naive implementation, setting an alpha, using setAlpha(), will cause the command to propagate to all the child views, the ImageViews in this case. Later, these ImageViews will be drawn with that alpha value to the frame buffer. The result:

[![img](/assets/images/{{ page.slug }}/alpha-direct.png)](/assets/images/{{ page.slug }}/alpha-direct.png)

That's not what we want to see.

Since each ImageView was drawn with an alpha value, all the overlapping images blend together. Luckily, the OS has a solution to this problem. The layout will be copied to an off-screen buffer, the alpha will be applied to that buffer as a whole and the result will be copied to the frame buffer. The result:

[![img](/assets/images/{{ page.slug }}/alpha-complex.png)](/assets/images/{{ page.slug }}/alpha-complex.png)

But..we payed a price for that.

Drawing the view on an off-screen buffer, before drawing it on the frame buffer, is virtually adding another __undetected__ overdraw layer. The OS doesn't know when exactly to use this approach or the direct approach shown earlier, so the default is to always do the complex one. But there are still ways to set alpha and avoid the complexity the off-screen buffer adds:

- TextViews - Use __setTextColor()__ instead of setAlpha(). Using the alpha channel for the text color, will cause the text to be drawn directly using it.
- ImageView - Use __setImageAlpha()__ instead of setAlpha(). Same reason as for the TextView.
- Custom Views - If your custom view doesn't support overlapping views, this complex behavior is irrelevant to us. There's no way our child views will blend together, as seen in the example above. By overriding the __hasOverlappingRendering()__ method to return false, we're signaling the OS to take the direct/naive path with our view. There's also an option to manually handle what happens when setting an alpha, by overriding the __onSetAlpha()__ method to return true. 

## Hardware Acceleration

When Hardware Acceleration was introduced in Honeycomb, we got a [new drawing model](http://developer.android.com/guide/topics/graphics/hardware-accel.html) to render our app to the screen. It introduced the DisplayList structures, which records the view's drawing commands for faster rendering. But there's another great feature that developers sometimes miss or don't use properly - The View layers.

Using a View layer, we can render the View into an off-screen buffer (as seen earlier, when applying an Alpha channel) and manipulate it as we like. This feature is mainly great for animations, because we can animate complex Views quicker. Without layers, animating a View will invalidate it after changing the animated property (e.g. x coordinate, scale, alpha value etc.). For complex views, this invalidation propagates to all the child views, and they in turn will redraw themselves, a costly operation. Using a View layer, backed by Hardware, a texture is created in the GPU for our view. There are several operations we can apply on that texture without invalidating it, such as x/y position, rotation, alpha and more. All that means, that we can animate a complex view on our screen without invalidating it at all during the animation! This will make the animation much smoother. Here's a code example how to do this:

{% highlight java %}
// Using the Object animator
view.setLayerType(View.LAYER_TYPE_HARDWARE, null);
ObjectAnimator objectAnimator = ObjectAnimator.ofFloat(view, View.TRANSLATION_X, 20f);
objectAnimator.addListener(new AnimatorListenerAdapter() {
    @Override
    public void onAnimationEnd(Animator animation) {
        view.setLayerType(View.LAYER_TYPE_NONE, null);
    }
});
objectAnimator.start();

// Using the Property animator
view.animate().translationX(20f).withLayer().start();
{% endhighlight %}

Simple, right?

Yes, but there are a few things to remember when using hardware layers:

- __Clean up__ after your view - hardware layers consume space on a limited memory component, your GPU. Try and use them only for the time they needed, like an animation, and clean them up afterwards. In the ObjectAnimator example above, I applied a listener to remove the layer when the animation ends. On the Property animator example, I used the withLayers() method, which automatically creates the layer at the beginning and removes it when the animation ends.

- If you __change your View__ after applying a hardware layer, it will invalidate the hardware layer and will render the view to that off-screen buffer all over again. That will happen when changing a property that's not optimized for hardware layers (for now, these are optimized: rotation, scale, x/y, translation, pivot and alpha). For example, if you're animating a View, backed by hardware layer, by moving it across the screen while updating the View's background color as it moves, it will result with constant updating of the hardware layer. Updating the hardware layer has an overhead that could make using it not worthwhile 

For the second problem, there's a way to visualize these hardware layer updates. Using the Developer Options, we can enable the "Show hardware layers updates".

[![img](/assets/images/{{ page.slug }}/hwl-devoptions2.png)](/assets/images/{{ page.slug }}/hwl-devoptions2.png)

When enabled, the View is being flashed with a green color as it updates its hardware layer. I used it a while ago, when I had a ViewPager that didn't scroll as smooth as I expected. After enabling this developer option, I went ahead and scroll the ViewPager, and this is what I saw:

[![img](/assets/images/{{ page.slug }}/hwl-calproblem.gif)](/assets/images/{{ page.slug }}/hwl-calproblem.gif)

Both pages were green for the entire scroll!

That means that there's a hardware layer created for them, and the pages are also invalidated as we scroll the ViewPager. I did update the pages as I scroll them, using a parallax effect on the background and gradually animating the items in the page. What I didn't do, is create a hardware layer for the ViewPager's pages. After reading the ViewPager's source code, I found that when the user starts scrolling, a hardware layer is created for both pages and removed after the scrolling stops.

While it makes complete sense to create hardware layers to the pages as we scroll, it was bad for me. Usually, these pages aren't changing as we scroll the ViewPager, and since they can be pretty complex - hardware layers help rendering them much quicker. In the app I was working on, that wasn't the case and I had to remove these hardware layer, using a [small hack that I wrote](http://blog.udinic.com/2013/09/16/viewpager-and-hardware-acceleration). 

Hardware layers are not a silver bullet. It's important to understand how they work and use them properly, or you can find yourself with a bigger problem.

## DIY

In preparation for all the examples I showed here, I wrote a lot of code to simulate these situations. You can find everything in this [Github repository](https://github.com/Udinic/PerformanceDemo), and also on [Google Play](https://play.google.com/store/apps/details?id=com.udinic.perfdemo). I splitted different scenarios to different activities, and tried to document them as much as possible to help understand what kind of problems you can find using that Activity. Read the Activities' javadoc, open the tools and play with app.

## More info

As the Android OS evolves, so are the ways you can optimize your apps. New tools are being introduced with the Android SDK, and new features are added to the OS (such as the hardware layers). It's important to stay up-to-date and examine the trade-offs before choosing to change something.

There's a great YouTube playlist, called [Android Performance Patterns](https://www.youtube.com/playlist?list=PLOU2XLYxmsIKEOXh5TwZEv89aofHzNCiu), with many short videos from Google, explaining different subjects related to performance. You can find comparisons between different data structures (HashMap vs ArrayMap), Bitmaps optimizations and even how to optimize your network requests. I highly recommend watching all of them.

Join the [Android Performance Patterns Google+ community](https://plus.google.com/communities/116342551728637785407) and talk about performance with others, including Googlers, to share ideas, articles and questions.

More interesting links:

- Learn how the [Graphics Architecture in Android](http://source.android.com/devices/graphics/architecture.html) works. It has everything you need to know about how Android renders your UI, explaining the different system components, such as SurfaceFlinger, and how they all talk to each other. It's long, but worth it.

- A [talk from Google IO 2012](https://www.youtube.com/watch?v=Q8m9sHdyXnE), showing how the drawing model works and how/why we get a jank when our UI renders.

- An [Android Performance Workshop talk](https://www.parleys.com/tutorial/part-2-android-performance-workshop), from Devoxx 2013, showing some of the optimizations that were made in Android 4.4 to the drawing model, and demoing the different tools to optimize performance (Systrace, Overdraw etc.).

- Great post about [Preventative Optimizations](https://medium.com/google-developers/the-truth-about-preventative-optimizations-ccebadfd3eb5), and why they are different than Premature Optimizations. Many developers don't optimize parts of their code, because they think the impact is not noticeable. One thing to keep in mind, is that everything add up to become a [bigger problem](https://plus.google.com/105051985738280261832/posts/YDykw2hstUu). If you have an opportunity to optimize a small part, that may seem negligible, don't rule it out.

- [Memory managment in Android](https://www.youtube.com/watch?v=_CruQY55HOk) - an old video from Google IO 2011, that's still relevant. It showcases how Android manages the memory of our apps, and how to use tools such as Eclipse MAT to find problems.

- [Case study](http://www.curious-creature.com/docs/android-performance-case-study-1.html) done by Google Engineer, Romain Guy, to optimize a popular twitter client. In this case study, Romain shows how he found performance issues in the app and what he recommends on doing to fix them. There's a [follow-up post](http://www.curious-creature.com/2015/03/25/android-performance-case-study-follow-up/), showing other issues for the same app, after it was redesigned.

I hope you now have enough information, and more confidence, to start optimizing your apps today!

Start a trace, or enable some of the relevant Developer Options, and just go from there. You're welcome to share some of the things you've found in the comments or in the Android Performance Patterns Google+ community.
