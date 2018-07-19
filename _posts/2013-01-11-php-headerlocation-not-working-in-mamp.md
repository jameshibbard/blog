---
title: "PHP – header('Location: ...') not working in MAMP"
layout: post
permalink: /php-headerlocation-not-working-in-mamp/
tags:
  - mamp
  - php
  - servers
excerpt_separator: <!--more-->
old-comments: php-headerlocation-not-working-in-mamp.html
comments: false
---

Recently, I was trying to help someone build a simple contact form in PHP.

This person was running everything locally on a [MAMP server](http://www.mamp.info/en/index.html "MAMP homepage") (Macintosh, Apache, Mysql and PHP) and everything was going swimmingly until we ran into a strange issue: PHP's `header('Location: ...')` wasn't working as expected. In fact, it wasn't working at all.

This is how we solved the problem.

<!--more-->

It turned out that `output_buffering` was disabled in his `php.ini` configuration file. All we had to do is turn it back on.

If this issue is affecting you, here are the steps you can take to correct it.


## Check the Current output_buffering Value

To do this, add the following code to a blank text file, then save it as `info.php` in the root folder of your MAMP installation (by default this is the MAMP `htdocs` folder which is located in the MAMP Application directory `/Applications/MAMP`).

```php
<?php
  phpinfo ();
?>
```

Now navigate to [http://localhost:8888/info.php](http://localhost:8888/info.php) and do an in-page search for the term "output_buffering".

If the value displaying next to it is "no value" or something similar, then it's turned off.

## Turn output_buffering on:

Go to your MAMP folder > conf > php4 or/and php5 -> `php.ini`

Make a copy of this file in case something goes wrong.

Open `php.ini` in a text editor.

Find this part of the file:

```ini
; Output buffering allows you to send header lines (including cookies) even
; after you send body content, at the price of slowing PHP's output layer a
; bit. You can enable output buffering during runtime by calling the output
; buffering functions. You can also enable output buffering for all files by
; setting this directive to On. If you wish to limit the size of the buffer
; to a certain size - you can use a maximum number of bytes instead of 'On', as
; a value for this directive (e.g., output_buffering=4096).
output_buffering = Off
```

The text might not be an exact match, but you can search for "output_buffering"

Change the last line to: `output_buffering = 4096`

Save the file.

Restart MAMP.

Go to [http://localhost:8888/info.php](http://localhost:8888/info.php) again to check the value is 4096.

## Test it's Working

Here's a mini script for you to copy and paste, just to check that everything is good.

Save this as `redirect.php` in your `htdocs` folder.

```php
<?php
  header("Location: http://www.hibbard.eu");
?>
```

Navigate to [http://localhost:8888/redirect.php](http://localhost:8888/redirect.php)  and you should be redirected back to my site.

Sorted!
