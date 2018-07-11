---
title: Authentication with Devise and cancancan in Rails 4.2
layout: post
permalink: /authentication-with-devise-and-cancancan-in-rails-4-2/
tags:
  - authentication
  - authorization
  - cancancan
  - devise
  - ruby
  - rails
excerpt_separator: <!--more-->
old-comments: authentication-with-devise-and-cancancan-in-rails-4-2.html
---

This is a beginner level tutorial on how to set up authentication (verifying who you are) and authorization (what you are permitted to do) using Ruby 2.2, Rails 4.2 and two popular Ruby gems: [Devise](https://github.com/plataformatec/devise "Flexible authentication solution for Rails with Warden") and [cancancan](https://github.com/CanCanCommunity/cancancan "Continuation of CanCan, the authorization Gem for Ruby on Rails").

The code for this tutorial is on GitHub: [https://github.com/hibbard-eu/authentication-with-devise-and-cancancan](https://github.com/hibbard-eu/authentication-with-devise-and-cancancan "Project code")

<!--more-->

## The Scenario

The app we'll be coding is a store. In order for people to use the store, they'll need to register an account. The store will also have sellers (otherwise it would be a rubbish store) and an admin.

This means we'll need the following resources: `Item`, `User`, `Role`.

Here's a UML diagram showing how they relate to one another:

![UML diagram depicting the assosciiations between the three resources in the app](https://res.cloudinary.com/hibbard/image/upload/v1530101542/uml-diagram-devise-cancancan.png "UML diagram depicting the assosciiations between the three resources in the app")

Note that users can have a maximum of one role. The permissions for each of these users will break down as follows:

-  Unregistered users are redirected to the sign up page
-  Registered users: can view items
-  Sellers: can view items, create items, as well as update and destroy any items that belong to them
-  Admin: can perform any CRUD operation on any resource

So let's get started.

```rb
rails new store
cd store
rails g scaffold user name:string role:belongs_to
rails g scaffold role name:string description:string
rails g scaffold item name:string description:text 'price:decimal{5,2}', user:belongs_to
rake db:migrate
```

If you're wondering what `price:decimal{5,2}` does, it generates the following in the migration file:

```rb
.decimal :price, precision: 5, scale: 2
```

A decimal with a precision of 5 and a scale of 2 can range from -999.99 to 999.99.

This will create a bunch of boilerplate code. We'll need to edit the view templates as shown below:

Remove `<p id="notice"><%= notice %></p>` from the top of the "index" and "show" templates in items, users and roles.

In `items/index.html.erb` and `items/show.html.erb`, change:

"User" to "Seller" and `item.user` to `item.user.name`

In `items/_form.html.erb` remove the complete user_id field (including the surrounding div tags)

In `users/index.html.erb` and `users/show.html.erb` change `user.role` to `user.role.name`

In `users/_form.html.erb` change `<%= f.text_field :role_id %>` to:

```erb
<%= collection_select(:user, :role_id, Role.all, :id, :name, {prompt: true}) %>
```

At this point if you start up Webbrick (`rails s`) and visit `http://localhost:3000/items`, `/roles`, or `/users`, you can see that our basic scaffolding is working (albeit without any data) and all of the pieces are in place for implementing the authentication logic.

## Authentication with Devise

Devise is a full-featured authentication solution for Rails based on Warden. One of the things I like about it most is that it is built on a modularity concept which makes it easy to include only those features you need in your applications (although in this tutorial I will go with the default configuration).

To get started with Devise, add it to our project's Gemfile:

```sh
gem "devise"
```

and run `bundle install`.

Then run the Devise's installation generator:

```sh
rails g devise:install
```

This command generates a couple of files, an initializer and a locale file that contains all of the messages that Devise needs to display.

As per the post installation message, we'll need to make a couple of alterations to our config files.

Add the following line to the bottom of `config/environments/development.rb`:

```rb
config.action_mailer.default_url_options = {:host => 'localhost:3000'}
```

And set a root route in `/config/routes.rb`

```rb
root to: "items#index"
```

We're going to use the `User` model to handle authentication and Devise provides a generator for doing just that:

```sh
rails g devise User
rake db:migrate
```

Devise won't override our current `User` model – it will simply add a bunch of attributes to it.

Edit the `/app/views/layouts/application.html.erb` to include the following immediately before the `<%= yield %>`:

```erb
<% if user_signed_in? %>
  Signed in as <%= current_user.email %>. Not you?
  <%= link_to "Edit profile", edit_user_registration_path %>
  <%= link_to "Sign out", destroy_user_session_path, :method => :delete %>
<% else %>
  <%= link_to "Sign up", new_user_registration_path %> or
  <%= link_to "sign in", new_user_session_path %>
<% end %>

<% flash.each do |name, msg| %>
  <%= content_tag :div, msg, id: "flash_#{name}" %>
<% end %>
```

And add the following line to the top of `ItemsController`, `RolesController` and `UsersController`:

```rb
before_filter :authenticate_user!
```

This will ensure that the user is logged in before being able to access any of these resources.

Now is the time to add some data. Alter `db/seeds.rb` thus:

```rb
r1 = Role.create({name: "Regular", description: "Can read items"})
r2 = Role.create({name: "Seller", description: "Can read and create items. Can update and destroy own items"})
r3 = Role.create({name: "Admin", description: "Can perform any CRUD operation on any resource"})

u1 = User.create({name: "Sally", email: "sally@example.com", password: "aaaaaaaa", password_confirmation: "aaaaaaaa", role_id: r1.id})
u2 = User.create({name: "Sue", email: "sue@example.com", password: "aaaaaaaa", password_confirmation: "aaaaaaaa", role_id: r2.id})
u3 = User.create({name: "Kev", email: "kev@example.com", password: "aaaaaaaa", password_confirmation: "aaaaaaaa", role_id: r2.id})
u4 = User.create({name: "Jack", email: "jack@example.com", password: "aaaaaaaa", password_confirmation: "aaaaaaaa", role_id: r3.id})

i1 = Item.create({name: "Rayban Sunglasses", description: "Stylish shades", price: 99.99, user_id: u2.id})
i2 = Item.create({name: "Gucci watch", description: "Expensive timepiece", price: 199.99, user_id: u2.id})
i3 = Item.create({name: "Henri Lloyd Pullover", description: "Classy knitwear", price: 299.99, user_id: u3.id})
i4 = Item.create({name: "Porsche socks", description: "Cosy footwear", price: 399.99, user_id: u3.id})
```

Then run:

```sh
rake db:seed
```

Now, we can start up webbrick and log in using the email address and password of one of the users we defined in the seeds file. Exciting times, huh?


## A Bit More About Devise

As I mentioned briefly above, Devise is based on a modularity concept. To elucidate that, take a peek in the `User` model. You should see:

```rb
class User < ActiveRecord::Base
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable
  belongs_to :role
end
```

In its default configuration, Devise comes with six of ten modules enabled.

You can see these in action in our app – for example Rememberable remembers the user's authentication in a saved cookie (embodied by the checkbox on the login form that says "Remember me").

If you didn't require this functionality, simply comment out `:rememberable` and it won't be included.

I would encourage you to take the time to read through what all of the modules do. They are listed on [the project's readme](https://github.com/plataformatec/devise#readme "Devise's readme on its GitHub project page")

Now, to get our app working properly, we'll going to need to make a few tweaks.

Let's start by namespacing the CRUD interface. This is necessary as otherwise [the user registration routes and user managing routes can conflict](https://github.com/plataformatec/devise/wiki/How-To:-Manage-users-through-a-CRUD-interface "How To: Manage users through a CRUD interface"). Alter `config/routes.rb` like so:

```rb
devise_for :users
scope "/admin" do
  resources :users
end
```

Then, in the `User` model, we can add the missing association (`app/models/user.rb`):

```rb
has_many :items
```

And in the `ItemsController`, make sure that a user is assosciated with each item before it is saved:

```rb
def create
  @item.user_id = current_user.id
  ...
end
```

Once you have restarted the server, you will be able to manage users at `localhost:3000/admin/users`.

Now, if you click on the "sign up" or "edit profile" links, you'll notice that the user's name attribute is missing. It would be nice to have users be able to specify their names when registering, so let's fix that.

Devise comes with many views built-in. If we would like to customize these pages, we must transfer the Devise view files into our project so we can modify them. Luckily, Devise has a generator to do just that:

```rb
rails g devise:views
```

This will copy across a bunch of templates to the `app/views/devise` directory.

Cd into this folder, then into the sub folder `registrations`. Locate the files `new.html.erb` and `edit.html.erb` and add the following just after `<%= devise_error_messages! %>`:

```erb
<div class="field">
  <%= f.label :name %><br />
  <%= f.text_field :name %>
</div>
```

This will add the correct fields to the appropriate forms, but were you to attempt to fill them out at this point, you would notice that Devise isn't saving your changes.

The reason for this is that the extra parameter you are passing to the controller (`name` in this case) needs to be whitelisted.

You can do this as follows in the `ApplicationController`:

```rb
before_filter :configure_permitted_parameters, if: :devise_controller?

protected
def configure_permitted_parameters
  devise_parameter_sanitizer.for(:sign_up) << :name
  devise_parameter_sanitizer.for(:account_update) << :name
end
```

Also, when a user signs up, they need to be assigned a role. We can make this default to "Regular". And while we're at it, we can also add some validation to make sure a user enters a name.

To do this, alter `/app/models/user.rb` as follows:

```rb
validates_presence_of :name
before_save :assign_role

def assign_role
  self.role = Role.find_by name: "Regular" if self.role.nil?
end
```

Take a moment at this point to start up the app again and make sure that everything is working. It is? Good.

The final thing we're going to do, is give the admin the power to create new users via the admin interface, as well as to edit any of the existing users' attributes.

First off, let's add the user's email address to the views.

`app/views/users/index.html.erb`:

```erb
<th>Email</th>
<td><%= user.email %></td>
```

`app/views/users/show.html.erb`:

```erb
<p>
  <strong>Email:
  <%= @user.email %>
</p>
```

`app/views/users/_form.html.erb`:

```erb
<div class="field">
  <%= f.label :email %><br>
  <%= f.text_field :email %>
</div>
```

In the form partial, we can also add a password field:

```erb
<div class="field">
  <%= f.label :password %><br>
  <%= f.password_field :password, placeholder: "Leave blank if unchanged" %>
</div>
```

Now things get a little complicated, so as to be able to create a new user via our admin interface (`http://localhost:3000/admin/users/new`), we need to whitelist the attributes that Devise requires:

`/app/controllers/users_controller.rb`

```rb
def user_params
  params.require(:user).permit(
    :email,
    :password,
    :password_confirmation,
    :name,
    :role_id
  )
end
```

That's ok, but if we try and edit an existing user (e.g. `http://localhost:3000/admin/users/1/edit`), then we are prompted to set a new password every time we want to update something. This is not ideal.

To get around this, we can take advantage of Devise's [update_without_password](http://www.rubydoc.info/github/plataformatec/devise/Devise/Models/DatabaseAuthenticatable:update_without_password "Method: Devise::Models::DatabaseAuthenticatable#update_without_password") method.

Alter the update method in `UsersController` as shown (making sure to include the protected method `needs_password?`):

```rb
def update
  if user_params[:password].blank?
    user_params.delete(:password)
    user_params.delete(:password_confirmation)
  end

  successfully_updated = if needs_password?(@user, user_params)
                           @user.update(user_params)
                         else
                           @user.update_without_password(user_params)
                         end

  respond_to do |format|
    if successfully_updated
      format.html { redirect_to @user, notice: 'User was successfully updated.' }
      format.json { head :no_content }
    else
      format.html { render action: 'edit' }
      format.json { render json: @user.errors, status: :unprocessable_entity }
    end
  end
end

protected
def needs_password?(user, params)
  params[:password].present?
end
```

Before we finish, to give us a little more info on the user, add the following to `/app/views/users/show.html.erb`:

```erb
<p>
  <strong>Joined on:</strong>
  <%= @joined_on %>
</p>
<p>
  <strong>Last logged in on:</strong>
  <%= @last_login %>
</p>
<p>
  <strong>No. times logged in:</strong>
  <%= @user.sign_in_count %>
</p>
```

And the logic to the `UsersController`:

```rb
def show
  @joined_on = @user.created_at.to_formatted_s(:short)
  if @user.current_sign_in_at
    @last_login = @user.current_sign_in_at.to_formatted_s(:short)
  else
    @last_login = "never"
  end
end
```

Also, to show us which users are associated with a particular role add the association to the `Role` model.

`app/models/role.rb`:

```rb
has_many :users
```

Add the following to app/views/roles/show.html.erb:

```erb
<p>
  <strong>Assosciated users:</strong>
  <%= @assosciated_users %>
</p>
```

And the logic to the `RolesController`:

```rb
def show
  if @role.users.length == 0
    @assosciated_users = "None"
  else
    @assosciated_users = @role.users.map(&:name).join(", ")
  end
end
```

And there we go. In not very much code we have implemented a robust authentication solution for our app, as well as building a simple interface to administer users.

Take a moment to restart the app and have a play with what we've got so far.

## Authorization With cancancan

[Cancancan](https://github.com/CanCanCommunity/cancancan "Continuation of CanCan, the authorization Gem for Ruby on Rails") is an authorization library for Ruby on Rails which restricts what resources a given user is allowed to access. It is the continuation of the now defunct cancan project started by Ryan Bates (of Railscast fame).

To install it, add the following line to your Gemfile:

```sh
gem 'cancancan', '~> 1.10'
```

and run `bundle install`.

CanCanCan expects a `current_user` method to exist in the controller, which we have thanks to Devise.

It also expects user permissions to be defined in an Ability class, for which it includes a generator:

```sh
rails g cancan:ability
```

This will generate a file at `app/models/ability.rb`. Edit it like so:

```rb
class Ability
  include CanCan::Ability

  def initialize(user)
    if user.admin?
      can :manage, :all
    else
      can :read, :all
    end
  end
end
```

As you can see, abilities are defined with the method `can`. This method takes two parameters: the first is the action that we want to perform and the second is the model class that the action applies to. In the above example we are permitting the admin to perform any CRUD action on any resource within our app and restricting all other users to just being able to read.

We don't care about the permissions of those users who have not yet logged in (you remember they should just be directed to the sign in page), but if we did (for example, if they could view items), you would have to initialize the user variable to something sensible, or you would end up checking for nil values all over the place.

```rb
def initialize(user)
  user ||= User.new # Guest user
  ...
end
```

Next we need to define an `admin?` method in our `User` model.

`app/models/user.rb`:

```rb
def admin?
  self.role.name == "Admin"
end
```

And in the `UsersController` we need to specify which users are authorized to do what. You can do this on a per action basis using the `authorize!` method, which in turn performs the `can?` check and raises an exception if needed.

For example to check if a given user can edit an item:

```rb
def edit
  authorize! :edit, @item
end
```

However, repeating this across every action in our app would soon become tiresome. Luckily there is an easier way to do this if you are using RESTful style controllers, namely with the method `load_and_authorize_resource`. As the name suggests, this method loads the appropriate resource and authorizes it in a before filter.

Place it at the top of your `ItemsController`.

```rb
load_and_authorize_resource
```

As this method loads the necessary resource for us based on the action we are performing, we can also remove any lines of code that set the instance variable in each action.

In the case of our `ItemsController` that would mean removing the following lines:

```diff
class ItemsController < ApplicationController
-  before_action :set_item, only: [:show, :edit, :update, :destroy]

  def new
-   @item = Item.new
  end

  def create
-    @item = Item.new(item_params)
    ...
  end

  private
-  def set_item
-    @item = Item.find(params[:id])
-  end
end
```

If you now restart the server and access `localhost:3000/items` as a regular user or as a seller, you will be in read only mode. As an admin, you will still be able to create, edit and delete records. Cool!

Now we need to repeat the above steps for our other two resources:

`app/controllers/users_controller.rb`

```diff
class UsersController < ApplicationController
-  before_action :set_user, only: [:show, :edit, :update, :destroy]
  load_and_authorize_resource

  def new
-    @user = User.new
  end

  def create
-    @user = User.new(user_params)
    ...
  end

  private
-  def set_user
-    @user = User.find(params[:id])
-  end
end
```

`app/controllers/roles_controller.rb`

```diff
class RolesController < ApplicationController
-  before_action :set_role, only: [:show, :edit, :update, :destroy]
  load_and_authorize_resource

  def new
-    @role = Role.new
  end

  def create
-    @role = Role.new(role_params)
    ...
  end

  private
-  def set_role
-    @role = Role.find(params[:id])
-  end
endv
```

This ensures that regular users and sellers are also in read-only mode when accessing Roles and Users.

Next, we can ensure that our users see a nicer error page than the one with the AccessDenied exception they currently see when attempting to access something they are not authorized to.

We can do this using a method called [rescue_from](http://api.rubyonrails.org/classes/ActiveSupport/Rescuable/ClassMethods.html "ActiveSupport::Rescuable::ClassMethods") that we can place in our `ApplicationController`. We'll pass it a block inside of which we'll make the application show a flash error message and redirect to the home page.

`app/controllers/application_controller.rb`

```rb
rescue_from CanCan::AccessDenied do |exception|
  flash[:error] = "Access denied!"
  redirect_to root_url
end
```

Now all that remains to do is to set the permissions of regular users and sellers to something sensible.

Let's define two more methods to check the user's current role:

`app/models/user.rb`

```rb
def seller?
  self.role.name == "Seller"
end
def regular?
  self.role.name == "Regular"
end
```

Then it's a matter of updating the abilities in the Ability class.

`app/models/ability.rb`

```rb
if user.admin?
  can :manage, :all
elsif user.seller?
  can :read, Item
  can :create, Item
  can :update, Item do |item|
    item.try(:user) == user
  end
  can :destroy, Item do |item|
    item.try(:user) == user
  end
elsif user.regular?
  can :read, Item
end
```

The permissions for admins and regular users should be straight forward.

In the case of sellers however, things become a little more complicated when dealing with the update and destroy actions (you remember that sellers should only be able to update and delete their own items).

Here we pass `can` a block which will pass in the instance of the model we're checking. The block should return `true` or `false` depending on whether the action should be allowed, or not. Within the block we'll use Rails' [try](http://apidock.com/rails/Object/try "Invokes the public method whose name goes as first argument. If the receiver does not respond to it the call returns nil rather than raising an exception") method to check that the item's `user` attribute is the current user. Using `try` covers the eventuality that the item is `nil` and will prevent an exception being raised.

Finally, so that regular users and sellers don't see any links to actions they are not permitted to perform, alter `app/views/items/index.html.erb` like so:

```erb
<td>
  <% if can? :update, item %>
    <%= link_to 'Edit', edit_item_path(item) %>
  <% end %>
</td>

<td>
  <% if can? :destroy, item %>
    <%= link_to 'Destroy', item, method: :delete, data: { confirm: 'Are you sure?' } %>
  <% end %>
</td>

<% if can? :create, Item %>
  <%= link_to 'New Item', new_item_path %>
<% end %>
```

## The Finishing Touches

The very last thing I want to examine (promise!) is a slightly lesser documented feature of Devise. You remember that we initially wanted users who are not logged in to be redirected to the sign-in page? Well, wouldn't it be nicer if we directed them to a welcome page with a link to either register or sign in?

To do this we need a `WelcomeController` and a corresponding view:

`app/controllers/welcome_controller.rb`

```rb
class WelcomeController < ApplicationController
  def index
  end
end
```

`app/views/welcome/index.html.erb`

```html
<h1>Welcome to the Store!</h1>
<h2>Selling you useless crap since 2015</h2>
```

Once that is in place, alter your `config/routes.rb` to take advantage of Devise's `authenticated` route thus:

```rb
authenticated :user do
  root :to => 'items#index', as: :authenticated_root
end
root :to => 'welcome#index'
```

Restart your server and we're done!

In case you missed it at the beginning, the code for this tutorial is on GitHub: [https://github.com/hibbard-eu/authentication-with-devise-and-cancancan](https://github.com/hibbard-eu/authentication-with-devise-and-cancancan "Project code")

### Reference

-  [Railscast#209 Introducing Devise](http://railscasts.com/episodes/209-introducing-devise "Devise is a full-featured authentication solution which handles all of the controller logic and form views for you. Learn how to set it up in this episode.")
-  [Railscasts#192 Authorization with CanCan](http://railscasts.com/episodes/192-authorization-with-cancan "CanCan is a simple authorization plugin that offers a lot of flexibility. See how to use it in this episode.")
-  [Add Custom Field/Column to Devise with Rails 4](http://stackoverflow.com/questions/16297797/add-custom-field-column-to-devise-with-rails-4 "Add a full_name field/column to `User` model (using the Devise gem) and Rails 4.")
-  [How to Use Devise in Rails for Authentication](http://www.gotealeaf.com/blog/how-to-use-devise-in-rails-for-authentication "Covers testing")
-  [How to manage users in Rails 4 (using Devise)](http://andowebsit.es/blog/noteslog.com/post/how-to-manage-users-in-rails-4-using-devise/ "Advise on changing the update action")
-  [Strong parameters in Devise](https://github.com/plataformatec/devise#strong-parameters "When you customize your own views, you may end up adding new attributes to forms")
-  [Devise Gem – How to Manage Users through CRUD interface](http://stackoverflow.com/questions/5721276/rails-3-devise-gem-how-to-manage-users-through-crud-interface "More tips")
-  [Redirect to log in page if user is not authenticated with Devise](http://stackoverflow.com/questions/23555618/redirect-to-log-in-page-if-user-is-not-authenticated-with-devise "The recommended way to redirect unauthenticated users to the sessions#new page if they attempt to access a page that requires authentication")
-  <a href="http://stackoverflow.com/questions/5725094/different-root-path-for-users-depending-if-they-are-authenticated-using-dev" target="_blank">Different '/' root path for users depending if they are authenticated (using Devise)</a>
-  [Rails Tip #5: Authenticated Root and Dashboard Routes With Devise](http://excid3.com/blog/rails-tip-5-authenticated-root-and-dashboard-routes-with-devise/ "Authenticated routes in Devise")
-  [Devise – How To: Disable user from destroying their account](https://github.com/plataformatec/devise/wiki/How-To:-Disable-user-from-destroying-their-account "This might be quite useful")
-  [Devise – How To: Allow users to edit their account without providing a password](https://github.com/plataformatec/devise/wiki/How-To%3a-Allow-users-to-edit-their-account-without-providing-a-password "Because sometimes developers want to create other actions that allow the user to change their information without requiring a password.")
-  [Preventing user deletion in Ruby On Rails and Devise Application](http://stackoverflow.com/questions/16249776/preventing-user-deletion-in-ruby-on-rails-and-devise-application "Solution from StackOverflow")

So, this was quite a long post — I hope it proves useful for people. If you have any questions or observations, I'd be glad to hear them in the comments.
