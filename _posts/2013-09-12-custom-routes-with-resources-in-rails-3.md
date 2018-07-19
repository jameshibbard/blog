---
title: Custom Routes and RESTful Resources in Rails 3
layout: post
permalink: /custom-routes-with-resources-in-rails-3/
tags:
  - rails
  - routing
  - ruby
excerpt_separator: <!--more-->
comments: false
---

The problem: you have a RESTful resource with default routes. The form to create a new item is located at `http://mydomain.com/resource/new/`. When you submit the form with valid input, a new item is created and that works fine.

However, when you submit the form with invalid input, the controller re-renders the form and the URL is changed to `http://mydomain.com/resource`. This is potentially confusing for end users.

How can we avoid this?

<!--more-->

Well, before we start talking about solutions, let me first say that this behaviour is actually correct on the part of Rails.

You create a new item by sending a HTTP POST request to the `create` action in your controller. When validation fails, you're still in the `create` action, but you're rendering the `new` view. Rails does not redirect you to the `new` action.

## A Concrete Example

Now that that is out of the way, let's look at how we can configure an app, so that re-loading the form to create a new item, appears to happen at the same URL as it was originally loaded at.

Let's start off by creating a new project and generating a scaffold:

```sh
rails new Contacts
cd Contacts
rails g scaffold Contact name:string email:string
rake db:migrate
```

Let's add some validation to the model:

```ruby
class Contact > ActiveRecord::Base
  attr_accessible :email, :name
  validates_presence_of :name
end
```

Now we need to alter our `routes.rb` file, thus:

```ruby
Contacts::Application.routes.draw do
 resources :contacts
 match 'contacts/new', :to => 'contacts#create',
                       :via => :post,
                       :as => :post_contact
end
```

This maps both the `new` action and the `create` action to the same URL.

The last thing to do is to update the `_form` partial (found in `/app/views/contacts/_form.html.erb`):

```erb
<%= form_for @contact, :url => post_contact_path do |f| %>
 ...
<% end %>
```

Restart your server and you're done!

Well, almost done. Unfortunately, you'll find that if you now create an item, then try to edit it, your app dies with the error message:

```
ActiveRecord::RecordNotFound in ContactsController#update
Couldn't find Contact with id=new
```

This is because your `new` action and your `edit` action are using the same partial to create/edit items, but the form generated in this partial can no longer submit to the same URL on both occasions.

No problem, let's just pass that in. Alter your Contacts controller thus:

```ruby
def new
  @contact = Contact.new
  @url = post_contact_path

  respond_to do |format|
    format.html # new.html.erb
    format.json { render json: @contact }
  end
end

def edit
  @contact = Contact.find(params[:id])
  @url = @contact
end
```

your `new` and `edit` views thus:

```erb
<%= render 'form', :url => @url %>
```

and your `_form` partial thus:

```erb
<%= form_for @contact, :url => @url do |f| %>
```

Now everything really will work as expected.

## Mapping the New View to a Different URL

In the case of a contact form (for example), it might be useful to customize the URL, so that the form is found at `http://mydomain.com/contact/`, instead of `http://mydomain.com/contacts/new/`.

This too, is quite possible, just alter your routes file like so:

```ruby
Demo::Application.routes.draw do
  resources :contacts
  match '/contact', :to => 'contacts#new', :via => :get
  match '/contact', :to => 'contacts#create',
                    :via => :post,
                    :as => :post_contact
end
```

It's important to add the `:via => get`, otherwise your data won't persist if the form is submitted with errors.

### References

- <a title="StackOverflow discussion" href="http://stackoverflow.com/questions/14575442/rails-routing-prob-on-failing-create-re-renders-the-form-as-it-should-but" target="_blank">Rails routing prob: on failing "create", re-renders the form (as it should) but not at the URL in my routes</a>
- <a title="StackOverflow discussion" href="http://stackoverflow.com/questions/14490098/in-rails-3-when-a-resource-create-action-fails-and-calls-render-new-why-must" target="_blank">In Rails 3, when a resource create action fails and calls render :new, why must the URL change to the resource's index url?</a>
- <a title="StackOverflow discussion" href="http://stackoverflow.com/questions/4932771/rails-why-does-custom-url-change-when-render-new-is-called" target="_blank">Rails: Why does custom url change when "render new" is called?</a>
- <a title="RailsGuides" href="http://guides.rubyonrails.org/routing.html#customizing-resourceful-routes" target="_blank">Rails Routing from the Outside In &#8211; Customizing Resourceful Routes</a>
