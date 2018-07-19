---
title: Simple Ajax Unit Converter
layout: post
permalink: /simple-ajax-unit-converter/
tags:
  - ajax
  - javascript
  - jquery
  - php
excerpt_separator: <!--more-->
old-comments: simple-ajax-unit-converter.html
comments: false
---

I've been doing quite a bit with jQuery and Ajax as of late and have not only been impressed at the ease of implementation, but also the snappy and responsive feel that it brings to your web apps.

To demonstrate this I'm going to show you how to make a simple unit converter, which will allow you to convert units of digital storage without having to make a single page refresh.

<!--more-->

Here's [a demo of what we'll end up with](http://hibbard.eu/demos/unit-converter/ "A demo of the finished unit converter").

## First, the Form

The first thing we'll need is a form. It should have a text input field into which the user can enter the amount of units they want to convert, as well as two drop-downs, from which they can select the unit to convert from and the unit to convert into

Here's the mark-up for that, with a minimum amount of styling applied. Note the `<p>` element below the form which will be used to display the result of the conversion.

```html
<!DOCTYPE HTML>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Ajax Unit Converter</title>
    <style>
      #myForm div{ margin-bottom:15px; }
      #result{
        border:solid 1px green;
        background: #6F9;
        margin-top:25px;
        padding:7px;
        display:none;
      }
    </style>
  </head>
  <body>
    <h1>Digital Storage Converter</h1>
    <form action="convert.php" method="post" id="myForm">
      <div>
        <label for="amount">Amount:</label>
        <input type="text" id="amount" name="amount" />
      </div>
      <div>
        <label for="from">From:</label>
        <select name="from" id="from">
          <option value="0" selected="selected">Select unit</option>
          <option value="1">kilobytes</option>
          <option value="2">megabytes</option>
          <option value="3">gigabytes</option>
          <option value="4">terabytes</option>
          <option value="5">petabytes</option>
          <option value="6">exabytes</option>
        </select>
        <label for="into">Into:</label>
        <select name="into" id="into">
          <option value="0" selected="selected">Select unit</option>
          <option value="1">kilobytes</option>
          <option value="2">megabytes</option>
          <option value="3">gigabytes</option>
          <option value="4">terabytes</option>
          <option value="5">petabytes</option>
          <option value="6">exabytes</option>
        </select>
      </div>
      <input type="submit" value="Convert">
    </form>

    <p id="result"></p>
  </body>
</html>
```

## Then the jQuery

Having included the latest version of the jQuery library in the foot of our document, we can attach an event listener to the form, which will fire when the form is submitted:

```js
$("#myForm").submit(function(e) {
  e.preventDefault();
  // Do some Ajax magic
});
```

The argument `e` that we pass to the anonymous function is a reference to the submit event. By writing `e.preventDefault();` we can prevent our form's default submit behaviour.

Next, let's get a reference to some variables and implement some basic error checking:

```js
var amount = $("#amount").val();
var from = $("#from").val();
var into = $("#into").val();
if (amount === "" || from === "0" || into === "0"){
alert("Please fill out all of the fields!");
  return false;
}
```

Here we get the value of the three form elements. If the user has left the amount blank or has not selected either conversion unit, we display an alert and stop the execution of any further JavaScript.

Finally comes the Ajax:

```js
$.ajax({
  type : "POST",
  url : "convert.php",
  data : 'amount=' + amount + '&from=' + from + '&into=' + into,
  success : function(res) {
    $("#result").html(res);
  }
});
```

To perform the Ajax call I'm using jQuery's [.Ajax()](http://api.jquery.com/jQuery.Ajax/ "jQuery's API documentation - .Ajax()") method. There's quite a lot going on here, so take a minute to understand everything.

First off, we are telling jQuery to submit the form data via a POST request.

We specify where the data should be sent (in this case a PHP script in the same folder entitled `convert.php`) and we also specify what the data should be. Note that the second, third and subsequent parameters need to be prefixed with an `&`.

Finally we are declaring a success callback. This is an anonymous function that is to be run upon the successful completion of the request. In our case we are inserting the answer from the PHP script into the `<p>` element below the form.


## Lastly, the PHP

Let's start off by retrieving the data which was submitted by the jQuery script, from the `$_POST` superglobal array

```js
$amount = $_POST['amount'];
$from = $_POST['from'];
$into = $_POST['into'];
```

Now we need to work out how to convert one unit to another.

We can start by ascertaining  that one kilobyte is 1024 bytes, or two to the power of ten bytes (2^10).

Consequently, one megabyte is 2^10 kilobytes, one gigabyte is 2^20 kilobytes, one terabyte is 2^30 kilobytes and so on…

Therefore to convert one unit to another we need to know which power to raise it to, then we need to multiply it by the amount of units to be converted.

I don't want to go too deeply into the maths at this point, but that works out as this:

```js
$step = $from - $into;
echo pow( (2), $step*10 ) * $amount;
```

And that's all there is to it. As we are echoing the result of the calculation, it is picked up as the parameter `res` in our success callback and can be inserted into the correct position on the page.

## Some Final Touches

Here's a complete listing of the code. I've added a little formatting to the output of the result.

And don't forget, you can see [the script running here](http://hibbard.eu/demos/unit-converter/ "A demo of the finished unit converter").

```html
<!DOCTYPE HTML>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Ajax Unit Converter</title>
    <style>
      #myForm div{ margin-bottom:15px; }
      #result{
        border:solid 1px green;
        background: #6F9;
        margin-top:25px;
        padding:7px;
        display:none;
      }
    </style>
  </head>

  <body>
    <h1>Digital Storage Converter</h1>
    <form action="convert.php" method="post" id="myForm">
      <div>
        <label for="amount">Amount:</label>
        <input type="text" id="amount" name="amount" />
      </div>
      <div>
        <label for="from">From:</label>
        <select name="from" id="from">
          <option value="0" selected="selected">Select unit</option>
          <option value="1">kilobytes</option>
          <option value="2">megabytes</option>
          <option value="3">gigabytes</option>
          <option value="4">terabytes</option>
          <option value="5">petabytes</option>
          <option value="6">exabytes</option>
        </select>
        <label for="into">Into:</label>
        <select name="into" id="into">
          <option value="0" selected="selected">Select unit</option>
          <option value="1">kilobytes</option>
          <option value="2">megabytes</option>
          <option value="3">gigabytes</option>
          <option value="4">terabytes</option>
          <option value="5">petabytes</option>
          <option value="6">exabytes</option>
        </select>
      </div>
      <input type="submit" value="Convert">
    </form>

    <p id="result"></p>

    <script src="http://code.jquery.com/jquery-latest.min.js"></script>
    <script>
      $(document).ready(function() {
        $("#myForm").submit(function(e) {
          e.preventDefault();
          var amount = $("#amount").val();
          var from = $("#from").val();
          var into = $("#into").val();
          if (amount === "" || from === "0" || into === "0"){
            alert("Please fill out all of the fields!");
            return false;
          }

          $.ajax({
            type : "POST",
            url : "convert.php",
            data  : 'amount=' + amount +
                    '&from=' + from +
                    '&into=' + into,
            success : function(res) {
              $("#result").html(
                amount
                + " "
                + $('#from option:selected').text()
                + " = "
                + res
                + " "
                + $('#into option:selected').text()
              )
              .css("display", "inline-block");
            }
          });
        });
      });
    </script>
  </body>
</html>
```

```php
<?php
  $amount = $_POST['amount'];
  $from = $_POST['from'];
  $into = $_POST['into'];

  $step = $from - $into;
  echo pow((2),$step*10)*$amount;
?>
```
