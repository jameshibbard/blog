---
title: "Quick Tip: Persist Checkbox Checked State after Page Reload"
layout: post
permalink: /persist-checkbox-checked-state-after-page-reload/
tags:
  - javascript
  - jquery
  - local storage
  - events
excerpt_separator: <!--more-->
canonical:
  url: 'https://www.sitepoint.com/quick-tip-persist-checkbox-checked-state-after-page-reload/'
---

<p class="callout originally-posted-elsewhere">This article was <a href="https://www.sitepoint.com/quick-tip-persist-checkbox-checked-state-after-page-reload/">first published on SitePoint</a> and has been republished here with permission.</p>

This quick tip describes how to have your browser remember the state of checkboxes once a page has been refreshed or a user navigates away from your site to come back at a later date.

It might be useful to persist checkbox checked state if, for example, you use checkboxes to allow your users to set site-specific preferences, such as opening external links in a new window or hiding certain page elements.

<!--more-->

For the impatient among you, there's [a demo of the technique](#demo) at the end of the article.

## The Checkbox Markup

So, the first thing we'll need are some checkboxes. Here are some I made earlier:

```html
<div id="checkbox-container">
  <div>
    <label for="option1">Option 1</label>
    <input type="checkbox" id="option1">
  </div>
  <div>
    <label for="option2">Option 2</label>
    <input type="checkbox" id="option2">
  </div>
  <div>
    <label for="option3">Option 3</label>
    <input type="checkbox" id="option3">
  </div>

  <button>Check All</button>
</div>
```

You'll notice that I've included a *Check All* button to allow a user to select or deselect all of the boxes with one click. We'll get to that later.

