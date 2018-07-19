---
title: Display UI-blocking Overlay on Page Load
layout: post
permalink: /display-ui-blocking-overlay-on-page-load/
tags:
  - javascript
  - jquery
  - jquery plugins
  - lightbox
  - local storage
excerpt_separator: <!--more-->
old-comments: display-ui-blocking-overlay-on-page-load.html
comments: false
---

A client asked me to add an announcement to their website, informing visitors that their business would be shut during the holidays.

"Uh, ok", I said, thinking that I could place an announcement in the sidebar, but the client wanted more. They wanted it visible, like really, really visible.

The solution we ended up with was to have the announcement displayed in an interface-blocking overlay when the site loaded. This would be shown to the user only once.

Although not overly user-friendly, some people might find this useful, so here's how I coded it.

<!--more-->

First off, let's start with [a demo of what we'll end up with](http://hibbard.eu/demos/ui-blocking-overlay/ "UI-blocking overlay on page load").

Still with me? Good.

Now the way this is going to work, is that we will create the announcement box on the page, then hide it using CSS:

```css
#announcement { display: none; }
```

```html
<h1>A visible heading</h1>
<p>Some visible content</p>

<div id="announcement">
  <p>An important announcement</p>
  <a id="close" href="#">Close</a>
</div>
```

Then, when the page loads, we can create an overlay and append it to the body:

```css
#overlay{
  position: fixed;
  top: 0%;
  left: 0%;
  width: 100%;
  height: 100%;
  background-color: black;
  opacity:.80;
  z-index:1001;
}
```

```js
$('<div>', { id : 'overlay' }).appendTo('body');
```

Be sure to give the overlay a relatively high `z-index` value, to ensure it sits on top of everything.

After that we'll need to show the announcement and position it correctly on the screen.

I'm using a slightly modified function, [picked off of Stack Overflow](http://stackoverflow.com/questions/210717/using-jquery-to-center-a-div-on-the-screen "Using jQuery to center a DIV on the screen") to center the box.

```css
#announcement{
  display: none;
  position: absolute;
  width: 250px;
  height: 120px;
  padding: 0 16px;
  border: 16px solid orange;
  background-color: white;
  z-index:1002;
}
```

```js
jQuery.fn.center = function () {
  var w = $(window);
  this.css("position","absolute");
  this.css("top", Math.max(0, (
    (w.height() - $(this).outerHeight()) / 2) + w.scrollTop()
  ) + "px");
  this.css("left", Math.max(0, (
    (w.width() - $(this).outerWidth()) / 2) + w.scrollLeft()
  ) + "px");
  return this;
}

$("#announcement").fadeIn('slow').center();
```

Notice that we give the announcement box a higher `z-index` value than that of the overlay.

Finally, we need to attach an event handler to "Close" link, so that when a user clicks on it, the announcement and the overlay are removed:

```js
$("#close").click(function(e){
  $("#announcement").remove();
  $("#overlay").remove();
  e.preventDefault();
});
```

## Showing it Only Once

Although we now have a functioning script, you will notice that the overlay displays whenever we reload the page. This can quickly become rather annoying.

We can safely assume that when the user has shut the overlay once, they have read and understood our message.

It would therefore be a good idea to add some functionality that displays the announcement only once. We can do this using [HTML5 storage](http://diveintohtml5.info/storage.html "Introducing HTML5 Storage").

```js
$("#close").click(function(e){
  localStorage.setItem("readAnnouncement", "true");
  ...
});
```

Then, when the page loads, we can check for the presence of this key/value pair and display the message accordingly:

```js
var readAnnouncement = localStorage.getItem("readAnnouncement");
if (!readAnnouncement){
  $('<div>', {id : 'overlay'}).appendTo('body');
  $("#announcement").fadeIn('slow').center();
}
```

And there you go, all done.

To be complete, here is the code in its entirety, as well as a link to the [finished demo](http://hibbard.eu/demos/ui-blocking-overlay/ "Display UI-blocking overlay on page load").

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <style>
      #overlay{
        position: fixed;
        top: 0%;
        left: 0%;
        width: 100%;
        height: 100%;
        background-color: black;
        -moz-opacity: 0.8;
        filter: alpha(opacity=80);
        opacity:.80;
        z-index:1001;
      }
      #announcement{
        display: none;
        position: absolute;
        width: 250px;
        height: 120px;
        padding: 0 16px;
        border: 16px solid orange;
        background-color: white;
        z-index:1002;
      }
      #close{
        display: inline;
        float: right;
      }
    </style>
  </head>
  <body>
    <h1>Some content</h1>
    <button>Clear storage</button>

    <div id="announcement">
      <p>This is an important announcement.</p>
      <p>Seriously!</p>
      <a id="close" href="#">Close</a>
    </div>

    <script src="https://code.jquery.com/jquery-latest.js"></script>
    <script>
      jQuery.fn.center = function () {
        var w = $(window);
        this.css("position","absolute");
        this.css("top", Math.max(0, (
          (w.height() - $(this).outerHeight()) / 2) + w.scrollTop()
        ) + "px");
        this.css("left", Math.max(0, (
          (w.width() - $(this).outerWidth()) / 2) + w.scrollLeft()
        ) + "px");
        return this;
      }
      var hasReadAnnouncement = localStorage
                                  .getItem("hasReadAnnouncement");
      if (!hasReadAnnouncement){
        $('<div>', {id : 'overlay'}).appendTo('body');
        $("#announcement").fadeIn('slow').center();
      }
      $("#close").click(function(e){
        localStorage.setItem("hasReadAnnouncement", "true");
        $("#announcement").remove();
        $("#overlay").remove();
        e.preventDefault();
      });
      $("button").on("click", function(){
        localStorage.removeItem("hasReadAnnouncement");
        alert("Storage cleared");
      })
    </script>
  </body>
</html>
```

I hope this proved useful for people. If you have any questions, I'd be glad to hear them in the comments.
