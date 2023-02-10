---
author: udinic
comments: true
date: 2011-10-04 00:07:56+00:00
layout: post
slug: package-name-changer
title: Package name changer
wordpress_id: 101
categories:
- Java
tags:
- java
- package name
- python
---

Sorry for the long wait, I was on a small vacation lately with with my love. But don't you worry! All those nice views got me thinking about some other great posts I can write here. And yes, "my love" = "Girlfriend", and not "Galaxy S".


The post today will be about something not directly related to Android code, but to a very useful utility I wrote recently. Whether you develop at home, or for some people in suits, you don't always have enough phones to test your apps. I think that the best way to find bugs - is to actually USE IT. If you're using the same phone to develop and as your primary phone, you'll need to install and uninstall the app lots of times, making it hard to use it for personal purposes (with all the data in it been trashed or deleted). You can try and use that awful emulator that comes with the SDK, or the cool [Android x86 VM](http://www.android-x86.org/), but working on the real thing is always the best way to see performance/convenience issues. Unfortunately, The Android system won't allow 2 apps with the same package name. I tried to find some automated way to change the package name for a whole project - but got nothing. So as a good engineer - I just wrote one!




<!-- more -->




Changing the package name is not as simple as it may seem. It's not just ordinary find&replace. There're lots of places where the package name is written in a different way than we used to (e.g. "com.blabla" can be found as "com_blabla"). After some trial and error, I got all the places where the package name needed be to changed (at least for my project, which is pretty big).


The script I wrote is in Python, and the only stuff you need to change is the package names you want to switch between, the base folder for your project and the app name that you'll probably want to change as well (how else will you know how to distinguish between those versions?). If you have a big project, with sub modules, no problem! Just call the handleModule for each one.

{% highlight python %}
def handleModule(moduleBaseFolder, codeFolder)
{% endhighlight %}

This will rename the appropriate code folders with the new package name. For example:

{% highlight python %}
handleModule(basePath + "\server-side\\", "src\\main\\java\\com\\")
handleModule(basePath + "\android-client\\", "src\\com\\")
{% endhighlight %}

After that, just run the script with the desired package name as an argument, refresh your project on the IDE to recognize the changed files' tree, and rebuild the app, that's it! To revert back, just pass the other package name as an argument to the script. This can be also used for ordinary java project, though I haven't tested it for that purpose.

The script can be improved to take any package name as an argument, or to identify all the sub modules on its own. Feel free to fork my git project and add more functionality. Also, corrections will be welcomed as well.

You can find the full code in here:


[https://github.com/Udinic/SmallExamples/tree/master/PackageNameChanger](https://github.com/Udinic/SmallExamples/tree/master/PackageNameChanger)
