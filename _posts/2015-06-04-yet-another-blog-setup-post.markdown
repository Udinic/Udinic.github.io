---
author: udinic
comments: true
date: 2015-06-04 13:30:10+00:00
layout: post
slug: yet-another-blog-setup-post
title: Yet another blog setup post
categories:
- General
tags:
- Jekyll
- Wordpress
- wordpress.com
- WYSIWYG
- Import Export
- CSS
- Octopress
- Syntax Highlighting
- Bootstrap
- Heroku
- Windows Live Writer
- disqus
- static web site
- Ruby
- git push
- Chrome DevTools
- Exitwp
- CNAME
- domain
- Github pages
- BitBucket
---


After several years of using Wordpress.com for my blog - I decided I need a change.

I wanted to move to something different for a while, but kept postponing it because that would require some time and effort to do. In this post, I'll summarize my transition experience and elaborate on problems I had and how I solved them. Hopefully, this information will help others with similar ambitions. 

<!--break-->

## First thing first - why leaving?

Wordpress.com is a good solution if you want a hassle free blog. You get a nice WYSIWYG editor, free hosting and variety of themes. However, after using it for a while, I came across many problems that got me to hate it and get annoyed by. These are my main reasons:

- __Editor__. I'm a power user. While WYSIWYG editor is great for many people, I prefer to have more control on the way my posts are formatted. Going back and forth between the HTML view and the Visual view, caused some html tags to disappear for some reason, especially code snippets. In 2 other occasions, my __entire post disappeared__! I was able to recover an old version for one of them, but the second one I had to rewrite (and if you've seen some of my posts, you'd know it's a lot of work). For a while, I used [Windows Live Writer](http://windows.microsoft.com/en-us/windows-live/writer-basics) for Windows, which is..OK, but it's far from perfect. I couldn't find a good editor for Mac.

- __Syntax Highlighting__. I use a lot of code snippets in my posts. I used a wordpress plugin for syntax highlighting, but I never really liked it. There's no option to theme it and it gave me hard time when posting code with the "<"/">" characters. I couldn't find any convenient alternatives.  

- __Design__. When I first created my blog, I spent a good hour to browse the themes wordpress has to offer. I liked a few, but I wasn't free to customize them (at least without paying for it). 

I had a few other reasons, such as using Google Analytics for stats and [Disqus](https://disqus.com/) for comments, but these are minor in compare to the others.


## Where should I go now?

It is decided!

I'm leaving Wordpress, but where? After doing some research, I wanted to try Octopress.

I was told Octopress is the "blogging framework for hackers", which sounds like something I want. I installed it and played around with it. The main problem I had with Octopress is the customization. My web design experience is not great, and Octopress uses Sass stylesheets, instead of the traditional CSS. While Sass is more powerful than using CSS directly, my customizations are relatively simple and I didn't want to invest time learning Sass at this time. Plus, I wanted to gain more experience with CSS, and designing my own blog is a great opportunity for that. I decided to pass on Octopress and try its base framework - Jekyll.

Since Octopress was created from Jekyll, they are very similar. Here are some of their common characteristics:

- They both generate a static website, instead of generating content on the fly. Static websites have better performance and easier to host (no need to maintain a database). Generating the static website is done by a single Ruby command.

- The posts are written in Markdown, a more convenient way to write posts than pure html.

- Start a local server to show a preview of the site. It will auto-generate pages as they are edited.

- Since everything is stored in plain files, Git can be used for reliable versioning.

I chose Jekyll because I can have all these features and also use CSS to design it. Now I can start a local server and work offline, whether to write a post or to design it. In fact, I'm writing these lines [from a plane!](https://plus.google.com/+UdiCohen/posts/VGJGiGYgY8U).

I need to design how my new blog is going to look like, and since my CSS skills are relatively low, I decided to find a theme that I like and modify it.  


## Design

After doing some field work to find interesting themes, which wasn't as easy as I assumed it'll be, I found the [dbyll](https://github.com/dbtek/dbyll) theme. It has the structure that I was looking for, and it looked nice, so I decided to use it, but not before I'll change a few things. 

[![img](/assets/images/{{ page.slug }}/css.gif)](/assets/images/{{ page.slug }}/css.gif)

Ah...CSS.

The theme I modified uses [Bootstrap](http://getbootstrap.com/), and it overrides Bootstrap's defined styles in its own CSS file. I made my changes on that file, and added some styles of my own to override other Bootstrap styles. I'm not a web developer/designer, but I didn't run into any problem that Google and the [Chrome DevTools](https://developer.chrome.com/devtools) couldn't help me solve. For developers who don't have much experience with these great tools, here are some tips that I found useful while working with:

- There are a few ways for a property to get overridden, and it's not always clear where the final value is coming from. Using the "Computed" tab, you can see all the final properties' values and their origin. Example:
<!-- [![img](/assets/images/{{ page.slug }}/devtools-computed.png)]() -->

- Using Cmd+Shift+C (or Ctrl+Shift+C if !MacOS) you can select a section in your page, and see in the "Styles" tab all the styles that are used for this specific section. You can edit and add new styles, just to see how things will look like, without updating any CSS files. Just don't forget to copy your changes to the CSS file afterwards.
	
- Using the "Device mode" button, you can change the layout to see how the page will render in other devices, such as Tablets or Phones. This is a great way to test your design's responsiveness for different screen sizes.

Intuition plays a large rule in design. The color palette, the button's design and other design elements, were things I just thought will fit together (with a __ton__ of trial and error). I asked some friends, some are designers and some are just people with good taste, to give me their opinions and I got a great feedback to take into account. The result is what you see in front of you. I still have some tweaks I want to do, but that's for later..

## Migrating the old posts from Wordpress

All my old posts are hosted in Wordpress.com, and I plan to leave no post behind. The migration process was a little bumpy, and required some manual work, but it didn't take too long.

### Import/Export

Luckily, Wordpress allows exporting the site to an XML file, which then can be converted to whatever we want. I researched a few scripts, some converts the content to pure html, some to markdown. I wanted to use only markdown in my posts, so I decided to go with that approach. 

I found a few automated solutions, such as the one described on this [elaborate post](http://vitobotta.com/how-to-migrate-from-wordpress-to-jekyll/index.html#converting-to-markdown), to convert my posts to markdown. After trying a few, I picked [Exitwp](https://github.com/thomasf/exitwp), which stated that the images will also be converted. In practice, only a few images were downloaded and I had to manually download the others. For every post, a folder was created in the "assets" folder to hold all the relevant images. The conversion script is also having some [escaping issues](https://github.com/thomasf/exitwp/issues/42), that got me running a find&replace on all my posts to fix the image links. I don't have tons of images on my posts, so it didn't require a lot of manual work. Other than that, I had to fix a few formatting issues after the conversion. I manually went over all of my posts, some had misplaced line breakers or italic/bold styles applied where shouldn't, not a big deal but required some time to inspect and fix.

My new blog uses Disqus to manage the comments, where in the old blog I used the built-in comments system. Fortunately, there's an export/import option when moving from Wordpress, using the same XML we exported earlier, and importing it into your Disqus account. I had to edit the wordpress exported xml file, since all the comments are linked to their respective posts by the url. I wanted to use my domain's url, so I needed to do a find&replace here too, to change all the URLs from _udinic.wordpress.com_ to _blog.udinic.com_. After importing the modified xml to my Disqus account - I had all my comments restored.

### Mastering my own domain

Speaking of domain issues, I also had a domain problem that almost aborted my entire migration process, but then I discovered there's a very simple solution to it.

When I set up my wordpress blog, I didn't set up my domain mapping properly. I only redirected my domain to go to the main page, but didn't set up a CNAME record to map internal links, such as posts. I was lazy and didn't think ahead (which is really not typical of me). As a result, all the links to my posts are in the format of _http://udinic.wordpress.com/year/month/day/post-title.html_, and these are the links in many Stackoverflow answers and other blogs that reference my posts. I didn't want to lose these links since, obviously, they bring a lot of traffic to my blog. I was mad at myself at first for not thinking about it when I initially set up my blog, but then I found out that wordpress provides a [solution](https://en.support.wordpress.com/site-redirect/) to this exact problem! I need to pay a yearly subscription fee for this service, of course, but there's always a price you pay when you don't think ahead (remember kids..).

Phew..that process was a bit longer than I wished, but thanks to all the scripts I used, written by great people with similar needs, this process was less excruciating. Now that I had all my old posts imported with a fresh design - I can go ahead and publish this thing.

But where?


## Publish

Jekyll can easily be run on your local environment, and thus can run on any machine with the right dependencies installed. One option is to get a server and set everything myself, or to just use a 3rd party service.

My first option was [GitHub Pages](https://pages.github.com/). It's easy, free and you can set up your own domain. The problem with that - your blog's content is a repository as any other under your account. If you want it to be private - you must pay Github a monthly fee. Since I wanted privacy, but wanted to minimize my blog's cost, I looked for other options.

I found [Heroku](https://www.heroku.com/) to be a good solution for my needs. There's a free tier, it's reliable and it's easy to use. Deploying new changes is done using a simple "git push" command. It has some limitations, such as shutting down if it hasn't been used for 30 mins. It comes up relatively quick after that, so it's not a big deal for me now. I really wanted to mess around with Heroku for some time now, and this was a great excuse to try it. I think that, at some point, I'll want to increase the blog's uptime and will find a solution for that. 

I also noticed that BitBucket can support hosting Jekyll, and it also provides free private repositories. The instructions I found are a little cumbersome in compare to the others, so I decided to pass on that option for now.

## Finally

I have a new blog, with a new design. Now for the real challenge - writing more.



