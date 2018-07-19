---
title: Using Cookies with jQuery to Make a Simple Style Switcher
layout: post
permalink: /using-cookies-with-jquery-to-make-a-simple-style-switcher/
tags:
  - css
  - cookies
  - javascript
  - jquery
old-comments: using-cookies-with-jquery-to-make-a-simple-style-switcher.html
comments: false
---

Short for **H**yper**T**ext **T**ransfer **P**rotocol, HTTP is the underlying protocol of the World Wide Web. It is used when a browser requests a web page from a server and it is used when the server responds to the browser's request. HTTP is also stateless, which means that once a server has sent a page to a browser, it forgets having done so straight away. This can be a problem, for example, if a user has identified themselves in order to access protected pages, or if a user wants to customize some aspect of a website, as the server simply cannot remember a thing about it!

## Sounds Like a Job for Cookies!

Invented by Netscape in 1994, cookies are small text files which are stored on a user's computer. They are designed to hold a modest amount of information specific to a particular client and website, and can be accessed either by the server or the client. A typical cookie will contain:

- A name-value pair containing the actual data
- An expiry date after which it is no longer valid
- The domain and path of the server it should be sent to

JavaScript doesn't offer any native support for cookies, but nonetheless there are plenty of good scripts out there that can add this functionality to your web pages. Here's one of my favourites: [http://www.quirksmode.org/js/cookies.html](http://www.quirksmode.org/js/cookies.html "All about cookies")

And the same goes for jQuery. Cookie support is not part of the jQuery core, but instead requires a plugin: [https://github.com/carhartl/jquery-cookie](https://github.com/carhartl/jquery-cookie "A simple, lightweight jQuery plugin for reading, writing and deleting cookies")

Today I want to demonstrate how to use this plugin and with a short, but practical example demonstrate how useful cookies can be.

---

## Three Basic Cookie Operations

### Setting a cookie

Setting a cookie is as easy as this:

```js
$.cookie("demonstration", "foobar");
```

This will create a cookie called "demonstration" with a value of "foobar", which will  last for the duration of the session (and will be destroyed when the user exits their browser).

To increase the duration of the cookie, do this:

```js
$.cookie("demonstration", "foobar", { expires:30 });`
```

This will make the cookie valid for thirty days.

You can also limit the cookie to a specific path on your domain:

```js
$.cookie("demonstration", "foobar", { expires:30, path: '/admin' });
```

Or make it valid across your entire site:

```js
$.cookie("demonstration", "foobar", { expires:30, path: '/' });
```

### Reading a cookie

This is a no brainer. The following will log the value of our cookie to the console:

```js
console.log($.cookie("demonstration"));
```

### Deleting a cookie

This is also very easy. To delete a cookie, just set its value to null.

```js
$.cookie("demonstraion", null);
```

### Additional options

To be complete, there are two additional options that I didn't cover above. Namely, that of `domain` and that of `secure`.

Domain is the value of the domain attribute of the cookie and can be set thus: `{ domain: 'jquery.com' }`. If not set, this will default to the domain of page that created the cookie.

Secure states whether the cookie transmission will require a secure protocol. It can be set thus: `{ secure: true }`. If not set, this will default to false.

### Two gotchas

Cookies won't work locally on some browsers (e.g. Chrome) unless you specifically enable them (e.g. with the command line flag [--enable-file-cookies](http://www.ericdlarson.com/misc/chrome_command_line_flags.html  "Google Chrome Command Line Switches"))

You can't delete a cookie by setting it to an empty string. This will just delete the value, not the cookie.

---

## A Practical Example – a Basic Style Switcher:

So, now we know how to set, read and delete cookies, here's a real world example – a basic style switcher. In this example we'll create two select elements with which the user can select the background and the text clour independently of each other. Here's the HTML:

```html
<p>Select the background colour:
  <select class="selector">
    <option value="background-default">Default</option>
    <option value="background-blue">Blue</option>
    <option value="background-green">Green</option>
    <option value="background-red">Red</option>
  </select>
</p>

<p>Select the text colour:
  <select class="selector">
    <option value="color-default">Default</option>
    <option value="color-yellow">Yellow</option>
    <option value="color-white">White</option>
    <option value="color-purple">Purple</option>
  </select>
</p>

<button id="reset">Reset ALL the styles</button>
```

The idea here is to attach an onChange event to the two select elements so that whenever they change a different class is applied to the document's `<body>` element.

Here's the CSS we'll use to apply the styles. As you can see the class names map to the values of the `<option>` tags:

```css
body { color:#000; background:#fff; }
.background-blue { background:blue; }
.background-red { background:red; }
.background-green { background:green;}
.color-yellow, .color-yellow a { color:yellow; }
.color-white, .color-white a { color:white; }
.color-purple, .color-purple a { color:purple }
```

Now for the onChange handlers. This is quite easy – all we'll do is call a custom function passing in the selected value every time the menu changes:

```js
$(".selector").on('change', function(){
  changeStyles(this.value);
});
```

Defining the `changeStyles` function gets a little trickier. Here we're going to want to split the value passed in at the minus sign, so that, for example "background-blue" would give us "background" and "blue". This'll make it easier if any styles have already been applied to know which one to overwrite. Then we're going to want to add our new class to the body and store it in a cookie.

```js
function changeStyles(optValue){
  var property = optValue.split("-")[0];
  removeOldClass(property);
  $("body").addClass(optValue);
  $.cookie('bodyClassList', $("body").attr('class'));
}

function removeOldClass(property){
  var classList = $("body").attr('class').split(/\s+/);
  $.each(classList, function(index, item){
    if (item.match(property)){
      $("body").removeClass(item);
    }
  });
}
```

I've put the logic for removing the current body class into its own function.

Here I get a list of all of the classes that are currently applied to the `<body>` element, iterate over them and see if they match the class which the user wants to apply. If so, they are removed.

Now let's give the user the possibility of resetting the page to its default style. We can do this by deleting our `bodyClassList` cookie when the user clicks the "Reset" button.

```js
$("#reset").click(function(){
  $("body").removeClass();
  $(".selector").each(function(){
    $(this[0]).attr('selected', true);
  })
  $.cookie('bodyClassList', null);
});
```

Now all we need to do is reapply to the `<body>` element, any classes we might have stored in our cookie, when a user navigates from one page to the next. For good measure we can also make sure that whatever style the user has selected previously, is also pre-selected in the drop-down menu when they navigate to a new page.

```js
$(".selector &gt; option").each(function() {
  if($.cookie('bodyClassList') != null &&
     $.cookie('bodyClassList').match($(this).val())){
       $(this).attr("selected","selected");
  }
});

if($.cookie('bodyClassList') != null) {
  $("body").removeClass();
  $("body").addClass($.cookie('bodyClassList'));
}
```

And that's it. Here's the complete code listing:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>Style Changer</title>
    <style>
      body { color:#000; background:#fff; }
      .background-blue { background:blue; }
      .background-red { background:red; }
      .background-green { background:green;}
      .color-yellow, .color-yellow a { color:yellow; }
      .color-white, .color-white a { color:white; }
      .color-purple, .color-purple a { color:purple }
    </style>
  </head>

  <body class="background-default">
    <h1>Page 1</h1>

    <p>Select the background colour:
      <select class="selector">
        <option value="background-default">Default</option>
        <option value="background-blue">Blue</option>
        <option value="background-green">Green</option>
        <option value="background-red">Red</option>
      </select>
    </p>

    <p>Select the text colour:
      <select class="selector">
        <option value="color-default">Default</option>
        <option value="color-yellow">Yellow</option>
        <option value="color-white">White</option>
        <option value="color-purple">Purple</option>
      </select>
    </p>

    <button id="reset">Reset ALL the styles</button>

    <script src="http://code.jquery.com/jquery-latest.min.js"></script>
    <script src="jquery.cookie.js"></script>
    <script>
      $(document).ready(function() {
        function changeStyles(optValue){
          var property = optValue.split("-")[0];
          removeOldClass(property);
          $("body").addClass(optValue);
          $.cookie('bodyClassList', $("body").attr('class'));
        }

        function removeOldClass(property){
          var classList = $("body").attr('class').split(/\s+/);
          $.each(classList, function(index, item){
            if (item.match(property)){
              $("body").removeClass(item);
            }
          });
        }

        $(".selector").change(function(){
          changeStyles(this.value);
        });

        $(".selector > option").each(function() {
          if(
              $.cookie('bodyClassList') != null &&
              $.cookie('bodyClassList')
               .match($(this).val())
            ){
            $(this).attr("selected","selected");
          }
        });

        $("#reset").click(function(){
          $("body").removeClass();
          $(".selector").each(function(){
            $(this[0]).attr('selected', true);
          })
          $.cookie('bodyClassList', null);
        });

        if($.cookie('bodyClassList') != null) {
          $("body").removeClass();
          $("body").addClass($.cookie('bodyClassList'));
        }
      });
    </script>
  </body>
</html>
````

### References / Useful resources

- [http://stackoverflow.com/questions/335244/why-does-chrome-ignore-local-jquery-cookies](http://stackoverflow.com/questions/335244/why-does-chrome-ignore-local-jquery-cookies "Stack Overflow - Why does Chrome ignore local cookies")
- [http://www.electrictoolbox.com/jquery-cookies/](http://www.electrictoolbox.com/jquery-cookies/ "Useful tutorial on setting cookies with jQuery")
- [http://stackoverflow.com/questions/1458724/how-to-set-unset-cookie-with-jquery](http://stackoverflow.com/questions/1458724/how-to-set-unset-cookie-with-jquery "A discussion of cookies with and without jQuery on Stack Overflow")
