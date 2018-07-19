---
title: How to Clear Your Browser's Cache
layout: post
permalink: /how-to-clear-your-browsers-cache/
tags:
  - browser
  - chrome
  - internet explorer
  - firefox
  - opera
excerpt_separator: <!--more-->
comments: false
---

It often occurs that I make changes to an existing website for a client, then send the client a mail informing them of what I have done.

However, it is not an infrequent occurrence that I then receive a mail back from the (sometimes disgruntled) client, saying that they cannot see any changes.

Hmmm… I know I made the changes and I'm viewing them on the live site, so why can't the client see the same thing?

The answer is because of browser caching.

<!--more-->

## Browser Caching 101

Each time you access a file through your web browser (if you don't know what a web browser is, read [this](http://googleblog.blogspot.de/2009/10/what-is-browser.html "Google Blog: What is a browser?")), the browser caches (i.e. stores) it in a temporary internet files folder on your hard drive.

The reason that it does this, is so that the next time you open the same page, it doesn't have to retrieve the file from a remote web server (the computer that hosts the web site), only from the local hard drive.

This results in a big increase in speed for you, the end user.

Browsers cache all kinds of files, ranging from HTML to CSS to images and this is normally a good thing. However, in some situations (such as the one described above), it can lead to problems.

## So, How Do I View Changes to a Web Page When My Browser Has It Cached?

Well, the simple answer is to reload the web page in question.

You can normally do this by pressing the <kbd>F5</kbd> button on your keyboard, or by right clicking on the page and selecting reload from the context menu, as is shown in the screen shot (click image to enlarge).

![Chrome's page reload dialogue](https://res.cloudinary.com/hibbard/image/upload/v1529427402/context_menu_reload.png "Chrome's page reload dialogue")

If this doesn't work for you, try holding down the <kbd>Control key</kbd> (bottom left hand corner of your keyboard) and pressing <kbd>F5<kbd>. This will force a cache refresh.

For those of you that want a little more technical information on the subject, check out [this discussion](http://stackoverflow.com/questions/385367/what-requests-do-browsers-f5-and-ctrl-f5-refreshes-generate "What requests do browsers' F5 and Ctrl + F5 refreshes generate?") on StackOverflow which goes into detail about what happens behind the scenes.

## Clearing Your Cache Completely

A slightly more draconian (but nonetheless equally as effective) solution is to clear your browser's cache completely.

**Be warned**: doing this may also result in your browsing history, download history, saved form data and cookies being erased (depending on the options you select).

You also need to know which browser you are running, as the instructions vary slightly from browser to browser.

If you don't know which browser you are using, visit [http://whatbrowser.org/](http://whatbrowser.org/ "What browser am I using?")

## Chrome

1. In the browser bar, enter: `chrome://settings/clearBrowserData`
2. Select the items you want to clear by checking the appropriate boxes
3. Select the period of time for which you want to clear cached information from the drop-down menu
4. Click _Clear browsing data_

## Firefox

1. From the _Tools_ or _History_ menu, select _Clear Recent History_
2. From the _Time range to clear_ drop-down menu, select the desired range
3. Click the down arrow next to _Details_ to choose which elements of the history to clear.
4. Click _Clear Now_

## Internet Explorer

1. Select _Tools_ > _Safety_ > _Delete browsing history..._
2. Deselect _Preserve Favorites website data_, then select any of _Temporary Internet files_, _Cookies_, and _History_
3. Click _Delete_

## Opera

1. From the _Opera_ menu, select _Settings_, and then _Delete Private Data..._
2. In the dialog box that opens, select the items you want to clear
3. Click _Delete_.

## And What If My Browser Isn't Listed Above?

If you don't see instructions above for your specific browser, then search your browser's Help menu for &#8220;clear cache&#8221;.

Failing that, ask Google. A typical query could be:

`How to clear cache in <browser name> <version number>`

It is also worth noting that you can do all of this and a lot more with the mighty [CCleaner](http://www.piriform.com/ccleaner "CCleaner - Optimization and Cleaning").
