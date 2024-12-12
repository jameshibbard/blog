---
title: "Quick-Tip: Show Modal Popup after Time Delay"
layout: post
permalink: /show-modal-popup-after-time-delay/
tags:
  - javascript
  - jquery
  - cookies
  - events
excerpt_separator: <!--more-->
canonical:
  url: 'https://www.sitepoint.com/show-modal-popup-after-time-delay/'
---

<p class="callout originally-posted-elsewhere">This article was <a href="https://www.sitepoint.com/quick-tip-persist-checkbox-checked-state-after-page-reload/">first published on SitePoint</a> and has been republished here with permission.</p>

In the following quick tip, I'm going to show you how to open a modal window on a web page after a short time deay. This might be useful to highlight a particular call to action, such as signing up for a newsletter or for getting likes on Facebook. Some sites also use this technique to display advertising.

<!--more-->

![An untimely popup between racket and shuttle cock](https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2016/09/1477382703popup.png)

But before continuing, take a second to ask yourself if this is something you really need to do. Whenever a site I'm browsing opens a modal without me having clicked on something, I almost always close it immediately and get annoyed that my attention was jerked away from whatever it was I was looking at. In my opinion such techniques can detract from the overall experience of a site and there are better ways to make visitors aware of your content.

## A Basic Implementation

Still reading? Ok, I guess you're set on doing this, so let's get to it. For the impatient, there is a [working demo](#demo) at the end of the article.

For this quick tip I'll use the [Colorbox plugin](http://www.jacklmoore.com/colorbox/) to display the modal. Colorbox relies on jQuery, so you'll have to add that to your page as well. Here's a template.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset=utf-8 />
    <title>Delayed modal demo</title>
    <link rel="stylesheet" href="https://cdn.rawgit.com/jackmoore/colorbox/master/example1/colorbox.css" />
  </head>
  <body>

    <script src="https://code.jquery.com/jquery-2.2.4.min.js"></script>
    <script src="https://cdn.rawgit.com/jackmoore/colorbox/master/jquery.colorbox-min.js"></script>
    <script>
      <!-- Code here -->
    </script>
  </body>
</html>
```

_Note that I'm using various CDNs to include the scripts here, but you could also install the dependencies using a package manager such as npm or bower._

### Displaying the Modal

Normally, we would assign Colorbox to an HTML element and pass in any settings as key/value pairs inside an object:

```js
$(selector).colorbox({
  key:value,
  key:value
});
```

However, we want to call colorbox directly (without assigning it to anything), so the syntax is slightly different:

```js
$.colorbox({
  key:value,
  key:value
});
```

Colorbox has a bunch of options (many related to displaying images) which allow you to customize the modal. In the following example I am specifying its dimensions, giving it a class name (which allows me to style it using CSS) and passing it a string of HTML to display. You can find a full list of options on the page linked to above.

```js
$.colorbox({
  html:"<h2>Call For a Free Quote</h2>",
  className: "cta",
  width: 350,
  height: 150
});
```

Then all that we need to do is use JavaScript's `setTimeout` function to call this code after the required period of time has elapsed. `setTimeout()` is a native JavaScript function, which calls a function or executes a code snippet after a specified delay in milliseconds. If you'd like to get up to speed with the ins and outs of `setTimeout()`, then you can read [this SitePoint tutorial](https://www.sitepoint.com/jquery-settimeout-function-examples/).


```js
setTimeout(function(){
  $.colorbox({
    html:"<h2>Call For a Free Quote</h2>",
    className: "cta",
    width: 350,
    height: 150
  });
}, 10000);
```

The popup will now open after your visitor has been browsing the site for ten seconds.

## Accessibility Concerns

There are a number of accessibility concerns surrounding modal windows, for example: can keyboard users interact with them? Is the markup semantic? Are they easy to dismiss? You can find a more thorough discussion of the subject here: [Making Modal Windows Better For Everyone](https://www.smashingmagazine.com/2014/09/making-modal-windows-better-for-everyone/).

Although, Colorbox comes with a lot of these features out of the box, there are still a couple of things we can improve.

###  Shifting Focus

When the modal opens, Colorbox shifts the focus to the window itself. This is good, but if we have any interactive elements in the modal (e.g. an `<input>` element) we could consider setting the focus to this instead. This will mean one less mouse click for mouse users and less button presses for those people using a keyboard. We can do this using JavaScript's [focus](https://developer.mozilla.org/en/docs/Web/API/HTMLElement/focus) method.

We'll also need to make use of the `onComplete` event that Colorbox fires, to ensure that our content has been loaded.


```js
$.colorbox({
  ...
  onComplete: function(){ $("#myInput").focus(); }
});
```

### Remembering where the user was previously

After the user has dismissed out popup, it is only polite to return them to where they were previously on the page. To do this, we need to keep track of the most recent element the user has interacted with and reset the focus to this element once the modal closes.

```js
var lastFocus;

setTimeout(function(){
  lastFocus = document.activeElement;

  $.colorbox({
    ...
    onClosed: function(){ lastFocus.focus(); }

  });
}, 2000);
```

## Displaying the Popup Once Every X Hours

For the sake of usability, it's not a good idea to have the modal open every single time the user visits your site. Instead consider showing it once every X hours, or once every X days.

One way to do this would be to set a cookie once the modal has been shown, which expires after an allotted time. You can then check for the cookie's presence on page load and act accordingly.

To do this, we will need a set of functions for handling cookies. I recommend [js-cookie](https://github.com/js-cookie/js-cookie) for this task.

Include it in your page after the Colobox library:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/js-cookie/2.1.3/js.cookie.min.js"></script>
```

By this point, it also makes sense to start moving the various pieces of functionality into their own functions.

```js
var lastFocus;

function onPopupOpen(){
  $("#myInput").focus();
}

function onPopupClose(){
  Cookies.set('colorboxShown', 'yes', { expires: 1 });
  lastFocus.focus();
}

function displayPopup(){
  $.colorbox({
    html:"<h2>Call For a Free Quote</h2>",
    className: "cta",
    width: 350,
    height: 250,
    onComplete: onPopupOpen,
    onClosed: onPopupClose
  });
}

setTimeout(function(){
  var popupShown = Cookies.get('colorboxShown');

  if(popupShown){
    console.log("Cookie found. No action necessary");
  } else {
    lastFocus = document.activeElement;
    displayPopup();
  }
}, 2000);

```

## Demo

And here's the whole thing working on CodePen. Run the embed and the popup will open three seconds later. It'll only show once every 24 hours, as it sets a cookie as demonstrated above. Saying that, I've added a _Clear Cookies_ button so that you can run the demo multiple times. You can rerun the embed by clicking on the _Rerun_ button in the bottom right corner.

<p data-height="500" data-theme-id="6441" data-slug-hash="gwWpQX" data-default-tab="result" data-user="SitePoint" data-embed-version="2" data-preview="true" class="codepen">See the Pen <a href="http://codepen.io/SitePoint/pen/gwWpQX/">gwWpQX</a> by SitePoint (<a href="http://codepen.io/SitePoint">@SitePoint</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

## Conclusion

In this quick tip I have demonstrated how to open a pop up once a user has been browsing your site for a specified time. I have also highlighted usability and accessibility concerns surrounding this approach.

If you have any remarks or questions, I'd love to hear them in the comments below.

If you have any questions concerning the code, or are getting stuck implementing this on your own site, I would recommend that you post a question in the [JavaScript category](https://www.sitepoint.com/community/c/javascript) of SitePoint's forums.
