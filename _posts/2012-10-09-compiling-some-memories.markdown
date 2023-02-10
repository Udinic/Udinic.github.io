---
author: udinic
comments: true
date: 2012-10-09 17:15:18+00:00
layout: post
slug: compiling-some-memories
title: Compiling some memories
wordpress_id: 331
categories:
- C++
- FAT32
- Java
- off-topic
tags:
- C++
- chat
- context menu
- FAT32
- Fat32 Sorter
- favorites
- file extension
- FlashLight
- j2me
- java
- midlet
- midp
- nokia
- nostalgy
- organize
- rename
- RenameExt
- symbian
- windows
---

This week I decided to [publish the code](https://github.com/Udinic/FAT32-Sorter) from an old project I did couple of years ago, the **FAT32 Sorter**. I wrote a little about it in [here](http://udinic.wordpress.com/2011/10/24/fat32-sorter/), and a lot about it in [there](http://www.codeproject.com/Articles/95721/FAT-32-Sorter). As I went through the code, make final adjustments before releasing it to the wild, I saw other old projects of mine. Just by reading those folder names, my mind has started reciting the story behind each and one of them. 

Ah… memories...

When I started coding, around the age of 14, I wasn’t attracted to write small games, as my classmates were doing from time to time. I wanted to solve problems that I find myself get annoyed by, or to make stuff easier for me. So whenever I got frustrated by something, I immediately started thinking – “How can I create a better experience?”. 

And I did.

I decided to release another project to the open source community. Before I’ll introduce it –I’ll explain the story behind it: As a heavy Windows user, I tried to tweak it as much as I can, making the glove better fit my hand. I wanted to enable the “Hide extension for knows file types" property. This helps me rename the file name without messing with its extension. But, I also wanted to be able to rename the file’s extension easily (Yes! I did rename extensions quite often. How did you write .BAT files if not from a .TXT file?!). Since Windows’ file properties didn’t give an option to edit the extension, I created **RenameExt**, which adds a context menu option for that:

[![right_click](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/right_click_thumb1.png)](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/right_click1.png)

The new option raises a minimal dialog to do the operation I needed

[![ext_dlg](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/ext_dlg_thumb1.png)](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/ext_dlg1.png)

Of course I handled all the cases where you select multiple files, and not necessarily with the same extension. Since Windows Vista, when you rename a file, the file’s extension is excluded from the selection, so it’s easier to rename just the file name. I don’t use RenameExt anymore, but if you want to check out the code – [here it is](https://github.com/Udinic/RenameExt).

Another utility that I wrote, came due to a computer failure, where the only option was to format the hard-drive and start fresh. The problem – I had a big and organized Favorites folder, with all the websites I kept close. Back than, we didn’t have auto-sync to all the favorites, like those spoiled kids have now, so I had to back them up myself. Since I couldn’t get Windows to load, I did that using DOS. The problem: DOS does not support long file names, at least not the DOS I had at that time. This means all the favorites’ titles just got truncated to shit like “HOWSTU~1.url”, which ruins any chance to understand what it links to. The solution: **FavoriteOrganizer**.

[![screenshot](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/screenshot_thumb.png)](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/screenshot1.png)

This small utility scans all the .url files, go to the website they refer to, get the title from the HTML file and rename the .url file to that title. It identifies dead links and webhosting service pages, which probably means that this site is no longer exist (Well, excluding some ACTUAL webhosting sites I kept there..). My favorites were saved, only to be completely deleted a few years after that.

There’s also a VB project that I wrote to solve a very specific problem I had with my brother – music. My brother and I have different taste in music, but some genres are common to both of us. To save space, we had one music folder with all the music, but each have his own Winamp playlist with all the songs he listens to. It’s not like today, where I have over 10,000 songs, we had maybe a 100, each and one of them were handpicked. You can’t afford downloading full discographies when you download in a rate of 7 KB/s at best. Downloading new songs and moving them to the common music directory caused a lot of problems, since we forgot to let each other know about new stuff that the other party might find interesting in, causing duplicates and stupid arguments. That’s why I wrote the **SongsOrganizer**:

[![songsorgnizer](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/songsorgnizer_thumb.png)](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/songsorgnizer.png)

It’s in Hebrew, but I’ll explain: The screen on the left side presents all the newly downloaded songs that are still in the download folder, waiting to join their friends in the music folder. Currently – there are no songs to transfer. Each user can see the song’s details and even play it. If the current user wants it, he presses the button that says “Transfer song” which causes this chain of events: Moving the song to the music folder, add the song to the current user’s playlist and save this song’s information in the internal DB. When the other user logs in, He can see all the songs the other user has moved, and click on the “I want too” button, which add this songs to his playlist too.   
Neat huh? This really made our lives much easier, at least until we each got a computer of his own :).

After a while, I got attracted to mobile development. J2ME was the first platform I learned, followed by Symbian and of course – Android. Using the mobile phone, I could solve a different kind of problems I encountered. The first JAVA supporting device I had was the Nokia 3510i

![http://www.gsmarena.com/nokia_3510i-344.php](http://cdn2.gsmarena.com/vv/bigpic/no3510i.gif)

It was really cool, it even had colored screen! The first problem I solved with it, was reading the menu at dark bars. We all used the light from our mobile devices, but it kept turning off, and was not bright enough. So I thought – why not create an app to light the screen all the time with white color? And that’s how **FlashLight** was born. Couple of months after I released it to the internet, I saw other apps do the same, and now we have dozens of those, but back then – I was the first one. No app stores were exist, so I can’t really know that for sure, but I did my research before that.

I also researched for an idea to create an app to keep all my passwords on my mobile device, and be able to sync them with my computer. I wrote the stuff I need to investigate on this To-Do list I created:

[![image](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image_thumb.png)](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image.png)

Even back then I was into To-Do lists :) Eventually I didn’t complete this project due to technological difficulties. Today I’m a happy user of KeePass on my PC and Android, and I sync the data using Dropbox. Much easier!

Last but not least, is my University project I did with a close friend of mine. We wrote a chat app called **ChatWithME****,** allowing PC and mobile clients to chat together. We wanted to make something cooler than just a regular server-client chat, so besides writing a beautiful J2ME chat client a person can write in a week, we give the mobile users the option to chat using a server connection or Bluetooth, which saves precious KBs if we want to speak with our classmates during a lecture.

[![image](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image_thumb1.png)](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image1.png) [![image](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image_thumb2.png)](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image2.png)

We got an A+ for that project.  
I also found a few small Symbian apps that I wrote, C# projects and more, but they are less interesting than those are. 

Hope you had fun like I did, taking this trip down memory lane. My only takeaway here - make the time to work on the things you love.