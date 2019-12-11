---
title: How to Make a Simple Visitor Counter Using PHP
layout: post
permalink: /how-to-make-a-simple-visitor-counter-using-php/
tags:
  - images
  - php
  - working with files
excerpt_separator: <!--more-->
old-comments: how-to-make-a-simple-visitor-counter-using-php.html
comments: false
---

Imagine you want a visitor counter on your homepage. Sure, there are plenty of [free ones out there](http://www.google.com/search?q=free+visitor+counter "A Google search for 'Free visitor counter'"), but I'm never happy about embedding random third-party code in my website, and besides wouldn't it be more fun to make your own?

Of course it would!

<!--more-->

---

### Here's what we'll be making

Number of visitors to this page so far:

![Visitor counter](https://hibbard.eu/demos/visitor-counter/)

---

### Count all the visitors!

In order to do this, the first thing we'll need is a method of keeping track of the number of visitors to the site. You could store this number in a database, but that'll make things a bit more complicated. Therefore, for the purposes of this demonstration, I'll use a plain and simple text file.

In addition to this, we'll also need a method of identifying unique visitors, as the counter should only increment once per visitor, not once per page view. For this we'll use a [session variable](http://php.net/manual/en/features.sessions.php "PHP Manual - Sessions").

```php
<?php
session_start();
$counter_name = "counter.txt";

// Check if a text file exists.
// If not create one and initialize it to zero.
if (!file_exists($counter_name)) {
  $f = fopen($counter_name, "w");
  fwrite($f,"0");
  fclose($f);
}

// Read the current value of our counter file
$f = fopen($counter_name,"r");
$counterVal = fread($f, filesize($counter_name));
fclose($f);

// Has visitor been counted in this session?
// If not, increase counter value by one
if(!isset($_SESSION['hasVisited'])){
  $_SESSION['hasVisited']="yes";
  $counterVal++;
  $f = fopen($counter_name, "w");
  fwrite($f, $counterVal);
  fclose($f);
}

echo "You are visitor number $counterVal to this site";
```

As you can see I'm using four of [PHP's filesystem functions](http://uk3.php.net/ref.filesystem "PHP Manual - Filesystem Functions") to create, read from, write to and close the text file. I'm also checking if our session variable is set before incrementing the counter. So far, so good…

## Display the Number as an Image

Now all we need to do is read the number stored in the text file and display it on our web page as an image. Here are the steps I'll take to accomplish this:

- Pad the counter value with zeros, so that it has a length of five digits in total. For example a value of one will display as 00001.
- Split the counter value into an array of individual digits. This means that &#8220;00001&#8221; would become `["0", "0", "0", "0", "1"]`
- Using PHP's `imagecreatefrompng` function, create a reference to six png files (one to act as the canvas and five individual digits).
- Using- PHP's `imagecopymerge` function, combine these six png files into one.
- Output the finished image an free it from memory.

The code for this is a little more complicated, so let's step through it bit by bit. Steps one and two are achieved thus:

```php
$counterVal = str_pad($counterVal, 5, "0", STR_PAD_LEFT);
$chars = preg_split('//', $counterVal);
```

Now, before we can start manipulating images with PHP we need an image set to work with. I've chosen to go with the Danielle Young's free Otto number set (since taken offline), which I'm using with her kind permission. If this isn't your kind of thing, a quick Google search should uncover a few more, for example: [http://www.bittbox.com/freebies/free-glass-number-icons](http://www.bittbox.com/freebies/free-glass-number-icons).

Whatever you decide to use, please make sure that you abide by the author's terms and conditions.

In preparation for dynamically creating the final image, I have made one transparent png file (`canvas.png`) which will act as a canvas. This measures 296px by 75px. I have also resized the individual images to a dimension of 56px by 75px and saved them with the same file name as the digit they represent. These will be placed onto the canvas as shown in the diagram:

![The positioning of the digits upon the canvas](https://res.cloudinary.com/hibbard/image/upload/v1528970017/counter_example.png "The positioning of the digits upon the canvas")

I can now use `imagecreatefrompng` to get a reference to `canvas.png`, as well as to the individual digits, stored in the elements of the `$char` array.

```php
$im = imagecreatefrompng("canvas.png");
$src1 = imagecreatefrompng("$chars[1].png");
$src2 = imagecreatefrompng("$chars[2].png");
$src3 = imagecreatefrompng("$chars[3].png");
$src4 = imagecreatefrompng("$chars[4].png");
$src5 = imagecreatefrompng("$chars[5].png");
```

Then we need to fit the individual digits onto the canvas and send the result to the browser to display.

We can use `imagecopymerge` to accomplish the first part. The parameters it takes are as follows:

- destination image link resource
- source image link resource
- x-coordinate of destination point
- y-coordinate of destination point
- x-coordinate of source point
- y-coordinate of source point
- source width
- source height
- pct (we can just leave this at 100).

```php
imagecopymerge($im, $src1, 0, 0, 0, 0, 56, 75, 100);
imagecopymerge($im, $src2, 60, 0, 0, 0, 56, 75, 100);
imagecopymerge($im, $src3, 120, 0, 0, 0, 56, 75, 100);
imagecopymerge($im, $src4, 180, 0, 0, 0, 56, 75, 100);
imagecopymerge($im, $src5, 240, 0, 0, 0, 56, 75, 100);
```

With that done we need to send the image to the browser and free it from memory:

```php
header('Content-Type: image/png');
echo imagepng($im);
imagedestroy($im);
```

Here's the complete listing:

```php
<?php
  session_start();
  $counter_name = "counter.txt";

  // Check if a text file exists.
  //If not create one and initialize it to zero.
  if (!file_exists($counter_name)) {
    $f = fopen($counter_name, "w");
    fwrite($f,"0");
    fclose($f);
  }
  // Read the current value of our counter file
  $f = fopen($counter_name,"r");
  $counterVal = fread($f, filesize($counter_name));
  fclose($f);

  // Has visitor been counted in this session?
  // If not, increase counter value by one
  if(!isset($_SESSION['hasVisited'])){
    $_SESSION['hasVisited']="yes";
    $counterVal++;
    $f = fopen($counter_name, "w");
    fwrite($f, $counterVal);
    fclose($f);
  }

  $counterVal = str_pad($counterVal, 5, "0", STR_PAD_LEFT);
  $chars = preg_split('//', $counterVal);
  $im = imagecreatefrompng("canvas.png");

  $src1 = imagecreatefrompng("$chars[1].png");
  $src2 = imagecreatefrompng("$chars[2].png");
  $src3 = imagecreatefrompng("$chars[3].png");
  $src4 = imagecreatefrompng("$chars[4].png");
  $src5 = imagecreatefrompng("$chars[5].png");

  imagecopymerge($im, $src1, 0, 0, 0, 0, 56, 75, 100);
  imagecopymerge($im, $src2, 60, 0, 0, 0, 56, 75, 100);
  imagecopymerge($im, $src3, 120, 0, 0, 0, 56, 75, 100);
  imagecopymerge($im, $src4, 180, 0, 0, 0, 56, 75, 100);
  imagecopymerge($im, $src5, 240, 0, 0, 0, 56, 75, 100);

  // Output and free from memory
  header('Content-Type: image/png');
  echo imagepng($im);
  imagedestroy($im);
?>
```

Now you can write the following in your HTML code and have it display your very own visitor counter:

```html
<p>Number of visitors to this page so far:</p>
<img alt="Visitor counter" src="counter.php" />
```

You can see the finished script in action at the top of the page. I hope you have found this useful!
