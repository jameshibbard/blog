---
title: Inspect Object in PHP
layout: post
permalink: /inspect-object-in-php/
excerpt_separator: <!--more-->
tags:
  - php
  - ruby
comments: false
---

To display the contents of an object in Ruby is really simple.

All you do is store the object in a variable, then pass the variable as an argument to the Kernel method `p()`, which in turn writes `obj.inspect` to the standard output.

<!--more-->

For example:

```ruby
class Person
  attr_accessor :name, :age
  def initialize(name, age)
    @name, @age = name, age
  end
end

bob= Person.new("Robert DeNiro", 68)
p bob
```

Outputs: `#<Person:0x1a58d8 @age=68, @name="Robert DeNiro">`

I was recently doing something in WordPress where it would have been really useful to see the contents of an object I was fetching.

Here's how I solved the problem:

```php
<?php
  // get_term_by returns an object
  $term = get_term_by(
    'slug',
    get_query_var( 'term' ),
    get_query_var( 'taxonomy' )
  );

  $array = get_object_vars($term);
  while (list($key, $val) = each($array)) {
    echo "$key => $val"."<br />";
  }
?>
```

This does what it should, but I still think the Ruby way is less cumbersome though &#8230;

N.B. Also useful was PHP's `count()` function, which returns the length of an array.

This is documented here: [http://phparraylength.com/](http://phparraylength.com/ "Find the Length of an Array in PHP").

---

**Edit**: since getting a bit more into PHP, I have discovered the nifty function `print_r()`, which outputs human-readable information about a variable. You can read more about this here: [http://php.net/manual/en/function.print-r.php](http://php.net/manual/en/function.print-r.php "PHP: print_r - Manual")
