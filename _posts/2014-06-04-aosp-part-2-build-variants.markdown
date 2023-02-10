---
author: udinic
comments: true
date: 2014-06-04 16:40:43+00:00
layout: post
slug: aosp-part-2-build-variants
title: 'AOSP Part 2: Build variants'
wordpress_id: 609
categories:
- Android
- AOSP
tags:
- /vendor/asus/flo
- /vendor/broadcom/flo
- /vendor/qcom/flo
- adb reboot bootloader
- adb sideload
- Android
- Android ROM
- ANDROID_PRODUCT_OUT
- AOSP
- aosp_flo
- aosp_flo-eng
- boot-loader
- bootloader
- build number
- build variant
- build/target/product
- buildtype
- ccache
- ccache -C
- ccache -M
- ccache -s
- CCACHE_DIR
- ClockworkMod
- CM updater
- CM_updater
- custom recovery
- debug
- download mode
- embedded android
- emulator
- eng
- envsetup.sh
- fastboot
- fastboot flashall -w
- fastboot mode
- fastboot oem lock
- fastboot oem unlock
- full
- full_flo
- generic
- karim yaghmour
- LOCAL_MODULE_TAGS
- lunch
- lunch full_eng
- make
- make -j16
- make -j8
- make clean
- make clobber
- make otapackage
- nexus
- odin mode
- ota pacakge
- ota wifi mac
- prebuilts/misc/darwin-x86/ccache/ccache
- prebuilts/misc/linux-x86/ccache/ccache
- recovery
- recovery console
- RecoverySystem
- sdk
- TARGET_BUILD_VARIANT
- test-keys
- touch wiz
- user
- userdebug
- USE_CCACHE
- vendor
- version_defaults.mk
- watch -n1 -d prebuilts/misc/darwin-x86/ccache/ccache -s
---

