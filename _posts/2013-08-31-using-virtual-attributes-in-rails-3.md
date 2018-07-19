---
title: Using Virtual Attributes in Rails 3
layout: post
permalink: /using-virtual-attributes-in-rails-3/
tags:
  - dates
  - rails
  - ruby
excerpt_separator: <!--more-->
comments: false
---

In Rails, virtual attributes allow you to create form fields that do not map directly to the database.

This can be useful in a variety of situations and can help you customize your interface to make it more intuitive and user-friendly.

In this short tutorial I will show you how to create three text fields to allow a user to enter a date (day, month, year), then combine the values entered into these 'virtual' fields and save the new value to the database.

<!--more-->

Let's start off by creating a new project called `birthdays`:

```sh
rails new birthdays
cd birthdays
bundle install
```

Now let's make use of Rails' [scaffolding generator](http://guides.rubyonrails.org/v3.2.13/getting_started.html#getting-up-and-running-quickly-with-scaffolding "Getting Up and Running Quickly with Scaffolding"):

```sh
rails generate scaffold Birthdays name:string date:string
rake db:migrate
rails s
```

If you navigate to `http://localhost/birthdays/` you should now see your newly created Rails app, with the option to add a "New Birthday".

But, before we click that, let's make a couple of changes to the code that Rails generated for us.

Open the `_form` partial (located in `/app/views/birthdays/`) in your favourite editor and change this:

```erb
<div class="field">
  <%= f.label :date %><br />
  <%= f.text_field :date %>
</div>
```

into this:

```erb
<div class="field">
  <%= f.label :day %><br />
  <%= f.text_field :day %>-
  <%= f.text_field :month %>-
  <%= f.text_field :year %>
</div>
```

Now open up your model `birthday.rb`. You should see something like this:

```ruby
class Birthday < ActiveRecord::Base
  attr_accessible :date, :name
end
```

For the virtual attributes to work, we need to create [getter and setter methods](http://en.wikipedia.org/wiki/Accessor "Mutator method") for `day`, `month` and `year`. We then need to create a `before_validation` callback, to ensure that the correct value is assigned to the model's `date` attribute when the form is submitted.

Initially I used `:attr_accessor` to create the getter and setter methods, but this has a caveat: the getter methods work just fine when we're submitting the form to create a record, however when we are reading a record from the database and using the same form to edit it, the date is displayed as blank.

To get around this, I check if `date` is set. If it is, I split it and return the appropriate element. If it isn't, I proceed as normal.

Here's the complete code:

```rb
class Birthday < ActiveRecord::Base
  attr_accessible :date, :name, :day, :month, :year
  before_validation :make_date

  def day
    (date.nil?)? @day : date.split("-")[0]
  end

  def month
   (date.nil?)? @month : date.split("-")[1]
  end

  def year
    (date.nil?)? @year : date.split("-")[2]
  end

  def day=(val)
    @day = val
  end

  def month=(val)
    @month = val
  end

  def year=(val)
    @year = val
  end

  def make_date
    birthday = [@day, @month, @year].join("-")
    self.date = (birthday == "--")? "" : birthday
  end
end
```

And that's all there is to it. Now when you create a new record, you'll be able to enter the day, month and year of the birthday individually, yet when you look at the database, the date will be stored in one column.

### References

- [Railscast #16 Virtual Attributes](http://railscasts.com/episodes/16-virtual-attributes "Keep your controllers clean and forms flexible by adding virtual attributes to your model.")
- [Railscast #167 More On Virtual Attributes](http://railscasts.com/episodes/167-more-on-virtual-attributes "Use a virtual attribute to implement a simple tagging feature.")
