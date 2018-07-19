---
title: How to Install rbenv on Linux Mint 16
layout: post
permalink: /how-to-install-rbenv-on-linux-mint-16/
tags:
  - linux
  - rbenv
  - ruby
  - version managers
excerpt_separator: <!--more-->
comments: false
---

Last year I switched from Windows to Linux Mint as my main operating system and wanted to install a Ruby version manager.

I weighed up the pros and cons of what was available and eventually opted for [rbenv](https://github.com/sstephenson/rbenv "rbenv GitHub page") as it seemed more lightweight, would let me compile my Rubies myself and (in contrast to [RVM](http://rvm.io/ "Ruby Version Manager")) didn't come with any way of managing gems.

I searched Google and came up with a couple of tutorials to follow, such as [this one](http://developwithguru.com/how-to-install-ruby-on-linux-mint-or-ubuntu-linux/ "How to install Ruby on Linux Mint or Ubuntu Linux") and [this one](http://linuxrails.blogspot.de/2013/12/how-to-install-ruby-with-rbenv-and.html "How to install Ruby with Rbenv and tools in Ubuntu 13.10 / Mint 16 ") which described how to install rbenv, but unfortunately they didn't work for me.

<!--more-->

---

**Update 03.01.2015**: I just installed Mint 17.1 and the process of installing rbenv was considerably easier. [Read more about that here](http://hibbard.eu/how-to-install-rbenv-on-linux-mint-17-1/).

---

The point at which they failed was when I cloned the rbenv repo, made the necessary changes to my bash_profile, then ran the command `rbenv -v`.

Apparently this should have displayed the appropriate rbenv version information, but no matter what I tried, I kept getting the error `The program 'rbenv' is currently not installed..`.

Changing tack I  decided to install rbenv from the repositories and typed:

```sh
sudo apt-get install rbenv
```

This pulled in the following additional packages: `libruby1.9.1`, `libyaml-0-2`, `ruby`, `ruby1.9.1`.

After this, typing `rbenv` resulted in:

```sh
rbenv 0.3.0
usage: rbenv <command> [<args>]
```

Progress!!

I then installed [ruby-build](https://github.com/sstephenson/ruby-build "An rbenv plugin that provides an rbenv install command")  which is an rbenv plugin which enables us to install any versions of Ruby, by just issuing a `rbenv install` command:

```sh
sudo apt-get install ruby-build
```

I could then do:

```sh
rbenv install -l
```

and was presented with a list of available Ruby versions which rbenv could install for me. Sweet!

Now, by installing rbenv on my machine, I had also installed Ruby version 1.9.3 in the process.

It therefore seemed like a good test to use rbenv to install Ruby 1.9.2

```sh
rbenv install 1.9.2-p180
```

```
Downloading http://pyyaml.org/download/libyaml/yaml-0.1.4.tar.gz...
Installing yaml-0.1.4...
Installed yaml-0.1.4 to /home/me/.rbenv/versions/1.9.2-p180
Downloading http://ftp.ruby-lang.org/pub/ruby/1.9/ruby-1.9.2-p180.tar.gz..
Installing ruby-1.9.2-p180...
Installed ruby-1.9.2-p180 to /home/me/.rbenv/versions/1.9.2-p180
Downloading http://production.cf.rubygems.org/rubygems/rubygems-1.8.23.tgz
Installing rubygems-1.8.23...
Installed rubygems-1.8.23 to /home/me/.rbenv/versions/1.9.2-p180
```

Let's test that:

First, it's important to ensure that `PATH` contains `$HOME/.rbenv/shims`

```sh
env | grep PATH
```

If it doesn't, then run the following command and restart the console:

```sh
echo 'export PATH="$HOME/.rbenv/shims:$PATH"' >> ~/.bashrc
```

Then:

```sh
ruby -v
=> 1.9.3p194

rbenv versions
=> 1.9.2-p180

rbenv global 1.9.2-p180
ruby -v
=> 1.9.2-p180
```

Success!

You might notice the `rbenv global` command above, which sets the Ruby version at a global level. It's counterpart is `rbenv local`, which is used in the same way and sets the ruby version on a per project basis.

## Compiling Your Own Ruby Versions

Now there might be occasions when you want to download, compile and install your own versions of Ruby. Luckily with rbenv, that's no problem.

Head over to ruby-lang.org and visit the [downloads section](https://www.ruby-lang.org/en/downloads/ "Download Ruby") and find the version you want. I'm going to go with [Current stable](http://cache.ruby-lang.org/pub/ruby/2.1/ruby-2.1.0.tar.gz "Ruby 2.1.0").

Download the tarball to your PC, unpack it and change to the newly created directory:

```sh
tar -xvzf ruby-2.1.0.tar.gz
cd ruby-2.1.0
```

Having done that, run the following commands:

```sh
./configure -prefix=$HOME/.rbenv/versions/2.1.0
make
make install
rbenv rehash
```

And that's all there is to it.

## Making a Launcher

A small issue that I ran into using rbenv, was that I had an FXRuby app that I wanted to make a desktop launcher for.

I could launch this app from the terminal with the following command:

```sh
ruby /mnt/files/Ruby/FXRuby/password_generator.rbw
```

However, when it came to making a launcher, the same command failed silently.

After much head scratching I stumbled on the solution, namely that instead of entering the launch command as `ruby /path/to/file.rbw`, I had to specify the complete path to the Ruby version I wanted to use.

Here's the complete launcher file:

```sh
[Desktop Entry]
Comment=Generates a random password
Terminal=false
Name=password_generator
Exec=/home/jim/.rbenv/versions/1.9.3-p484/bin/ruby /mnt/files/Ruby/FXRuby/password_generator.rbw
Type=Application
Icon=/mnt/files/Linux/Icons/gnome-mime-application-x-ruby.png
```

I hope this proved useful for people. If you have any questions, I'd be glad to hear them in the comments.