You'll hopefully also notice that I'm using labels for the text pertaining to each of the boxes. It goes without saying that [form fields should have labels](http://www.coolfields.co.uk/2011/04/accessible-forms-should-every-input-have-a-label/) anyway, but in the case of checkboxes this is particularly important, as it allows users to select/deselect the box by clicking on the label text.

Finally, you'll see that I'm grouping the labels and check boxes inside block-level elements (`<div>` elements in this case), so that they appear beneath each other and are easier to style.

## Responding to Change

Now let's bind an event handler to the checkboxes, so that something happens whenever you click them. I'm using [jQuery](http://jquery.com/) for this tutorial although, of course, this isn't strictly necessary. You can include it via a CDN thus:

```html
<script src="https://code.jquery.com/jquery-2.2.3.min.js"></script>
```

Now the JavaScript:

```javascript
$("#checkbox-container :checkbox").on("change", function(){
  alert("The checkbox with the ID '" + this.id + "' changed");
});
```

Here I'm using jQuery's psuedo-class [:checkbox selector](https://api.jquery.com/checkbox-selector/), preceded by the ID of the containing `<div>` element (`checkbox-container`). This allows me to target only those checkboxes I am interested in and not all of the checkboxes on the page.

## Persist Checkbox Checked State

As you're probably aware HTTP is a [stateless protocol](https://en.wikipedia.org/wiki/Stateless_protocol). This means that it treats each request as an independent transaction that is unrelated to any previous request. Consequently if you check a checkbox then refresh the page, the browser has no way of remembering the checkbox's prior state and—in the example above—will render it in its unchecked form (although [some browsers will cache the value](http://stackoverflow.com/questions/299811/why-does-the-checkbox-stay-checked-when-reloading-the-page)).

To circumvent this, we're going to store the checkbox states on a user's computer using  local storage (which is part of the [Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API/Using_the_Web_Storage_API)). It has pretty good browser support:

<p class="ciu_embed" data-feature="namevalue-storage" data-periods="future_1,current,past_1,past_2">   <a href="http://caniuse.com/#feat=namevalue-storage">Can I Use namevalue-storage?</a> Data on support for the namevalue-storage feature across the major browsers from caniuse.com. </p>

<script src="//cdn.jsdelivr.net/caniuse-embed/1.0.1/caniuse-embed.min.js"></script>

To modify the `localStorage` object in JavaScript, you can use the `setItem` and `getItem` methods:

```javascript
localStorage.setItem("favoriteVillan", "Dr. Hannibal Lecter");

console.log(localStorage.getItem("favoriteVillan"));
=> Dr. Hannibal Lecter
```
To remove an item from local storage, use the `removeItem` method.

```javascript
localStorage.removeItem("favoriteVillan");

console.log(localStorage.getItem("favoriteVillan"));
=> null
```

You can read more about local storage here: [HTML5 Web Storage](http://www.sitepoint.com/html5-web-storage/)

With respect to our code, it seems that as we want to persist all of the checkboxes, a logical choice would be to make key/value pairs consisting of the checkbox IDs and their corresponding checked state. We can save these in an object literal which we can add to local storage. However, as local storage can only handle key/value pairs, we'll need to stringify the object before storing it and parse it upon retrieval.

Let's look at how we might do that:

```javascript
var checkboxValues = JSON.parse(localStorage.getItem('checkboxValues')) || {};
var $checkboxes = $("#checkbox-container :checkbox");

$checkboxes.on("change", function(){
  $checkboxes.each(function(){
    checkboxValues[this.id] = this.checked;
  });
  localStorage.setItem("checkboxValues", JSON.stringify(checkboxValues));
});
```

Notice the logical OR operator (`||`) which returns the value of its second operand, if the first is falsy. We're using this to assign an empty object to the `checkboxValues` variable, if no entry could be found in local storage. This is effectively the same as writing:

```javascript
var checkboxValues = JSON.parse(localStorage.getItem('checkboxValues');
if (checkboxValues === null){
  checkboxValues = {};
}
```

Also notice that we are caching a reference to the checkboxes, so that we don't have to keep querying the DOM. This is overkill in such a small example, but is a good habit to get into as your code grows.

The last thing to do is to iterate over the `checkboxValues` variable when the page loads and set the value of the boxes accordingly:

```javascript
$.each(checkboxValues, function(key, value) {
  $("#" + key).prop('checked', value);
});
```

And that's it. We now have persistent checkboxes.

<p data-height="250" data-theme-id="6441" data-slug-hash="xVevNw" data-default-tab="result" data-user="SitePoint" data-embed-version="2" class="codepen">See the Pen <a href="http://codepen.io/SitePoint/pen/xVevNw/">xVevNw</a> by SitePoint (<a href="http://codepen.io/SitePoint">@SitePoint</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

If you try refreshing the Code Pen (by clicking the *Rerun* button), you'll see that the checkboxes remain selected, whereas if you enter something into the text input, that will vanish on reload.

## Checking and Unchecking All the Boxes

To round this off, let's implement the *Check All* button, so that users can select or deselect everything in one go.

When clicked, this button should check all of the checkboxes. When all of the checkboxes are checked its text should change to *Uncheck All* and its state should be stored in local storage, too.

Let's start off like this:

```javascript
var $button = $("#checkbox-container button");

function allChecked(){
  return $checkboxes.length === $checkboxes.filter(":checked").length;
}

function updateButtonStatus(){
  $button.text(allChecked()? "Uncheck all" : "Check all");
}

$checkboxes.on("change", function(){
  ...
  updateButtonStatus();
});
```

This will cause the button's text to change when all of the check boxes are in a checked or unchecked state.

The only thing that might be slightly unusual here is the use of a [ternary conditional](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator) in the `updateButtonStatus` method. This is a shortcut for an `if` statement and is the equivalent to:

```javascript
if(allChecked()){
  $button.text("Uncheck all");
} else {
  $button.text("Check all");
}
```

Now a function to do the selecting and deselecting:

```javascript
function handleButtonClick(){
  $checkboxes.prop("checked", allChecked()? false : true)
}

$("button").on("click", function() {
  handleButtonClick();
});
```

Finally, we'll add a new value to the object in local storage, which I have renamed to reflect the fact that it now stores different types of values. I have also moved the logic for updating local storage into its own function:

```javascript
var formValues = JSON.parse(localStorage.getItem('checkboxValues')) || {};

function updateStorage(){
  $checkboxes.each(function(){
    formValues[this.id] = this.checked;
  });

  formValues["buttonText"] = $button.text();
  localStorage.setItem("formValues", JSON.stringify(formValues));
}
```

And on page load:

```javascript
$button.text(formValues["buttonText"]);
```

## Demo

Here's what we end up with.

<p data-height="280" data-theme-id="6441" data-slug-hash="grJYvd" data-default-tab="result" data-user="SitePoint" data-embed-version="2" class="codepen">See the Pen <a href="http://codepen.io/SitePoint/pen/grJYvd/">grJYvd</a> by SitePoint (<a href="http://codepen.io/SitePoint">@SitePoint</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

## Conclusion

In this tutorial I have demonstrated how to persist form values to local storage and how to implement _Check/Uncheck All_ functionality for checkboxes in a simple form.

If you have any questions or comments, I'd be glad to hear them below.
