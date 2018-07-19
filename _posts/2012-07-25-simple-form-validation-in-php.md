---
title: Simple Form Validation in PHP
layout: post
permalink: /simple-form-validation-in-php/
excerpt_separator: <!--more-->
tags:
  - forms
  - input validation
  - php
old-comments: simple-form-validation-in-php.html
comments: false
---

Server-side validation of user input is something you run into once in a while and although it is not an overly complicated subject, there are a couple of gotchas to be aware of.

This post demonstrates how to validate a simple form in PHP.

<!--more-->

First we need a form. Something like this:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Form validation example</title>
  <style>input{ display:block; margin-bottom:10px; }</style>
</head>

<body>
  <h1>Form Validation Example</h1>
  <h2>Please enter your name</h2>

  <form action="." method="post">
    <label for="first_name">First name:</label>
    <input id="first_name" type="text" />

    <label for="last_name">Last name:</label>
    <input id="last_name" type="text" />

    <input type="submit" value="Submit" />
  </form>
</body>
</html>
```

There are a couple of things to notice here.

- First off, the use of the `<label>` element which defines a label for an `<input>` element. It provides a usability improvement for mouse users, as if the user clicks on the text within the `<label>` element, focus will jump to the `<input>` element associated with it.
- Secondly, what the form does when it's submitted. It simply posts back any values it may have received to itself. This is pretty useless in our case, but demonstrates one of the aforementioned gotchas &#8211; any input the user might have entered disappears once the form is submitted! This will annoy users if all correct input is erased when a form is re-rendered owing to a validation error.

So, now let's make our form do something useful. When the form is submitted I want to access whatever values the user has entered into the 'first name' and 'last name' fields and display an appropriate greeting. However, if our form is being rendered for the first time, I just want to display the blank form and no greeting.

We can tell if the form is being rendered for the first time by adding the following line of HTML just before the submit button:

```html
<input type="hidden" name="process" value="1" />
```

Then we use the following PHP at the top of our file to check if the `process` variable is set to '1'. This will only be the case if the user has clicked the submit button:

```php
<?php
  if ($_POST['process'] == 1) {
    //do some stuff
  }
?>
```

In the following example I then set the variables `$first_name` and `$last_name` to whatever it was the user entered. Check out the PHP.net site for <a href="http://www.php.net/manual/en/reserved.variables.post.php" target="_blank">more information on how to retrieve values in a form</a>, submitted with `method="post"`, using the super global `$_POST` variable.

If both variables are empty, I display the message "Howdy, stranger" otherwise I greet them by name.

```php
<?php
  if ($_POST['process'] == 1) {
    $first_name = $_POST['first_name'];
    $last_name = $_POST['last_name'];
    if (empty($first_name) && empty($last_name)){
      echo "Howdy, stranger";
    }else{
      echo "Hello there, ".$first_name." ".$last_name;
    }
  }
?>

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Form validation example</title>
  <style>input{ display:block; margin-bottom:10px; }</style>
</head>

<body>
  <h1>Form Validation Example</h1>
  <h2>Please enter your name</h2>

  <form action="." method="post">
    <label for="first_name">First name:</label>
    <input name="first_name" id="first_name" type="text" />

    <label for="last_name">Last name:</label>
    <input name="last_name" id="last_name" type="text" />

    <input type="hidden" name="process" value="1" />
    <input type="submit" value="Submit" />
  </form>
</body>
</html>
```

[View demo](https://hibbard.eu/demos/php-form-validation/1/ "Simple HTML form with basic PHP validation")

So, now we know how to catch user input, let's make our 'last name' field compulsory and check if the user has filled it out. If he or she hasn't, we want to display an error message and re-render the form, otherwise we want to display our welcome message.

In the following example I have added a class of `error` to the error message. This is to be preferred over inline CSS and is more semantic to boot.

I have also wrapped the assignment of the `$first_name` and `$last_name` variables in a call to the function `htmlentities()`. This avoids a so-called cross-site scripting attack (XXS), and prevents a potential hacker injecting JavaScript into our web page.

Finally, I have added the following attributes to our input boxes `value="<?php echo($first_name); ?>"` and `value="<?php echo($last_name); ?>"`. This means that whatever value a user enters into either of the input fields is re-rendered in case of a validation error.

You can find out more about XXS here: [http://jstiles.com/Blog/How-To-Protect-Your-Site-From-XSS-With-PHP](http://jstiles.com/Blog/How-To-Protect-Your-Site-From-XSS-With-PHP)

Upon successful submission, the form could do something a little more complicated than regurgitating the user's input. For example it could send an email, and/or could redirect to another page by setting the headers. For example: `header("location: http://hibbard.eu/success");`

So, here's the final code. I hope you find this useful.

```php
<?php
  if ($_POST['process'] == 1) {
    $first_name = htmlentities($_POST['first_name']);
    $last_name = htmlentities($_POST['last_name']);
    if (empty($last_name)){
      echo "<p class='error'>Your last name cannot be blank</p>";
    } else {
      echo "<p>Hello there, ".$first_name." ".$last_name."</p>";
    }
  }
?>

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Form validation example</title>
  <style>
    input{ display:block; margin-bottom:10px; }
    .error{ color:red; }
  </style>
</head>

<body>
  <h1>Form Validation Example</h1>
  <h2>Please enter your name</h2>

  <form action="." method="post">
    <label for="first_name">First name:</label>
    <input name="first_name"
           id="first_name"
           type="text"
           value="<?php echo($first_name); ?>" />

    <label for="last_name">Last name:</label>
    <input name="last_name"
           id="last_name"
           type="text"
           value="<?php echo($last_name); ?>" />

    <input type="hidden" name="process" value="1" />
    <input type="submit" value="Submit" />
  </form>
</body>
</html>
```

[View demo](https://hibbard.eu/demos/php-form-validation/2/ "Simple HTML form with slightly more complicated PHP validation")

**Update (2014.04.11)**: I was contacted by Gajus, who has written an [input validation library](https://github.com/gajus/vlad "Input validation library promoting succinct syntax with extendable validators and multilingual support.") called Vlad, which has a simple and intuitive API. I recommend checking this out to see if this is a good fit for your project.
