---
title: How to Update Phusion Passenger When Installed via RubyGems
layout: post
permalink: /update-passenger/
excerpt_separator: <!--more-->
tags:
  - apache
  - passenger
  - rails
  - ruby
  - ruby gems
  - servers
twitter:
  title: "How to Update Phusion Passenger When Installed via RubyGems"
  description: "When I tried to update Phusion Passenger my Rails app crashed, and Apache's logs pointed to a segmentation fault. Here's how I fixed it."
  image_url: https://res.cloudinary.com/hibbard/image/upload/f_auto,w_800/v1535634471/stock/passenger.jpg
---

<figure>
  <img src="https://res.cloudinary.com/hibbard/image/upload/f_auto,w_800/v1535634471/stock/passenger.jpg" alt="Pair of feet sticking out of a car window">
  <figcaption>Photo by <a href="https://unsplash.com/photos/Jx39BWpv6xs">Erik Odiin</a> on <a href="https://unsplash.com/search/photos/passenger">Unsplash</a></figcaption>
</figure>

Phusion Passenger (a.k.a. mod_rails) is a module for the Apache HTTP Server which can (among other things) be used to deploy Rails apps.

As with any piece of software, from time to time [security vulnerabilities will be discovered](https://blog.phusion.nl/tag/security%20advisory/) and Passenger will need to be updated.

Although the project's homepage offers [some excellent documentation on how to do this](https://www.phusionpassenger.com/library/install/apache/upgrade/), the steps they describe didn't work for me and resulted in my app crashing.

Inspecting the Apache error logs informed me that a segmentation fault had occurred and that I may have encountered a bug in the Ruby interpreter. This was accompanied by a 5,000 line stack trace.

Oh dear!

<!--more-->

## Let's Start at the Beginning

If you installed Passenger via RubyGems, the update guide (linked to above) recommends that you simply repeat the normal installation process.

This is as follows:

### Install the Gem

Easy enough.

```sh
gem install passenger --no-rdoc --no-ri
```

Note that the `--no-rdoc --no-ri` argument makes installation faster by skipping generation of API documentation. You can also avoid specifying this every time you install a gem by adding this to a `.gemrc` file in your home directory and restarting your terminal.

```sh
cd && touch .gemrc
echo "gem: --no-rdoc --no-ri" > .gemrc
source ~/.bashrc
```

### Run the Passenger Apache Module Installer

Step 2 takes you into an install wizard, where you will be asked what you want to install (Ruby, Node etc).

```sh
passenger-install-apache2-module
```

Passenger will then compile the Apache module. Depending on how much RAM you have available, this might take some time.

Once Passenger is done, it'll spit out a configuration snippet which you should paste into your Apache configuration file.

Finally, it'll ask you to restart Apache with `sudo service apache2 reload`.

This is where my problems started ...

## Segmentation Fault

When I attempted to restart Apache, my app went into 404 mode and became unreachable.

Inspecting `/var/log/apache2/error.log` revealed a very long error message.

```sh
App 108939 output: /var/www/app/shared/bundle/ruby/2.5.0/gems/celluloid-0.17.3/lib/celluloid/mailbox.rb:41:
App 108939 output: [BUG] Segmentation fault at 0x0000557b8e8ba731
App 108939 output: ruby 2.5.1p57 (2018-03-29 revision 63029) [x86_64-linux]
App 108939 output:
App 108939 output: -- Control frame information -----------------------------------------------
App 108939 output: c:0016 p:---- s:0076 e:000075 CFUNC  :signal
App 108939 output: c:0015 p:0075 s:0072 e:000071 METHOD /var/www/app/shared/bundle/ruby/2.5.0/gems/celluloid-0.17.3/lib/celluloid/mailbox.rb:41
App 108939 output: c:0014 p:0055 s:0067 e:000066 METHOD /var/www/app/shared/bundle/ruby/2.5.0/gems/celluloid-0.17.3/lib/celluloid/proxy/actor.rb:35
...
```

This had me scratching my head until about three quarters of the way down I found the following lines mentioning a "graceful restart":

```bash
[Sat Jun 02 06:25:01.765420 2018] [mpm_prefork:notice] [pid 83397] AH00171: Graceful restart requested, doing restart

[ N 2018-06-02 06:25:01.7953 108669/T4 age/Cor/CoreMain.cpp:615 ]: Signal received. Gracefully shutting down... (send signal 2 more time(s) to force shutdown)
[ N 2018-06-02 06:25:01.7954 108669/T1 age/Cor/CoreMain.cpp:1148 ]: Received command to shutdown gracefully. Waiting until all clients have disconnected...
[ N 2018-06-02 06:25:01.7954 108669/T1 age/Cor/CoreMain.cpp:1062 ]: Checking whether to disconnect long-running connections for process 127768, application /var/www/app/current (production)
```

I'd never heard of a graceful restart, but a quick Google search turned up [this page](https://benohead.com/apache2-graceful-restart-seg-fault-or-similar-nasty-error-detected-in-the-parent-process/) which stated:

> The graceful signal causes the parent process to advise the children to exit after their current request (or to exit immediately if they're not serving anything). The parent re-reads its configuration files and re-opens its log files. When a child dies off the parent replaces it with a child of the new generation of the configuration. This immediately begins serving new requests.

So I tried sending a graceful signal to Apache, like so:

```sh
apachectl graceful
```

and to my huge relief, the site came back up again.

After that I could complete the final step listed in the Passenger docs and verify the install:

```sh
sudo passenger-config validate-install
```

to which Passenger replied "Everything looks good. :-)"

Thanks passenger, I love you, too...
