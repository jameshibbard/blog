---
title: How to Detect a Touch Device Using JavaScript
layout: post
permalink: /how-to-detect-a-touch-device-using-javascript/
tags:
  - device detection
  - javascript
  - modernizr
excerpt_separator: <!--more-->
old-comments: how-to-detect-a-touch-device-using-javascript.html
comments: false
---

A while back I had to make a navigation menu which played a sound when you moused over the various menu points.

The problem was, that this feature didn't play nicely with touch devices, as they don't really have a concept of hover.

What happened instead was that the sound would play when the user selected one of the menu points, but would get cut off as the new page loaded – eurgh!

<!--more-->

So, I started looking for a way to detect a touch device and to disable this functionality accordingly.

I tried quite a few little hacks, from user agent sniffing:

```js
if(/Android|webOS|iPhone|iPad|iPod|BlackBerry/i.test(navigator.userAgent))
{
  // some code..
}
```

to creating and detecting a `TouchEvent`:

```js
function is_touch_device() {
  try {
    document.createEvent("TouchEvent");
    return true;
  } catch (e) {
    return false;
  }
}
```

While these kind of worked, they either felt kind of hacky or returned too many false positives.

I eventually settled on a nifty solution using  [modenizr](http://modernizr.com/ "Modernizr is a JavaScript library that detects HTML5 and CSS3 features in the user's browser."), a tiny feature detection library.

With this included in my page, I could then write:

```js
if (Modernizr.touch){
   // disable sound
} else {
   // attach sound to menu
}
```

I tested this on the iPad1, iPad2, iPhone3, iPhone4 and the latest version of Android and it worked just as expected.

### Resources

  * [What's the best way to detect a &#8216;touch screen' device using JavaScript?](http://stackoverflow.com/questions/4817029/whats-the-best-way-to-detect-a-touch-screen-device-using-javascript "StackOverflow")
  * [How to determine if the client is a touch device](http://stackoverflow.com/questions/6262584/how-to-determine-if-the-client-is-a-touch-device "StackOverflow")
