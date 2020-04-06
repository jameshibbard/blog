---
title: How to Set up Rails with PostgreSQL on Linux Mint
layout: post
permalink: /how-to-setup-rails-with-postgresql-on-linux-mint/
tags:
  - linux
  - postgres
  - ruby
  - rails
excerpt_separator: <!--more-->
old-comments: how-to-setup-rails-with-postgresql-on-linux-mint.html
comments: false
---

Rails is database agnostic, meaning that it can talk to different databases with little more than a couple of configuration changes. Here's how to install [PostgreSQL](http://www.postgresql.org/ "PostgreSQL homepage") on Linux Mint 16 and configure it for use with Rails.

<!--more-->

The first thing to do is to install the Postgres packages from the repositories using `apt-get`.

```sh
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib
```

As you might gather, postgresql is the main package, [postgresql-contrib](https://www.postgresql.org/docs/current/static/contrib.html "Additional Supplied Modules") is an additional package that adds further utilities and functionality.

The next thing to do is to install the [pg gem](https://bitbucket.org/ged/ruby-pg/wiki/Home "pg - project homepage"), which is the Ruby interface to the PostgreSQL RDBMS:

```sh
sudo apt-get install libpq-dev
```

This installs the header files and static libraries necessary for compiling C programs to link with the libpq library, so as to communicate with the PostgreSQL backend.

```sh
gem install pg
```

PostgreSQL works with the concept of "roles" to aid in authentication and authorization. You can [read more about them here](https://www.postgresql.org/docs/current/static/user-manag.html "Database Roles and Privileges"). For now it is sufficient to create a role with the same name as your current user:

```sh
sudo -u postgres psql
```

This will take us to a shell prompt for the "postgres" role.

```sh
create role jim with createdb login password 'pass';
```

This creates a new role (jim) with a password of "pass". Obviously, swap out "jim" for the name of your current user.

You can verify the success of this command thus:

```sh
SELECT rolname FROM pg_roles;
```

Sample output:

```
postgres=# SELECT rolname FROM pg_roles;
       rolname
----------------------
 postgres
 pg_monitor
 pg_read_all_settings
 pg_read_all_stats
 pg_stat_scan_tables
 pg_signal_backend
 jim
(7 rows)

```

## A Simple Rails App

Now, let's build a simple Rails app to ensure that everything is working as expected.

```sh
rails new testapp --database=postgresql
```

This will create a new Rails app with the name of "testapp". Change directory into testapp and edit `config/database.yml` with the details of the role you previously created. For me this would be:

```sh
default: &default
  adapter: postgresql
  encoding: unicode
  # For details on connection pooling, see rails configuration guide
  # http://guides.rubyonrails.org/configuring.html#database-pooling
  pool: 5

development:
  <<: *default
  database: testapp_development

  # The specified database role being used to connect to postgres.
  # To create additional roles in postgres see `$ createuser --help`.
  # When left blank, postgres will use the default role. This is
  # the same name as the operating system user that initialized the database.
  username: jim

  # The password associated with the postgres role (username).
  password: pass
```

You should then run:

```sh
rake db:migrate
```

to create testapp/db/schema.rb

Then:

```sh
rake db:setup
```

This creates development and test databases, sets their owners to the user specified, and creates "schema_migrations" tables in each.

Now you can start the server using `rails s` and head to [http://localhost:3000/](http://localhost:3000/) where you can inspect your applications details.

Once you have verified that it is set up correctly, hop back onto the console, stop the server and enter:

```sh
rails g scaffold User name:string description:text
rake db:migrate
```

Restart the server, then head to [http://localhost:3000/users](http://localhost:3000/users) and you should see your app running as expected.

You can now perform CRUD operations on the PostgreSQL database.


### Useful links

-  [How To Install and Use PostgreSQL on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04 "How to install Postgres on an Ubuntu 14.04")
-  [How To Setup Ruby on Rails with Postgres](https://www.digitalocean.com/community/tutorials/how-to-setup-ruby-on-rails-with-postgres "Create a Rails application that uses a Postgres database")
-  [Installing PG gem - failure to build native extension](http://stackoverflow.com/questions/19262312/installing-pg-gem-failure-to-build-native-extension "Troubleshooting gem issues")
-  [Postgresql: password authentication failed for user "postgres"](http://stackoverflow.com/questions/7695962/postgresql-password-authentication-failed-for-user-postgres "Troubleshooting connectivity issues")
