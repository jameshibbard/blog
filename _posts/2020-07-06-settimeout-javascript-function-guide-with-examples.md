---
title: "setTimeout JavaScript Function: Guide with Examples"
layout: post
permalink: /javascript-settimeout-function-examples/
tags:
  - jquery
  - javascript
  - setTimeout
excerpt_separator: <!--more-->
canonical:
  url: 'https://www.sitepoint.com/javascript-settimeout-function-examples/'
---

<p class="callout originally-posted-elsewhere">This article was <a href="https://www.sitepoint.com/javascript-settimeout-function-examples/">first published on SitePoint</a> and has been republished here with permission.</p>

`setTimeout` is a native JavaScript function (although it can be used with a library such as jQuery, as we’ll see later on), which calls a function or executes a code snippet after a specified delay (in milliseconds). This might be useful if, for example, you wished to display a popup after a visitor has been browsing your page for a certain amount of time, or you want a short delay before removing a hover effect from an element (in case the user accidentally moused out).

<!--more-->

<img class="aligncenter size-full wp-image-141586" src="https://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/08/1476830355set_timeout_2-01.png" alt="set_timeout_2-01" width="900" height="500" loading="lazy"/>

## Basic setTimeout Example

To demonstrate the concept, the following demo displays a popup, two seconds after the button is clicked.

