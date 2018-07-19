---
title: Nested Attributes and Updating Without a Password in Devise
layout: post
permalink: /nested-attributes-and-updating-without-a-password-in-devise/
tags:
  - authentication
  - devise
  - forms
  - ruby
  - rails
excerpt_separator: <!--more-->
old-comments: nested-attributes-and-updating-without-a-password-in-devise.html
comments: false
---

I've spent all day trying to get Devise and nested attributes to play nicely together. This and giving the user the ability to update parts of their profile without providing a password proved kind of tricky. Here's how I got things working.

The code for this tutorial is on GitHub: [https://github.com/hibbard-eu/authentication-with-devise-and-cancancan](https://github.com/hibbard-eu/nested-attributes-with-devise "Project code")

<!--more-->

---

I have a standard Devise `User` model which contains of the usual email and password fields. The only deviation from Devise's standard configuration is that a user has an associated profile which is specified using `belongs_to` in the `User` model (as [this is where I want to place the foreign key](https://stackoverflow.com/questions/3808926/whats-the-difference-between-belongs-to-and-has-one "Differences Between Belongs_to and Has_one in Ruby on Rails")). The profile will contain a bunch of information about the user and should be nested in the form which is shown when a user accesses Devise's `edit_user_registration_path`.

To start with lets create a new project and generate the resources:

```sh
rails new users && cd users
rails g scaffold User profile:belongs_to
rails g scaffold Profile name:string age:integer hobbies:text
rake db:migrate
```

Add the Devise gem to the Gemfile, run bundle, then initialize the gem:

```sh
gem 'devise'
```

```sh
bundle install
rails g devise:install
rails g devise User
rake db:migrate
```

Make the usual Devise specific configuration:

Add the following line to the bottom of `config/environments/development.rb`:

```rb
config.action_mailer.default_url_options = {:host => 'localhost:3000'}
```

And set a root route in `/config/routes.rb`

```rb
root to: "welcome#index"
```

We'll also need to create a `WelcomeController` which contains an index action, as well as the corresponding view:

```rb
class WelcomeController < ApplicationController
  def index
  end
end
```

```html
<h1>Welcome</h1>
<p>This is the welcome page</p>
```

Finally, add the following to layouts/application.html.erb:

```erb
<% if user_signed_in? %>
  Signed in as <%= current_user.email %>&nbsp;&nbsp;
  <%= link_to "Edit profile", edit_user_registration_path %>&nbsp;|&nbsp;
  <%= link_to "Sign out", destroy_user_session_path, :method => :delete %>
<% else %>
  <%= link_to "Sign up", new_user_registration_path %> or <%= link_to "sign in", new_user_session_path %>
<% end %>

<% flash.each do |name, msg| %>
  <%= content_tag :div, msg, id: "flash_#{name}" %>
<% end %>
```

Before we can get on to customizing the "Edit profile" page, we'll need to create a user and a profile, then associate the one with the other:

```sh
rails c
pw = "12345678"
p = Profile.create(name: "Jack")
u = User.create(email: "a@b.com", password: pw, password_confirmation: pw, profile: p)
```

## Nested Attributes

Now if you go to `http://localhost:3000` you should be able to sign in with the above credentials. Once you've done that, click the "Edit profile" link in the top right and you should see a form like this:

![Devise standard Edit profile view](https://res.cloudinary.com/hibbard/image/upload/v1530103834/devise-edit-profile-view.png "Devise standard Edit profile view")

In order to customize this view, we need to have Devise copy a few files into our project:

```sh
rails g devise:views
```

And tell our `User` model that we will be requiring nested attributes:

```rb
accepts_nested_attributes_for :profile
```

Now we can edit `app/views/devise/registrations/edit.html.erb`:

```erb
<h2>Login Details</h2>

<%= form_for(resource, as: resource_name, url: registration_path(resource_name), html: { method: :put }) do |f| %>
  <%= devise_error_messages! %>

  <div class="field">
    <%= f.label :email %><br />
    <%= f.email_field :email, autofocus: true %>
  </div>

  <% if devise_mapping.confirmable? && resource.pending_reconfirmation? %>
    <div>Currently waiting confirmation for: <%= resource.unconfirmed_email %></div>
  <% end %>

  <div class="field">
    <%= f.label :password, "New password" %><br />
    <%= f.password_field :password, autocomplete: "off" %>
  </div>

  <div class="field">
    <%= f.label :password_confirmation, "New password confirmation"  %><br />
    <%= f.password_field :password_confirmation, autocomplete: "off" %>
  </div>

  <div class="field">
    <%= f.label :current_password %> <i>(to confirm changes)</i><br />
    <%= f.password_field :current_password, autocomplete: "off" %>
  </div>

  <%= f.fields_for :profile do |builder| %>
    <hr />

    <h2>Profile Details</h2>

    <div class="field">
      <%= builder.label :name %><br />
      <%= builder.text_field :name %>
    </div>
    <div class="field">
      <%= builder.label :age %><br />
      <%= builder.text_field :age %>
    </div>
    <div class="field">
      <%= builder.label :hobbies %><br />
      <%= builder.text_area :hobbies, :rows => 5  %><br />
    </div>
  <% end %>

  <div class="actions">
    <%= f.submit "Update" %>
  </div>

<% end %>

<%= link_to "Back", :back %>
```

Now in order for Devise to register these changes at all, we need to update our `ApplicationController`:

```rb
before_action :configure_permitted_parameters, if: :devise_controller?

protected

def configure_permitted_parameters
  devise_parameter_sanitizer.for(:account_update) {|u| u.permit(
    :email,
    :password,
    :password_confirmation,
    :current_password,
    profile_attributes: [:id, :name, :age, :hobbies]
  )}
end
```

At this point if you attempt to edit the profile (including any of the nested attributes), it'll work, but Devise will steadfastly insist on you entering a password. Let's change that.

## Allowing Users to Update Things Without a Password

In the last part of this post, I would like to demonstrate how to have Devise permit updates to any of the nested attributes without a user having to enter their password. However, it should still ask for password confirmation and/or a password if a user tries to update either the `email` or `password` fields.

The first step to achieving this is to alter our routes file:

```rb
devise_for :users, :controllers => {:registrations => "registrations"}
```

This overrides Devise's `RegistrationsController` and allows us to add our own logic to handle updates.

Now we need to create the file `app/controllers/registrations_controller.rb` and add the following:

```rb
class RegistrationsController < Devise::RegistrationsController
  def update
    account_update_params = devise_parameter_sanitizer.sanitize(:account_update)
    @user = User.find(current_user.id)

    if needs_password?
      successfully_updated = @user.update_with_password(account_update_params)
    else
      account_update_params.delete('password')
      account_update_params.delete('password_confirmation')
      account_update_params.delete('current_password')
      successfully_updated = @user.update_attributes(account_update_params)
    end

    if successfully_updated
      set_flash_message :notice, :updated
      sign_in @user, :bypass => true
      redirect_to edit_user_registration_path
    else
      render 'edit'
    end
  end

  private

  def needs_password?
    @user.email != params[:user][:email] || params[:user][:password].present?
  end
end
```

This checks to see if the user is attempting to update the email or password field. If so, and if validation passes, then it calls [update_with_password](http://www.rubydoc.info/github/plataformatec/devise/Devise/Models/DatabaseAuthenticatable:update_with_password "Method: Devise::Models::DatabaseAuthenticatable#update_with_password") which will carry out the necessary password checks, otherwise it calls update_attributes, which doesn't.

And that's everything.

The code for this tutorial is on GitHub: [https://github.com/hibbard-eu/authentication-with-devise-and-cancancan](https://github.com/hibbard-eu/nested-attributes-with-devise "Project code")

### Reference

-  [Railscasts#196 Nested Model Form Part 1](http://railscasts.com/episodes/196-nested-model-form-part-1 "How to use 'accepts_nested_attributes_for' to handle nested model fields.")
-  [A Rule of Thumb for Strong Parameters](http://patshaughnessy.net/2014/6/16/a-rule-of-thumb-for-strong-parameters "Strong Parameters and Nested Attributes")
-  [Forum Series Part 3: Nested Attributes and fields_for](https://gorails.com/episodes/forum-nested-attributes-and-fields-for "Learn how to use accepts_nested_attributes_for and fields_for to create forms that include associated models")
-  [Devise - Requiring password on email change](http://stackoverflow.com/questions/5413549/ruby-on-rails-devise-requiring-password-on-email-change "How to require the user to enter a password when updating certain attributes")
-  [Still getting "Current password can't be blank" in Registration Edit](http://stackoverflow.com/questions/18153978/still-getting-current-password-cant-be-blank-in-registration-edit-after-follo "Allow users to update custom fields without a password (Rails 4)")
