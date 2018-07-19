---
title: How to Manage Multiple Versions of Ruby on Windows
layout: post
permalink: /how-to-manage-multiple-versions-of-ruby-on-windows/
tags:
  - ruby
  - ruby gems
  - version managers
  - windows
excerpt_separator: <!--more-->
old-comments: how-to-manage-multiple-versions-of-ruby-on-windows.html
comments: false
---

I had recently started work on a Rails project and was setting up my local dev environment which I wanted to make it as similar as possible to the environment on the server I will deploy to.

The remote server currently runs Ruby 1.9.2 and Rails 3.x, so this is what I installed on my local machine.

<!--more-->

Whilst ensuring that everything worked as expected, I noticed was that [WEBrick](http://en.wikipedia.org/wiki/WEBrick "About WEBrick") was very slow to boot. However, as this only needs to be done once (or occasionally twice) per session, I didn't think much more of it.

I then started coding, wrote some simple tests then ran `rake test` from the console. And waited, and waited, and waited…

My minimal set of tests took over one and a half minutes to run. WTF?!? This was something that would seriously slow me down.

I did some Googling and found out that I wasn't the only one who was having issues with Rails 3 and Ruby 1.9.2. I read through a [couple](http://stackoverflow.com/questions/4789248/rails-3-initializes-extremely-slow-on-ruby-1-9-2 "Rails 3 initializes extremely slow on Ruby 1.9.2") of [questions](http://stackoverflow.com/questions/4461346/slow-rails-stack "Slow rails stack") on StackOverflow and found that one proposed solution was to upgrade to 1.9.3.

Hmmm… I wanted to keep Ruby 1.9.2 installed, but needed a way to speed things up, so I started looking for ways to manage multiple Ruby versions on Windows (yes, I know, [Windoof](http://www.urbandictionary.com/define.php?term=windoof "Explination of the term"), but just remember that you cannot play Skyrim on Linux without performing a backwards somersault through your own sphincter).

That's when I discovered [pik](https://github.com/vertiginous/pik "Project homepage").

## Installing pik

pik bills itself as a tool to manage multiple versions of Ruby on Windows which can be used from the Windows command line (cmd.exe), Windows PowerShell, or Git Bash. It sounded ideal for the job.

In order to install pik you need a working version of Ruby. No problemo, I had already installed 1.9.2 using the mighty [RubyInstaller](http://rubyinstaller.org/ "The easy way to install Ruby on Windows").

After that, as you can install pik via rubygems,  it's a matter of opening a command prompt and typing:

```sh
$ gem install pik
```

If the gem installs successfully, you'll see a message telling you to use the `pik_install` script to install the pik executable. You need to install it somewhere that's located in your `PATH`, but not your `ruby/bin` directory.

I did this (taking care with the backslash):

```sh
$ pik_install C:\pik
```

As `C:/pik` isn't in my `PATH`, I had to add it manually and restart the PC. Here's a brief tutorial on how to [do that](http://www.computerhope.com/issues/ch000549.htm "How to set the path and environment variables in Windows") (alter your `PATH` variable, not restart the PC).

So following the reboot, it was time to install Ruby 1.9.3. I opened a command prompt and typed:

```sh
$ pik
```

And pik obligingly added my current Ruby version:

```sh
** Adding:  192: ruby 1.9.2p290 (2011-07-09) [i386-mingw32]
Located at:  C:\Ruby192\bin
```

Then:

```sh
$ pik install ruby 1.9.3
** Adding:  193: ruby 1.9.3p429 (2013-05-15) [i386-mingw32]
 Located at:  C:\Users\Me\.pik\rubies\Ruby-193-p429\bin
```

And that was all there was to it.

## Configuring pik

To list the ruby versions recognized by pik I typed:

```sh
$ pik list
* 192: ruby 1.9.2p290 (2011-07-09) [i386-mingw32]
193: ruby 1.9.3p429 (2013-05-15) [i386-mingw32]
```

And could switch between them thus:

```sh
$ pik 193
$ ruby -v
ruby 1.9.3p429 (2013-05-15) [i386-mingw32]
```

Feeling pretty happy I navigated to my Rails project directory, switched to Ruby 1.9.3 and tried to start the server with `rails s`, yet to my dismay I saw:

```sh
The command "rails" is either spelt incorrectly or could not be found
```

It seems that Ruby 1.9.3 had failed to recognize the gems I had installed using Ruby 1.9.2. Drat!!

I did some Googling and found that by setting `gem_home` in pik's configuration, you could share gem sets.

```sh
pik use 192
gem env
=>  - GEM PATHS: C:/ruby192/lib/ruby/gems/1.9.1

pik config gem_home=C:/ruby192/lib/ruby/gems/1.9.1

pik use 193
pik config gem_home=C:/ruby192/lib/ruby/gems/1.9.1
```

Following this, my Ruby 1.9.3 version recognized all of the gems installed under 1.9.2 and although I could run scripts which required other gems (FXRuby for example), annoyingly I still got the same error message when I tried to start the rails server.

So, I reverted `gem_home` to its original value for both Ruby versions, switched to Ruby 1.9.3 and installed Rails a second time.

This was a bit of a clunky solution, but it worked and for that I was grateful. I posted a [question](https://groups.google.com/forum/#!msg/discuss_pik/lfSJW-ZNWJ0/PJCRtgUJ6NEJ "pik mailing list") on the pik mailing list asking how I could solve this issue, but as of yet, I have received no reply.

Back in my Rails project directory I  ran my tests again (using Ruby 1.9.3) and they completed within 20 seconds.

What a difference!

### Further pik commands

`$ pik help commands` – Lists all available commands

`$ pik list -r` – Lists available remote Ruby versions

`$ pik 192` – Short cut for `$ pik use 192`

## Useful references

- [How to have multiple versions of Ruby AND Rails, and their combinations on Windows?](http://stackoverflow.com/questions/3648744/how-to-have-multiple-versions-of-ruby-and-rails-and-their-combinations-on-windo "StackOverflow")
- [How can I use pik in Win7?](http://stackoverflow.com/questions/6778160/how-can-i-use-pik-in-win7 "StackOverflow")
- [Pik on Windows 7 doesn't remember the selection](http://stackoverflow.com/questions/10607114/pik-on-windows-7-doesnt-remember-the-selection "StackOverflow")
