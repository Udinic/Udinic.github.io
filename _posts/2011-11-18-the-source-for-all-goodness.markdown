---
author: udinic
comments: true
date: 2011-11-18 17:57:53+00:00
layout: post
slug: the-source-for-all-goodness
title: The source of all Goodness
wordpress_id: 144
categories:
- Android
- AOSP
tags:
- Android
- Any.DO
- AOSP
- source-code
---

We have 2 reasons to celebrate this week!

The first one - Any.DO, my baby, the Android app I've been working on for the past 9 months, has launched with great success! Over 50K users in a week, rating of 4.5, numerous articles on [famous websites](http://techcrunch.com/2011/11/10/any-do-launches-a-social-to-do-list-app-with-1-million-in-funding/) and awesome feedback! If you have an Android device - come and join the party! [Market Link](https://market.android.com/details?id=com.anydo) or [Any.DO website](http://any.do). Give us a feedback!

The second reason - The source code for Android 4.0, IceCreamSandwich (or ICS), was released! After a long wait for the new Android versions' source code, we finally get it. As you might recall, Google refused to release the Honeycomb's source code, due to that fact it's designed for large screen devices. As far as I know, they didn't want people to take the code and build a modified version for small screen devices, as mobile phones, hurting their brand.

The source code is very useful for debugging purposes. You can access the lower parts of the Stack-Trace, and get a better understanding of the situation. However, downloading the source-code under Windows is not trivial, as all the scripts to do that are designed to work under Linux or Mac OS. I've tried modifying the scripts to work under windows using Cygwin. I've managed to get pass a few obstacles on the way, but after that I just couldn't make it work. I decided to try a different approach.

Since I don't have Linux installed on my computer, I've downloaded [Ubuntu 11](http://www.ubuntu.com/download/ubuntu/download) and used it as a live CD. That way, I don't need to install new operation system solely for the purpose of downloading source files. Since the amount of space needed is a few GBs, I decided to mount my NTFS drive, and download everything there. In order to do that, I've opened the Terminal, and unleashed some bash commands. First, we'll get the device name for the partition we want to mount. Its name should be "sdaX", where X is some number. In order to retrieve the correct name, you need to run:

{% highlight bash %}

sudo fdisk -l

{% endhighlight %}

and you'll get a list like that:

{% highlight bash %}

Device Boot      Start         End      Blocks   Id  System
/dev/sda1              63       80324       40131   de  Dell Utility
/dev/sda2   *       81920    23670783    11794432    7  HPFS/NTFS/exFAT
/dev/sda3        23670784   238710783   107520000    7  HPFS/NTFS/exFAT
/dev/sda4       238710784   625139711   193214464    7  HPFS/NTFS/exFAT

{% endhighlight %}

These are all the partitions on the system. Don't know which one is drive D on your Windows? Look at the number of blocks and a little trial and error to get the right one.

Now, armed with the device name, we'll start the magic:

{% highlight bash %}
# Entering god-mode
sudo su

mkdir /media/udinic

# Mounting the partition to be accessed from /media/udinic
mount /dev/sdaX /media/udinic -o exec
{% endhighlight %}

Note that I've used the "exec" option for the mount command. This is because one of the downloaded scripts will be executed as part of the process, and Ubuntu by default doesn't mount devices with the option to execute files (That is due to lots of complaints by new Linux users, started with Ubuntu, who got into a mess by that).

Now, go to /media/udinic, which directs you to your windows partition, and create your own folder for this process. From here you can just follow the [instructions](http://source.android.com/source/downloading.html) on the Android site. You'll need to use the "apt-get" command to install a few missing components, but you'll be given the exact command for that when needed. You can download a specific branch, as the Gingerbread branch instead of the ICS branch. After some downloading time (6 GB of download), you'll have a nice folder with all the source code for that Android version. You'll also get all the tools for building your own Android version.

Most devices run a custom ROM, which is a modified version of Android. Popular ROMs as [CyanogenMod](http://www.cyanogenmod.com/), publish their source code, and gives you the ability to download it. If the device you're working on is running CM, you'll probably want that more than the
original Android source code. The instructions for downloading the CM source code are [here](http://wiki.cyanogenmod.com/wiki/Building_from_source), and they are similar to the Google's instructions.

After the download is complete, you can restart your computer back into Windows, and attach to source (positioned in the partition you've selected, in the folder you've created, using Ubuntu) to your favorite IDE. Happy debugging for us all!