On the [previous post](http://udinic.wordpress.com/2014/05/24/aosp-part-1-get-the-code-using-the-manifest-and-repo/), we got the AOSP code into our environment and learned how that process works. The _repo _tool and the Manifest project goes hand-in-hand, managing the 200+ git repositories, all part of the AOSP.

Now that we have all the code locally, we need to get all those lines of code into a nice package we can eventually install on a device. The nice guys in Google brought us some nice utilities to help with that. To initialize your environment with those new functions and environment variables, you need to run the command:

{% highlight bash %}

$ source <working dir>/build/envsetup.sh
{% endhighlight %}


Note: I’m using Mac OS with “[oh-my-zsh](http://ohmyz.sh/)” and “zsh" as my default shell. When running this command from a shell other than “bash”, you’ll get a warning that only bash is supported. From my experience, I can tell that I never ran into a problem. If you know otherwise - let me know, otherwise - I highly recommend this configuration regardless of AOSP work.

Now, we can get this party started..

<!--break-->


## Build targets


There are several devices we can build our new Android ROM for ([remember what ROM is](http://droidlessons.com/what-are-roms-for-android/)?), with different feature combinations and build types that we can use. In order to pick the desired configuration, we’ll use _lunch_. The _lunch_ command can take the configuration name as an argument, or picked by the user from a list. A configuration name is structured this way:


PRODUCT-BUILDTYPE


**PRODUCT** - This part describes the set of modules to be included, among other configurations. We can include only some of the packages in the system, or all of them, depending on the “flavor” we want our new ROM to be. A “package”, for this matter, can be an app, a system component, sounds, locales, fonts or even just a regular files that are copied to the final product. We can also set different system properties for a specific product. There are several predefined products we can use:


generic - default set of packages.




_full_ - a full set of packages, with most necessary apps and all locales




_aosp_ - Basically inherits everything from _full_.




sdk - packages needed to build the SDK. I'll elaborate about the SDK on a separate post.


There are also sub-products for specific uses. For instance, you can have _full_flo_ to build the full product for the Nexus 7 (codename “flo”), providing you have the necessary drivers (more on that later). To see what’s included in each product, check the .mk files under \<working dir\>/build/target/product and also under /device/\<vendor\>/\<device codename\>.

**BUILDTYPE** - Selects the modules to install and set some default settings or behaviors of the product. Each variant can accept only modules who set, in their Android.mk file, the LOCAL_MODULE_TAGS to at least one of the tags it approves (tag examples: _debug_, _samples_, _tests_ and more) . The valid build types are:


_eng_ - this variant will have some development tools and predefined settings that developers need (e.g. USB debugging enabled).




_user_ - this is the variant for a product that suppose to go out to the users. Will use only modules that the end-user will need and more restricted settings.




_userdebug_ - A combination of the _user_ and _eng_ variants. It's intended for users testing your environment, but not developing it.


You can see in more detail the different handling for these variants in the \<working dir\>/build/core/main.mk file, by following the use of the TARGET_BUILD_VARIANT env variable.

After selecting our preferred product and build variant, we can apply it using the _lunch_ command. For this demonstration, I want to run my new ROM on an emulator and have access to every advanced feature available. For that, I’ll use the “full-eng” configuration and I’ll apply it by running:

{% highlight bash %}

$ lunch full-eng
{% endhighlight %}





## Build flash-able images


The basic build artifacts are system images, one for each partition on the device. To build our images, we need to use the _make_ command. To build my ROM on my computer, I’ll use this command:

{% highlight bash %}

$ make -j16
{% endhighlight %}


The _-jN_ switch is defining the amount of threads to use when making the project. That’s a standard "GNU make” switch, not unique to AOSP builds. To get the maximum out of your machine, the recommended value for N is CPU_cores*2, so in a single quad-core CPU machine, you should probably use _N=8_, but you can try and play with the values to get the most out of your machine. My Macbook Pro has one quad-core CPU with Hyper-Threading, meaning it can run 2 threads simultaneously on each core, providing 8 “virtual” cores. I use mostly N=16 and sometimes N=8, depends if I’m still working on the laptop while building.

The build process is long, could take anywhere between 30mins to 3 hrs. We’ll talk more about time comparisons and performance boosting later on. If you need something to do while it’s building, I can recommend watching my new favorite TV show “Rick and Morty”, it’s a combination of “Familty Guy” and “Back to the Future” but also cleaver as “South Park". You can [stream it for free from their official website](http://video.adultswim.com/rick-and-morty/) and it works smoothly even if using _make -j16 _(tested it myself!)_._

After the process is complete, the emulator’s path is automatically added to your terminal's PATH, meaning we can start it by running:


{% highlight bash %}
$ emulator
{% endhighlight %}


To check that I’m looking at the image that I just built, I’ll go to the “About Phone” screen and check the "Build Number”.





![](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/wpid-screen-shot-2014-05-22-at-1-36-15-am23.png)


I can see the build configuration that I selected - “full-eng”, and my name as the user who built it. I can also see the date/time of the build and that I used the default test key_ _to sign all the modules (which, of course, needs to be changes before releasing). Check the _<working dir>/build/core/version_defaults.mk_ file to see how the build number is formatted.












Seeing it runs on an emulator is nice, seeing an actual device running our sweet new ROM - even better.







## Building for actual devices




Let’s start with an important fact - Every device needs to have its **own build** of our ROM.







You cannot take a build intended for Nexus 7 and flash it on a Galaxy Note 8, mainly because different devices need different drivers. So, the first step we need to do - importing the device’s drivers into our system.




### Drivers




Acquiring device drivers is tricky. Not all manufacturers are willing to give away their precious drivers. They don’t want brats like you creating your own OS for their hardware, beyond their control. They do this mainly for commercial/business reasons (how else can they put bloatware on your device?). That’s why you can’t simply build KitKat for, let’s say, Samsung Galaxy S3. All Google sponsored devices, called the "Nexus series", have their drivers [available for free](https://developers.google.com/android/nexus/drivers).







Every set of drivers binaries will probably have its own installation process. For the Nexus drivers, we get an extractable file that we need to run on the root of the working directory, and part of the process will be to accept the licensing agreement. After that - the _<working dir>/vendor_ folder will get a little heavier. Doing the above for the Nexus 7 Wifi 2013 model (Codename “flo”), will produce the following new folders:

\<working dir\>/vendor/asus/flo

\<working dir\>/vendor/broadcom/flo

\<working dir\>/vendor/qcom/flo


Inside these folders, we’ll find binary files and .mk files. It’s best to clean the build environment after extracting new drivers in the system. Use:






{% highlight bash %}

make clean
      or
make clobber # Both are basically the same
{% endhighlight %}







### Building




In order to create a build for our device, we need to initialize the build system with suitable build configurations. I mentioned earlier that we can use sub-products for the main build products, in example for a specific device. In this case, we can use the _aosp_flo_ sub-product with the _eng_ build type. We’ll run:






{% highlight bash %}

$ lunch aosp_flo-eng
{% endhighlight %}







Note: Since the _full_ and _aosp_ products are basically the same, we could've use _full_flo-eng_ and get the same results.




After that, we’ll start the build as usual, with a _make -jN _command:






{% highlight bash %}

$ make -j16
{% endhighlight %}







And now - we wait again. You can watch more “[Rick and Morty](video.adultswim.com/rick-and-morty/)", as suggested earlier.




### Go with the Flo




At the end of the build process, we’ll have a series of .img files that we flash to the device from the boot-loader. Before doing that - a few words about **how** it’s done:






</br>

#### Bootloader, Fastboot and Download Mode




So..who is this “boot” fella and why do we need to load it?







Well..you can see the _boot-loader_ as a small part in the system that’s in charge of loading basically everything. The bootloader is the initial sequence of code that executes the operation system. When flashing a new OS on the device, some changes needs to be done to the bootloader, which explains why it's initially in a locked state in most devices. This is intended to “protect” the device from flashing custom ROMs (mostly for commercial reasons). Unlocking the boot-loader is a process simple as a running a shell command in some devices, and using external software in other. When unlocking the bootloader, ALL USER DATA ON THE DEVICE IS DELETED as a security measure, consider that fact when you decide to do that.







**Fastboot/Bootloader mode** is a state in which your device can be flashed with new images. You can access it by a [key combination](http://www.droidviews.com/how-to-boot-android-devices-in-fastboot-download-bootloader-or-recovery-mode/) or using the command: _adb reboot bootloader_. Communicating with a device in this state is done using the _fastboot_ command that comes with the Android SDK, but can also be [downloaded](https://code.google.com/p/adb-fastboot-install/) in separate (the ADB command is also included). Not all devices have fastboot mode.







**Download mode** is a mode that exists on Samsung devices to replace the fastboot mode. Communicating with the device in this mode is done using a software called Odin, which explains the other name to this mode - “Odin mode”. Using Odin is out of this post’s scope, but the internet is full of Odin how-to’s.







To Unlock a Nexus device’s bootloader, you can follow [these instructions](http://source.android.com/source/building-devices.html#unlocking-the-bootloader). Other devices may require different methods, you can search and see the appropriate method for your device.

<br/>



#### Flash…A-ha!


![flashGordonAndroid](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/flashgordonandroid.jpg){: .center-image }









Get your device into Fastboot/Bootloader mode and flash all the created images using this command, to be run from the root of your working directory:






{% highlight bash %}

$ fastboot flashall -w
{% endhighlight %}







**flashall** - This _fastboot_ command flashes all the images that were created. It looks for them in the path stored in the $OUT environment variable, which is initialized by the _lunch_ command.







**-w** - resets the device's _/data _partition, with all the user’s date, so backup your stuff first. Not mandatory after the first time you flash this ROM.







After the new images got flashed into my Nexus 7, let’s take a look at the “About” screen:













![](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/wpid-screen23.png)





The "Build number" has our lunch target (aosp_flo-eng), my name, the signing keys and the date/time of the build. This screen is the best place to check if your flash process was successful.









Victorious!




## Creating an OTA Package


In contrary to image files, where the entire partition is replaced with the content of the image files, an OTA package performs an update to the system’s files. It’s smaller in size and can be incremental from the last build. You can check the _<working dir>/build/tools/releasetools/ota_from_target_files_ file, to see how the OTA package is actually created

To create an OTA package, we need to first pick a build configuration, using _lunch_ as before, and then use

{% highlight bash %}

$ make otapackage -j16
{% endhighlight %}


After the build finishes, the env variable $OUT or $ANDROID_PRODUCT_OUT will be initialized to the path where the product can be found, that’s where the OTA zip file will be. It should be around 200 MB in our case, less than the sum of all image files (> 400MB). Installing the OTA can be done using the Recovery system.

What is a _Recovery_ you ask?

Remember the bootloader I mentioned earlier? Well..it can boot our device into the Android OS, that we all know and love, **or** into another partition with a Recovery console. The latter has tools to help the user recover or repair a problem he’s having with the OS. The basic recovery on AOSP is very limited, it offers basic operation such as deleting user data, upgrading the firmware and is being controlled using the volume up/down and power keys. You can use a custom recovery such as the popular [ClockworkMod Recovery](https://www.clockworkmod.com/rommanager), which supports more functions and touch screen controls. The easiest way to flash a new custom recovery, is using the app [ROM Manager](https://play.google.com/store/apps/details?id=com.koushikdutta.rommanager), but there are other methods simple as a one _fastboot_ command. Different devices require different steps to do that, search the appropriate method for your device. You can read more about the Recovery [here](http://wiki.cyanogenmod.org/w/All_About_Recovery_Images).

In both Stock and Custom recoveries we can flash a new OTA [using the adb sideload method](http://android-revolution-hd.blogspot.com/2013/12/ow-to-use-adb-sideload.html). The stock recovery will check the package’s signature, a check you can bypass on a custom recovery. So, if you didn’t sign your OTA package correctly - you can use a custom recovery instead.

If the previous ROM on the device was different than the one you installing, it’s encouraged to do a factory reset and clear the cache, to clean up the place and avoid upgrading issues. This can be done from the Recovery console as well. After getting the device into “sideload mode”, we install the OTA package with this command:


{% highlight bash %}
$ adb sideload \<path to OTA package\>
{% endhighlight %}


The sideload command copies the OTA file to the device and install it. After the process completes - our new ROM is now installed. It is also possible to write code that does the updating for us, you can read about the [RecoverySystem class](http://developer.android.com/reference/android/os/RecoverySystem.html) and take an example from [Cyanogenmod’s updater](https://github.com/CyanogenMod/android_packages_apps_CMUpdater)

**An important note for Mac users**: After building an OTA package for my Nexus 7, I noticed that the WiFi isn’t working at all. After doing a bit of research, it seems like there’s a problem with the WiFi driver on OTA packages **if built on Mac**. Creating the OTA package on a Linux system resulted in a fully working WiFi. I found [another person with the same problem](http://grokbase.com/t/gg/android-building/128v3423db/unable-to-make-working-ota-zip) as I’m having, but for a Galaxy Nexus device, which means this problem may occur on other devices too.


## Build time and performance boost using CCache




Building Android ROM is a slow process. Many projects are involved and it requires a lot of computing power to crunch all that code together. Strong hardware is the best solution for a fast build - many CPU cores, SSD hard drive and a lot of RAM will increase the build time. Other than that, there is one optimization you can do that doesn’t involve opening your wallet - using CCache.







CCache stands for “Compiler Cache”. What it does is caching the compilation of C\C++ files, in favor of reusing them in future compilations of the same files. In most cases, C/C++ files aren’t changed between builds, since most of the changes we do are in Java files. This means that this cache could really help speed up the process.







How faster can it take us?







I’m using Macbook Pro 2013 with a quad-core CPU + Hyper Threading. A full build after cleanup takes about an **hour and a half.** Using a “warm” ccache, the process takes about **30 mins**. That’s **a third** of the original time! Your results may vary.







The ccache binary file is located at \<working dir\>/prebuilts/misc/darwin-x86/ccache/ccache (if you’re using linux, then s/darwin-x86/linux-x86). There are a few important switches to that command:







**-M \<size\>** - Initializes the max size of the cache. You can use values as 200M, 30G etc.
**-s** - See statistics of the cache. The amount of files, total size and more.
**-C** - Clears the cache







In order to set up your system to use ccache, you first need to initialize the cache's max size. The recommended size is between 50-100G. I can say that after cleaning the cache and running a full build I see that only 6G are used, but before I cleaned the cache it was about 45G (I played around a lot with different repos, so maybe that’s why). To initialize the max size, I ran the command only once, before the first time I ran a build:






{% highlight bash %}

$ \<working dir\>/prebuilts/misc/darwin-x86/ccache/ccache -M 50G
{% endhighlight %}







You also need to set the following env variable, showing the build system that you want to use ccache. I suggest putting this setting with the other global variables that you set in your environment:






{% highlight bash %}

$ export USE_CCACHE=1
{% endhighlight %}







The location of the ccache is by default _~/.ccache. _If you want to change this, you can set it using:






{% highlight bash %}

$ export CCACHE_DIR=<\path_of_your_choice\>/.ccache
{% endhighlight %}




If you want to watch the ccache being used as you build, you can run






{% highlight bash %}

$ watch -n1 -d prebuilts/misc/darwin-x86/ccache/ccache -s
{% endhighlight %}


I highly recommend using the ccache for your builds. It really is improving the build time and I never had any problems with using the cache. But then again - I don’t normally change C/C++ files in the AOSP. If you find yourself in a weird compilation problem, and you want to just clean everything up, use _make clean_ **and** also run this to clear the ccache:

{% highlight bash %}

$ \<working dir\>/prebuilts/misc/darwin-x86/ccache/ccache -C
{% endhighlight %}








## Conclusion







At the end of this part, we got our new ROM packaged and flashed into an actual device. Exciting times indeed.. but this is just the beginning. What we learned is just the overhead of creating an AOSP product, the real work is ahead of us. We still have a huge amount of repositories that we need to steer to our direction, and some roads are treacherous. In the last part of this series, we’ll discuss how to create new features on AOSP by maintaining changes on multiple repositories, easily deploying new changes and debugging your code. It’ll give you the jump-start you need before doing any any AOSP work.







## Further reading




[Embedded Android by Karim Yaghmour](http://amzn.to/1kJfajE) - This book explains the AOSP from the A to the P. The different components, the way the build system works, the many “Hacks” we can do there and some really great How-to’s (adding new device, adding new apps/libraries etc.). I don’t read technical books at all, but for this subject - it’s a life saver.
