---
title: Authentication with Devise and CanCanCan in Rails 8
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
  image_url: https://res.cloudinary.com/hibbard/image/upload/f_auto,q_auto,c_limit,w_800/v1593027720/stock/lock.jpg
image_shadow: true
---

This is a tutorial on how to set up **authentication** (verifying who you are) and **authorization** (what you're allowed to do) in a Ruby on Rails app using [Devise](https://github.com/heartcombo/devise "Flexible authentication solution for Rails with Warden") and [CanCanCan](https://github.com/CanCanCommunity/cancancan "The authorization Gem for Ruby on Rails"). We'll use Rails 8 and Ruby 3.4, and build everything locally—no external auth services required.

The code for this tutorial is on GitHub: [https://github.com/hibbard-eu/authentication-with-devise-and-cancancan](https://github.com/hibbard-eu/authentication-with-devise-and-cancancan "Project code")

<!--more-->

---

**Note:** this post was updated on 7<sup>th</sup> June, 2025 to use the latest versions of Ruby, Rails, Devise, and CanCanCan..

---

## The Scenario

The app we'll be building is a store. In order for people to use the store, they'll need to register an account. The store will also have sellers (otherwise it would be a rubbish store) and an admin.

This means we'll need the following resources: `Item`, `User`, `Role`.

Here's a UML diagram showing how they relate to one another:

![UML diagram illustrating the relationships between models](https://res.cloudinary.com/hibbard/image/upload/f_auto,q_auto,c_limit/rails-devise-cancancan/uml-diagram.png "UML diagram of application model structure"){: .no-shadow}

Note that users can have a maximum of one role. The permissions for each of these users will break down as follows:

-  **Unregistered users** are redirected to the sign up page
-  **Registered users** can view items
-  **Sellers** can view items, create items, as well as update and destroy any items that belong to them
-  **Admins** can perform any CRUD operation on any resource

So let's get started.

## Installing the Dependencies

To follow along you will need Ruby and Rails installed on your machine.

For Ruby the exact steps you follow will depend upon your system. If in doubt you can consult the [official Ruby install guide](https://www.ruby-lang.org/en/documentation/installation "Installing Ruby").

If you are using a Unix-like system, I recommend [rbenv](https://github.com/rbenv/rbenv " Manage your app's Ruby environment"). rbenv is a Ruby version manager and is extremely useful for switching between multiple Ruby versions on the same machine. With rbenv installed, pulling in the latest version of Ruby is as simple as:

```bash
rbenv install 3.4.4
rbenv global 3.4.4
```

Then install Rails globally for this Ruby version:

```bash
gem install rails
```

For more info, see the [official Rails installation guide](https://guides.rubyonrails.org/getting_started.html#installing-rails "Getting Started with Rails").

You can check the version of each like so:

```bash
$ ruby -v
ruby 3.4.4 (2025-05-14 revision a38531fd3f) +PRISM [x86_64-linux]

$ rails -v
Rails 8.0.2
```

## Generating the Project Files

Let’s start by creating a new Rails app. We will skip using the [Jbuilder gem](https://github.com/rails/jbuilder "Jbuilder: generate JSON objects with a Builder-style DSL"), as we won’t be rendering any JSON responses.

```bash
rails new store --skip-jbuilder
```

Rails 8 uses [Import Maps](https://github.com/rails/importmap-rails "Use ESM with importmap to manage modern JavaScript in Rails without transpiling or bundling.") by default, which means you don’t need Node.js or any JavaScript bundler to get started.

Once the project is created, change into the `store` directory and optionally perform an initial git commit:

```bash
cd store
git add --all
git commit -m "Initial commit"
```

Now generate the basic scaffolding:

```bash
rails g scaffold user name:string role:belongs_to
rails g scaffold role name:string description:string
rails g scaffold item name:string description:text 'price:decimal{5,2}', user:belongs_to
```

Here `price:decimal{5,2}` sets up a decimal field with a precision of 5 and a scale of 2. This allows values from -999.99 to 999.99.

The `user:belongs_to` defines a foreign key relationship, meaning each item will be associated with a specific user.

Finally, run the migrations with the following command:

```bash
rake db:migrate
```

### Changes to Rails Scaffolding in Rails 7+

Rails 7 fundamentally changed how scaffold-generated views are structured. Older versions used simple tables in full-page templates. Rails 7 ditched this for a more modular approach with partials and div-based layouts, driven by Turbo Frames and Hotwire.

This means, for example, that index pages no longer use `<table>` rows but instead render each item via partials, often wrapped in divs. While this works better with Turbo, it broke expectations for many developers used to tabular views.

If you prefer the old approach, you can still create your own table-based views or [override the scaffold templates](https://www.youtube.com/watch?v=3129Se2NcOc "SupeRails #164 The illegal loophole to scaffold Tables in Rails 7"). These discussions go into the rationale and community response:

- Ruby on Rails Discussions – [Rails 7 - scaffolds? - table view?](https://discuss.rubyonrails.org/t/back-again-after-a-long-time-rails-7-scaffolds-table-view/80967)
- Reddit – [Why did Rails 7 kill scaffolding?](https://www.reddit.com/r/rails/comments/1bb0ixo/why_did_rails_7_kill_scaffolding/)
- Reddit – [Scaffold Index no longer generating HTML Tables](https://www.reddit.com/r/rails/comments/pztx9g/scaffold_index_no_longer_generating_html_tables/)

## Editing the Boilerplate

Now we can tailor the files that Rails has generated to suit our needs.

Start by removing the following line from the top of the `index` and `show` templates for the `Item`, `Role` and `User` resources. These are in the `app/views` folder.

```diff
- <p style="color: green"><%= notice %></p>
```

In `items/_item.html.erb` change:

- "User" to "Seller"
- `<%= item.price %>` to `<%= "%.2f" % item.price %>`
- `item.user_id` to `item.user.name`.

In `items/_form.html.erb` remove the complete `user_id` field (including the surrounding div tags).

```diff
- <div>
-   <%= form.label :user_id, style: "display: block" %>
-   <%= form.text_field :user_id %>
- </div>
```

In `users/_user.html.erb` change `user.role_id` to `user.role.name`

In `users/_form.html.erb` change `<%= form.text_field :role_id %>` to:

```erb
<%= collection_select(
  :user, :role_id, Role.all, :id, :name, { prompt: true }
) %>
```

At this point if you start up Puma (`rails s`) and visit [http://localhost:3000/items](http://localhost:3000/items), [http://localhost:3000/roles](http://localhost:3000/roles), or [http://localhost:3000/users](http://localhost:3000/users), you can see that our basic scaffolding is working (albeit without any data)

![New item form before any styling has been applied](https://res.cloudinary.com/hibbard/image/upload/f_auto,q_auto,c_limit/rails-devise-cancancan/new-item-unstyled.png "Unstyled new item form")

### Adding Some Styles

As you can see, the default Rails scaffolding comes with no styling at all—just raw HTML elements stacked on a blank page. To improve the appearance of our app, copy the [CSS styles from the project repository](https://github.com/jameshibbard/authentication-with-devise-and-cancancan/blob/main/app/assets/stylesheets/application.css "View base styles in application.css") and paste them into `app/assets/stylesheets/application.css`.

These styles are a simple way to smarten up the interface and make things easier to work with during development. They're not designed for production, but they provide a clean starting point until you're ready to implement a proper design.

## Authentication with Devise

Now let's turn our attention to Devise and the authentication logic.

[Devise](https://github.com/heartcombo/devise) is a full-featured authentication solution for Rails based on Warden. One of the things I like about it most (aside from the ease of use) is that it is built on a modularity concept. This makes it easy to include only those features you need in your application.

### Why Not Use Built-in Rails Authentication?

Rails 8 ships with a built-in authentication generator that gives developers full control over the auth stack. It’s ideal if you prefer to manage your own logic and don’t mind writing some boilerplate.

That said, Devise remains the better choice for most real-world apps. It provides a robust, out-of-the-box solution with minimal setup, and comes packed with features like user confirmation, session timeouts, and OAuth support—saving time and reducing complexity.

If you’d like to explore the built-in option, see the official guide: [Securing Your App with the Default Authentication Generator (Rails 8 Unpacked)](https://www.youtube.com/watch?v=4q1RWZABhKE "Learn how to add authentication to your Rails 8 app with the new built-in Authentication Generator")

### Installing And Configuring Devise

To get started with Devise, add it to our project's Gemfile:

```bash
bundle add devise
```

Then run Devise's installation generator:

```bash
rails g devise:install
```

This command generates a couple of files: an initializer and a locale file that contains all of the messages that Devise needs to display.

![Terminal output after running devise:install showing setup instructions](https://res.cloudinary.com/hibbard/image/upload/f_auto,q_auto,c_limit/rails-devise-cancancan/devise-install-terminal.png "Terminal output after Devise install")

As per the post installation message, we'll need to make a couple of alterations to our config files.

Add the following line to the bottom of `config/environments/development.rb`:

```rb
config.action_mailer.default_url_options = { host: "localhost", port: 3000 }
```

And set a root route in `/config/routes.rb`

```rb
root "items#index"
```

We're going to use the `User` model to handle authentication and Devise provides a generator for doing just that:

```bash
rails g devise User
rake db:migrate
```

Devise won't override our current `User` model – it will simply add a bunch of attributes to it.

Now let's update our application layout to include a header and navigation. In `/app/views/layouts/application.html.erb` replace:

```erb
<body>
  <%= yield %>
</body>
```

With the following:

```erb
<body>
  <div class="container">
    <header class="site-header">
      <div class="branding">
        <%= link_to "MyStore", root_path %>
      </div>
        <div class="header-right">
        <nav class="user-nav">
          <% if user_signed_in? %>
            Signed in as <%= current_user.email %> |
            <%= link_to "Edit profile", edit_user_registration_path %> |
            <%= link_to "Sign out", destroy_user_session_path, data: { "turbo-method": :delete } %>
          <% else %>
            <%= link_to "Sign up", new_user_registration_path %> or
            <%= link_to "sign in", new_user_session_path %>
          <% end %>
        </nav>

        <nav class="admin-nav">
          <% if user_signed_in? %>
            <%= link_to "Items", items_path %> |
            <%= link_to "Users", users_path %> |
            <%= link_to "Roles", roles_path %>
          <% end %>
        </nav>
      </div>
    </header>

    <% flash.each do |name, msg| %>
      <%= content_tag :div, msg, class: "flash #{name}" %>
    <% end %>

    <main>
      <%= yield %>
    </main>
  </div>
</body>
```

The new layout includes a top navigation bar showing the current user’s authentication status. If the user is signed in, it displays their email address along with links to edit their profile or sign out. If not signed in, it shows links to sign up or log in. Flash messages are displayed directly beneath the header.

We've also added an admin navigation bar with links to `Items`, `Users`, and `Roles`. This is primarily for convenience during development, allowing quick access across key resources. It’s currently visible to all users, but we’ll later restrict it to admins only.

Next, add the following line to the top of `ItemsController`, `RolesController` and `UsersController`:

```rb
before_action :authenticate_user!
```

This will ensure that the user is logged in before being able to access any of these resources.

Now, populate your database with some initial records. Alter `db/seeds.rb` thus:

```rb
r1 = Role.create({ name: "Regular", description: "Can read items" })
r2 = Role.create({ name: "Seller", description: "Can read and create items. Can update and destroy own items" })
r3 = Role.create({ name: "Admin", description: "Can perform any CRUD operation on any resource" })

u1 = User.create({ name: "Sally", email: "sally@example.com", password: "aaaaaaaa", password_confirmation: "aaaaaaaa", role_id: r1.id })
u2 = User.create({ name: "Sue", email: "sue@example.com", password: "aaaaaaaa", password_confirmation: "aaaaaaaa", role_id: r2.id })
u3 = User.create({ name: "Kev", email: "kev@example.com", password: "aaaaaaaa", password_confirmation: "aaaaaaaa", role_id: r2.id })
u4 = User.create({ name: "Jack", email: "jack@example.com", password: "aaaaaaaa", password_confirmation: "aaaaaaaa", role_id: r3.id })

i1 = Item.create({ name: "Rayban Sunglasses", description: "Stylish shades", price: 99.99, user_id: u2.id })
i2 = Item.create({ name: "Gucci watch", description: "Expensive timepiece", price: 199.99, user_id: u2.id })
i3 = Item.create({ name: "Henri Lloyd Pullover", description: "Classy knitwear", price: 299.99, user_id: u3.id })
i4 = Item.create({ name: "Porsche socks", description: "Cosy footwear", price: 399.99, user_id: u3.id })
```

Then run:

```bash
rake db:seed
```

Finally, we can restart the Rails server and log in using the email address and password of one of the users we defined in the seeds file.

Notice that if we are logged out and try and access any of the protected resources, we are redirected to a log in page with the message "You need to sign in or sign up before continuing."

![Devise login form with branding and fields for authentication](https://res.cloudinary.com/hibbard/image/upload/f_auto,q_auto,c_limit/rails-devise-cancancan/devise-login.png "Devise login form")

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

Now, to get our app working properly, we need to make a few tweaks.

## Adding an Admin Namespace

Let's start by namespacing the CRUD interface. This is necessary as otherwise [the user registration routes can conflict with the user management routes](https://github.com/heartcombo/devise/wiki/How-To:-Manage-users-through-a-CRUD-interface "How To: Manage users through a CRUD interface"). Alter `config/routes.rb` like so:

```rb
devise_for :users
scope "/admin" do
  resources :users
end

resources :items
resources :roles
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
  @item.user = current_user
  ...
end
```

Once you have restarted the server, you will be able to (kinda) manage users at [http://localhost:3000/admin/users](http://localhost:3000/admin/users). I write kinda, as you'll not yet be able to create new users. We'll get to that a little later.

![Admin index page showing all users with their assigned roles](https://res.cloudinary.com/hibbard/image/upload/f_auto,q_auto,c_limit/rails-devise-cancancan/admin-users-index.png "Admin users index with roles")

## Customizing Devise

If you click on the "sign up" or "edit profile" links, you'll notice that the user's name attribute is missing. It would be nice for users to specify their names when registering, so let’s fix that.

Devise comes with many built-in views. To customize these pages, we must transfer the Devise view files into our project so we can modify them. Luckily, Devise has a generator to do just that:

```rb
rails g devise:views
```

This will copy across a bunch of templates to the `app/views/devise` directory.

Change into this folder, then into the `registrations` sub-folder. Locate the `new.html.erb` and `edit.html.erb` files and after this line:

```erb
<%= render "devise/shared/error_messages", resource: resource %>`:
```

Add the following:

```erb
<div class="field">
  <%= f.label :name %><br />
  <%= f.text_field :name, autofocus: true %>
</div>
```

And remove the `autofocus: true` attribute from the email field.

This adds the name field to the registration and edit forms. But if you fill them out at this point, you will notice that Devise doesn't save your changes.

The reason for this is that the extra parameter you are passing to the controller (`name` in this case) needs to be expressly permitted.

You can do this as follows in the `ApplicationController`:

```rb
class ApplicationController < ActionController::Base
  before_action :configure_permitted_parameters, if: :devise_controller?

  # Only allow modern browsers supporting webp images, web push, badges, import maps, CSS nesting, and CSS :has.
  allow_browser versions: :modern

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [ :name ])
    devise_parameter_sanitizer.permit(:account_update, keys: [ :name ])
  end
end
```

Now Devise will accept and save the `name` field when users register or edit their account.

![Devise sign-up form showing a validation error for missing name](https://res.cloudinary.com/hibbard/image/upload/f_auto,q_auto,c_limit/rails-devise-cancancan/devise-sign-up-error.png "Sign-up form with Devise validation error")

We also want to assign a default role to new users. Rather than requiring the user to select one, we can assign it automatically using a callback. Update the `User` model like so:

```rb
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable

  belongs_to :role
  has_many :items, dependent: :destroy
  before_validation :assign_role
  validates :name, presence: true

  def assign_role
    self.role = Role.find_by name: "Regular" if role.nil?
  end
end
```
This ensures that any new user is automatically given the “Regular” role if no other role is set.

Notice how we are assigning the role in a `before_validation` callback. One might consider using `before_create` or `before_save` instead, but since Rails 5, `belongs_to` enforces presence by default, and those callbacks fire after validations have run — too late to meet the requirement. Using `optional: true` would suppress the error, but would contradict the application’s intent, as every user should have a role.

Take a moment at this point to start up the app again and make sure that everything is working. You should be able to specify your name when signing up as a new user.

## Creating New Users

Next we're going to give the admin the power to create new users via the admin interface, as well as to edit any of the existing users' attributes.

First off, let's add the user's email address to the views.

`app/views/users/_user.html.erb`:

```erb
<p>
  <strong>Email:</strong>
  <%= user.email %>
</p>
```

`app/views/users/_form.html.erb`:

```erb
<div class="field">
  <%= form.label :email %>
  <%= form.text_field :email %>
</div>
```

In the form partial we can also add password and confirmation fields:

```erb
<div class="field">
  <%= form.label :password %>
  <%= form.password_field :password, placeholder: "Leave blank if unchanged" %>
</div>

<div class="field">
  <%= form.label :password_confirmation %>
  <%= form.password_field :password_confirmation %>
</div>
```

Now things get a little complicated. So as to be able to create a new user via our admin interface ([http://localhost:3000/admin/users/new](http://localhost:3000/admin/users/new)), we need to permit the attributes that Devise requires:

`/app/controllers/users_controller.rb`

```rb
def user_params
  params.expect(user: [
    :name,
    :email,
    :role_id,
    :password,
    :password_confirmation
  ])
end
```

That works, but if we try and edit an existing user (e.g. [http://localhost:3000/admin/users/1/edit](http://localhost:3000/admin/users/1/edit)), then we are prompted to set a new password every time we want to update something. This is not ideal.

To get around this, we can take advantage of Devise's [update_without_password](https://www.rubydoc.info/gems/devise/4.9.4/Devise/Models/DatabaseAuthenticatable:update_without_password "Method: Devise::Models::DatabaseAuthenticatable#update_without_password") method.

Alter the update method in `UsersController` as shown (making sure to include the protected method `needs_password?`):

```rb
def update
  successfully_updated = if needs_password?(@user, user_params)
    @user.update(user_params)
  else
    @user.update_without_password(user_params)
  end

  if successfully_updated
    redirect_to @user, notice: "User was successfully updated."
  else
    render :edit, status: :unprocessable_entity
  end
end

private

def needs_password?(_user, params)
  params[:password].present?
end
```

And with that we can now create, update and delete users.

![User show page showing successful creation of a new user with details](https://res.cloudinary.com/hibbard/image/upload/f_auto,q_auto,c_limit/rails-devise-cancancan/user-show-success.png "User show page after successful creation")

## Enabling the Trackable Module

Before we finish looking at Devise and authentication, let's enable the trackable module, to give us a little more information on our users.

In `/app/models/users.rb` alter the code like so:

```rb
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable, :trackable
  ...
end
```

Create a new migration:

```bash
rails g migration AddDeviseTrackableColumnsToUsers
```

This will create a new file in the `db/migrate` folder. Alter it like so:

```ruby
class AddDeviseTrackableColumnsToUsers < ActiveRecord::Migration[8.0]
  def change
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
<div class="user-meta">
  <p><strong>Joined on:</strong> <%= @joined_on %></p>
  <p><strong>Last logged in on:</strong> <%= @last_login %></p>
  <p><strong>No. times logged in:</strong> <%= @user.sign_in_count %></p>
</div>
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

Also, to show us which users are associated with a particular role, add the association to the `Role` model.

`app/models/role.rb`:

```rb
has_many :users, dependent: :restrict_with_exception
```

The `restrict_with_exception` option will cause an `ActiveRecord::DeleteRestrictionError` exception to be raised if you try to delete a `Role` record, but it has associated `User` records.

Next, add the following to `app/views/roles/show.html.erb`:

```erb
<div class="user-meta">
  <p>
    <strong>Associated users:</strong>
    <%= @associated_users %>
  </p>
</div>
```

And the logic to the `RolesController`:

```rb
def show
  if @role.users.empty?
    @associated_users = "None"
  else
    @associated_users = @role.users.map(&:name).join(", ")
  end
end
```

And there we go. In not very much code we have implemented a robust authentication solution for our app, as well as building a simple interface to administer users.

![User profile displaying trackable Devise data like login count and timestamps](https://res.cloudinary.com/hibbard/image/upload/f_auto,q_auto,c_limit/rails-devise-cancancan/user-show-trackable.png "User profile with trackable Devise information")

Take a moment to restart the app, have a play with what we've got so far and assure yourself that it is working.

## Authorization With CanCanCan

[CanCanCan](https://github.com/CanCanCommunity/cancancan "The authorization Gem for Ruby on Rails") is an authorization library for Ruby on Rails that restricts which resources a given user is allowed to access. It is the continuation of the now defunct cancan project started by Ryan Bates (of Railscast fame).

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

We don't care about the permissions of those users who have not yet logged in (you remember they should just be directed to the sign in page), but if we did—for example, if they could view items—you would need to initialize the `user` variable to something sensible. Otherwise you would end up checking for nil values all over the place.

```rb
def initialize(user)
  user ||= User.new # guest user

  if user.admin?
    ...
  elsif user.seller?
    ...
  elsif user.regular?
    ...
  else
    # guest permissions
  end
ens
```

Next we need to define an `admin?` method in our `User` model.

`app/models/user.rb`:

```rb
def admin?
  role&.name == "Admin"
end
```

And in the `UsersController` we need to specify which users are authorized to do what. You can do this on a per action basis using the `authorize!` method, which in turn performs the `can?` check and raises an exception if needed.

For example to check if a given user can edit an item:

```rb
def edit
  authorize! :edit, @item
end
```

However, repeating this check across every action would quickly become tedious. Fortunately, if your controller follows RESTful conventions, CanCanCan provides a shortcut: `load_and_authorize_resource`.

This method serves two purposes. First, it automatically loads the relevant resource into an instance variable based on the controller name and action—for example, loading the correct `Item` into `@item` in an `ItemsController`. Second, it performs an authorization check against the current user using the rules defined in your `Ability` class.

In short, it reduces boilerplate by handling both resource loading and permission checks for you.

Place it at the top of your ItemsController like so:

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

![CanCanCan access denied error displayed when unauthorized action is attempted](https://res.cloudinary.com/hibbard/image/upload/f_auto,q_auto,c_limit/rails-devise-cancancan/access-denied-error.png "CanCanCan access denied error page")

## Better Error Handling For CanCanCan

Next, we can ensure that our users see a nicer error page than the one with the `AccessDenied` exception they currently see when attempting to access something they are not authorized to.

We can do this using a method called [rescue_from](http://api.rubyonrails.org/classes/ActiveSupport/Rescuable/ClassMethods.html "ActiveSupport::Rescuable::ClassMethods") that we can place in our `ApplicationController`. We'll pass it a block inside of which we'll make the application show a flash error message and redirect to the home page.

`app/controllers/application_controller.rb`

```rb
rescue_from CanCan::AccessDenied do
  flash[:error] = "Access denied!"
  redirect_to root_url
end
```

Now all that remains to do is to set the permissions of regular users and sellers to something sensible.

Let's define two more methods to check the user's current role:

`app/models/user.rb`

```rb
def seller?
  role&.name == "Seller"
end

def regular?
  role&.name == "Regular"
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

In the case of sellers however, things become a little more complicated when dealing with the update and destroy actions—remember that sellers should only be able to update and delete their own items.

Here we pass `can` a block which will receive the instance of the model we're checking. The block should return `true` or `false` depending on whether the action should be allowed, or not. Within the block we'll use Rails' [try](https://api.rubyonrails.org/classes/Object.html#method-i-try "Invokes the public method whose name goes as first argument. If the receiver does not respond to it the call returns nil rather than raising an exception") method to check that the item's `user` attribute is the current user. Using `try` covers the eventuality that the item is `nil` and will prevent an exception being raised.

Finally, so that regular users and sellers don't see any links to actions they are not permitted to perform, alter `app/views/items/show.html.erb` like so:

```erb
<% if can? :update, @item %>
  <%= link_to "Edit this item", edit_item_path(@item) %> |
<% end %>
<%= link_to "Back to items", items_path %>
<% if can? :destroy, @item %>
  <%=
    button_to "Destroy this item", @item,
    data: { turbo_method: :delete, turbo_confirm: "Are you sure?" }
  %>
<% end %>
```

And `app/views/items/index.html.erb` like so:

```erb
<% if can? :create, Item %>
  <%= link_to 'New Item', new_item_path %>
<% end %>
```

## The Finishing Touches

Before we end, let's add a couple of finishing touches. First, let's adjust the admin navigation so that it is only displayed for the admin.

```erb
<nav class="admin-nav">
  <% if @current_user&.admin? %>
    <%= link_to "Items", items_path %> |
    <%= link_to "Users", users_path %> |
    <%= link_to "Roles", roles_path %>
  <% end %>
</nav>
```

Next, we can enhance our flash messages by making them fade out and disappear after a short delay. In Rails 8, the idiomatic way to handle this is with [Stimulus](https://stimulus.hotwired.dev/), which is included by default via Importmap.

First, generate a new Stimulus controller:

```bash
rails g stimulus flash
```
This will create a file at `app/javascript/controllers/flash_controller.js`. Edit it to contain:

```js
import { Controller } from '@hotwired/stimulus';

export default class extends Controller {
  connect() {
    setTimeout(() => {
      this.element.classList.add('opacity-0');
      setTimeout(() => this.element.remove(), 1500);
    }, 3500);
  }
}
```

Next, ensure the flash messages are rendered with the appropriate `data-controller` attribute.

In `app/views/layouts/application.html.erb`:

```erb
<% flash.each do |name, msg| %>
  <%= content_tag :div, msg,
      class: "flash #{name}", data: { controller: "flash" } %>
<% end %>
```

This will automatically fade out and remove flash messages after 3.5 seconds.

Note that the necessary transition styles were included earlier when we copied [the CSS from the project repository](https://github.com/jameshibbard/authentication-with-devise-and-cancancan/blob/main/app/assets/stylesheets/application.css "View base styles in application.css"). These utility classes handle the fade-out animation applied by the Stimulus controller. If you skipped that step, be sure to add them manually or copy them from below.

```css
.flash {
  ...
  opacity: 1;
  transition: opacity 1.5s ease;
}

.flash.opacity-0 {
  opacity: 0;
}
```
Finally, the very last thing I want to examine (promise!) is a slightly lesser documented feature of Devise. You remember that we initially wanted users who are not logged in to be redirected to the sign-in page? Well, wouldn't it be nicer if we directed them to a welcome page with a link to either register or sign in?

To do this we need a `WelcomeController` and a corresponding view. These can be generated with the following command:

```bash
rails g controller welcome index
```

Now edit the newly generated view template at `app/views/welcome/index.html.erb`:

```html
<h1>Welcome to MyStore™</h1>
<h2>Supplying unnecessary objects to discerning procrastinators since 2020</h2>
<p>
  At MyStore™, we specialise in products you didn’t ask for, solving problems you don’t really have. From elegantly useless kitchen gadgets to beautifully confusing tech accessories, we’re here to complicate your life with style.
</p>
<p>
  Our mission? To ensure your shelves are full, your drawers are cluttered, and your bank account just slightly emptier than it was five minutes ago. Because nothing says satisfaction like a parcel you forgot you ordered.
</p>
<p>
  We understand your needs. Or at least we pretend to. That’s why we’ve created a shopping experience that’s slick, efficient, and only mildly intrusive. You click, we ship. You regret, we thrive.
</p>
<p>
  So settle in, scroll aimlessly, and add with abandon. MyStore™: because owning things is easier than finding meaning.
</p>
```

Once that is in place, alter your `config/routes.rb` to take advantage of Devise's `authenticated` route thus:

```rb
authenticated :user do
  # Authenticated users see this
  root "items#index", as: :authenticated_root
end

## Everyone else sees this
root "welcome#index"
```

This gives us:

```rb
Rails.application.routes.draw do
  devise_for :users

  authenticated :user do
    root "items#index", as: :authenticated_root
  end
  root "welcome#index"

  scope "/admin" do
    resources :users
  end

  resources :items
  resources :roles

  get "up" => "rails/health#show", as: :rails_health_check
end
```

Restart your server and we're done! We have now tidied up the navigation, we have disappearing flash messages and any users who are not logged in will be directed to the front-facing page of our app.

![Screenshot of the landing page of the MyStore app showing a welcome message](https://res.cloudinary.com/hibbard/image/upload/f_auto,q_auto,c_limit/rails-devise-cancancan/landing-page.png "Landing page of the MyStore app")

In case you missed it at the beginning, the code for this tutorial is on GitHub: [https://github.com/hibbard-eu/authentication-with-devise-and-cancancan](https://github.com/hibbard-eu/authentication-with-devise-and-cancancan "Project code")

This was quite a long post — I hope it proves useful for people. If you have any questions or comments, I'd be glad to hear them below.
