---
title: Promises with jQuery
layout: post
permalink: /promises-with-jquery/
tags:
  - javascript
  - jquery
  - promises
excerpt_separator: <!--more-->
comments: false
---

Promises are starting to change the way we write asynchronous JavaScript. They allow us to write clearer, shorter callbacks and to separate application logic from DOM interaction.

jQuery's implementation of the Promise pattern (introduced in version 1.5) centres around the [Deferred object](http://api.jquery.com/category/deferred-object/ "jQuery API docs: Deferred Object") , the predominate use of which is to attach success and failure callbacks to Ajax requests.

<!--more-->

But the fun doesn't stop there! Promises and Deferred objects aren't just limited to AJAX requests and can be employed throughout your code to make it  more expressive and easier to maintain.

## A Simple Example

You create a new Deferred object like so:

```js
var deferred = new $.Deferred();
```

This is a Promise with methods that allow its owner to resolve or reject it.

We can then create a "pure" Promise by calling the Deferred's `promise()` method.

```js
var promise = deferred.promise();
```

The resultant Promise is identical to the Deferred, except that the `resolve()` and `reject()` methods are missing.

This is important for purposes of encapsulation, for example if you wish to return a Promise from a function, and only allow the caller to read its state or to attach callbacks to it.

This would be done like so:

```js
function successCallback(){
  console.log("This will run if this Promise is resolved.");
}

function failCallback(){
  console.log("This will run if this Promise is rejected.");
}

function alwaysCallback(){
  console.log("And this will run either way.");
}

promise.done(successCallback);
promise.fail(failCallback);
promise.always(alwaysCallback);
```

Finally we can resolve or reject the Deferred object:

```js
deferred.resolve();
```

Outputs:

```sh
This will run if this Promise is resolved.
And this will run either way.
```

Whereas:

```js
deferred.reject();
```

Outputs:

```sh
This will run if this Promise is resolved.
And this will run either way.
```

It is also worth noting, that there is a shorthand for attaching the success and failure callbacks using `.then()`.

```js
promise.then(successCallback, failCallback);
```

## Talking of .then()

