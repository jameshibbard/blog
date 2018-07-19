---
title: Making a Very Simple WordPress Theme for a Static Site
layout: post
permalink: /making-a-very-simple-wordpress-theme-for-a-static-site/
tags:
  - wordpress
  - wordpress themes
old-comments: making-a-very-simple-wordpress-theme-for-a-static-site.html
comments: false
---

Despite starting life as a blogging platform, WordPress can also be used as a fully functional CMS. The following steps describe how to skin WordPress for use as a CMS with a simple brochure-style website.

I will assume that you have WordPress installed and running. If this is not the case, then you can find more than enough help in the following places:

- [http://wordpress.org/about/requirements/](http://wordpress.org/about/requirements/ "What your host needs to run WordPress")
- [http://codex.wordpress.org/Installing_WordPress](http://codex.wordpress.org/Installing_WordPress "The famous 5 Minute Installation, or the more detailed installation guide?")
- [http://codex.wordpress.org/Editing_wp-config.php](http://codex.wordpress.org/Editing_wp-config.php "How to modify the wp-config.php file")

Also, please note that you can [find the files for this tutorial on GitHub](https://github.com/jameshibbard/simple-wp-theme).

---

The first thing to do is create a template for the new site. In my example, I'll use a div container measuring 960px, with a header area, a sidebar for the navigation and a footer. The code for such a template might look something like this:

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Simple template</title>
  <style>
    *{
      margin: 0;
      padding: 0;
    }
    body {
      font: 100%/1.4 Verdana, Arial, Helvetica, sans-serif;
      background:#633232;
      color: #000;
    }
    .container {
      width: 960px;
      background: #FFF;
      margin: 0 auto;
    }
    .header {
      padding:10px;
      background:#999;
    }
    .sidebar {
      float: left;
      width: 190px;
      padding:10px 0 10px 10px;
      background: #ccc;
    }
    .sidebar ul{
      list-style:none;
    }
    .content {
      padding: 10px 25px 25px 15px;
      width: 720px;
      float: left;
    }
    .footer {
      background: #999;
      position: relative;/* Gives IE6 the hasLayout property, so that clear:both will work */
      clear: both;
    }
    .footer p{
      padding:10px;
    }
  </style>
</head>
<body>
  <div class="container">
    <div class="header">
      <h1>Site Heading</h1>
    </div> <!-- end .header -->

    <div class="sidebar">
      <ul class="nav">
        <li><a href="#">Hyperlink 1</a></li>
        <li><a href="#">Hyperlink 2</a></li>
        <li><a href="#">Hyperlink 3</a></li>
        <li><a href="#">Hyperlink 4</a></li>
      </ul>
    </div> <!-- end .sidebar -->

    <div class="content">
      <h2>Page Heading</h2>
      <p>
        Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
        Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.
        Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.
        Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
      </p>
    </div> <!-- end .content-->

    <div class="footer">
      <p>Footer text here.</p>
    </div> <!-- end .footer -->
  </div> <!-- end .container -->
</body>
</html>
```

The next thing to do is to create a folder for your theme. I'll call mine `simple-theme`.

In this folder create a file named `style.css` and a file named `screenshot.png` (you can do this with any decent graphics program, e.g. Adobe's Fireworks).

In `style.css` enter the following CSS comment at the top of the file (adapting it where necessary):

```css
/*
  Theme Name: Simple Theme
  Theme URI: http://hibbard.eu/
  Author: James Hibbard
  Author URI: http://hibbard.eu/
  Description: A really simple theme for demonstrative purposes
  License: MIT
  License URI: https://opensource.org/licenses/MIT
  Tags: simple, easy, garish, pointless
*/
```

Then add the CSS code contained in the header section of the above template.

The file `screenshot.png` will appear under _Available Themes_ in the WordPress backend. I've edited mine to look like this:

![screenshot.png as it will appear in the WordPress backend](http://res.cloudinary.com/hibbard/image/upload/v1528728957/screenshot.png "screenshot.png as it will appear in the WordPress backend")

Next, we have to split the sections of the template into three PHP files, namely `header.php`, `sidebar.php` and `footer.php`.

Create these files in your theme folder, then edit them like so:

### header.php

```php
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>
    <?php wp_title('|', 1, 'right'); ?>
    <?php bloginfo('name'); ?>
  </title>
  <link rel="Stylesheet" href="<?php echo get_stylesheet_directory_uri(); ?>/style.css" />
  <?php wp_head(); ?>
</head>

<body>
  <div class="container">
    <div class="header">
      <h1><?php bloginfo('name'); ?></h1>
    </div>
```

### sidebar.php

```php
<div class="sidebar">
  <?php if ( !function_exists('dynamic_sidebar') || dynamic_sidebar('Sidebar') ) : ?>
  <?php endif; ?>
</div>

<div class="content">
```

### footer.php

```php
      </div> <!-- end .content-->

      <div class="footer">
        <p>Footer text here.</p>
      </div>
    </div> <!-- end .container -->
  </body>
</html>
```

Now we need to tie all of this together and display the result. To do this, create a file called `index.php` in your theme folder and edit it like so:

### index.php

```php
<?php get_header(); ?>
<?php get_sidebar(); ?>

<?php if ( have_posts() ) : while ( have_posts() ) : the_post(); ?>
  <h1><?php the_title(); ?></h1>
  <?php the_content('Read more...'); ?>
<?php endwhile; endif; ?>

<?php get_footer(); ?>
```

The final thing that remains to be done is to add the code which will display our menu in the sidebar. To do this create a file called `functions.php` and add the following code:

### functions.php

```php
<?php
  if ( function_exists('register_sidebar') ) {
    register_sidebar(array('name' => 'Sidebar',
                           'description' => 'Sidebar for navigation',
                           'before_widget' => '<div class="widget">',
                           'after_widget' => '</div>',
                           'before_title' => '',
                           'after_title' => ''));
  }
?>
```

Now upload the folder you created earlier (`simple-theme` in my case), via FTP to the themes folder of your WordPress installation. Typically this is `wp-content/themes`.

Now switch to your WordPress backend and click on _Appearance_ > _Themes_. You should see your theme listed under available themes. Activate it.

Now let's add some pages to our site: Home, About Us and Contact.

Create and save these pages via _Pages_ > _Add New_.

To add these pages to our site's navigation click on _Appearance_ > _Menus_.

Create a new menu, giving it a name, then select the pages you just created and add them to the menu. Update the menu by clicking _Save_.

Finally, you have to embed the menu within the sidebar.

This you can do by going to _Appearance_ > _Widgets_ and dragging the custom menu you just created, from the 'Available Widgets' area on the left, to the 'Main Sidebar' area on the right. Then click _Save_ and you're good to go.

This is what the theme looks like on my site. Not pretty, I know, but hopefully it serves to demonstrate how easy it is to get going with WordPress as a CMS.

![A Preview of the simple-theme](http://res.cloudinary.com/hibbard/image/upload/v1528729753/simple-theme-preview.png "A Preview of the simple-theme")

There are of course lots more things you can do to enhance your site's functionality, for example add a 404 error page, or a search box to the side bar, but this I leave up to you.

And don't forget, you can [find all of the files used in this tutorial on GitHub](https://github.com/jameshibbard/simple-wp-theme).