<p class="codepen" data-height="300" data-theme-id="6441" data-slug-hash="Ejxvjr" data-default-tab="result" data-user="SitePoint" data-pen-title="Delayed Magnific Popup modal" data-preview="true" style="margin-top: 55px;">See the Pen <a href="https://codepen.io/SitePoint/pen/Ejxvjr/">Delayed Magnific Popup modal</a> by SitePoint (<a href="https://codepen.io/SitePoint">@SitePoint</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async="" src="https://static.codepen.io/assets/embed/ei.js"></script>

If you don’t see the popup open, please visit [CodePen](http://codepen.io/SitePoint/pen/Ejxvjr/) and run the demo there.

## Syntax

From the [MDN documentation](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout), the syntax for `setTimeout` is as follows:

```js
var timeoutID = scope.setTimeout(function[, delay, arg1, arg2, ...]);
var timeoutID = scope.setTimeout(function[, delay]);
var timeoutID = scope.setTimeout(code[, delay]);
```

where:

- `timeoutID` is a numerical ID, which can be used in conjunction with [clearTimeout](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/clearTimeout) to cancel the timer.
- `scope` refers to the [Window interface](https://developer.mozilla.org/en-US/docs/Web/API/Window) or the [WorkerGlobalScope interface](https://developer.mozilla.org/en-US/docs/Web/API/WorkerGlobalScope).
- `function` is the function to be executed after the timer expires.
- `code` is an alternative syntax that allows you to include a string instead of a function, which is compiled and executed when the timer expires.
- `delay` is the number of milliseconds by which the function call should be delayed. If omitted, this defaults to 0.
- `arg1, ..., argN` are additional arguments passed to the function specified by `function`.

_Note: the square brackets `[]` denote optional parameters._

### setTimeout vs window.setTimeout

You’ll notice that the syntax above uses `scope.setTimeout`. Why is this?

Well, when running code in the browser, `scope` would refer to the global `window` object. Both `setTimeout` and `window.setTimeout` refer to the same function, the only difference being that in the second statement we are referencing the `setTimeout` method as a property of the `window` object.

In my opinion, this adds complexity for little or no benefit. If you’ve defined an alternative `setTimeout` method which would be found and returned in priority in the scope chain, then you’ve probably got bigger problems to worry about.

For the purposes of this tutorial, I’ll omit `window`, but ultimately, which syntax you choose is up to you.

## Examples of Use

`setTimeout` accepts a reference to a function as the first argument.

This can be the name of a function:

```js
function greet(){
  alert('Howdy!');
}
setTimeout(greet, 2000);
```

A variable that refers to a function (a function expression):

```js
const greet = function(){
  alert('Howdy!');
};
setTimeout(greet, 2000);
```

Or an anonymous function:

```js
setTimeout(() => { alert('Howdy!'); }, 2000);
```

As noted above, it’s also possible to pass `setTimeout` a string of code for it to execute:

```js
setTimeout('alert("Howdy!");', 2000);
```

However, this is not advisable for the following reasons:

- It’s hard to read (and thus hard to maintain and/or debug).
- It uses an implied `eval`, which is a potential security risk.
- It’s slower than the alternatives, as it has to invoke the JS interpreter.

This [Stack Overflow question](http://stackoverflow.com/questions/6081560/is-there-ever-a-good-reason-to-pass-a-string-to-settimeout) offers more information on the above points.

## Passing Parameters to setTimout

In a basic scenario, the preferred, cross-browser way to pass parameters to a callback executed by `setTimeout` is by using an anonymous function as the first argument.

In the following example, we select a random animal from an `animals` array and pass this random animal as a parameter to a `makeTalk` function. The `makeTalk` function  is then executed by `setTimeout` with a delay of one second:

```js
function makeTalk(animal){
  const noises = {
    cat: 'purr',
    dog: 'woof',
    cow: 'moo',
    pig: 'oink',
  }

  console.log(`A ${animal} goes ${noises[animal]}.`);
}

function getRandom (arr) {
  return arr[Math.floor(Math.random()*arr.length)];
}

const animals = ['cat', 'dog', 'cow', 'pig'];
const randomAnimal = getRandom(animals);

setTimeout(() => {
  makeTalk(randomAnimal);
}, 1000);
```

*Note: I've used a regular function (`getRandom`) to return a random element from an array. It would also be possible to write this as a function expression using an arrow function:*

```js
const getRandom = arr => arr[Math.floor(Math.random()*arr.length)];
```

We’ll get to [arrow functions](https://www.sitepoint.com/arrow-functions-javascript/) in the next section. <a href="https://codepen.io/SitePoint/pen/BaELpRj?editors=1111">Here’s a CodePen that contains the above code</a>.

### An Alternative Method

As can be seen from the [syntax at the top of the article](#syntax), there’s a second method of passing parameters to a callback executed by `setTimeout`. This involves listing any parameters after the delay.

With reference to our previous example, this would give us:

```js
setTimeout(makeTalk, 1000, randomAnimal);
```

Unfortunately, this doesn’t work in IE9 or below, where the parameters come through as `undefined`. If you’re in the unenviable position of having to support IE9, there is [a polyfill available on MDN](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout#Polyfill).

## The Problem with `this`

Code executed by `setTimeout` is run in a separate execution context to the function from which it was called. This is problematic when the context of the `this` keyword is important:

```js
const dog = {
  sound: 'woof',
  bark() {
    console.log(`Rover says ${this.sound}!`);
  }
};

dog.bark();
// Outputs: Rover says woof!

setTimeout(dog.bark, 50);
// Outputs: Rover says undefined!
```

The reason for this output is that, in the first example, `this` points to the `dog` object, whilst in the second example `this` points to the global `window` object (which doesn’t have a `sound` property).

To counteract this problem, there are various measures …

### Explicitly Set the Value of `this`

You can do this using [bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind), a method which creates a new function that, when called, has its `this` keyword set to the provided value (in our case, the `dog` object). This would give us:

```js
setTimeout(dog.bark.bind(dog), 50);
```

_Note: `bind` was introduced in ECMAScript 5, so will only work in [more modern browsers](http://kangax.github.io/compat-table/es5/#Function.prototype.bind). You can read more about it (and other methods of setting the value of `this`) [in this SitePoint article](http://www.sitepoint.com/inner-workings-javascripts-this-keyword#how-this-can-be-successfully-manipulated)._

### Use a Library

Many libraries come with built-in functions to address this issue. For example, jQuery’s [jQuery.proxy()](http://api.jquery.com/jQuery.proxy/) method. This takes a function and returns a new one that will always have a particular context. In our case, that would be:

```js
setTimeout($.proxy(dog.bark, dog), 50);
```

## Using Arrow Functions with setTimeout

Arrow functions were introduced with ES6. They have a much shorter syntax than a regular function:

```js
(param1, param2, paramN) => expression
```

You can, of course, use them with `setTimeout`, but there’s one gotcha to be aware of — namely, that arrow functions don’t have their own `this` value. Instead, they use the `this` value of the enclosing lexical context.

Using a regular function:

```js
const dog = {
  sound: 'woof',
  bark() {
    console.log(`Rover says ${this.sound}!`);
  }
};

dog.bark();
// Rover says woof!
```

Using an arrow function:

```js
const dog = {
  sound: 'woof',
  bark: () => {
    console.log(`Rover says ${this.sound}!`);
  }
};

dog.bark();
// Rover says undefined!
```

In the second example, `this` points to the global `window` object (which again, doesn’t have a `sound` property).

This can trip us up when using arrow functions with `setTimeout`. Previously we saw how we can supply a function called in a `setTimeout` with the correct `this` value:

```js
setTimeout(dog.bark.bind(dog), 50);
```

This won’t work when using an arrow function in the `introduce` method, as the arrow function doesn’t have its own `this` value. The method will still log `undefined`.

### Cleaner Code with Arrow Functions and setTimeout

However, because arrow functions don’t have their own `this` value, it can also work to our advantage.

Consider code like this:

```js
const dog = {
  sound: 'woof',
  delayedBark() {
    setTimeout(
      function() {
        console.log(`Rover says ${this.sound}!`);
      }
      .bind(this)
    , 1000);
  }
}

dog.delayedBark();
```

It can be rewritten more concisely with an arrow function:

```js
const dog = {
  sound: 'woof',
  delayedBark() {
    setTimeout(
      () => { console.log(`Rover says ${this.sound}!`); }, 1000
    );
  }
}

dog.delayedBark();
```

If you’d like a primer on arrow functions, please read “[ES6 Arrow Functions: Fat and Concise Syntax in JavaScript](https://www.sitepoint.com/es6-arrow-functions-new-fat-concise-syntax-javascript/)”.

## Canceling a Timer

As we learned at the start of the article, the return value of `setTimeout` is a numerical ID which can be used to cancel the timer in conjunction with the `clearTimeout` function:

```js
const timer = setTimeout(myFunction, 3000);
clearTimeout(timer);
```

Let’s see this in action. In the following Pen, if you click on the _Start countdown_ button, a countdown will begin. If the countdown completes, the kittens get it. However, if you press the _Stop countdown_ button, the timer will be halted and reset. (If you don’t see a cool effect when the countdown reaches zero, re-run the Pen using the button in the bottom right of the embed.)

<p class="codepen" data-height="450" data-theme-id="6441" data-slug-hash="dPxEPJ" data-default-tab="result" data-user="SitePoint" data-pen-title="SetTimeout Kittens" data-preview="true">See the Pen <a href="https://codepen.io/SitePoint/pen/dPxEPJ/">SetTimeout Kittens</a> by SitePoint (<a href="https://codepen.io/SitePoint">@SitePoint</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async="" src="https://static.codepen.io/assets/embed/ei.js"></script>

## Wrapping Up

One potential caveat to be aware of is the fact that `setTimeout` is asynchronous. It queues the function reference it receives to run once the current call stack has finished executing. It doesn’t, however, execute concurrently, or on a separate thread (due to JavaScript’s single-threaded nature).

```js
console.log(1);
setTimeout(() => { console.log(2); }, 0);
console.log(3);

// Outputs: 1, 3, 2
```

Although we’re calling `setTimeout` with a zero second delay, the numbers are still logged out of order. This is because when `setTimeout`'s timer has expired, the JavaScript engine places its callback function in a queue, _behind_ the other `console.log` statements, to be executed.

If you'd like to learn more about what happens when JavaScript runs, I highly recommend this video from JSConf 2014: [What the heck is the event loop anyway?](https://www.youtube.com/watch?v=8aGhZQkoFbQ)

### requestAnimationFrame()

You should also be aware of [requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame). This method tells the browser that you wish to call a specified function before the next repaint.

When making animations, we should favor `requestAnimationFrame` over using `setTimeout`, as it will fire roughly sixty times a second, as opposed to `setTimeout`, which is called after a minimum of `n` milliseconds. By using `requestAnimationFrame` we can avoid changing something twice between two frame updates.

Here’s an example of how to use `requestAnimationFrame` to animate a `div` element across the screen:

```js
const div = document.querySelector('#rectangle');
let leftPos = 0;

function animateDiv(){
  leftPos += 1;
  div.style.left = `${leftPos}px`;
  if (leftPos < 100) requestAnimationFrame(animateDiv);
}

requestAnimationFrame(animateDiv);
```

You could, of course, achieve the same thing using `setTimeout`:

```js
const div = document.querySelector('#rectangle');
let leftPos = 0;

function animateDiv(){
  leftPos += 1;
  div.style.left = `${leftPos}px`;
  if (leftPos < 100) setTimeout(animateDiv, 1000/60);
}

animateDiv();
```

But as mentioned, using `requestAnimationFrame` offers various advantages, such as allowing the browser to make optimizations and stopping animations in inactive tabs.

<p class="codepen" data-height="346" data-theme-id="6441" data-default-tab="js,result" data-user="SitePoint" data-slug-hash="LYGLmrX" data-preview="true" data-pen-title="Animation with requestAnimationFrame">
  <span>See the Pen <a href="https://codepen.io/SitePoint/pen/LYGLmrX">
  Animation with requestAnimationFrame</a> by SitePoint (<a href="https://codepen.io/SitePoint">@SitePoint</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

### jQuery.delay()

Finally, I'd like to clear up any confusion between the use of the native JavaScript `setTimeout` function and [jQuery’s delay method](https://api.jquery.com/delay/).

The `delay` method is meant specifically for adding a delay between methods in a given jQuery queue. There is no possibility to cancel the delay. For example, if you wanted to fade an image into view for one second, have it visible for five seconds, and then fade it out for a period of one second, you could do the following:

```js
$('img').fadeIn(1000).delay(5000).fadeOut(1000);
```

`setTimeout` is best used for everything else.

_Note: if you need to repeatedly execute code after a specified delay, then `setInterval` is more suited to the job. You can read [more about this function here](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setInterval)._

## Conclusion

In this article, I’ve demonstrated how to use `setTimeout` to delay the execution of a function. I have also shown how to pass parameters to `setTimeout`, maintain the `this` value inside its callback and also how to cancel a timer.

 If you run into a coding problem regarding the use of `setTimeout` (or anything else, really), then please head to the [SitePoint forums](http://community.sitepoint.com/) where we'll be happy to help.

Alternatively, if you have any remarks or questions, I'd love to hear them in the comments below.
