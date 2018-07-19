---
title: Tampermonkey Tutorial
layout: post
permalink: /tampermonkey-tutorial/
tags:
  - javascript
  - jquery
  - tampermonkey
  - userscripts
old-comments: tampermonkey-tutorial.html
comments: false
---

In an effort to stay abreast of developments on the World Wide Web, I subscribe to the CodeProject's excellent newsletter: [The Daily Insider](http://www.codeproject.com/Feature/Insider/ "The Daily Insider. Know it All."). Now, when I open this newsletter in my browser, I'm presented with a whole bunch of links (usually 12) that I want to open. It was getting a bit bothersome to open all of these manually, so I decided to write a userscript to do it for me.

---

**Update 22.08.2013**: I published a [second Tampermonkey](http://hibbard.eu/tampermonkey-tutorial-2/ "Tampermonkey Tutorial (2)") tutorial.

Be sure to check that out, too.

---

For those of you who don't know, userscripts are small snippets of JavaScript code that change the behaviour of a website. Google Chrome (my browser of choice) supports userscripts out of the box, however, to make the whole process of writing and testing one a little more comfortable, the first thing I did, was to install [Tampermonkey](https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo "Tampermonkey on the Google Chrome web store"), a userscript manager for Google Chrome.

To install Tampermonkey, head on over to the [Chrome web store](https://chrome.google.com/webstore/category/home "Chrome web store"), search for Tampermonkey, then click _Add to Chrome_. Once the extension is installed, you'll see a little icon in the top right hand corner of your browser window that looks like this:

![Tampermonkey browser icon](https://res.cloudinary.com/hibbard/image/upload/v1528910417/tampermonkey_icon.png "Tampermonkey browser icon")

To create a new script click on this icon, then select _Add a new script..._ A new tab will open which looks like this:

![Tampermonkey screenshot - create new script](https://res.cloudinary.com/hibbard/image/upload/v1528910435/tampermonkey_screenshot.png "Tampermonkey screenshot - create new script")

So, let's fill those details out. They're all fairly self explanatory, except perhaps line 6 (which begins `@match`). Here, using a regular expression, you can specify a full or partial URL. When this regular expression matches your current URL, Tampermonkey will fire the script.

```js
// ==UserScript==
// @name Open CodeProject Links
// @namespace http://hibbard.eu/
// @version 0.1
// @description  Opens links from the CodeProject newsletter
// @match http://www.codeproject.com/script/Mailouts/*
// @copyright 2012+, hibbard.eu
// @require http://code.jquery.com/jquery-latest.js
// ==/UserScript==
```

As you can see, I have specified that this script should run when I visit any site beginning with `http://www.codeproject.com/script/Mailouts/`. All CodeProject newsletters start with this address, you can see a sample one here:

[http://www.codeproject.com/script/Mailouts/View.aspx?mlid=9776&_z=6134961](http://www.codeproject.com/script/Mailouts/View.aspx?mlid=9776&_z=6134961 "The CodeProject Insider")

Also worthy of note is the fact that I have required the latest version of jQuery in the final line.

So, with this in place we're ready to begin. The first thing to do is to get a list of all of the links that we want to open. Luckily, each link we're interested in is enclosed within a div which has a class of headline.

```js
var hrefs = new Array();
var elements = $('.headline > a');
elements.each(function() {
  hrefs.push($(this).attr('href'));
});
```

Now let's add a button to the page and position it in the top left corner:

```js
$('body').append('<input type="button" value="Open" id="CP">')
$("#CP").css("position", "fixed").css("top", 0).css("left", 0);
```

Now all we need to do is attach an event listener, so that when the button is clicked, all of the links are opened in new tabs.

```
$('#CP').click(function(){
  $.each(hrefs, function(index, value) {
    setTimeout(function(){
      window.open(value, '_blank');
    },1000);
  });
});
```

You might wonder why I've used JavaScript's `setTimeout` function. Well, the reason is that otherwise Chrome opens the first two links as tabs, then the remaining ten links as popups. I don't know why this is, but in this case, setting a small delay seemed to solve the problem.

And there you have it, a small script that saves me twelve additional clicks per day. It might not seem much, but that's 60 unnecessary clicks per week, or 3,120 unnecessary clicks per year. I guess I might be able to avoid [RSI](http://en.wikipedia.org/wiki/Repetitive_strain_injury "Repetitive strain injury") for a while longer.

Here's a listing of the complete code:

```js
// ==UserScript==
// @name       Open CodeProject Links
// @namespace  http://hibbard.eu/
// @version    0.1
// @description  Opens links from the CodeProject newsletter
// @match      http://www.codeproject.com/script/Mailouts/*
// @copyright  2012+, hibbard.eu
// @require http://code.jquery.com/jquery-latest.js
// ==/UserScript==

$(document).ready(function() {
  var hrefs = new Array();
  var elements = $('.headline > a');
  elements.each(function() {
    hrefs.push($(this).attr('href'))
  });

  $('body').append('<input type="button" value="Open Links" id="CP">')
  $("#CP").css("position", "fixed").css("top", 0).css("left", 0);
  $('#CP').click(function(){
    $.each(hrefs, function(index, value) {
      setTimeout(function(){
       window.open(value, '_blank');
      },1000);
    });
  });
});
```

And here's proof that it actually works:

![The finished product](https://res.cloudinary.com/hibbard/image/upload/v1528910463/newsletter_with_button.png "The finished product")
