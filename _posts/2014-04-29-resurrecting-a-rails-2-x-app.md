---
title: Resurrecting a Rails 2.x App
layout: post
permalink: /resurrecting-a-rails-2-x-app/
tags:
  - linux
  - mysql
  - rails
  - ruby
excerpt_separator: <!--more-->
old-comments: resurrecting-a-rails-2-x-app.html
comments: false
---

This morning I had to give a client an estimate for some work they wanted doing on a Rails 2.1.2 app.

The work itself wasn't overly complicated, but getting a system up and running using Ruby 1.8 and Rails 2.1 proved to be somewhat of a challenge.

Here's how I did it.

<!--more-->

First things first, I needed an OS running Ruby 1.8 (as this was the most popular, stable release when Rails 2.1 appeared in June 2008), so I started by installing the 64 bit version of [Ubuntu](http://www.ubuntu.com/ "Ubuntu - a Debian-based Linux operating system") (13.04) on Virtual Box.

When that was up and running, I installed the basics (everything was run as root):

```sh
apt-get update
apt-get -y install build-essential zlib1g zlib1g-dev libxml2 libxml2-dev libxslt-dev sqlite3 libsqlite3-dev locate git-core
apt-get -y install curl wget
```

Then installed Ruby 1.8 (MRI):

```sh
apt-get -y install ruby1.8-dev ruby1.8 ri1.8 rdoc1.8 irb1.8 libreadline-ruby1.8 libruby1.8 libopenssl-ruby
```

Then it was time to install RubyGems. I remember reading that Rails 2.x had compatibility issues with RubyGems > 1.6, so I opted to download v1.3.5 from source:

```sh
curl http://rubyforge.org/frs/download.php/60718/rubygems-1.3.5.tgz | tar -xzv
cd rubygems-1.3.5 && ruby setup.rb install
cd .. && rm -rf rubygems-1.3.5
ln -s /usr/bin/gem1.8 /usr/local/bin/gem
```

I then added Github as a gem source:

```sh
gem sources -a http://gems.github.com
```

Ok, so far so good:

```sh
ruby -v
=> ruby 1.8.7 (2012-02-08 patchlevel 358) [x86_64-linux]

gem -v
=> 1.3.5
```

Now it was time to install the correct version of Rails:

```sh
gem install rails -v2.1.2
```

Next MySQL plus adapter:

```sh
apt-get install mysql-server
apt-get install libmysqlclient15-dev

gem install mysql
```

Note: you get prompted for a root password during the installation process.

```sh
mysql -v
=> Server version: 5.5.34-0ubuntu0.13.04.1 (Ubuntu)
```

Looking good. Now it's time to create the databases (having first altered myApp/config/database.yml):

```sh
cd myApp
rake db:create:all

=> rake aborted! ERROR: 'rake/rdoctask' is obsolete and no longer supported.
```

Oops. Looks like I should downgrade rake.

```sh
gem uninstall rake
gem install rake -v0.9.0
```

Now I can start the server with `./script/server start` and am greeted by this lovely sight at http://localhost:3000/

![Riding the Rails!](https://res.cloudinary.com/hibbard/image/upload/v1530016646/riding-the-rails.png "Riding the Rails!"")

## A Few Final Tweaks

The app in question also relied on the [rmagick](https://github.com/rmagick/rmagick "An interface to the ImageMagick and GraphicsMagick image processing libraries.") gem.

This gem is an interface to the ImageMagick image processing library, so obviously we'll need that.

```sh
apt-get install imagemagick libmagickwand-dev
gem install rmagik
gem list- ** LOCAL GEMS ***

actionmailer (2.1.2)
actionpack (2.1.2)
activerecord (2.1.2)
activeresource (2.1.2)
activesupport (2.1.2)
mysql (2.9.1)
rails (2.1.2)
rake (0.9.0)
rmagick (2.13.2)
```

Sorted!

### Reference

-  [How To Install A Ruby 1.8 Stack on Ubuntu 8.10 From Scratch](http://www.rubyinside.com/how-to-install-a-ruby-18-stack-on-ubuntu-810-from-scratch-1566.html "Ruby Inside")

I hope this proved useful for people. If you have any questions, I'd be glad to hear them in the comments.
