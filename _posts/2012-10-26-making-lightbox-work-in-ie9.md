---
title: Making Lightbox Work in IE9
layout: post
permalink: /making-lightbox-work-in-ie9/
tags:
  - browser
  - internet explorer
  - lightbox
comments: false
---

I'm a big fan of [Lokesh Dhakar's Lightbox script](http://lokeshdhakar.com/projects/lightbox2/ "Lightbox is a simple, unobtrusive script used to overlay images on top of the current page."), which is an elegant and easy to use method of displaying images against a darkened background on a web page. I've used this script quite heavily in the past, so I was somewhat alarmed when reviewing some of my old projects, to observe that it was broken in Internet Explorer 9.

A quick perusal of the changelog on the Lightbox site soon brought the problem to light. Version 2.05 of the script had been published with the message "Upgraded Prototype (now works in IE9) and Scriptaculous". I had a look at my code and sure enough, I was using version 2.04.

"Never mind", I thought "I'll just download the latest release and swap out the affected files. That should fix the problem". However, as you might already have guessed, this would have been too simple. Looking at the download section of the website I saw that the current version of the script was 2.51 and in version 2.5, Lokesh had switched from Prototype to jQuery. Although this in itself is a good thing, changing JavaScript libraries would mean revisiting every one of my sites which was broken in IE9, finding every page which used the Lightbox script and altering the code by hand.

So, it seemed that it was version 2.05 which I needed, yet this was nowhere to be found on the website. I did see however, that the Lightbox code now lived on GitHub at the following url: [https://github.com/lokesh/lightbox2](https://github.com/lokesh/lightbox2 "lokesh / lightbox2").

Finding myself in Windows, I then fired up GIT bash (see note at end of post) and typed:

```sh
git clone https://github.com/lokesh/lightbox2/
cd lightbox2
git checkout v2.05
```

This downloaded the Lightbox repo from GitHub into a folder on my desktop and checked out the version of the project I was after. I could then examine the relevant JavaScript files and soon concluded that the only difference between my broken projects and Lightbox v2.05 was the updated version of the Prototype library.

Armed with this new and dangerous knowledge, all that I then had to do was drop the more recent Prototype version into the JavaScript folder of any of my projects which had been affected by this, and the problem was solved. Yay!

---

**Edit**: It also occurred to me that GitHub  lets you download a complete repository as a zip file. Here's the link for the Lightbox project: [https://github.com/lokesh/lightbox2/zipball/master](https://github.com/lokesh/lightbox2/zipball/master "Lightbox 2 repository").

You can download this file, unzip it and look in the `releases` folder for a file named `lightbox2.05.zip`. This will also contain the files that you need.

---

**Note**: GIT Bash comes bundled with Msysgit. This is one the best packages for installing GIT for windows and can be found here: [http://msysgit.github.com/](http://msysgit.github.com/ "The home page of Git for Windows"). After installation it should be available under Start > Programs > Git > Git Bash.

Alternatively, for those who prefer GUIs, check out this tutorial: [http://net.tutsplus.com/tutorials/tools-and-tips/git-on-windows-for-newbs/](http://net.tutsplus.com/tutorials/tools-and-tips/git-on-windows-for-newbs/ "Nettuts+ - Git on Windows for Newbs")
