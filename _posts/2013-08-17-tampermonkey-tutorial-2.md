---
title: Tampermonkey Tutorial (2)
layout: post
permalink: /tampermonkey-tutorial-2/
tags:
  - forms
  - javascript
  - jquery
  - tampermonkey
  - userscripts
excerpt_separator: <!--more-->
old-comments: tampermonkey-tutorial-2.html
comments: false
---

At work, I'm currently working on a rather large form. And when I say large, I mean seriously large – I just counted, it has 136 fields.

Now when a user submits the form, the input is validated. If everything is correct the user's data is saved to a data base, otherwise the form re-renders, displaying appropriate error messages.

I have written unit tests to test the form's logic, but nonetheless I still want to test the form manually, just to be 100% sure that things are displaying as expected.

But there's no way that I'm going to manually fill out 136 fields over and over again. No Sir! A much better solution would be to use Tampermonkey to do it for me.

<!--more-->

So that I can demonstrate how to do this, we're going to need a simple form.

Here's one I made earlier:

```html
<!DOCTYPE HTML>
<html>
  <head>
    <meta charset="utf-8">
    <title>Tampermonkey form auto-fill</title>
    <style>
      label{ display: inline-block; margin:5px 0px; }
    </style>
  </head>

  <body>
    <form>
      <div>
        <label for="name">Your name:</label>
        <input type="text" id="name">
      </div>
      <div>
        <label for="email">Your email:</label>
        <input type="text" id="email">
      </div>
      <div>
        <label for="comments">Your comments:</label><br>
        <textarea id="comments" rows="15" cols="50"></textarea>
      </div>
      <input type="submit" value="Submit">
    </form>
  </body>
</html>
```

Were also going to need to install [Tampermonkey](http://tampermonkey.net/ "Tampermonkey homepage") (a userscript manager for [Blink-based](http://en.wikipedia.org/wiki/Blink_(layout_engine) "Blink (layout engine)") Browsers such as Chrome and Opera). Luckily, my [previous Tampermonkey tutorial](http://hibbard.eu/tampermonkey-tutorial/ "Tampermonkey Tutorial") explains how to do this.

Now, with Tampermonkey installed, click on the Tampermonkey icon in the top right hand corner of your browser, then select _Add a new script …_. A new tab will open which looks like this:

![Tampermonkey screenshot - create new script](https://res.cloudinary.com/hibbard/image/upload/v1528910435/tampermonkey_screenshot.png "Tampermonkey screenshot - create new script")

So, let's fill those details out. They're all fairly self explanatory, except perhaps line 6 (which begins ‘@match'). Here, using a regular expression, you can specify a full or partial url. When this regular expression matches your current url, Tampermonkey will fire the script.

```js
// ==UserScript==
// @name Auto-fill my massive form
// @namespace http://hibbard.eu/
// @version 0.1
// @description Autofills fields in my massive form
// @match http://hibbard.eu/*
// @copyright 2013+, hibbard.eu
// @require http://code.jquery.com/jquery-latest.js
// ==/UserScript==
```

As you can see, I have specified that this script should run when I visit any page beginning with `http://hibbard.eu/`. Obviously, anyone following this tutorial will need to adapt this path accordingly.

Also worthy of note is the fact that I have required the latest version of jQuery in the final line.

So, with that out of the way, we can get on to the exciting bit, writing the script.

Let's start off by adding a button to the page and attaching some click behaviour to it:

```js
$('body').prepend('
  <input type="button" value="Populate" id="populate">
');

$("#populate").on("click", function() {
  console.log("I was clicked");
});
```

To populate the form, I am going to create an object-literal whose properties correspond to the ids of the respective form fields. Then, when the user clicks the _populate_ button, I am going to set the values of the form fields to the value of the corresponding attribute.

```js
var values = {
  name: "Fred Blogs",
  email: "freddyboy@gmail.com",
  comment: "Your tutorial is simply the best!"
}

$('body').prepend('
  <input id="populate" type="button" value="Populate"/>
');

$("#populate").on("click", function(){
  Object.keys(values).forEach(function(key){
    $("#" + key).val(values[key]);
  });
});
```

And that's it! Now, when you visit the page with the form on, you will see a button, that when clicked, will auto-fill your form for you – quite pointless in this small example, but when you have 136 form fields to deal with, it's pretty cool!

As a side note, in the above code we are combining `Object.keys()` and `Array.prototype.forEach()` to iterate over the object's attributes.

You can read more about this here: [How to Loop through JavaScript object literal with objects as members?](http://stackoverflow.com/questions/921789/how-to-loop-through-javascript-object-literal-with-objects-as-members "Stackoverflow discussion")

So, I hope this tutorial was useful for people. I noticed in my analytics that I'm getting a lot of traffic for "Tampermonkey tutorial", so if there is some aspect of this extension that I didn't cover, that people would like to learn about, be sure to let me know in the comments.
