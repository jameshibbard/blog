---
title: Prevent Double Data Submission in Rails
layout: post
permalink: /prevent-double-data-submission-in-rails-3/
tags:
  - forms
  - javascript
  - rails
  - ruby
excerpt_separator: <!--more-->
comments: false
---

I was recently working on an application, that lets people from all over the world apply for a programme at the place I work.

The application works well and does what it should, but we were seeing quite a lot of double submissions, that is to say, the same person applying multiple times in short succession.

Here's how I fixed that.

<!--more-->
## Easy Win — Disable the Submit Button
The first I did was to use JavaScript to disable the submit button once it had been pressed.

This is as simple as this (using jQuery):

```js
$("myForm").on("submit", function(){
  $("input:submit").prop("disabled", true);
});
```

And this should be enough in most cases.

As an alternative to that, if you are using Rails `button_tag`, you can pass it a `:disable_with` parameter, the value of which will be used as the value for a disabled version of the submit button when the form is submitted.

This feature is provided by the unobtrusive JavaScript driver.

```erb
button_tag "Submit", data: { disable_with => "Please wait..." }
```

[API docs](http://api.rubyonrails.org/classes/ActionView/Helpers/FormTagHelper.html#method-i-button_tag "API docs for the button_tag method")

## But This Was Not Enough…

Although implementing the above stopped most double submissions, one or two were still getting through.

My theory on this is that our application gets a fair amount of hits from people in developing countries, who don't always have a particularly good internet connection.

They hit _Submit_, their data gets processed and stored, but the connection times out before they are redirected to the 'success' page, so thinking that something has gone wrong, they hit back, then hit submit again.

Duplicate data sets were not acceptable from my boss's point of view, so I was charged with finding a solution.

What I did was as follows:

-  I created a column in the database named `hash`.
-  When an applicant submits the form, in my model I use the `content_columns` method to iterate over all of the columns in the applicants table.
-  I ignore the `created_at`, `updated_at` and `hash` columns, for obvious reasons.
-  I then use the column names to reference the form values that my model has received.
-  If a particular field is blank, I skip it, otherwise I shovel its value into an array.
-  I then use the `Array#hash` method to compute a hash-code for the array. This is useful because two arrays with the same content will have the same hash-code.

This looks like this:

```ruby
def make_hash
  res = []
  columns_to_skip = ["created_at", "updated_at", "hash"]
  for column in Applicant.content_columns
    next if columns_to_skip.include?(column.name) or
            self[column.name].nil? or
            self[column.name].blank?
    res << self[column.name]
  end

  res.hash.to_s
end
```

Then, in my controller, before a new applicant is saved, I grab the hash of the last successful applicant and compare it with the incoming one.

If the two match, I do nothing.

```ruby
def create
  @applicant = Applicant.new(params[:applicant])
  a = Applicant.last
  @previous_applicant_hash = (a.nil?)? 0 : a.applicant_hash
  if @applicant.valid?
    unless @applicant.applicant_hash == @previous_applicant_hash
      @applicant.save
      ApplicantMailer.admin_notification(@applicant).deliver
      ApplicantMailer.application_confirmation(@applicant).deliver
    end
    redirect_to :action => "success"
  else
    render action: "new"
  end
end
```

I'm sure this is probably overkill, but it has so far proved 100% effective at preventing double submissions.
