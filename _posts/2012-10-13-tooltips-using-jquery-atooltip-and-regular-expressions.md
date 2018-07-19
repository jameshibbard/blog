---
title: Tooltips Using jQuery, aToolTip and Regular Expressions
layout: post
permalink: /tooltips-using-jquery-atooltip-and-regular-expressions/
tags:
  - javascript
  - jquery
  - regular expressions
  - tooltips
comments: false
---

Imagine you have a page of text and you want each occurrence of a certain word to display a tooltip whenever a visitor hovers over it with their mouse. What's the best way to go about implementing this? Well, you could edit your HTML by hand and add the appropriate tags to every single occurrence of the term. Or, you could just specify your term once and then let jQuery do the work for you.

The first thing we'll need for this demonstration is a HTML document with some random text. Here's one I made earlier (you'll notice I included jQuery for good measure):

```html
<!DOCTYPE HTML>
  <html>
    <meta charset="UTF-8">
    <title>Tooltip demo</title>
    <style>
      #container {
        width:600px; margin: 0 auto; padding-top:50px;
      }
    </style>
  </head>
  <body>
    <div id="container">
      <p>
        Lorem ipsum dolor sit amet, cat consectetur adipisicing elit, sed do eiusmod tempor cat incididunt ut labore
        et dolore magna aliqua. Cat ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip
        ex ea commodo cat consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu cat
        fugiat nulla pariatur cat. Cat Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt
        mollit anim id est laborum cat, cats and cat.
      </p>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
  </body>
</html>
```

Now let's say that we want to display a tooltip for every occurrence of the word "cat" or "Cat" in this page. The first thing we need to do is to get a reference to the div they are stored in. Using jQuery this is as easy as:

```js
$('#container')
```

The thing I love about jQuery is the ability to chain its methods together. So building on the above, we can access (and later set) the contents of the container div, thus:

```js
$('#container').html();
```

Now we can use JavaScript's `replace` method to wrap an anchor tag around each occurrence of the word "cat" in order to display the tooltip. We do this by looking for "cat" and replacing it with `<a href="#" title="Cats are great">cat</a>`.

The syntax of this method call is as follows: `(string).replace(searchValue, newValue)`.

```js
$('#container')
  .html()
  .replace(
    /\b(cat)\b/ig,
    '<a href="#" title="Cats are great!">$1</a>'
  )
```

Now you might be wondering what's going on in the second part of this method call, so let's break it down:

As the `searchValue` for `replace`, we have used a regular expression. This is denoted by the two forward slashed (also called delimiters). Within these two slashes comes the pattern we are searching for, which in our case this is the word "cat" preceeded and followed by a word boundary (denoted by the backslash b). The inclusion of the word boundaries means that we will match "cat," "cat." and "cat", but not "cats" or "scat". We have also put the string we are searching for ("cat") in brackets, which means that we can refer back to the match in the `newValue` part of the method call.

After the second delimiter come two modifiers: `i` and `g`. The `g` modifier stands for "global" and means that we will match and replace all occurrences of the regular expression within the string (as opposed to only the first). The `i` modifiers stands for "case insensitive" and means that we match "cat" as well as "Cat" (and "cAt" too, if we really want).

The `newValue` for `replace`, is hopefully a little more straight forward. It is simply an HTML string and the `$1` variable which refers back to what we matched in the first part of the statement.

So, let's finish this off by replacing the current contents of the container div, with our modified content, which will then display the tooltips.

```js
$('#container')
  .html(
    $('#container')
      .html()
      .replace(
        /(cat)\b/ig,
        '<a href="#" title="Cats are great!">$1</a>'
      )
  );
```

Now this code would already work if you were to run it in your browser and would display the browser's native tooltip.

![Screenshot of the word 'cat' highlighted, displaying a native tooltip](https://res.cloudinary.com/hibbard/image/upload/v1528961806/tooltips_screenshot.png "Screenshot of the word 'cat' highlighted, displaying a native tooltip")

However, as we already have jQuery included in our page, it would be a little more stylish if we were to display our tooltips using a jQuery tooltip library. I'm going to use [aToolTip](https://github.com/ItsMeAra/aToolTip "aToolTip home page"), as it is both stylish and lightweight.

If you are following along at home, you'll have to download the library from the above address. After that you just need to include the relevant JavaScript and CSS files, give your tooltips a class of (for example) `normalTip`, and kick everything off with the following line of code:

```js
$(function(){
  $('a.normalTip').aToolTip();
});
```

Here's everything put together:

```html
<!DOCTYPE HTML>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Tooltip demo</title>
    <link href="css/atooltip.css" rel="stylesheet" />
    <style>
      #container {
        width:600px; margin: 0 auto; padding-top:50px;
      }
    </style>
  </head>

  <body>
    <div id="container">
      <p>
        Lorem ipsum dolor sit amet, cat consectetur adipisicing elit, sed do eiusmod tempor cat incididunt ut labore
        et dolore magna aliqua. Cat ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip
        ex ea commodo cat consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu cat
        fugiat nulla pariatur cat. Cat Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt
        mollit anim id est laborum cat, cats and cat.
      </p>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
    <script type="text/javascript" src="js/jquery.atooltip.min.js"></script>
    <script>
      $(function(){
        $('#container').html(
          $('#container')
            .html()
            .replace(
              /\b(cat)\b/ig,
              '<a href="#" class="normalTip" title="Cats are great!">$1</a>'
            )
          );

        $('a.normalTip').aToolTip();
      });
    </script>
  </body>
</html>
```

And Here's a demo of what [the finished page](http://hibbard.eu/demos/aToolTip/ "The finished tooltip demo") looks like.

**Edit**: here's a useful link to a discussion of the same subject on StackOverflow: [http://stackoverflow.com/questions/2558278/how-to-show-mouseover-tooltip-on-selected-word-on-web-page-using-javascript-con/12707462](http://stackoverflow.com/questions/2558278/how-to-show-mouseover-tooltip-on-selected-word-on-web-page-using-javascript-con/12707462 "Tooltips with jQueryon StackOverflow")