Deferred objects start to get interesting when you realise that you can filter the status and values of a Deferred through a function, using the method [.then()](http://api.jquery.com/deferred.then/ "jQuery API docs: deferred.then()")

This method replaces the now-deprecated `deferred.pipe()` method.

Let's see it in action:

```js
function successCallback(msg){
  return msg;
}

function failCallback(msg){
  return msg;
}

var def = new $.Deferred();
var newDef = def.then(successCallback, failiureCallback);

newDef.always(function(retValue){
  console.log("I was called with " + retValue);
});

def.resolve("Success!");
```

Now when we resolve our original Deferred object, the success callback will be invoked and is passed any parameters received by the `.resolve()` method (a string containing 'Success!' in this case).

The Deferred object now returns a resolved Promise, which is assigned to the variable `newDef`, following which the always callback which we attached to newDef can fire.

The always callback receives the return value of whichever callback our original Deferred invoked and can output its message accordingly.

In this case: `I was called with Success!`

## Introducing $.when()

Before looking at a real world example of when and how this might be useful, we need to spend a minute understanding the [$.when()](http://api.jquery.com/jQuery.when/ "jQuery API docs: jQuery.when()") method.

In its simplest form, this method provides a way to execute callback functions based on one or more Deferred objects that represent asynchronous events.

If a single Deferred is passed to `$.when()`, the Deferred's Promise object is returned by the method.

If it is passed multiple Deferred objects, it returns a new "master" promise which:

- will be resolved when _all_ of the given Promises are resolved
- will be rejected when _any_ of the given Promises are rejected.

Here's an example:

```js
function randomResolve(obj){
 var i = Math.random();

 if (i<0.1){
   console.log(obj.name + " resolved with " + i);
   obj.def.resolve();
 } else {
   setTimeout( function(){randomResolve(obj)}, 500);
 }

 return obj.def.promise();
}

var d1 = new $.Deferred(),
    d2 = new $.Deferred();

// Asynchronous events
randomResolve({name: "d1", def: d1});
randomResolve({name: "d2", def: d2});

$.when(d1, d2).then(function(){console.log("Both resolved")});
```

In the above code, the Deferred objects are resolved at random, but the `$.when()` method will wait for both of them before it fires.

## A Real World Example

I recently helped someone implement a countdown script where, when the timer reached zero, a song started playing.

Of course you could do this with callbacks, but it seemed too good an opportunity to pass up to use Deferred objects.

First our HTML:

```html
<button id="myButton">Start Countdown</button>
<div id="count"></div>
```

Now we need to attach some behaviour to the button, so that when it is clicked, it starts a countdown:

```js
function countdown(number){
  $("#count").html(number);

  if (number === 0){
    // Countdown finished
  } else {
    number -= 1;
    window.setTimeout(function() {
      countdown(number);
    }, 1000);
  }
}

$("#myButton").on("click", function(){
  countdown(5);
});
```

So far, so good.. Now what I would like to do is to use $.when() to wait for the countdown to finish, then to start a song playing.

To do this we will need to create a Deferred object and have the countdown method resolve it on completion, then return its promise.

```js
function countdown(number){
  $("#count").html(number);

  if (number === 0){
    d.resolve();
  } else {
   number -= 1;
   window.setTimeout(function() {
     countdown(number);
   }, 1000);
 }

 return d.promise();
}

$("#myButton").on("click", function(){
  d = new $.Deferred();
  $.when(countdown(10)).then(function() {
    playSong("clip.mp3");
  });
});
```

Now all we need is a function to create an audio element on the fly, set its src attribute acordingly, then start it playing:

```js
function playSong(src){
  var audioElement = document.createElement('audio');
  audioElement.setAttribute('src', src);
  audioElement.setAttribute('autoplay', 'autoplay');
  audioElement.play();
}
```

This works already, but as you will maybe notice, there is nothing to stop the user hitting play multiple times.

This would result in multiple countdowns and lots of different audio elements, playing at different intervals.

Therefore as a final touch I have added a function `waitForAudioToFinish()` which takes advantage of the fact that an audio element has two states – paused and not paused. It checks this state at half second intervals and when the track has finished, resolves a second Deferred object, allowing us to chain a second `.then()` method call to the first.

You can find the demo here (sound clip licensed under the Public Domain).

You can [find the demo here](https://hibbard.eu/demos/countdown-sound/ "Using jQuery Deferred to chain asynchronous tasks") ([sound clip](http://soundbible.com/1686-Appear.html "Appear Sound") licensed under the Public Domain).

The complete code is listed below:

```html
<!DOCTYPE HTML>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Play sound after countdown</title>
    <style>
      #count{
        color: blue;
        font-size: 70px;
        padding: 15px 0 0 30px;
      }
    </style>
  </head>

  <body>
    <button id="myButton">Start Countdown</button>
    <div id="count"></div>

    <script src="http://code.jquery.com/jquery-latest.min.js"></script>
    <script>
      function countdown(number){
        $("#count").html(number);

        if (number === 0){
          d.resolve();
        } else {
          number -= 1;
          window.setTimeout(function() {
            countdown(number);
          }, 1000);
        }
        return d.promise();
      }

      function waitForAudioToFinish(audioElement){
        if (!audioElement.paused){
          setTimeout(function(){
            waitForAudioToFinish(audioElement);
          }, 500);
        } else {
          d1.resolve();
        }
      }

      function playSong(src){
        var audioElement = document.createElement('audio');
        audioElement.setAttribute('src', src);
        audioElement.setAttribute('autoplay', 'autoplay');
        audioElement.play();
        waitForAudioToFinish(audioElement);
        return d1.promise();
      }

      $("#myButton").on("click", function(){
        window.d = new $.Deferred();
        window.d1 = new $.Deferred();
        $(this).prop("disabled", true);
        $.when(countdown(5)).then(function() {
          $("#count").empty();
          playSong("clip.mp3")
            .then(function() {
              $("#myButton").prop("disabled", false);
            });
        });
      });
    </script>
  </body>
</html>
```

### Further reading

- [Understanding JQuery.Deferred and Promise](http://joseoncode.com/2011/09/26/a-walkthrough-jquery-deferred-and-promise/ "Jose on Code!")
- [Wrangle Async Tasks With JQuery Promises](http://net.tutsplus.com/tutorials/javascript-ajax/wrangle-async-tasks-with-jquery-promises/ "nettuts+")
- [Promise-Based Validation](http://net.tutsplus.com/tutorials/javascript-ajax/promise-based-validation/ "nettuts+")
- [How do I animate in jQuery without stacking callbacks?](http://stackoverflow.com/questions/10370298/how-do-i-animate-in-jquery-without-stacking-callbacks "Promise-based animation")
