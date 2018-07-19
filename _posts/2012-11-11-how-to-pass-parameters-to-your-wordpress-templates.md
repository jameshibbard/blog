---
title: How to Pass Parameters to Your WordPress Templates
layout: post
permalink: /how-to-pass-parameters-to-your-wordpress-templates/
tags:
  - wordpress
  - wordpress templates
comments: false
---

I currently find myself in the middle of redesigning this site, as I want to move away from the generic WordPress theme to something a little more individual. That my new design should be responsive was a no-brainer, but this time round I decided to try something new (for me, at least) and that was to design for mobile first.

In the course of doing this, I ran into an interesting problem – how to display different elements in my side bar, depending upon which page was being viewed.

Let me elaborate:

As those of you who are familiar with WordPress will know, your average sidebar is nothing more than a PHP file which gets included into (almost) every page on your site, using the method `get_sidebar()`. Usually the sidebar defines a widget area, which can be filled with widgets via the dashboard, but in this case I had decided to hard code mine, having read somewhere that this gives you a small performance boost.

Applying the mobile first approach, I started by designing for a screen measuring 320×480 pixels (the size of your average iPhone 3) . For a screen this size, I decided to push the sidebar under the main content area, which was easy to do using CSS3 media queries and looked just fine for the index page of the site. So far, so good…

However, for some of the other pages (such as the search results page, or the 404 error page) I didn't want to display all of the sidebar elements, just some of them. And for other pages (such as the single post page, or normal static pages) I didn't want to display the sidebar at all.

The problem was that I wanted to do all of this, but only when the site was being viewed on a phone. When the site was viewed on a normal desktop PC, I wanted the entire sidebar to sit next to the content area (not beneath it), regardless of whichever page was being viewed. Oh dear…

It seemed that the ideal solution to this problem, would be to apply a class to the sidebar, depending upon which page it was being included from. Then, in my style sheet, I could target the individual widgets using css.

For example:

```css
/* Hide blogroll on 404 page for small screen */
.error-page span.blogroll{
  display:none;
}
```

Or:

```css
/* Display blogroll on 404 page for desktop */
@media all and (min-width:960px){
  .error-page span.blogroll{
    display:block;
  }
}
```

However, I had a quick look at the [Codex documentation for the get_sidebar() method](http://codex.wordpress.org/Function_Reference/get_sidebar "Function Reference/get sidebar"), but the only parameter it accepts, is `$name`, which is only useful if you want to include, for example, two different sidebars.

I thought about this a little more, then when I stumbled on the solution, it was blindingly obvious! Including a PHP file is the same as copy-pasting the code from the include into the position where the include-statement stands. This means that you inherit the current scope. Bingo!

That meant, I could write the following in my 404.php:

```php
<?php
  $sidebarClass="error-page":
  get_sidebar();
?>
```

And then in sidebar.php:

```php
<div id="sidebar"
  <?php if (isset($sidebarClass)){echo("class=\"$sidebarClass\"");} ?>
>
```

And Bob's your uncle! Using CSS, I could now target whichever sidebar elements I wanted, or even hide the sidebar completely, yet at the same time restrict all of that to a device of a certain width.

**Reference:**
This post at Stack Overflow was very helpful - [How to pass parameters to PHP template rendered with 'include'?](http://stackoverflow.com/questions/1312300/how-to-pass-parameters-to-php-template-rendered-with-include "How to pass parameters to PHP template rendered with 'include'?")
