---
title: How to Create a Custom Filter Selector with jQuery
layout: post
permalink: /how-to-create-a-custom-filter-selector-with-jquery/
tags:
  - javascript
  - jquery
  - selectors
excerpt_separator: <!--more-->
comments: false
---

A little jQuery trick I learned recently, was how to create a reusable, custom filter to target specific elements based on their characteristics.

I was kind of surprised that I hadn't heard of this before, so thought I'd jot it down here, in case it is of use to anyone else.

<!--more-->

## A Quick Refresher

By way of a refresher, the [filter()](http://api.jquery.com/filter/ "jQuery API Documentation: .filter()") method in jQuery reduces the set of matched elements to those that match the selector or pass the function's test.

For example, you can apply a specific style to evenly indexed items in a list, thus:

```js
$("li").filter(":even").css("background-color", "red");
```

or, using the shorthand version:

```js
$("li:even").css("background-color", "red");
```

Filters are handy for a whole bunch of things, such as selecting hidden elements:

```js
$("div:hidden").show();
```

Or selecting an element's first child:

```js
$("div.myClass div:first-child").fadeIn("slow");
```

And you can even filter against a function rather than a selector. For each element, if the function returns `true` (or a "truthy" value), the element will be included in the filtered set; otherwise, it will be excluded:

```js
$("li").filter(function(index) {
  return index % 2 === 0;
}).addClass("green");
```

### Rolling Your Own

You can extend jQuery's selector expressions under the `jQuery.expr[':']` object (an alias for `Sizzle.selectors.filters`). Each new filter expression is defined as a property of this object, like so:

```js
jQuery.expr[':'].myFilter = function(elem, index, match){
  // Return true or false
};
```

The function will be run on all elements in the current collection. Returning `true` will keep the element in the collection, whereas `false` will remove it.

As you can see, the function receives three parameters: the element in question, the index of this element in the collection, and a match array which is useful for more complex expressions.

You can read more about creating custom filters in the following links:

-  [Creating Your Own Custom jQuery Filters](http://sampsonblog.com/279/creating-your-own-custom-jquery-filters "Jonathan Sampson - Creating Your Own Custom jQuery Filters")
-  [Creating a Custom Filter Selector with jQuery](http://answers.oreilly.com/topic/1055-creating-a-custom-filter-selector-with-jquery/ "An excerpt from the jQuery Cookbook")
-  [Make jQuery :contains Case-Insensitive](http://css-tricks.com/snippets/jquery/make-jquery-contains-case-insensitive/ "CSS-Tricks > Code Snippets > jQuery")

### A Proper Example

Armed with this new and dangerous knowledge, I decided to make a filter to select all green elements on a page.

Admittedly, this is rather pointless, but hey, you never know…

The filter itself looks like this:

```js
jQuery.expr[':'].green = function(elem, index, match) {
  return $(elem).css("background-color") === "rgb(0, 128, 0)";
};
```

You might notice that I am using the rgb value to determine the colour. This works great in Chrome and Firefox, but YMMV using IE.

You can apply it to all of the elements on a page thus:

```js
$(":green")
```

or to a subset of elements (which is a better idea performance-wise), thus:

```js
$("#myDiv:green")
```

And finally, here's a working demo to play around with:

<iframe style="width:100%; height:400px; border:solid #4173A0 1px;" src="http://jsfiddle.net/hibbard_eu/VcxNC/embedded/result,js,html,css/light/" frameborder="0"></iframe>

I hope this proved useful for people. If you have any questions, I'd be glad to hear them in the comments.
