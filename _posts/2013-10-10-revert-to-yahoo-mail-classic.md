---
title: Revert to Yahoo! Mail Classic
layout: post
permalink: /revert-to-yahoo-mail-classic/
tags:
  - browser
  - email
  - internet explorer
excerpt_separator: <!--more-->
old-comments: revert-to-yahoo-mail-classic.html
comments: false
---

Today I got a phone call from a relative, who was rather annoyed that Yahoo! had upgraded their mail interface.

She complained that she didn't like the way that conversations were now arranged and that she was generally confused by the changes.

While I quite like the new look myself, I can appreciate that others might not, so I had a look at what can be done.

As it turns out, switching back to the old version isn't so difficult. Here's how to do it.

<!--more-->

## Spoofing Your User Agent String

First thing you will need is a browser extension to manipulate your user-agent sting (this is a text field in an HTTP request header that contains the name and version of the Web browser you are using).

Don't worry if this sounds complicated. It isn't!

My relative was using Chrome, so I headed to the Chrome Web store and discovered "[User-Agent Switcher for Chrome](https://chrome.google.com/webstore/detail/user-agent-switcher-for-c/djflhoibgkdhkhhcedjiklpkjnoahfmg "Quickly switch between user-agent strings on the fly")".

Here's a screen shot of this extension in the web store.

![Chrome Web Store - User Agent Switcher](https://res.cloudinary.com/hibbard/image/upload/v1529665469/user-agent-switcher-1.png "Chrome Web Store - User Agent Switcher")

To install the extension, click on the button in the top right-hand corner of the window that says _+ FREE_.

A dialogue will then open asking you to confirm the installation.

Click on "Add".

![Confirm extension installation](https://res.cloudinary.com/hibbard/image/upload/v1529665466/user-agent-switcher-2.png "Confirm extension installation")

After a short pause, you will see another dialogue telling you that the extension was installed successfully and you will see a little symbol (looking like a piece of paper with a Zorro mask on) in the top right of your browser window.

![The mask of Zorro](https://res.cloudinary.com/hibbard/image/upload/v1529665466/user-agent-switcher-3.png "The mask of Zorro")

## Time to Fool Yahoo!

Now, navigate to your Yahoo! Mail account and sign in.

When you are greeted with the new version, click on the User-Agent Switcher symbol in the top right of your browser.

From the resultant drop-down menu select "Internet Explorer".

![Select your user-agent](https://res.cloudinary.com/hibbard/image/upload/v1529665466/user-agent-switcher-4.png "Select your user-agent")

In the next step select "Internet Explorer 6".

![Select IE6](https://res.cloudinary.com/hibbard/image/upload/v1529665466/user-agent-switcher-5.png "Select IE6")

Yahoo! Mail will immediately throw a wobbly and confront you with the following screen, as it now thinks it is dealing with Internet Explorer 6, which it doesn't support.

![Oh Noes! Now we've broken the internet!](https://res.cloudinary.com/hibbard/image/upload/v1529665467/user-agent-switcher-6.jpg "Oh Noes! Now we've broken the internet!")

Click on _Continue without upgrading_ and you'll have your old Yahoo! Mail back!

You can now also see that you are browsing using the Internet Explorer 6 string, as the "Zorro" symbol will change.

![We're identifying ourselves as IE6](https://res.cloudinary.com/hibbard/image/upload/v1529665467/user-agent-switcher-7.png "We're identifying ourselves as IE6")

When you stop using Yahoo! Mail, set your user agent string back to that of

"Chrome" -> "Default".

I hope that helps someone.

If you have any questions, leave them in the comments.
