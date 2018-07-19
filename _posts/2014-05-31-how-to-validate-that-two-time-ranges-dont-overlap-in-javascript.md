---
title: How to Validate That Two Time Ranges Don't Overlap in JavaScript
layout: post
permalink: /how-to-validate-that-two-time-ranges-dont-overlap-in-javascript/
tags:
  - javascript
  - times
excerpt_separator: <!--more-->
comments: false
---

I was working on a form for a timekeeping app, where a user is able to enter their time worked, as well as breaks taken.

There is no limit on the amount of breaks that may be taken, but one of the validation requirements is that no two breaks may overlap.

Here's how I implemented this validation check.

<!--more-->

## First the Form

So let's start off with a form, that allows you to enter time worked, as well as add or remove breaks dynamically.

<iframe style="width:100%; height:400px; border:solid #4173A0 1px; margin: 15px 0;" src="http://jsfiddle.net/hibbard_eu/ULVmA/embedded/result,js,html,css/light/" frameborder="0"></iframe>

This is fairly standard stuff.

As you can see by clicking the JavaScript tab, I start off using `clone()` to clone the original select elements into a template.

After that, every time the _add_ link is clicked, I insert the template before the add/remove row using `insertBefore()`.

And when the user clicks _remove_, I remove the last occurrence of the break row using the `remove()` method.

## Creating Something We Can Compare

With this in place we need to turn our thoughts to how we can make the periods represented by a start and an end time comparable.

The best way to do this is to turn each time into an amount of minutes.

So, for example a break from 09:15 to 09:45 would be:

Start time: 9 x 60 + 15 = 555

End time: 9 x 60 + 45 = 585

It would also help to have a reference to the row in question. This is in case we need to mark it as an error if the times it contains overlap.

That means that when the form is submitted, we need to construct an array of arrays, thus:

```js
[
  [elem, start, end], [elem, start, end], [elem, start, end] ...
]
```

To implement this, I'll write a helper function that gets a reference to the select elements in question, then a second function that returns an array with the appropriate values.

Finally, in the submit handler, I'll add a reference to the containing row to the array using `unshift()`.

This is what that gives us. I'm alerting the values this produces to give us some visual feedback that things are working.

<iframe style="width:100%; height:400px; border:solid #4173A0 1px;" src="http://jsfiddle.net/hibbard_eu/ULVmA/3/embedded/result,js,html,css/light/" frameborder="0"></iframe>

## Making the Comparison

The function to determine whether ranges intersect actually quite simple:

```js
function intersect(x1,x2,y1,y2){
  return x1 < y2 && y1 < x2;
}
```

You can find a more in-depth discussion of it here: [What's the most efficient way to test two integer ranges for overlap?](http://stackoverflow.com/questions/3269434/whats-the-most-efficient-way-to-test-two-integer-ranges-for-overlap "Given two inclusive integer ranges [x1:x2] and [y1:y2], where x1 <= x2 and y1 <= y2, what is the most efficient way to test whether there is any overlap of the two ranges?")

We now need to iterate over our `breakTimes` array and check each of its elements for overlapping times using this formula.

If any are found, we need to highlight these fields and inform the user.

In the real world, we would need to perform additional checks, such as whether the start and end times of the breaks are chronological, or if the break duration is less that the total time worked.

For the purposes of this demo however, they have been omitted.

<iframe style="width:100%; height:400px; border:solid #4173A0 1px; margin: 15px 0;" src="http://jsfiddle.net/hibbard_eu/ULVmA/5/embedded/result,js,html,css/light/" frameborder="0"></iframe>

And there we have it – a fully functioning demo. You can test it out by entering any two breaks that intersect with each other, then pressing submit.

I hope this proved useful for people. If you have any questions, I'd be glad to hear them in the comments.
