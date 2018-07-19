---
title: DOM Selection Without jQuery
layout: post
permalink: /dom-selection-without-jquery/
tags:
  - dom
  - javascript
  - jquery
  - selectors
excerpt_separator: <!--more-->
comments: false
---

One of the main reasons behind jQuery's meteoric rise to popularity was its use of the CSS selector syntax to select DOM elements and the ease with which it could then traverse these and modify their content.

In this post I'd like to focus on how jQuery handles DOM selection under the hood and what native JavaScript functions we can use in its place.

<!--more-->

A quick glance at the [API docs](http://api.jquery.com/jQuery/ "jQuery selector documentation"), shows the syntax for jQuery DOM selection to be `jQuery( selector [, context ] )`.

The fact that this method takes an optional context parameter might come as a surprise to some, as we normally see it used without. However, passing a valid context parameter thus: `$('a', $nav)`, would be equivalent to calling `$nav.find('a')`.

When jQuery receives a valid selector as a first parameter it starts by identifying the string it was passed using a regular expression. The easiest scenario here is when this string is an id, in which case jQuery can call the traditional `document.getElementById` and wrap the returned element in a jQuery object.

In most other cases, jQuery will leverage the following native JavaScript methods where they are available:

- `document.querySelectorAll(selector)` — returns a non-live NodeList of all the matching element nodes
- `document.getElementsByTagName(tagname)` — returns a live HTMLCollection of matching elements ('live'meaning that it updates itself automatically to stay in sync with the DOM tree)
- `document.getElementsByClassName(class)` — returns a HTMLCollection of elements with a specific class name

Support for these methods is varied:

- `getElementById` was introduced in DOM Level 1 for HTML documents and works well in almost all browsers (with the [odd exception](http://www.quirksmode.org/bugreports/archives/2005/09/documentgetElementById_may_return_element_with_a_n.html "document.getElementById() returns element with name equal to id specified"))
- `getElementsByTagName` was introduced in DOM Level 2 and also works well in almost all browsers (again with the [odd exception](http://ejohn.org/blog/object-getelementsbytagname-ie7-bug/ "Object getElementsByTagName IE7 Bug")).
- `document.getElementsByClassName` is not as widely supported, most notably lacking support in IE8. [Support overview](http://caniuse.com/getelementsbyclassname "Compatibility table for support of getElementsByClassName in desktop and mobile browsers").
- `querySelectorAll` and the related `querySelector` (which returns the first element within the document that matches the specified group of selectors) were introduced with the [HTML5 Selectors API](http://www.w3.org/TR/selectors-api/ "W3C - Selectors API Level 1"). They have been around for some time now and are supported in all modern browsers and IE8 (although IE8 only supports up to CSS2.1 selectors). [Support overview](http://caniuse.com/queryselector "Compatibility table for support of querySelector/querySelectorAll in desktop and mobile browsers").

For the case that native support is not available, jQuery falls back to using the [Sizzle engine](http://sizzlejs.com/ "Sizzle - a pure-JavaScript CSS selector engine") which was written by John Resig (author of jQuery).

Sizzle is a pure-JavaScript CSS selector engine designed to be easily dropped in to a host library. It's lightweight (4kb when minified), easy to use and I encourage you to check it out.

E.g. to select all `<span>` elements which are a direct child of a `<p>` element, you would do:

```js
Sizzle('p > span');
```

or to select all inputs with an attribute name that starts with 'news':

```js
Sizzle('input[name^="news"]');
```

You can find the [Sizzle documentation here](https://github.com/jquery/sizzle/wiki/Sizzle-Documentation "Sizzle Documentation"), where you will also find details on additional selectors that are supported by sizzle too.

You can also search the [jQuery source code](http://code.jquery.com/jquery-latest.js "jQuery source code") for occurrences of the term "Sizzle" to get an idea of how it is used.

## Life Without jQuery

So, how to implement this in your next project?

Well, for projects which only require the support of modern browsers, you're good to go already – just use the HTML5 Selector's API as outlined above.

For projects which require the support of older browsers, a simple solution is to include the Sizzle library in your code, then map `querySelector` and `querySelectorAll` to Sizzle if native browser support wasn't detected.

You can do that, like this:

```js
// If there's native support for querySelector, don't load Sizzle.
if (undefined !== document.querySelector) {
  return;
}

...

document.querySelectorAll = function querySelectorAll(selector){
  return Sizzle(selector, this);
};

document.querySelector = function querySelector(selector){
  return (document.querySelectorAll.call(this, selector)[0] || null);
};
```

There are a couple of caveats to this approach. For example, IE7 doesn't have an Elements object from which to inherit things like `querySelector`, so you can't do:

```html
<ul>
  <li>One</li>
  <li>Two</li>
  <li class="highlight">Three</li>
  <li>Four</li>
  <li class="highlight">Five</li>
</ul>

var myList = document.getElementById("myList");
console.log(myList.querySelectorAll(".highlight").length);
```

In such a situation it is better to use `getElementsByTagName` instead.

**Note**: This post originally was adapted from a post I originally made on the [Sitepoint Forums](http://www.sitepoint.com/forums/showthread.php?1069204-JavaScript-Challenge-Convert-JQuery-to-Plain-JavaScript&p=5541688&viewfull=1#post5541688 "The original post"), where in conjunction with [Paul Wilkins](https://www.sitepoint.com/community/u/Paul_Wilkins "Paul's profile page"), I am running a JavaScript challenge to convert jQuery to plain JavaScript.
