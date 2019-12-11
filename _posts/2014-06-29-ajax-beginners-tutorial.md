---
title: Ajax Beginner's Tutorial
layout: post
permalink: /ajax-beginners-tutorial/
tags:
  - ajax
  - javascript
  - jquery
  - php
excerpt_separator: <!--more-->
old-comments: ajax-beginners-tutorial.html
comments: false
---

Ajax stands for Asynchronous JavaScript and XML and is used for allowing the client side of an application to communicate with the server side of an application.

This might be necessary in order to update the contents of a drop down menu, or to check the availability of a user name, all without reloading the whole page.

Using Ajax isn't very hard and in this tutorial I'll show you how to get up and running.

<!--more-->

## Bring on the jQuery

Although not strictly necessary, I always recommend using the [jQuery library](http://jquery.com/ "jQuery library") when dealing with Ajax. It simplifies the syntax considerably and covers various edge cases behind the scenes.

That said, let's begin. First of all, we'll need a simple HTML page which will submit an Ajax request to a PHP script when a button is pressed.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>AJAX demo</title>
  </head>
  <body>
    <input type="text" id="myTextField" />
    <button id="ajax">Submit AJAX request</button>

    <script src="https://code.jquery.com/jquery-latest.min.js"></script>
      $("#ajax").on("click", function(){
        var text = $("#myTextField").text();
        $.ajax({
          type: "POST",
          url: "submit.php",
          cache: false,
          data: { "value" : text },
          success: function(response){
            alert("The server says: " + response);
          },
          error: function(response){
            alert ("Ajax Error");
            console.log(response);
          }
        });
      });
    </script>
  </body>
</html>
```

As you can see, when the button is clicked, the contents of the text input are read into the variable called `text`.

After that a POST request is sent to submit.php. This request will pass the contents of the `text` variable to the PHP script. We also tell the browser not to cache the response.

You might take a minute to notice that the `data` attribute accepts an object literal (denoted by the `{}` syntax). This object literal can contain as many key/value pairs as desired.

The `success` attribute accepts a function which will be executed if the Ajax request completes successfully. Inside this function, the variable `response` will contain the server's response.

The `error` attribute also accepts a function which will be executed when the Ajax request has completed. This is useful for debugging purposes.


## On to the PHP

In our PHP script, we will extract the variable submitted via the form and simply return it to the client-side script:

```php
$value = $_POST["value"];
echo "I received '" . $value . "'";
```

The JavaScript will have access to this within the `success` callback and can alert it to the user.

In the real world, you would use this value to extract data from a database, or something similar, but for the purposes of this tutorial, I'm going to leave it here.

And that's how Ajax works. Simple.

You can try out our little demo for yourself [here](http://hibbard.eu/demos/basic-ajax-example/ "Demo fo our basic Ajax script").

I hope this proved useful for people. If you have any questions, I'd be glad to hear them in the comments.
