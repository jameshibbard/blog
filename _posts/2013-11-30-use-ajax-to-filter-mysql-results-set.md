---
title: Use Ajax to Filter MySQL Results Set
layout: post
permalink: /use-ajax-to-filter-mysql-results-set/
tags:
  - ajax
  - javascript
  - jquery
  - mysql
  - php
excerpt_separator: <!--more-->
old-comments: use-ajax-to-filter-mysql-results-set.html
comments: false
---

I recently helped someone with a project where they had to select a bunch of records from a database, then on the client side use AJAX to filter those records according to certain criteria.

This was a fun thing to work on and a good opportunity to demonstrate the power of AJAX, so I thought I'd write it all up in the form of a quick tutorial.

For the impatient among you, here's [a demo of what we'll end up with](https://hibbard.eu/demos/ajax-filter/ "A demo of the finished AJAX filter script").

<!--more-->

## The Temp Agency

In this tutorial we'll imagine we're creating an app for a temp agency. Our app should initially display a list of all available temporary workers with the option of filtering them according to whether they have a car, can speak a foreign language, can work nights, or are students.

So, let's create the database table:

```sql
CREATE TABLE IF NOT EXISTS `workers` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `address` varchar(255) DEFAULT NULL,
  `hasCar` tinyint(1) DEFAULT NULL,
  `speaksForeignLanguage` tinyint(1) DEFAULT NULL,
  `canWorkNights` tinyint(1) DEFAULT NULL,
  `isStudent` tinyint(1) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

You'll notice I'm using `tinyint` to store the Boolean values, whereby a value of zero is considered false and non-zero values are considered true.

You can read more on this here: [Which MySQL Datatype to use for storing boolean values?](http://stackoverflow.com/questions/289727/which-mysql-datatype-to-use-for-storing-boolean-values "StackOverflow discussion")

Now we'll need to populate our table with some data:

```sql
INSERT INTO `workers` (
  `id`, `name`, `age`, `address`, `hasCar`,
  `speaksForeignLanguage`, `canWorkNights`, `isStudent`
) VALUES
(1, 'Jim', 39, '12 High Street, London', 1, 1, 1, 1),
(2, 'Fred', 29, '13 High Street, London', 1, 1, 1, 0),
(3, 'Bill', 19, '14 High Street, London', 1, 1, 0, 0),
(4, 'Tom', 39, '15 High Street, London', 1, 0, 0, 0),
(5, 'Cathy', 29, '16 High Street, London', 1, 0, 0, 1),
(6, 'Petra', 19, '17 High Street, London', 1, 0, 1, 0),
(7, 'Heide', 39, '18 High Street, London', 1, 1, 0, 0),
(8, 'William', 29, '19 High Street, London', 1, 1, 0, 1),
(9, 'Ted', 19, '20 High Street, London', 0, 0, 0, 1),
(10, 'Mike', 19, '21 High Street, London', 1, 0, 0, 1),
(11, 'Jo', 19, '22 High Street, London', 0, 1, 0, 1);
```

To round it all off, we'll need a PHP script that connects with our database, fetches all of the worker records and returns them as JSON:

```php
<?php
  $pdo = new PDO(
    'mysql:host=localhost;dbname=db_name', 'user', 'pass'
  );
  $select = 'SELECT *';
  $from = ' FROM workers';
  $where = ' WHERE TRUE';
  $sql = $select . $from . $where;
  $statement = $pdo->prepare($sql);
  $statement->execute();
  $results=$statement->fetchAll(PDO::FETCH_ASSOC);
  $json=json_encode($results);
  echo($json);
?>
```

Here's [a demo](http://hibbard.eu/blog/pages/ajax-filter/1/index.php "A load of JSON") of what we've got so far. Exciting, huh?

Please note that for the purposes of this tutorial, I'm using `JSON_PRETTY_PRINT` to format the output. In real life this is not necessary.

## On the Client-side

So far everything we have done has been on the server. This is about to change.

The next step is to create an HTML page which when accessed in the browser, will fire off an Ajax request to our server-side PHP script and display the results in a predefined `<div>` element:

```html
<!DOCTYPE HTML>
<html>
  <head>
    <meta charset="utf-8">
    <title>AJAX filter demo</title>
  </head>

  <body>
    <h1>Temporary Worker Database</h1>
    <div id="workers"></div>

    <script src="https://code.jquery.com/jquery-latest.js"></script>
    <script>
      function updateEmployees(){
        $.ajax({
          type: "POST",
          url: "submit.php",
          dataType : 'json',
          cache: false,
          success: function(records){
            $('#workers').text(JSON.stringify(records, null, 4));
          }
        });
      }

      updateEmployees();
    </script>
  </body>
</html>
```

Hopefully there's nothing too complicated going on here. When the page loads, we're calling the function `updateEmployees()` which is making a POST request to submit.php and inserting the server's response into the page.

Here's [a demo](http://hibbard.eu/blog/pages/ajax-filter/2/index.php "A load of JSON") of what it looks like.

## Getting Stylish

Now we need to format the data, as the bunch of JSON we currently have is of no use to man nor beast.

The easiest way to do this is to use a table (as this is tabular data) which will replace our previous `<div>` element.

I'll hard-code the table structure into the page:

```html
<table id="employees">
  <thead>
    <tr>
      <th>ID</th>
      <th>Name</th>
      <th>Age</th>
      <th>Address</th>
      <th>Car</th>
      <th>Language</th>
      <th>Nights</th>
      <th>Student</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
```

Then use [a function that I found on StackOverflow](http://stackoverflow.com/questions/1051061/convert-json-array-to-an-html-table-in-jquery "Why reinvent the wheel?") to format the JSON correctly, before inserting it into the table body:

```js
function makeTable(data){
  var tbl_body = "";
  $.each(data, function() {
    var tbl_row = "";
    $.each(this, function(k, v) {
      tbl_row += "<td>"+v+"</td>";
    });
    tbl_body += "<tr>"+tbl_row+"</tr>";
  });
  return tbl_body;
}
...
$('#employees tbody').html(makeTable(records));
```

This is starting to look a bit more usable now. You can [see the results here](https://hibbard.eu/demos/ajax-filter/3/ "We have tables!").

However, some styles would be nice, wouldn't they?

A quick Google search brought [this page](http://coding.smashingmagazine.com/2008/08/13/top-10-css-table-designs/ "Top 10 CSS Table Designs") from Smashing Magazine to light. I went with the "Horizontal Minimalist" design.

I also found [this fine tutorial](http://tympanus.net/codrops/2012/11/02/heading-set-styling-with-css/ "Heading Set Styling with CSS"), which I used to style the heading.

Here's [the updated page](https://hibbard.eu/demos/ajax-filter/4/ "Looking good!") – looking quite snazzy, I'm sure you agree.

## Filtering the Results

So, we can display all the results from the database, all we need to do now is to filter them.

For this, we'll need some checkboxes.

```html
<div id="filter">
  <h2>Filter options</h2>
  <div>
    <input type="checkbox" id="car" name="hasCar">
    <label for="car">Has own car</label>
  </div>
  <div>
    <input type="checkbox" id="language" name="speaksForeignLanguage">
    <label for="language">Can speak foreign language</label>
  </div>
  <div>
    <input type="checkbox" id="nights" name="canWorkNights">
    <label for="nights">Can work nights</label>
  </div>
  <div>
    <input type="checkbox" id="student" name="isStudent">
    <label for="student">Is a student</label>
  </div>
</div>
```

Now we'll need to attach an onchange event handler to the checkboxes, so that our `updateEmployees()` function is fired every time a user selects or deselects anything.

We'll pass this function an array containing the names of whichever checkboxes were selected, which it can then send to the PHP script.

```js
function getEmployeeFilterOptions(){
  var opts = [];
  $checkboxes.each(function(){
    if(this.checked){
      opts.push(this.name);
    }
  });
  return opts;
}

var $checkboxes = $("input:checkbox");

$checkboxes.on("change", function(){
  var opts = getEmployeeFilterOptions();
  updateEmployees(opts);
});
```

Our final task is to adjust the PHP script so that it builds its query correctly:

```php
<?php
  $pdo = new PDO(
    'mysql:host=localhost;dbname=db_name', 'user', 'pass'
  );
  $select = 'SELECT *';
  $from = ' FROM workers';
  $where = ' WHERE TRUE';
  $opts = isset($_POST['filterOpts'])?
            $_POST['filterOpts'] :
            array('');

  if (in_array("hasCar", $opts)){
    $where .= " AND hasCar = 1";
  }

  if (in_array("speaksForeignLanguage", $opts)){
    $where .= " AND speaksForeignLanguage = 1";
  }

  if (in_array("canWorkNights", $opts)){
    $where .= " AND canWorkNights = 1";
  }

  if (in_array("isStudent", $opts)){
    $where .= " AND isStudent = 1";
  }

  $sql = $select . $from . $where;
  $statement = $pdo->prepare($sql);
  $statement->execute();
  $results = $statement->fetchAll(PDO::FETCH_ASSOC);
  $json = json_encode($results);
  echo($json);
?>
```

As you can see, we are now checking to see if our PHP script received a `filterOpts` parameter. If not, we set `$opts` to be an empty array.

We then check the `$opts` array for the presence of any elements we are interested in and extend the `WHERE` part of our query accordingly.

This might seem a little verbose, but this approach means that we don't open ourselves up to the possibility of an [SQL injection attack](http://en.wikipedia.org/wiki/SQL_injection "Nasty things!"), which we might otherwise do if we formed a query out of whatever we received from the client.

Here's a listing of the complete code:

```html
<!DOCTYPE HTML>
<html>
  <head>
    <meta charset="utf-8">
    <title>AJAX filter demo</title>
    <style>
      body {
        padding: 10px;
      }
      h1 {
          margin: 0 0 0.5em 0;
          color: #343434;
          font-weight: normal;
          font-family: 'Ultra', sans-serif;
          font-size: 36px;
          line-height: 42px;
          text-transform: uppercase;
          text-shadow: 0 2px white, 0 3px #777;
      }

      h2 {
          margin: 1em 0 0.3em 0;
          color: #343434;
          font-weight: normal;
          font-size: 30px;
          line-height: 40px;
          font-family: 'Orienta', sans-serif;
      }
      #employees {
        font-family: "Lucida Sans Unicode","Lucida Grande",Sans-Serif;
        font-size: 12px;
        background: #fff;
        margin: 15px 25px 0 0;
        border-collapse: collapse;
        text-align: center;
        float: left;
        width: 700px;
      }
      #employees th {
        font-size: 14px;
        font-weight: normal;
        color: #039;
        padding: 10px 8px;
        border-bottom: 2px solid #6678b1;
      }
      #employees td {
        border-bottom: 1px solid #ccc;
        color: #669;
        padding: 8px 10px;
      }
      #employees tbody tr:hover td {
        color: #009;
      }
      #filter {
        float:left;
      }
    </style>
  </head>
  <body>
    <h1>Temporary worker database</h1>

    <table id="employees">
      <thead>
        <tr>
          <th>ID</th>
          <th>Name</th>
          <th>Age</th>
          <th>Address</th>
          <th>Car</th>
          <th>Language</th>
          <th>Nights</th>
          <th>Student</th>
        </tr>
      </thead>
      <tbody>
      </tbody>
    </table>

    <div id="filter">
      <h2>Filter options</h2>
      <div>
        <input type="checkbox" id="car" name="hasCar">
        <label for="car">Has own car</label>
      </div>
      <div>
        <input type="checkbox"
               id="language"
               name="speaksForeignLanguage">
        <label for="language">Can speak foreign language</label>
      </div>
      <div>
        <input type="checkbox" id="nights" name="canWorkNights">
        <label for="nights">Can work nights</label>
      </div>
      <div>
        <input type="checkbox" id="student" name="isStudent">
        <label for="student">Is a student</label>
      </div>
    </div>

    <script src="http://code.jquery.com/jquery-latest.js"></script>
    <script>
      function makeTable(data){
       var tbl_body = "";
          $.each(data, function() {
            var tbl_row = "";
            $.each(this, function(k , v) {
              tbl_row += "<td>"+v+"</td>";
            })
            tbl_body += "<tr>"+tbl_row+"</tr>";
          })
        return tbl_body;
      }
      function getEmployeeFilterOptions(){
        var opts = [];
        $checkboxes.each(function(){
          if(this.checked){
            opts.push(this.name);
          }
        });
        return opts;
      }
      function updateEmployees(opts){
        $.ajax({
          type: "POST",
          url: "submit.php",
          dataType : 'json',
          cache: false,
          data: {filterOpts: opts},
          success: function(records){
            $('#employees tbody').html(makeTable(records));
          }
        });
      }
      var $checkboxes = $("input:checkbox");
      $checkboxes.on("change", function(){
        var opts = getEmployeeFilterOptions();
        updateEmployees(opts);
      });
      updateEmployees();
    </script>
  </body>
</html>
```

And for those who missed it the first time round, [the working page](http://hibbard.eu/demos/ajax-filter/ "The final thing").

I hope someone found this useful.

If you have any questions or comments, I'd be glad to hear them.
