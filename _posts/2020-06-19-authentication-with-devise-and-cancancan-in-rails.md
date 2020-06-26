---
title: Authentication with Devise and cancancan in Rails
layout: post
permalink: /authentication-with-devise-and-cancancan-in-rails/
tags:
  - authentication
  - authorization
  - cancancan
  - devise
  - ruby
  - rails
excerpt_separator: <!--more-->
twitter:
  title: "Authentication with Devise and cancancan in Rails"
  description: "Learn how to set up authentication (verifying who you are) and authorization (what you may do) in a Rails app with Devise and Cancancan."
  image_url: https://res.cloudinary.com/hibbard/image/upload/f_auto,w_800/v1593027720/stock/lock.jpg
---

This is a tutorial on how to set up authentication (verifying who you are) and authorization (what you are permitted to do) using Ruby 2.7, Rails 6.0.3 and two popular Ruby gems: [Devise](https://github.com/heartcombo/devise "Flexible authentication solution for Rails with Warden") and [cancancan](https://github.com/CanCanCommunity/cancancan "Continuation of CanCan, the authorization Gem for Ruby on Rails").

The code for this tutorial is on GitHub: [https://github.com/hibbard-eu/authentication-with-devise-and-cancancan](https://github.com/hibbard-eu/authentication-with-devise-and-cancancan "Project code")

<!--more-->

## The Scenario

The app we'll be coding is a store. In order for people to use the store, they'll need to register an account. The store will also have sellers (otherwise it would be a rubbish store) and an admin.

This means we'll need the following resources: `Item`, `User`, `Role`.

Here's a UML diagram showing how they relate to one another:

![UML diagram depicting the assosciiations between the three resources in the app](https://res.cloudinary.com/hibbard/image/upload/v1593028898/rails-devise-cancancan/uml-diagram.png "UML diagram depicting the assosciiations between the three resources in the app")

Note that users can have a maximum of one role. The permissions for each of these users will break down as follows:

-  Unregistered users are redirected to the sign up page
-  Registered users: can view items
-  Sellers: can view items, create items, as well as update and destroy any items that belong to them
-  Admin: can perform any CRUD operation on any resource

So let's get started.

## Generating the Project Files

Let's start by creating a new project.

```bash
rails new store
```

In its current version, Rails uses [Webpacker](https://github.com/rails/webpacker) as its default JavaScript compiler. Webpacker expects us to have both [Node.js](https://nodejs.org/en/) and the [Yarn package manager](https://yarnpkg.com/) installed. If you don't have Node installed already, I would recommend using a [version manager](https://www.sitepoint.com/quick-tip-multiple-versions-node-nvm/) which will let you switch between versions with ease and which also negates certain permission errors.

For this tutorial, I am using the current LTS version of Node (12.18.1) and the latest version of Yarn (1.22.4). Yarn should be installed globally using `npm -i g yarn`.

Once the project has been created, we can change into the `store` directory and remove the following line from our `Gemfile`:

```diff
- gem 'jbuilder', '~> 2.7'
```

[Jbuilder](https://github.com/rails/jbuilder) is used for generating and rendering JSON responses for API requests in Rails. We won't be needing this functionality.

Next, let's use the [scaffold generator](https://www.rubyguides.com/2020/03/rails-scaffolding/) to create our project files:

```bash
rails g scaffold user name:string role:belongs_to
rails g scaffold role name:string description:string
rails g scaffold item name:string description:text 'price:decimal{5,2}', user:belongs_to
```

If you're wondering what `price:decimal{5,2}` does, it adds the following to the migration file:

```rb
.decimal :price, precision: 5, scale: 2
```

A decimal with a precision of 5 and a scale of 2 can range from -999.99 to 999.99.

Finally, run the migrations with the following command.

```bash
rake db:migrate
```

## Editing the Boilerplate

We'll need to tailor the files that Rails has generated to suit our needs.

Start by removing the following line from the top of the `index` and `show` view templates for the `User`, `Role` and `Item` resources.

```diff
- <p id="notice"><%= notice %></p>
```

In `items/index.html.erb` change "User" to "Seller" and `item.user_id` to `item.user.name`.

In `items/show.html.erb` change "User" to "Seller" and `@item.user_id` to `@item.user.name`.

In `items/_form.html.erb` remove the complete `user_id` field (including the surrounding div tags).

```diff
- <div class="field">
-   <%= form.label :user_id %>
-   <%= form.text_field :user_id %>
- </div>
```

In `users/index.html.erb` change `user.role_id` to `user.role.name`

In `users/show.html.erb` change `@user.role_id` to `@user.role.name`

In `users/_form.html.erb` change `<%= form.text_field :role_id %>` to:

```erb
<%= collection_select(
  :user, :role_id, Role.all, :id, :name, { prompt: true }
) %>
```

At this point if you start up Puma (`rails s`) and visit [http://localhost:3000/items](http://localhost:3000/items), [http://localhost:3000/roles](http://localhost:3000/roles), or [http://localhost:3000/users](http://localhost:3000/users), you can see that our basic scaffolding is working (albeit without any data)

![Create a new item](https://res.cloudinary.com/hibbard/image/upload/v1593075472/rails-devise-cancancan/basic-scaffold.png "Rails CRUD interface to create a new item")

Everything is now setup to implement the authentication logic.

## Authentication with Devise

[Devise](https://github.com/heartcombo/devise) is a full-featured authentication solution for Rails based on Warden. One of the things I like about it most (aside from the ease of use) is that it is built on a modularity concept. This makes it easy to include only those features you need in your application.

To get started with Devise, add it to our project's Gemfile:

```bash
bundle add devise
```

Then run Devise's installation generator:

```bash
rails g devise:install
```

This command generates a couple of files: an initializer and a locale file that contains all of the messages that Devise needs to display.

As per the post installation message, we'll need to make a couple of alterations to our config files.

Add the following line to the bottom of `config/environments/development.rb`:

```rb
config.action_mailer.default_url_options = { host: 'localhost:3000' }
```

And set a root route in `/config/routes.rb`

```rb
root to: 'items#index'
```

We're going to use the `User` model to handle authentication and Devise provides a generator for doing just that:

```bash
rails g devise User
rake db:migrate
```

Devise won't override our current `User` model – it will simply add a bunch of attributes to it.

Edit the `/app/views/layouts/application.html.erb` to include the following immediately before the `<%= yield %>`:

```erb
<% if user_signed_in? %>
  Signed in as <%= current_user.email %>. Not you?
  <%= link_to "Edit profile", edit_user_registration_path %>
  <%= link_to "Sign out", destroy_user_session_path, method: :delete %>
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
before_action :authenticate_user!
```

This will ensure that the user is logged in before being able to access any of these resources.

Now is the time to add some data. Alter `db/seeds.rb` thus:

```rb
r1 = Role.create({ name: 'Regular', description: 'Can read items' })
r2 = Role.create({ name: 'Seller', description: 'Can read and create items. Can update and destroy own items' })
r3 = Role.create({ name: 'Admin', description: 'Can perform any CRUD operation on any resource' })

u1 = User.create({ name: 'Sally', email: 'sally@example.com', password: 'aaaaaaaa', password_confirmation: 'aaaaaaaa', role_id: r1.id })
u2 = User.create({ name: 'Sue', email: 'sue@example.com', password: 'aaaaaaaa', password_confirmation: 'aaaaaaaa', role_id: r2.id })
u3 = User.create({ name: 'Kev', email: 'kev@example.com', password: 'aaaaaaaa', password_confirmation: 'aaaaaaaa', role_id: r2.id })
u4 = User.create({ name: 'Jack', email: 'jack@example.com', password: 'aaaaaaaa', password_confirmation: 'aaaaaaaa', role_id: r3.id })

i1 = Item.create({ name: 'Rayban Sunglasses', description: 'Stylish shades', price: 99.99, user_id: u2.id })
i2 = Item.create({ name: 'Gucci watch', description: 'Expensive timepiece', price: 199.99, user_id: u2.id })
i3 = Item.create({ name: 'Henri Lloyd Pullover', description: 'Classy knitwear', price: 299.99, user_id: u3.id })
i4 = Item.create({ name: 'Porsche socks', description: 'Cosy footwear', price: 399.99, user_id: u3.id })
```

Then run:

```bash
rake db:seed
```

Finally, we can restart the Rails server and log in using the email address and password of one of the users we defined in the seeds file.

Notice that if we are logged out and try and access any of the protected resources, we are redirected to a log in page with the message "You need to sign in or sign up before continuing."

![Log in page with the message "You need to sign in or sign up before continuing."](https://res.cloudinary.com/hibbard/image/upload/v1593075954/rails-devise-cancancan/sign-in-with-devise.png "Devise prevents unauthorized users from accessing resources")

Exciting times, huh?

## A Bit More About Devise

As I mentioned briefly above, Devise is based on a modularity concept. To expand on that, take a peek at the `User` model. You should see:

```rb
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable
  belongs_to :role
end
```

In its default configuration, Devise comes with five of ten modules enabled.

You can see these in action in our app – for example `Rememberable` remembers the user's authentication in a saved cookie (embodied by the checkbox on the login form that says "Remember me").

If you didn't require this functionality, simply comment out `:rememberable` and it won't be included.

I would encourage you to take the time to read through what all of the modules do. They are listed on [the project's readme](https://github.com/heartcombo/devise#readme "Devise's readme on its GitHub project page")

Now, to get our app working properly, we'll going to need to make a few tweaks.

## Adding an Admin Namespace

Let's start by namespacing the CRUD interface. This is necessary as otherwise [the user registration routes and user management routes can conflict](https://github.com/heartcombo/devise/wiki/How-To:-Manage-users-through-a-CRUD-interface "How To: Manage users through a CRUD interface"). Alter `config/routes.rb` like so:

```rb
devise_for :users
scope '/admin' do
  resources :users
end
```

Then, in the `User` model, we can add the missing association (`app/models/user.rb`):

```rb
has_many :items, dependent: :destroy
```

We are specifying `dependent: :destroy` as if we delete a seller, it also makes sense to delete the items they are selling.

And in the `ItemsController`, make sure that a user is assosciated with each item before it is saved:

```rb
def create
  @item = Item.new(item_params)
  @item.user_id = current_user.id
  ...
end
```

Once you have restarted the server, you will be able to (kinda) manage users at [http://localhost:3000/admin/users](http://localhost:3000/admin/users). I write kinda, as you'll not yet be able to create new users. We'll get to that a little later.

## Customizing Devise

Now, if you click on the "sign up" or "edit profile" links, you'll notice that the user's name attribute is missing. It would be nice to have users be able to specify their names when registering, so let's fix that.

Devise comes with many views built-in. If we would like to customize these pages, we must transfer the Devise view files into our project so we can modify them. Luckily, Devise has a generator to do just that:

```rb
rails g devise:views
```

This will copy across a bunch of templates to the `app/views/devise` directory.

Change into this folder, then into the `registrations` sub-folder. Locate the `new.html.erb` and `edit.html.erb` files and add the following just after `<%= render "devise/shared/error_messages", resource: resource %>`:

```erb
<div class="field">
  <%= f.label :name %><br />
  <%= f.text_field :name %>
</div>
```

This will add the correct fields to the appropriate forms, but were you to attempt to fill them out at this point, you would notice that Devise isn't saving your changes.

The reason for this is that the extra parameter you are passing to the controller (`name` in this case) needs to be expressly permitted.

You can do this as follows in the `ApplicationController`:

```rb
class ApplicationController < ActionController::Base
  before_action :configure_permitted_parameters, if: :devise_controller?

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:name])
    devise_parameter_sanitizer.permit(:account_update, keys: [:name])
  end
end
```

Also, when a user signs up, they need to be assigned a role. We can make this default to "Regular". And while we're at it, we can also add some validation to make sure a user enters a name.

To do this, alter `/app/models/user.rb` as follows:

```rb
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable

  belongs_to :role, optional: true
  has_many :items, dependent: :destroy
  validates :name, presence: true
  before_save :assign_role

  def assign_role
    self.role = Role.find_by name: 'Regular' if role.nil?
  end
end
```

Take a moment at this point to start up the app again and make sure that everything is working. It is? Good.

## Creating New Users

The final thing we're going to do, is give the admin the power to create new users via the admin interface, as well as to edit any of the existing users' attributes.

First off, let's add the user's email address to the views.

`app/views/users/index.html.erb`:

```erb
<th>Role</th>
<th>Email</th>
<th colspan="3"></th>
```

```erb
<td><%= user.role.name %></td>
<td><%= user.email %></td>
<td><%= link_to 'Show', user %></td>
```

`app/views/users/show.html.erb`:

```erb
<p>
  <strong>Email:</strong>
  <%= @user.email %>
</p>
```

`app/views/users/_form.html.erb`:

```erb
<div class="field">
  <%= form.label :email %><br>
  <%= form.text_field :email %>
</div>
```

In the form partial, we can also add a password field:

```erb
<div class="field">
  <%= form.label :password %><br>
  <%= form.password_field :password, placeholder: "Leave blank if unchanged" %>
</div>
```

Now things get a little complicated, so as to be able to create a new user via our admin interface ([http://localhost:3000/admin/users/new](http://localhost:3000/admin/users/new)), we need to permit the attributes that Devise requires:

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

That's ok, but if we try and edit an existing user (e.g. [http://localhost:3000/admin/users/1/edit](http://localhost:3000/admin/users/1/edit)), then we are prompted to set a new password every time we want to update something. This is not ideal.

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

  if successfully_updated
    redirect_to @user, notice: 'User was successfully updated.'
  else
    render :edit
  end
end

private

def needs_password?(_user, params)
  params[:password].present?
end
```

And with that we can now create, update and delete users.

![Creating a new user](https://res.cloudinary.com/hibbard/image/upload/v1593076446/rails-devise-cancancan/create-user.png "Creating a new user via our admin interface")

## Enabling the Trackable Module

Before we finish looking at Devise and authentication, let's enable the trackable module, to give us a little more information on our users.

In `/app/models/users.rb` alter the code like so:

```rb
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable
  ...
end
```

Create a new migration:

```bash
rails generate migration AddDeviseTrackableColumnsToUsers
```

This will create a new file in the `db/migrate` folder. Alter it like so:

```ruby
class AddDeviseTrackableColumnsToUsers < ActiveRecord::Migration[6.0]
  def self.up
    change_table :users do |t|
      t.integer  :sign_in_count, default: 0, null: false
      t.datetime :current_sign_in_at
      t.datetime :last_sign_in_at
      t.string   :current_sign_in_ip
      t.string   :last_sign_in_ip
    end
  end
end
```

Then run the migration with `rake db:migrate`. As you can see from the migration file, this will add several columns to the user table.

Now, add the following to `/app/views/users/show.html.erb`:

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
    @last_login = 'never'
  end
end
```

Also, to show us which users are associated with a particular role, add the association to the `Role` model.

`app/models/role.rb`:

```rb
has_many :users, dependent: :restrict_with_exception
```

The `restrict_with_exception` option will cause an `ActiveRecord::DeleteRestrictionError` exception to be raised if you try to delete a `Role` record, but it has associated `User` records.

Next, add the following to `app/views/roles/show.html.erb`:

```erb
<p>
  <strong>Assosciated users:</strong>
  <%= @assosciated_users %>
</p>
```

And the logic to the `RolesController`:

```rb
def show
  if @role.users.empty?
    @assosciated_user = 'None'
  else
    @assosciated_users = @role.users.map(&:name).join(', ')
  end
end
```

And there we go. In not very much code we have implemented a robust authentication solution for our app, as well as building a simple interface to administer users.

Take a moment to restart the app, have a play with what we've got so far and assure yourself that it is working.

## Authorization With cancancan

[Cancancan](https://github.com/CanCanCommunity/cancancan "Continuation of CanCan, the authorization Gem for Ruby on Rails") is an authorization library for Ruby on Rails which restricts what resources a given user is allowed to access. It is the continuation of the now defunct cancan project started by Ryan Bates (of Railscast fame).

To install it, run:

```bash
bundle add cancancan
```

CanCanCan expects a `current_user` method to exist in the controller, which we have thanks to Devise.

It also expects user permissions to be defined in an `Ability` class, for which it includes a generator:

```bash
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

We don't care about the permissions of those users who have not yet logged in (you remember they should just be directed to the sign in page), but if we did (for example, if they could view items), you would have to initialize the `user` variable to something sensible. Otherwise you would end up checking for nil values all over the place.

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
  role.name == 'Admin'
end
```

And in the `UsersController` we need to specify which users are authorized to do what. You can do this on a per action basis using the `authorize!` method, which in turn performs the `can?` check and raises an exception if needed.

For example to check if a given user can edit an item:

```rb
def edit
  authorize! :edit, @item
end
```

However, repeating this across every action in our app would soon become tiresome. Luckily there is an easier way to do this if you are using RESTful style controllers, namely with the method `load_and_authorize_resource`. As the name suggests, this method loads the appropriate resource and authorizes it in a before action.

Place it at the top of your `ItemsController`.

```rb
class ItemsController < ApplicationController
  before_action :authenticate_user!
  load_and_authorize_resource
  ...
end
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

If you now restart the server and access [http://localhost:3000/items](http://localhost:3000/items) as a regular user or as a seller, you will be in read only mode. As an admin, you will still be able to create, edit and delete records.

Now we need to repeat the above steps for our other two resources:

`app/controllers/users_controller.rb`

```diff
class UsersController < ApplicationController
-  before_action :set_user, only: [:show, :edit, :update, :destroy]
  before_action :authenticate_user!
+  load_and_authorize_resource

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
  before_action :authenticate_user!
+  load_and_authorize_resource

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
end
```

This ensures that regular users and sellers are also in read-only mode when accessing Roles and Users.

![CanCan::AccessDenied error when accessing Roles](https://res.cloudinary.com/hibbard/image/upload/v1593077285/rails-devise-cancancan/access-denied.png "You will see an AccessDenied error when trying to access a protected resource without the correct priviledges")

## A Nicer Error Page

Next, we can ensure that our users see a nicer error page than the one with the AccessDenied exception they currently see when attempting to access something they are not authorized to.

We can do this using a method called [rescue_from](http://api.rubyonrails.org/classes/ActiveSupport/Rescuable/ClassMethods.html "ActiveSupport::Rescuable::ClassMethods") that we can place in our `ApplicationController`. We'll pass it a block inside of which we'll make the application show a flash error message and redirect to the home page.

`app/controllers/application_controller.rb`

```rb
rescue_from CanCan::AccessDenied do
  flash[:error] = 'Access denied!'
  redirect_to root_url
end
```

Now all that remains to do is to set the permissions of regular users and sellers to something sensible.

Let's define two more methods to check the user's current role:

`app/models/user.rb`

```rb
def seller?
  role.name == 'Seller'
end

def regular?
  role.name == 'Regular'
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

Here we pass `can` a block which will receive the instance of the model we're checking. The block should return `true` or `false` depending on whether the action should be allowed, or not. Within the block we'll use Rails' [try](http://apidock.com/rails/Object/try "Invokes the public method whose name goes as first argument. If the receiver does not respond to it the call returns nil rather than raising an exception") method to check that the item's `user` attribute is the current user. Using `try` covers the eventuality that the item is `nil` and will prevent an exception being raised.

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

...

<% if can? :create, Item %>
  <%= link_to 'New Item', new_item_path %>
<% end %>
```

## The Finishing Touches

Before we end, let's add some navigation to `app/views/layouts/application.html.erb` so as to help admin users switch between resources.

```erb
<body>
  <div class="flex-container">
    <header>
      <% if user_signed_in? %>
        Signed in as <%= current_user.email %>.<br>
        Not you?
        <%= link_to "Edit profile", edit_user_registration_path %>
        <%= link_to "Sign out", destroy_user_session_path, method: :delete %>
      <% else %>
        <%= link_to "Sign up", new_user_registration_path %> or
        <%= link_to "sign in", new_user_session_path %>
      <% end %>

      <nav>
        <% if @current_user&.admin? %>
          <%= link_to "Items", items_path %> |
          <%= link_to "Users", users_path %> |
          <%= link_to "Roles", roles_path %>
        <% end %>
      </nav>
    </header>

    <% flash.each do |name, msg| %>
      <%= content_tag :div, msg, id: "flash_#{name}" %>
    <% end %>

    <main>
      <%= yield %>
    </main>
  </div>
</body>
```

The extra markup lets us add some basic styling to make the app more visually appealing. I'm not going to list the CSS here, you can copy it out of the [scaffold.scss file on GitHub](https://github.com/jameshibbard/authentication-with-devise-and-cancancan/blob/master/app/assets/stylesheets/scaffolds.scss). The table styling is courtesy of [W3schools](https://www.w3schools.com/Css/css_table.asp).

When you've applied the styles, here's what the app should look like:

![The finished app listing items in the store](https://res.cloudinary.com/hibbard/image/upload/v1593077992/rails-devise-cancancan/finished-app.png "The finished app listing items in the store")

Next, we can add some JavaScript to fade out, then remove our flash messages after a delay of 3.5 seconds. To this end, make a new folder named `src` in the `app/javascript` folder. In `app/javascript/src` create a file named `index.js` and add the following:

```js
document.addEventListener('DOMContentLoaded', () => {
  const flashMessage = document.querySelector('div[id^="flash_"]');

  if (flashMessage) {
    setTimeout(() => {
      flashMessage.classList.add('hide');
      flashMessage.classList.remove('show');

      setTimeout(() => {
        flashMessage.parentElement.removeChild(flashMessage);
      }, 1500);
    }, 3500);
  }
});
```

Then require it in `app/javascript/packs/application.js` to ensure it is included with our JavaScript bundle.

```js
require('@rails/ujs').start();
require('turbolinks').start();
require('@rails/activestorage').start();
require('channels');

require('../src/index');
```

Finally, the very last thing I want to examine (promise!) is a slightly lesser documented feature of Devise. You remember that we initially wanted users who are not logged in to be redirected to the sign-in page? Well, wouldn't it be nicer if we directed them to a welcome page with a link to either register or sign in?

To do this we need a `WelcomeController` and a corresponding view. These can be generated with the following command:

```bash
rails g controller welcome index
```

Now edit the newly generated view template at `app/views/welcome/index.html.erb`:

```html
<h1>Welcome to the Store!</h1>
<h2>Selling you things you don't need since 2020</h2>
<p>
  Lorem ipsum dolor sit amet, consectetur adipisicing elit. Adipisci
  eveniet, repellat quae sed hic obcaecati nam exercitationem saepe
  quod totam dolore explicabo culpa iure deserunt? Dignissimos, fuga,
  adipisci. Temporibus, aliquid.
</p>
```

Once that is in place, alter your `config/routes.rb` to take advantage of Devise's `authenticated` route thus:

```rb
authenticated :user do
  root to: 'items#index', as: :authenticated_root
end
root to: 'welcome#index'
```

Restart your server and we're done!

In case you missed it at the beginning, the code for this tutorial is on GitHub: [https://github.com/hibbard-eu/authentication-with-devise-and-cancancan](https://github.com/hibbard-eu/authentication-with-devise-and-cancancan "Project code")

This was quite a long post — I hope it proves useful for people. If you have any questions or comments, I'd be glad to hear them below.
