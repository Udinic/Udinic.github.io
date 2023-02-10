---
author: udinic
comments: true
date: 2012-07-21 16:29:00+00:00
layout: post
slug: upgrade-your-galaxy-nexus-to-jellybean
title: Upgrade your Galaxy Nexus to JellyBean
wordpress_id: 265
categories:
- Android
tags:
- Android
- Galaxy nexus
- google io
- imm76i
- jelly-bean
- Jellybean
- ota
- upgrade
---

After seen the new features added to Android 4.1 (A.K.A. Jelly-bean), I immediately started seeking a way to get it into my phone. Waiting for the OTA (Over the Air) update is not acceptable! 

![](http://hothardware.com/newsimages/Item21886/Jelly_Bean.jpg)

I didn’t participate the Google I/O convention, which means I didn’t get the new Nexus 7 nor a Galaxy Nexus Device with the JellyBean already preloaded. But, I do have my own Galaxy Nexus with Android 4.0.4, and since I have a Nexus device – it should be fairly easy to flash it in. 

It wasn’t that far from the truth.

After reading [some information](http://www.randomphantasmagoria.com/firmware/galaxy-nexus/i9250/) about the different firmware the Galaxy Nexus has, I know that there are 2 main variant types: “**takju**” and “**yakju**”, where the first one is found on models bought in the USA, and the other is international. The first JellyBean build that I’ve found is **JRO03C**, but it’s not a full image, but an update to a specific build of Android 4.0.4. That required build was IMM76I and I had IMM76K. Problem.

On a side note: My phone just recently got the Android 4.0.4 OTA, even though this update is out for months now, it appears to be because I have a sub-variant of “yakju”. Since Google manage and support only the “takju” and “yakju” variants, all the sub-variants get updated later. it explains why I just now got my 4.0.4 OTA. Meaning, If you can’t wait for OTAs – get one of those variants on your phone, and you’ll be one of the first to get them! The needed IMM76I build is a “yakju” variant.

So, back to my problem. I needed to have the IMM76I build instead of my IMM76K, in order to apply the JellyBean updated. That image can be found on the [Google’s Factory Images for Nexus Devices](https://developers.google.com/android/nexus/images) site, or in [goo.im](http://goo.im/stock/maguro/stock), which also have a massive collection of ROMs for all phones/tablets. After downloading it – I have to flash it to my device. The easiest way I found was to use [Galaxy Nexus Toolkit](http://forum.xda-developers.com/showthread.php?t=1614827), which made my life much easier, even just for rooting purposes. There’s even an option to download the images straight from there, but I haven’t tried it. Just follow the instructions on the Toolkit and on that XDA post to get it done. WARNING: All your data will be deleted on the process, make sure you back up. Also, make sure you have the appropriate device, my device is a GSM, if you have CDMA/LTE, there are other images needed to be download.

After getting the “right” build for me, I only needed to update it with the new JellyBean OTA. After downloading the OTA (can be found in [goo.im](http://goo.im/stock/maguro/ota) or [here](http://www.randomphantasmagoria.com/firmware/galaxy-nexus/i9250/)) I copied it to my phone, and run it from the Phone’s Recovery. If you don’t have a recovery installed – you can use the Galaxy Nexus Toolkit I presented above. I installed the CWM touch recovery, and it’s very convenient in compare to the non-touch recovery that I used in the past.

[![image](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image_thumb.png)](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image.png)

Mission accomplished! 

I did a factory reset at the end of it, to get the full new-user-experience, and I got to say – It’s even better than the one on ICS.

Don’t forget to give some quick pressed on the “Android version” line on this “About phone” screen. This will give you a cute picture related to the current Android version. This is the one from the new JellyBean:

[![image](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image_thumb1.png)](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image1.png)

All that left to do now is to explore the new API, and find some cool stuff. My next post will be on one of those…
