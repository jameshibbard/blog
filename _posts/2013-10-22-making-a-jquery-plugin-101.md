---
title: Making a jQuery Plugin – 101
layout: post
permalink: /making-a-jquery-plugin-101/
tags:
  - dates
  - javascript
  - jquery
  - jquery plugins
excerpt_separator: <!--more-->
comments: false
---

The other day my colleague mentioned that she'd like a simple calendar on the front page of her website. This didn't sound too difficult, so being a nice chap I said I'd take a look.

Unfortunately, it turned out that my colleague works with this rather rigid [content management system](http://www.imperia.net/produkt/index.html.en "Shady, shady, shady ...") and beyond a few pre-defined widgets, she didn't have the ability to alter much on the page.

On closer inspection however, I found that one of these widgets allowed us to create arbitrary HTML elements, as well as to include script tags. This sounded like a perfect case for a jQuery plugin…

<!--more-->

OK, so it isn't _really_ a perfect case for a plugin, but I was interested to see how plugin development is done and doing it this way would allow me to test and debug everything on my own PC, then just drop it into my colleague's website with a couple of lines of code.

For the impatient, here's a [demo](http://hibbard.eu/demos/calendar-plugin/ "The calendar plugin in action") of what we'll end up with.

## So How Do Plugins Actually Work?

In its simplest form, you can create a plugin by extending `$.fn` (an alias for `jQuery.prototype`) with a function of your own, which will then be available just like any other jQuery object method.

For example, if you wanted to write a pointless function to add random amounts of padding to an element, you could do it like this:

```js
$.fn.animatePadding= function() {
  var randomPadding = Math.floor(Math.random()*101);
  this.animate({ 'padding' : randomPadding });
  return this;
};

$(selector).animatePadding()
```

Worthy of note is that jQuery is the parent scope to your function, so within the function `this` will refer to the jquery object.

Also, returning `this` allows you to chain your methods together.

You can read more about the basics of plugin creation on the jQuery site: [How to Create a Basic Plugin](http://learn.jquery.com/plugins/basic-plugin-creation/ "Basic Plugin Authoring")

## Back to My Colleague

Now what I would really like to end up with is the following:

```html
<div id="calendar"></div>

<script>
  $("#calendar").calendarize({ some options here });
</script>
```

The options shouldn't be anything to fancy: some styling for the current day, an address to which the current day should link (the events page) and the heading for the calendar.

Also, as my colleague has a multi-lingual website (English and German) and it would be nice to specify a language and have the month appear in the correct language.

```js
$.fn.calendarize = function( options ) {
  var settings = $.extend({
    styles: {'color': '#8DAE10'},
    link: "http://www.google.com",
    heading: "My great calendar",
    language: "EN"
  }, options );
}
```

The easiest way to do this is to this is with an object literal. You can specify sensible defaults in your plugin's settings and use the [$.extend()](http://api.jquery.com/jQuery.extend/ "API docs:  jQuery.extend()") method to overwrite them with anything that the user passes in.

```js
$("#calendar").calendarize({language: "DE"});
```

Next we're going to need a method of getting the number of days in a given month:

```js
function daysInMonth(month,year) {
  return new Date(year, month, 0).getDate();
}
```

As well as method of distinguishing between German and English month names:

```js
switch(settings.language){
  case "GER":
    month[0]="Januar";
    month[1]="Februar";
    ...
    break;
  default:
    month[0]="January";
    month[1]="February";
    ...
}
```

and references to the day and year:

```js
var today = new Date(),
    day = today.getDate(),
    month = new Array(),
    year = today.getFullYear(),
    noDays = daysInMonth(month, year);
```

## Outputting Something to the Screen

Let's start off with our heading.

There's nothing special about this: create a `<h3>` element, then set its html to the heading name as specified by the settings, followed by the current month and year, before appending it to the calendar div.

```js
$('<h3>', {
  html: settings.heading +
        '<br>' +
        month[today.getMonth()] +
        ', ' +
        year
}).appendTo(this);
```

Then comes the main loop.

What I would like to do, is create a `<span>` element for every day and have five days in one row followed by a separating `<div>`.

```js
for(i= 1; i < noDays +1; i++){
  daySpan = $('<span>', {
    class: 'day',
    text: i
  }).appendTo(this);

  if (i%5 === 0){
    $('<div>', {
      class: 'row'
    }).appendTo(this);
  }
}
```

Now all we have to do is wrap an anchor tag around the current day and apply any necessary styling to it:

```js
if (i === day){
  daySpan.text('');
  curDayLink = $('<a>', {
    href: settings.link,
    text: i
  })

  $.each( settings.styles, function( property, value ) {
    curDayLink.css( property, value );
  });

  curDayLink.appendTo(daySpan);
}
daySpan.appendTo(this);
```

## Wrapping It Up

There's one more little trick to employ before calling this a day, namely to add scope to the plugin, in order to protect the `$` alias.

This is necessary because the dollar variable is very popular amongst JavaScript libraries and consequently jQuery has a [noConflict()](http://api.jquery.com/jQuery.noConflict/ "API docs:  jQuery.noConflict()") method to release control of it.

This however, would break our plugin, as we are assuming that the dollar is available.

To work around this we need to put all of our code inside of an [Immediately Invoked Function Expression](https://medium.com/@vvkchandra/essential-javascript-mastering-immediately-invoked-function-expressions-67791338ddc6) (IIFE), and then pass the function `jQuery`, and name the parameter `$`:

```js
(function ($) {
  // Plugin code here
}(jQuery));
```

And there you have it, a simple jQuery plugin.

I hope this was of use to somebody somewhere.

If you have any questions, leave them in the comments.

Here's the complete code:

```html
<!DOCTYPE HTML>
<html>
  <head>
    <meta charset="utf-8">
    <title></title>
    <style>
      #calendar{ font-size:14px; }

      .day{
        display: inline-block;
        width:30px;
        text-align:center;
        margin-right:10px;
      }

      .row{
        display:block;
        border-top:solid 1px gray;
        height:3px;
        margin-top:3px;
        width: 190px;
      }
    </style>
  </head>

  <body>
    <div id="calendar"></div>

    <script src="http://code.jquery.com/jquery-latest.min.js"></script>
    <script>
      (function ( $ ) {
        $.fn.calendarize = function( options ) {

          function daysInMonth(month,year) {
            return new Date(year, month, 0).getDate();
          }

          var settings = $.extend({
                styles: {
                  'color': '#8DAE10',
                  'font-weight': 'bold',
                  'text-decoration': 'none'
                },
                link: "http://www.google.com",
                heading: "My Awesome Calendar",
                language: "EN"
              }, options ),

              i,
              heading,
              daySpan,
              curDayLink,
              today = new Date(),
              day = today.getDate(),
              month = today.getMonth() + 1,
              year = today.getFullYear(),
              noDays = daysInMonth(month, year),
              month = new Array();

              switch(settings.language){
                case "GER":
                  month[0]="Januar";
                  month[1]="Februar";
                  month[2]="März";
                  month[3]="April";
                  month[4]="Mai";
                  month[5]="Juni";
                  month[6]="Juli";
                  month[7]="August";
                  month[8]="September";
                  month[9]="Oktober";
                  month[10]="November";
                  month[11]="Dezember";
                  break;
                default:
                  month[0]="January";
                  month[1]="February";
                  month[2]="March";
                  month[3]="April";
                  month[4]="May";
                  month[5]="June";
                  month[6]="July";
                  month[7]="August";
                  month[8]="September";
                  month[9]="October";
                  month[10]="November";
                  month[11]="December";
              }

          $('<h3>', {
            html: settings.heading +
                  '<br />' +
                  month[today.getMonth()] +
                  ', '  +
                  year
          }).appendTo(this);

          for(i= 1; i < noDays +1; i++){
            daySpan = $('<span>', {
              class: 'day',
              text: i
            })

            if (i === day){
              daySpan.text('');

              curDayLink = $('<a>', {
                href: settings.link,
                text: i
              })

              $.each(settings.styles, function(property, value) {
                curDayLink.css( property, value );
              });

              curDayLink.appendTo(daySpan);
            }

            daySpan.appendTo(this);

            if (i%5 === 0){
              $('<div>', {
                class: 'row'
              }).appendTo(this);
            }
          }

          return this;
        };
      }( jQuery ));
      $("#calendar").calendarize({language: "DE"});
    </script>
  </body>
</html>
```
