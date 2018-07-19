---
title: Display a Random Thumbnail Image next to Gallery Excerpt in WordPress
layout: post
permalink: /display-a-random-thumbnail-image-next-to-gallery-excerpt-in-wordpress/
excerpt: <p>This post demonstrates how to customize the gallery preview on the front page of your WordPress blog and have it display a random image from the gallery every time the page is loaded.</p>
tags:
  - images
  - wordpress
  - wordpress themes
comments: false
---

I have a WordPress site (designed using a Twenty Eleven child theme), which has a whole bunch of image galleries on it.

By default, these galleries will display on the front page of the blog as follows:

- Heading: Gallery
- Post title
- Date published
- A thumbnail from the gallery
- Heading: This gallery contains x photos
- An excerpt of the post's content, followed by a truncating '&#8230;' and a link to the rest of the post
- Categories and tags

Here's an example:

![How the Twenty Eleven theme displays galleries on the site's main page](https://res.cloudinary.com/hibbard/image/upload/v1528880881/gallery_example.png "How the Twenty Eleven theme displays galleries on the site's main page")

I wasn't so keen on this standard format. I didn't want to display the 'Gallery' title or the date posted. I also though it might be nice to have WordPress show a different thumbnail image from the gallery every time the page loads.

Here's how to do that:

Copy the file `content-gallery.php` from the Twenty Eleven theme folder into your child theme folder. You can now edit this file without any fear of breaking anything in the original.

If you don't know how to make a child theme, see here: [http://wp.tutsplus.com/tutorials/theme-development/creating-a-simple-child-theme-using-twenty-eleven/](http://wp.tutsplus.com/tutorials/theme-development/creating-a-simple-child-theme-using-twenty-eleven/ "Creating a Simple Child Theme Using Twenty Eleven ")

In line 17 of `content-gallery.php`, comment out thefollowing to remove the 'Gallery' heading:

```php
<h3><?php _e( 'Gallery', 'twentyeleven' ); ?></h3>
```

In lines 20–22, comment out the following to remove the date published:

```php
<div class="entry-meta">
 <?php twentyeleven_posted_on();?>
</div><!-- .entry-meta -->
```

Then remove the unnecessary padding on the thumbnail by adding the following to your stylesheet:

```css
.entry-content, .entry-summary{padding: 0;}
```

To have WordPress display a random image from the gallery each time the page is loaded, you need to change lines 39 and 40 from this:

```php
$image = array_shift( $images );
$image_img_tag = wp_get_attachment_image( $image->ID, 'thumbnail');
```

to this:

```php
$image = array_rand( $images );
$image_img_tag = wp_get_attachment_image( $image, 'thumbnail' );
```

And there you go. Much nicer!

![What the gallery preview looks like after a little tweaking](https://res.cloudinary.com/hibbard/image/upload/v1528880918/revised_gallery.png "What the gallery preview looks like after a little tweaking")
