---
title: Getting Started with NodeGUI
layout: post
permalink: /node-gui/
excerpt_separator: <!--more-->
tags:
  - node
  - javascript
  - gui
twitter:
  title: "TODO"
  description: "TODO"
  image_url: TODO
---

[NodeGUI](https://github.com/nodegui/nodegui) is an open source library for building cross-platform native desktop applications with JavaScript and CSS-like styling.

In this article, I'm going to demonstrate how to get up and running with NodeGUI. We'll set up a development environment, take a look at several of the library's basic concepts, then finish off by creating a simple password generator app.

If you're curious to see what we'll end up with, [the finished code can be found on GitHub](https://github.com/jameshibbard/nodegui-password-generator).

<!--more-->

And this is what the app will look like:

![Screenshot of app](https://res.cloudinary.com/hibbard/image/upload/v1568030141/password-generator.png)

## Why Not Electron?

But before we get into it, let's look at why you might want to use NodeGUI, as opposed to one of the more popular Chromium-based solutions, such as [Electron](https://electronjs.org/).

### Electron apps are bloated

The [main criticism levelled at Electron apps](https://news.ycombinator.com/item?id=12119278), is that they are bloated and require way too much memory. This is because each Electron app ships with a fully featured version of the Chromium browser and is not in a position to share resources, as native apps would.

For larger apps, on a high-powered desktop machine, this is fine. But when it comes top anything I'm likely to write, shipping a whole browser to render my app feels very much like cheating. So much so, that it would probably be easier to write an [offline first](http://offlinefirst.org/) web app instead.

NodeGUI on the other hand, is powered by the [Qt framework](https://wiki.qt.io/About_Qt). This means that its widgets are rendered natively and that _it does not need to open up a browser instance to render the UI_. This makes it both CPU and memory efficient and much more suited to my needs.

### Privacy concerns

As mentioned, Electron apps are based on the [Chromium browser](https://www.chromium.org/Home) (the open-source version of Google Chrome) and, privacy-wise, this is not ideal. Google has become unbelievably data hungry in the past few years, and in my opinion Chromium cannot be trusted to not phone home in some way shape or form.

With NodeGUI, this is obviously a non-issue.

## Setting Up a Dev Environment

With that out of the way, let's get NodeGUI up and running.

### Node.js

NodeGUI requires Node.js version 12.x or above, so let's get that installed first.

Either head on over to the [project's home page](https://nodejs.org/en/download/) and download the correct binaries for your system, or use a version manager such as [nvm](https://github.com/creationix/nvm). I would recommend using a version manager where possible, as this will allow you to install different Node versions and switch between them at will. It will also negate a bunch of potential permissions errors.

You can check that the installation process went well by typing `node -v` to confirm the version you are running.

### Additional Dependencies

I'm running Linux, so the installation instructions in this section will reflect that. For other operating systems, please [check the documentation](https://nodegui.github.io/nodegui/#/tutorial/development-environment).

To get NodeGUI working, we'll need to install [Make](http://man7.org/linux/man-pages/man1/make.1.html), [GCC](https://gcc.gnu.org/) v7 and [CMake](https://cmake.org/). This can be done with:

```
sudo apt-get install make gcc cmake
```

On a standard Linux install the chances are that Make and GCC will be installed already. Once done, you can check the versions using:

```
make -v
$ GNU Make 4.1

gcc -v
$ gcc version 7.4.0 (Ubuntu 7.4.0-1ubuntu1~18.04.1)

cmake --version
$ cmake version 3.10.2
```

Finally, it is advisable (but probably not essential) to install the `pkg-config` and `build-essential` packages. You can do this using:

```
sudo apt-get install pkg-config build-essential
```

And that's it, we're good to go.

## Clone the Starter Repo

Next, let's clone the nodegui-starter repo. This will provide us with the minimal setup we need to start developing.

```
git clone https://github.com/nodegui/nodegui-starter
cd nodegui-starter
npm install
npm start
```
