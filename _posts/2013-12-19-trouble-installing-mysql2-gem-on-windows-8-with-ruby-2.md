---
title: Trouble Installing mysql2 Gem on Windows 8 with Ruby2
layout: post
permalink: /trouble-installing-mysql2-gem-on-windows-8-with-ruby-2/
tags:
  - mysql
  - ruby
  - ruby gems
  - windows
excerpt_separator: <!--more-->
old-comments: trouble-installing-mysql2-gem-on-windows-8-with-ruby-2.html
comments: false
---

I recently installed the 64 bit version of Ruby 2.0 on my Windows 8 machine.

All of my old projects I tested with it ran just fine and the world was a happy place.

Then I had to install Rails, so that I could make some minor changes to an existing app… and the pain began.

<!--more-->

## The problem

Cutting a long story short, I couldn't for the life of me get the mysql2 gem to compile and install.

This is the error message I got:

```sh
C:/Ruby200-x64/bin/ruby.exe extconf.rb
checking for main() in -llibmysql... no
*** extconf.rb failed ***
Could not create Makefile due to some reason,
probably lack of necessary libraries and/or headers.
Check the mkmf.log file for more details.
You may need configuration options.
```

I followed numerous instructions from across the internet ([example](http://stackoverflow.com/questions/16295011/error-during-install-of-mysql2-gem-for-ruby-2-0-0-on-windows "Error during install of mysql2 gem for ruby 2.0.0 on Windows")) as to how this might be achieved.

Most seemed to indicate that you could set the `--with-mysql-dir` flag accordingly and although this got me close, no dice.

## The solution

This is what worked for me. Hopefully it'll help someone else.

- Uninstall the 64 bit version of Ruby 2 (apparently, there are lot of libraries that haven't been tested against 64 bit Ruby on Windows and it can lead to errors)
- Download the latest 32 bit Ruby 2 version from [here](http://rubyinstaller.org/downloads/ "Ruby Installer for Windows") and install
- Download the 32 bit version of the [DevKit](http://rubyinstaller.org/add-ons/devkit/ "Meet the DevKit") from the same link as above and follow [these instructions](https://github.com/oneclick/rubyinstaller/wiki/Development-Kit "Development Kit") to have it enhance your previously installed Ruby version
- Download the 32 bit MySQL C connector (archive version) from [here](http://dev.mysql.com/downloads/connector/c/ "Download Connector/C") and unzip it to C:/mysql
- Open a command prompt and enter `gem install mysql2 --platform=ruby -- '--with-mysql-dir="C:/mysql"`
- Copy the file `libmysql.dll` from `C:/mysql/lib` to your Ruby `bin` folder
- Optional: delete `C:/mysql`

Reference: [https://github.com/oneclick/rubyinstaller/issues/191](https://github.com/oneclick/rubyinstaller/issues/191)
