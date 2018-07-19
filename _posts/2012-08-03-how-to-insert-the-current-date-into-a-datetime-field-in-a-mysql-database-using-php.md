---
title: How to Insert the Current Date into a DATETIME Field in a MySQL Database using PHP
layout: post
permalink: /how-to-insert-the-current-date-into-a-datetime-field-in-a-mysql-database-using-php/
excerpt: <p>This quick tip explains how to insert the current date into a table in a MySQL database from within a PHP script.</p>
tags:
  - dates
  - mysql
  - php
old-comments: how-to-insert-the-current-date-into-a-datetime-field-in-a-mysql-database-using-php.html
comments: false
---

This is a simple but effective tip that I picked up this week.

In the past, if I wanted to insert the current date into a table in a MySQL database from within a PHP script, I would always do something like this:

```php
<?php
  $current_date = date("Y-m-d H:i:s");
  ...
  $query = "INSERT INTO guestbook SET date = '$current_date'";
  $sql = mysql_query($query) or die(mysql_error());
?>
```

Then it was pointed out to me, that this is in fact an arse-backwards way of doing things, as the variable `$current_date` will enter MySQL as a string, which MySQL will have to parse back into a DATETIME type.

A much better / faster / cleaner way to do things is using [CURRENT_TIMESTAMP](https://www.w3schools.com/sql/func_mysql_current_timestamp.asp "The CURRENT_TIMESTAMP() function returns the current date and time"), which is a built-in MySQL function.

```php
<?php
  $query = "INSERT INTO guestbook SET date = CURRENT_TIMESTAMP";
  $sql = mysql_query($query) or die(mysql_error());
?>
```

You can take this even further by constructing the date field in your database table thus:

```sql
date TIMESTAMP NULL default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP
```

Which means that it is automatically populated with the current date, the minute a record is created or updated.
