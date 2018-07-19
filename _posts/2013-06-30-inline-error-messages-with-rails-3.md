---
title: Inline Error Messages with Rails 3
layout: post
permalink: /inline-error-messages-with-rails-3/
tags:
  - forms
  - input validation
  - rails
  - ruby
excerpt_separator: <!--more-->
old-comments: inline-error-messages-with-rails-3.html
comments: false
---

I was recently tasked with converting a Rails app from Rails 2.3.x to 3.2.x.

"Shouldn't be too difficult" I thought, but one of the first things that came out and bit me is that the `error_message_on` helper method, which was previously used to display error messages next to the form fields that had caused them, has been deprecated.

It took me a little while to figure out how to reinstate this functionality. Here's how I did it.

<!--more-->

First off, we should probably note, that although this method has been removed from Rails, it is now available as a plugin named [dynamic_form](https://github.com/joelmoss/dynamic_form "Helpers to deal with your model backed forms in Rails3"). This means that you could just add `dynamic_form` to your gem file, run bundler and everything would be the same as before.

However, as Ryan Bates notes in [Railscast 211](http://railscasts.com/episodes/211-validations-in-rails-3 "Rails 3 offers several new additions to validations. Here learn how to make a custom error_messages partial, reflect on validations, and clean up complex validations in a model.") (Validations in Rails 3), the reason that this and a couple of other methods (such as `error_messages_for`) have been removed, is that the display of error messages often needs to be customized and doing this through the old methods was a little bit cumbersome.

Instead we now have access to `@resource.errors` which is an instance of the class `ActiveModel::Errors` containing all errors for a particular resource, where each key is the attribute name and the value is an array of strings with all errors.

That means that we can now write:

```ruby
<% if @resource.errors[:field_name] %>
  <span class="formError">
    <%= @resource.errors[:field_name][0] %>
  </span>
<% end %>
```

and have our errors reappear back inline.

Note, that it is necessary to write `[:field_name][0]`, as `@resource.errors[:field_name]` is an array containing all available error messages and it is probably not a good idea to display all of these to your users in one go.

Now, this is all a bit verbose, so it is a good idea to move this code into a helper method.

```ruby
def error_message_for(field, options = {:prepend_text => ""})
  error_message = @resource.errors[field][0]
  if error_message
    raw "<span class='formError'>#{options[:prepend_text]} #{error_message}</span>"
  end
end
```

Now you can just write:

```ruby
<%= error_message_for(:field_name, :prepend_text => "Whatever ") %>
```

### References

- [Rails 3 – show error messages next to field ](http://stackoverflow.com/questions/10775407/rails-3-inline-errors "StackOverflow")
- [Rails 3 – inline errors](http://stackoverflow.com/questions/5646855/show-error-messages-next-to-field "StackOverflow")
- [Working with Validation Errors](http://guides.rubyonrails.org/active_record_validations.html#working-with-validation-errors "Active Record Validations")
