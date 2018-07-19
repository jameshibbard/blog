---
title: How to Install rbenv on Linux Mint 17.1
layout: post
permalink: /how-to-install-rbenv-on-linux-mint-17-1/
tags:
  - linux
  - rbenv
  - ruby
  - version managers
excerpt_separator: <!--more-->
comments: false
---

Last year I wrote about [installing rbenv on Linux Mint 16](http://hibbard.eu/how-to-install-rbenv-on-linux-mint-16/ "How to install rbenv on Linux Mint 16"). Back then the installation process as described on [the project's Github page](https://github.com/sstephenson/rbenv "rbenv - Groom your app's Ruby environment") didn't work for me and, after much frustration,  I ended up installing an older version of rbenv from the repositories.

Recently, I had to reinstall my operating system (upgrading to Mint 17.1) and decided to give the rbenv installation process another try. I'm happy to say that it worked entirely as expected and within a matter of minutes I had two Ruby versions installed on my system and could switch between them at will.

<!--more-->

Nonetheless, this still involved a little preparation and several steps which I want to summarize in this post.

## So, Let's Get Started!

As we will need to clone the rbenv repository, the first thing to do is ensure that git is installed on your system. To do this open a terminal and type:

```sh
git --version
```

If this prints out a version number, it's all good. If it doesn't you'll need to grab git from the repositories:

```sh
sudo apt-get install git
```

The developers of rbenv [recommend running the following command](https://github.com/sstephenson/ruby-build/wiki#suggested-build-environment "Suggested build environmen") before installing and using rbenv itself:

```sh
sudo apt-get install autoconf bison build-essential libssl-dev libyaml-dev libreadline6-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm3 libgdbm-dev
```

This will install everything necessary to create a sane build environment.

Next we need to clone the rbenv repo to our home directory:

```sh
git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
```

Then add `~/.rbenv/bin` to our `$PATH` for access to the rbenv command-line utility:

```sh
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
```

If you are following along with a different Linux distro, you will have to modify `~/.bash_profile` or `~/.zshrc` accordingly.

After that, add `rbenv init` to your shell to [enable shims and autocompletion](https://github.com/sstephenson/rbenv#understanding-shims "Understanding Shims"):

```sh
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
```

Finally, restart your shell:

```sh
exec bash
```

You can check if all has gone well with the following command:

```sh
type rbenv
```

This should output:

```sh
rbenv is a function
rbenv ()
{
...
}
```

Congratulations! You have now successfully installed rbenv.

## Install ruby-build

Now, a Ruby version manager is no good without a few Ruby versions to manage and the easiest way to compile and install different versions of Ruby on your system is with the aid of the [ruby-build plugin](https://github.com/sstephenson/ruby-build "ruby-build rbenv plugin").

The first thing to do is to install it. Like rbenv itself, it can be cloned from its Github repo:

```sh
git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
```

Once it's there, you can view available Ruby versions with the following command:

```sh
rbenv install --list
```

Choose the one you want and install it, thus:

```sh
rbenv install <version_number>
```

This will take a while (depending on your system). The Ruby version will be installed to `.rbenv/versions/version_number`.

Select the previously installed version:

```sh
rbenv global <version_number>
```

And that's all there is to it:

```sh
jim@friendly ~ $ irb
irb(main):001:0> puts "Woo hoo!"
Woo hoo!
=> nil
```

### Reference

- [Chruby and Rbenv Tips and Tricks](http://www.sitepoint.com/chruby-rbenv-tips-tricks/ "Some advanced rbenv and chruby usage to maximize your productivity")

I hope this proves useful for people. If you have any questions or observations, I'd be glad to hear them in the comments.
