---
title: Overlaying Lightbox with the HTML5 Canvas Element
layout: post
permalink: /overlaying-lightbox-with-the-html5-canvas-element/
tags:
  - html5
  - javascript
  - jquery
  - lightbox
old-comments: overlaying-lightbox-with-the-html5-canvas-element.html
comments: false
---

Here's how to use the HTML5 `<canvas>` element in conjunction with Lokesh Dhakar's excellent Lightbox2 script, to dynamically highlight a part of a static image, in this case individual offices on a company's floor plan.

I was making a "Team" page for a client recently. The purpose of this page was to list the contact details for each member of the client's (not-so-small) team. The client also wanted each team member's contact details to include their office number, which when clicked, would display an image showing a floor plan of their department, with the exact location of the member's office highlighted in red.

The client's idea was to use a different image for each member. Although this would essentially work, it would mean a whole bunch of almost identical images and it didn't really seem like the best solution to me. I therefore decided to take just the one image of the floor plan, display it in a Lightbox and overlay it with a dynamically generated HTML5 canvas element, which would display the correct office location for each member.

## Adding a Lightbox

I started off by downloading Lokesh Dhakar's excellent [Lightbox2](http://lokeshdhakar.com/projects/lightbox2/ "Lightbox2 - Homepage"). I then made a simple script that displayed a member's contact details. I made the member's office number a link, that when clicked, opened the client's floor plan in a Lightbox.

