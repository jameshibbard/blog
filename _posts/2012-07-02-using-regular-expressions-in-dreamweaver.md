---
title: Using Regular Expressions in Dreamweaver
layout: post
permalink: /using-regular-expressions-in-dreamweaver/
excerpt: <p>This time-saving tip explains how to use Dreamweaver's search/replace functionality combined with the power of regular expressions to manipulate HTML across a large number of pages.</p>
tags:
  - code editors
  - regular expressions
old-comments: using-regular-expressions-in-dreamweaver.html
comments: false
---

Recently, I had to change the following HTML:

```html
<h3>Short academic profile:</h3>
<p>
  2000 - 2005: Studies in something at a university somewhere<br />
  2005 - 2010: Different studies at a different uni<br />
  Since 2011: Very important studies somewhere else<br />
</p>
```

into this:

```html
<h3>Short academic profile:</h3>
<ul>
  <li>2000 - 2005: Studies in something at a university somewhere</li>
  <li>2005 - 2010: Different studies at a different uni</li>
  <li>Since 2011: Very important studies somewhere else</li>
</ul>
```

If I was using ruby to to do this, I would make use of `$1`, (a global variable, representing the content of the previous successful pattern match) and write something like:

```ruby
if line.match(/^\s+(.*)<br \/>/)
  line.sub!($1, "<li>#{$1}").sub!(/<br \/>/, "</li>")
end
```

As this is something that comes up quite often in the course of my work, it got me wondering if one can use Dreamweaver's search and replace dialogue to do something similar.

Happily, it turns out that you can.

Open the dialogue box using <kbd>Crtl</kbd> + <kbd>F</kbd>, make sure _Use regular expression_ is checked, then type:

_Search_: `\b(.*)<br \/>`
_Replace_: `<li>$1</li>`


![Dreamweaver's search/replace dialogue](https://res.cloudinary.com/hibbard/image/upload/v1528723571/dreamweaver_search_dialogue_with_regular_expression.png "Dreamweaver's search/replace dialogue")

This regular expression will match a word boundary, followed by any number of characters (except newline), followed by a `<br />`. The any number of characters (except newline) bit is placed in brackets, and can consequently be later referenced via `$1`.

Incidentally, a small annoyance is that Dreamweaver's search and replace function does not support using `^` and `$` to match the beginning and end of lines. If you want to match this, use `[\r\n]`

There is an excellent [article on using regular expressions in Dreamweaver](http://www.adobe.com/devnet/dreamweaver/articles/regular_expressions_pt1.html "Part 1 of a two-part tutorial series on using regular expressions with Dreamweaver") on the Adobe site.
