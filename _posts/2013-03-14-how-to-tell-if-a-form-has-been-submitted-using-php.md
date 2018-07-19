---
title: How to Tell If a Form Has Been Submitted Using PHP
layout: post
permalink: /how-to-tell-if-a-form-has-been-submitted-using-php/
tags:
  - forms
  - php
excerpt_separator: <!--more-->
old-comments: how-to-tell-if-a-form-has-been-submitted-using-php.html
comments: false
---

Consider this scenario:

You have a form which submits to itself (e.g. a simple contact form). The first time it loads you don't want to do anything other that render the form.

However, when a user hits _Submit_, you want to process whatever data you have received, then re-render the form, including (for example) error messages where appropriate.

<!--more-->

Until recently, I was doing it like this:

```php
<?php
  if (isset($_POST['submitted'])){
    // The form has been submitted
    // We can do stuff here
  }
?>

<form>
  <input type="hidden" value="1" name="submitted" />
  <input type="submit" />
</form>
```

I was using a hidden input field which, when the page first rendered would be not be set, yet after the form had been submitted would have a value of `1`.

Then it was pointed out to me that this is a bit clunky and can be done much more elegantly:

```php
<?php
  if ($_SERVER['REQUEST_METHOD'] == "POST"){
    // The form has been submitted
    // We can do stuff here
  }
?>
```

You don't need the hidden field at all. When the form first renders:

```php
$_SERVER['REQUEST_METHOD'] == "GET"
```

When the form is submitted:

```php
$_SERVER['REQUEST_METHOD'] == "POST"</pre>
```

Simples!