[You can see this in action here](http://hibbard.eu//demos/lightbox-canvas/1/ "Lightbox 2 with <canvas> overlay - Example 1").

After that I needed to write some JavaScript to the following:

  * When the page has loaded, it should check if the browser supports the canvas element
  * If it does, it should find all of the links to the floor plan and attach an event handler to them, so that when they are clicked a specific function fires
  * This function should receive the coordinates of a specific office as parameters
  * It should then create a canvas element
  * Draw a rectangle around the appropriate office
  * Super impose the canvas element on the floor plan, which by this time is being displayed in a Lightbox

The first step is fairly easy. I used jQuery's `$(document).ready` function, as I am including jQuery anyway (Lightbox2 requires it).

```js
function isCanvasSupported(){
  var elem = document.createElement('canvas');
  return !!(elem.getContext && elem.getContext('2d'));
}

$(document).ready(function() {
  if(isCanvasSupported()){
    alert("Yay!");
  }
});
```

I found the function to check for the canvas element here: [http://stackoverflow.com/questions/2745432/best-way-to-detect-that-html5-canvas-is-not-supported](http://stackoverflow.com/questions/2745432/best-way-to-detect-that-html5-canvas-is-not-supported "Best way to detect that HTML5 <canvas> is not supported")

To attach an event listener to all of the links pointing to the floor plan, I decided to wrap them in a div tag, which I gave the class of `highlight_room`, and then use jQuery's `click` method

```js
$(".highlight_room a").on("click", function() {
  // do stuff
});
```

So, now it started getting tricky. I had a few problems generating a canvas element on the fly and getting it to behave as I thought it should, so I cheated and hard-coded it into the page. I set its `display` property to `none` and its background image to that of the floor plan.

```html
<canvas width="750"
        height="548"
        class="plan"
        style="display:none; background-image: url(plan.jpg);">
</canvas>
```

I then fetch the canvas element using jQuery.

```js
var canvas = $(".plan");
```

To get the coordinates of the rectangle to draw on the canvas, I used an image map in DreamWeaver. I highlighted each office individually and took the coordinates from DreamWeaver's output. For example, for an image map covering room 395, DW outputs:

```html
<map name="Map">
  <area shape="rect" coords="158,82,35,65" href="#">
</map>
```

For more about creating image maps in DW, see here: [http://visibleranking.com/create-image-map-in-dreamweaver.php](http://visibleranking.com/create-image-map-in-dreamweaver.php "How to create a clickable image map in Dreamweaver CS4/CS5")

Once I had all of the coordinates I needed, I included them as global variables at the top of my JavaScript:

```js
var room_336 = [409,397,68,36];
var room_339 = [409,302,68,36];
...
var room_595 = [220,82,35,65];
var room_598 = [252,82,36,65];
```

Then, for every link pointing to the floor plan, I added a class to it, making the name of the class dependent on the office to be highlighted.

```html
<a href="images/plan.jpg" rel="lightbox" class="room_593">01/593</a>
```

Finally, I parse this class reference in my JavaScript to point to one of the previously-declared global variables.

```js
var coord = window[$(this).attr("class")];
```

## Drawing on the Canvas

Now I was in a position to draw a rectangle around the appropriate office on the canvas element. I did this with a function called `highlightRoom` to which I passed the four coordinates as parameters. This function also includes a call to `clearCanvas` which pretty much does what you'd expect. I found it here: [http://stackoverflow.com/questions/2142535/how-to-clear-the-canvas-for-redrawing](http://stackoverflow.com/questions/2142535/how-to-clear-the-canvas-for-redrawing "How to clear the canvas for redrawing")

```js
function clearCanvas(context, canvas) {
  context.clearRect(0, 0, canvas.width, canvas.height);
  var w = canvas.width;
  canvas.width = 1;
  canvas.width = w;
}

function highlightRoom(x, y, h, w){
  var can = $('canvas')[0];
  var ctx = can.getContext('2d');
  clearCanvas(ctx, can);
  ctx.strokeStyle = '#f00';
  ctx.lineWidth = 5;
  ctx.lineJoin = 'round';
  ctx.strokeRect(x,y,h,w);
}
```

After this, all that remained to be done now, was to swap out the image of the floor plan with the modified canvas element, whenever the user clicks on a link to a member's office.

To do this we can target the Lightbox2 div with the class of `lb-container` and use jQuery's `html` method to swap out its contents. It is prudent to wait a short while before doing this, so that Lightbox2 has the chance to create these elements, before we try and access their content.

```js
setTimeout(function(){
  clearInterval(imageLoaded);
}, 5000);

imageLoaded = setInterval(function(){
  if(imageReady()){
    $(".lb-image").hide();
    $(".lb-container").prepend(canvas);
    canvas.css("display", "block");
  }
}, 100);
```

Clearing the timer after 5 seconds, guarantees that it won't keep running if the image fails to load.

The finished script looks like this:

```html
<!doctype html>
  <html lang="en-us">
  <head>
  <meta charset="utf-8">
    <title>Lightbox 2 with <canvas> overlay</title>
    <link rel="stylesheet" href="css/lightbox.css" />
    <style>
      .plan{
        display:none;
        background-image: url(images/plan.jpg);
      }
    </style>
  </head>

  <body>
    <div class="highlight_room">
      <p>
        Prof. Dr. E. Wriston<br>
        Head of Department<br>
        Tel.: +44 (0)123 / 45-6789<br>
        Fax: +44 (0)123 / 45-6780 <br>
        Office:
          <a href="images/plan.jpg" rel="lightbox" class="room_593">
            01/593
          </a><br>
      </p>

      <p>
        Prof. Dr. B. Longboard<br>
        Second in command<br>
        Tel.: +44 (0)123 / 45-6789<br>
        Fax: +44 (0)123 / 45-6780 <br>
        Office:
          <a href="images/plan.jpg" rel="lightbox" class="room_336">
            01/336
          </a><br>
      </p>
    </div>

    <canvas width="750" height="548" class="plan"></canvas>

    <script src="js/jquery-3.3.1.min.js"></script>
    <script src="js/lightbox.js"></script>
    <script>
      var room_336 = [409,397,68,36];
      var room_339 = [409,302,68,36];
      var room_340 = [409,269,68,36];
      var room_342 = [409,207,68,36];
      var room_343 = [409,176,68,36];
      var room_348 = [379,82,35,65];
      var room_394 = [585,461,35,65];
      var room_588 = [92,82,35,65];
      var room_591 = [126,82,35,65];
      var room_593 = [158,82,35,65];
      var room_595 = [220,82,35,65];
      var room_598 = [252,82,36,65];
      var imageLoaded;

      function isCanvasSupported(){
        var elem = document.createElement('canvas');
        return !!(elem.getContext && elem.getContext('2d'));
      }

      function clearCanvas(context, canvas) {
        context.clearRect(0, 0, canvas.width, canvas.height);
        var w = canvas.width;
        canvas.width = 1;
        canvas.width = w;
      }

      function highlightRoom(x, y, h, w){
        var can = $('canvas')[0];
        var ctx = can.getContext('2d');
        clearCanvas(ctx, can);
        ctx.strokeStyle = '#f00';
        ctx.lineWidth = 5;
        ctx.lineJoin = 'round';
        ctx.strokeRect(x,y,h,w);
      }

      function imageReady(){
        if ($("img.lb-image").is(":visible")){
          clearInterval(imageLoaded);
        }

        return $("img.lb-image").is(":visible");
      }

      $(document).ready(function() {
        if(isCanvasSupported()){
          $(".highlight_room a").on("click", function() {
            var canvas = $(".plan");
            var coord = window[$(this).attr("class")];
            highlightRoom(coord[0], coord[1], coord[2], coord[3]);

            setTimeout(function(){
              clearInterval(imageLoaded);
            }, 5000);

            imageLoaded = setInterval(function(){
              if(imageReady()){
                $(".lb-image").hide();
                $(".lb-container").prepend(canvas);
                canvas.css("display", "block");
              }
            }, 100);
          });
        }
      });
    </script>
  </body>
</html>
```

You can see it in action here: [example 2](http://hibbard.eu//demos/lightbox-canvas/2/ "Lightbox 2 with <canvas> overlay - Example 2")
