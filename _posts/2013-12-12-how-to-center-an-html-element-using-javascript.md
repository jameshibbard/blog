---
title: How to Center an HTML Element Using JavaScript
layout: post
tags:
  - javascript
  - images
permalink: /how-to-center-an-html-element-using-javascript/
excerpt_separator: <!--more-->
comments: false
---

Imagine you have an element which is dynamically added to your web page (an image in a lightbox, for instance) and you want to centre it on the screen both horizontally and vertically.

You could do so with CSS (as [this article](http://designshack.net/articles/css/how-to-center-anything-with-css/ "How to Center Anything With CSS") explains), but if you don't know the dimensions of the element you want to centre or you need to support older browsers, things can become quite tricky quite quickly.

That's when you should think about doing the job with JavaScript.

<!--more-->

First we'll need an image:

```js
var img = new Image(),
    body = document.getElementsByTagName("body")[0];
img.src = "http://www.lolcats.com/images/u/13/39/tastegood.jpg";
body.appendChild(img);
```

Just the job.

Now to centre it, we'll need the image's dimensions:

```js
var imageWidth = img.offsetWidth,
    imageHeight = img.offsetHeight;
console.log(imageWidth, imageHeight);
```

But this just logs `0 0` to the console. What's going on?

Well, we need to wait for the image to load before we can access its properties:

```js
img.onload = function(){
  var imageWidth = this.offsetWidth,
      imageHeight = this.offsetHeight;
  console.log(imageWidth, imageHeight);
}
```

That's better.

Now we need to grab the viewport dimensions:

```js
var vpWidth = document.documentElement.clientWidth,
    vpHeight = document.documentElement.clientHeight;
```

Then all that remains to do is subtract the image's width from the viewport width, halve the difference and set the image's `left` property accordingly.

The height is a little trickier, as we have to keep in mind the fact that the user might have scrolled the page, so we subtract the image's height from the viewport height, halve it, then add that number to the window's [pageYOffset](https://developer.mozilla.org/en-US/docs/Web/API/Window.scrollY "Returns the number of pixels that the document has already been scrolled vertically.") property.

```js
img.onload = function(){
  var imageWidth = this.offsetWidth,
      imageHeight = this.offsetHeight,
      vpWidth = document.documentElement.clientWidth,
      vpHeight = document.documentElement.clientHeight;

  this.style.position = 'absolute'
  this.style.left = (vpWidth - imageWidth)/2 + 'px';
  this.style.top = (vpHeight - imageHeight)/2 +
                   window.pageYOffset + 'px';
}
```

Now this works, but it we wanted to centre two elements, we'd have an awful lot of duplicated code.

That's why it's better it put the code for centering things in it's own function.

In the following example I've extended HTMLElement's prototype for a nicer syntax.

```js
HTMLElement.prototype.centre = function(){
  var w = document.documentElement.clientWidth,
      h = document.documentElement.clientHeight;
  this.style.position = 'absolute';
  this.style.left = (w - this.offsetWidth)/2 + 'px';
  this.style.top = (h - this.offsetHeight)/2 +
                   window.pageYOffset + 'px';
}

var img = new Image(),
    body = document.getElementsByTagName("body")[0];
img.src = "http://www.lolcats.com/images/u/13/39/tastegood.jpg";
img.onload = function(){
  this.centre();
}

body.appendChild(img);
```

I hope this helps someone.

If you have any comments or questions, I'd be glad to hear them.
