---
title: Sliding Panels with jQuery
layout: post
permalink: /sliding-panels-with-jquery/
tags:
  - animation
  - css
  - images
  - javascript
  - jquery
excerpt_separator: <!--more-->
comments: false
---

Sliding panels are cool! jQuery is cool! Who doesn't love sliding panels and jQuery??

This tutorial demonstrates how to build a menu, where the items respond to a user click by sliding a panel into view containing relevant content.

Here's [a demo of what we'll end up with](https://hibbard.eu/demos/sliding-panels/4/ "The finished product").

<!--more-->

---

## Basic Structure

So, the first thing we'll need is some HTML.

We'll define a menu bar which will occupy 100% of the viewport height. This bar will contain the actual menu which will be marked up as an unordered list.

The panels themselves will be div elements with a width of 750px, and will sit on the left of the menu bar. We'll give them unique ids, so that we can reference them later from our jQuery.

```html
<!DOCTYPE HTML>
<html>
  <head>
    <meta charset="UTF-8">
    <title>jQuery sliding panels example</title>
  </head>

  <body>
    <div id="wrapper">
      <div id="menu_panel">
        <ul id="menu">
          <li><a href="#index" class="panel">Home</a></li>
          <li><a href="#about" class="panel">About</a></li>
          <li><a href="#contact" class="panel">Contact</a></li>
        </ul>
      </div>

      <div id="mask">
        <div id="index">
          <h2>Homepage</h2>
          <p>Lorem ipsum dolor sit amet...</p>
        </div>

        <div id="about">
          <h2>About Us</h2>
          <p>Lorem ipsum dolor sit amet...</p>
        </div>

        <div id="contact">
          <h2>Contact Us</h2>
          <p>Lorem ipsum dolor sit amet...</p>
        </div>
      </div>
    </div>
  </body>
</html>
```

Here's [a demo page of what we've got so far](https://hibbard.eu/demos/sliding-panels/1/ "First step - very unexciting").

If you look at it, you'll see that I've added some CSS styles to it. I don't want to list the CSS here, as it'll take up too much space, but if you're curious, be sure to inspect the page source.

## Bring on the jQuery

To start we need to hide all of the panels using `display: none`.

Then we need to add event handlers to our menu items, so that when they are clicked they slide the current panel out of view and slide the panel corresponding to whatever was clicked back into view.

```js
$('#menu a').click(function () {
  hidePanel($(this).attr('href'))
  return false;
});
```

What this will do is grab all of the anchor tags within our menu and attach an event handler to them that calls the function hidePanel when they are clicked.

This function is passed the href value of whichever link was clicked, before returning `false` so as to prevent the link from being followed.

The hidePanel function looks like this:

```js
function hidePanel(e){
  $('div.sel').animate({ left: -750 }, 900, function(){
    $(this).removeClass('sel').hide();
    showPanel(e);
  });
 }
```

It first grabs the div element which has the class `sel` (which we will assign in the next step). It then animates this div's `left` CSS property to a value of minus that of its width over a period of 900 milliseconds, effectively causing the panel to slide out of view.

Once the animation is complete, a callback function removes the class of `sel` and hides the div (assigning it the CSS property of `display: none`). This will make it easier to reference the "invisible" divs later.

It then calls the function `showPanel`, passing it the `href` value of whichever link was previously clicked, so that the script knows which panel to display.

```js
function showPanel(e){
  $(e).css("display", "block").animate({ left: 0 }, 900)
  $(e).addClass('sel');
}
```

This function sets the display property of the panel to be animated to that of `block`, then animates the panel's `left` CSS property to a value of zero, effectively sliding it back into view.

It then adds a class of `sel` to the panel which is now displaying.

All that now remains to be done is on page load, to assign all three of the "invisible" panels a `left` CSS property of -750px and then to animate one of them into view:

```js
$('.slider:hidden').css({ left: -750 });
$("#index").css("display", "block").animate({ left: 0 }, 900);
```

And there we are – [a basic working slider](https://hibbard.eu/demos/sliding-panels/2/ "A working slider").

## Pretty Backgrounds

To make the page less boring, it would be nice to add a full page background. To do this, we will need three pictures, in a decent resolution.

An excellent source of royalty free stock photography is [Wikimedia Commons](http://commons.wikimedia.org/wiki/Main_Page "Wikimedia Commons, a database of freely usable media files to which anyone can contribute.").

Here are the three photos I chose, with the correct attribution:

1. [Damselfly](http://commons.wikimedia.org/wiki/File:Damselfly_October_2007_Osaka_Japan.jpg#mw-page-base "Damselfly") by [Latiche](http://www.laitche.com/ "Laitche Studio Works")
2. [Autum Scenery](http://commons.wikimedia.org/wiki/File:Autumn_scenery.jpg#mw-page-base "Autumn Scenery") by Daniel Skorodjelow
3. [Landschaft bei Brand Erbisdorf](http://commons.wikimedia.org/wiki/File:Landschaft_bei_Brand_Erbisdorf.jpg#mw-page-base "Landschaft bei Brand Erbisdorf") by [Ingo](http://www.flickr.com/photos/44080248@N03 "Ingo's Flickr Stream")

To make these photos scale to the exact dimensions of the viewport, I am going to use a plugin called [jQuery Backstrectch](http://srobbin.com/jquery-plugins/backstretch/ "Backstretch home page"). This is a simple jQuery plugin that allows you to add a dynamically-resized background image to any page or element.

Wonderful!

Having downloaded it and included it in our page, it is just a simple matter of initializing it like so:

```js
$.backstretch("index.jpg");
```

We can also update our showPanel method to construct a file name from the argument it is passed (either `#index`, `#about` or `#contact`).

We do this by removing the first character (the hash symbol) and appending the file ending (in this case .jpg).

```js
function showPanel(e){
  $(e).css("display", "block").animate({ left: 0 }, 900, function(){
    bg = e.substring(1, e.length) + '.jpg'
    $.backstretch(bg, { speed: 1000 });
  })
  $(e).addClass('sel');
 }
```

We can do this inside of the animation's callback function and by passing `{ speed: 500 }`, to the `backstretch` function, we can ensure a nice fade in / fade out effect.

[Here's the result](https://hibbard.eu/demos/sliding-panels/3/ "With pretty backgrounds").

## Some Little Extras

As we have such pretty backgrounds, it would be nice to make the navigation bar semi-transparent, as well as offer the possibility of hiding the current panel altogether.

The first part is realised very easily:

```js
.transparent {
  zoom: 1;
  filter: alpha(opacity=50);
  opacity: 0.5;
}
```

and then:

```html
<div id="menu_panel" class="transparent">
```

The second part is a little more complicated. First off, we have to update our navigation with this option:

```html
<ul id="menu">
  <li><a href="#index" class="panel">Home</a></li>
  <li><a href="#about" class="panel">About</a></li>
  <li><a href="#contact" class="panel">Contact</a></li>
  <br /><br />
  <li><a href="#" id="panelHandle">Hide Content</a></li>
</ul>
```

Then attach an event handler to our new menu point, that adds the required functionality:

```js
$('#panelHandle').toggle(function() {
  $('div.sel').animate({ left: -750 }, 900, function(){
    $('#panelHandle').text("Show Content");
  });
  return false;
}, function() {
  $('div.sel').animate({ left: 0 }, 900, function(){
    $('#panelHandle').text("Hide Content");
  });
  return false;
});
```

This makes use of jQuery's toggle event handler which executes each passed function on successive clicks.

There is hopefully nothing surprising within the functions themselevs.

## A Final Refactoring

Hardcoding the width of the panels is not very future-proof. It is a better idea to determine the width of the panels once at runtime and save this width in a variable.

Therefore we can put this in our code:

```js
var w = $('.sel').width();
```

Then add a class of `sel` to the panel we want to open first, before replacing every occurrence of  the number '750' with the variable `w`.

[Here's the final result again, in case you missed it the first time](https://hibbard.eu/demos/sliding-panels/4/ "The final product").
