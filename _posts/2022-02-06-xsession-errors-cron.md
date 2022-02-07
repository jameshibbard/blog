---
title: How to Prevent the .xsession-errors File From Growing Too Large
layout: post
permalink: /prevent-xsession-errors-file-growing-too-large/
excerpt_separator: <!--more-->
tags:
  - bash
  - cron
  - linux
twitter:
  title: "How to Prevent the .xsession-errors File From Growing Too Large"
  description: "Automatically monitor the size of your .xsession-errors file and get notified if it becomes too large."
  image_url: https://res.cloudinary.com/hibbard/image/upload/v1644171525/stock/monitoring.jpg
---

Sometimes, without warning, your `.xsession-errors` file will grow to an enormous size. This can be inconvenient at best, or crash your operating system at worst.

In this post I explain how you can set up a cron job to automatically watch the file and get notified if it exceeds a certain size.

<!--more-->

My operating system of choice is Linux Mint, but with a little tweaking this should work on most distros.

If you'd like to skip straight to the solution, click <a href="#solution">here</a>.

## What Is This File and Why Does It Get So Huge?

Many Linux distributions use something called the [X Window System](https://en.wikipedia.org/wiki/X_Window_System) (also known as X11, or just X) to display graphical user interfaces. The `.xsession-errors` file is where X logs errors that occur within any of its client processes. These processes are regular GUI programs that you launch, such as a media player, image viewer, or file explorer.

The `.xsession-errors` file is located in your home directory. The dot at the beginning of the file name denotes that it is hidden by default, so you will have to make sure you can view hidden files (e.g. by pressing <kbd>Ctrl</kbd> + <kbd>H</kbd> in a file explorer) before accessing it.

The reason that it can grow so big, is that a large amount of applications can write to it and occasionally, one of these applications will go a little haywire and dump the same entry thousands of times over.

For example, this happened to me recently, when my `.xsession-errors` file grew to over 300GB and was packed full of entries like this:

```bash
[00007f600009b310] vdpau_chroma filter error: video mixer attributes failure: An invalid handle value was provided.
[00007f600009b310] vdpau_chroma filter error: video mixer rendering failure: An invalid handle value was provided.
[00007f600009b310] vdpau_chroma filter error: video mixer features failure: An invalid handle value was provided.
```

It turns out this was [being caused by VLC](https://unix.stackexchange.com/questions/561565/i-dont-know-what-is-producing-the-gigabytes-of-error-in-syslog).

Every time you reboot your system, the current `.xsession-errors` file is renamed to `.xsession-errors.old` and an empty `.xsession-errors` file will be created. However, if you don't often reboot your computer, this isn't very helpful and other measures are called for.

## How to Automatically Watch the Size of the `.xsession-errors` File

There are various posts around the internet that recommend truncating this file whenever it gets too big, or disabling writes on it altogether. And while that's fine, I would prefer to get notified instead (via a system notification), so that I can check which application is misbehaving and address the root of the problem.

Doing anything automatically in regular intervals on a Linux system sounds like a job for [cron](https://en.wikipedia.org/wiki/Cron), so let's look at that first.

### Setting up a Cron Job

Cron is a command-line utility and you can use it to schedule jobs (commands or shell scripts) to run periodically at fixed times, dates, or intervals. You can access it by typing `crontab -e` into a terminal, which will then open the crontab file for the current user in an editor (you will be asked to select an editor if you are running this for the first time).

<img class="shadow" alt="The crontab file open in the nano editor" src="https://res.cloudinary.com/hibbard/image/upload/v1644226027/xsession-errors/crontab.png">

Enter the following line at the bottom of the file:

```
* * * * * XDG_RUNTIME_DIR=/run/user/$(id -u) notify-send 'Hey' 'Hello, World!'
```

Save the file and exit. You should now recieve a "Hello, World!" system notification every minute. Nice, huh?

<img class="shadow" alt="Test notification reading 'Hello, World!'" src="https://res.cloudinary.com/hibbard/image/upload/v1644226757/xsession-errors/test-notification.png">

Satisfy yourself that this command works, then let's take a look at what's going on.

The above line breaks down as follows:

```
<frequency> <command-to-execute>
```

The frequency part is `* * * * *`. As we can read in the screenshot above, to define the time you can provide concrete values for minute, hour, day of the month, month, and day of the week. Or we can use `*` in these fields for 'any'. An asterisk in every field means run the given command every minute.

> If you'd like to run a command at a different interval, you can use [Cron Helper](https://cron.help/) to figure out the correct syntax.

The command part is `notify-send 'Hey' 'Hello, World!'`. This uses the current desktop environment's notification system to create a notification and is called thus: `notify-send 'Title' 'Message'`. You can also run this command straight from the command line to see it in action.

To [use this command with cron](https://stackoverflow.com/questions/16519673/cron-with-notify-send) however, we need to specify a `XDG_RUNTIME_DIR` environment variable. This tells `notify-send` where to find a user-specific directory in which to store small temporary files.

The value we set the `XDG_RUNTIME_DIR` variable to is `/run/user/$(id -u)`, where `$(id -u)` returns the current user id as a number. This usually starts at 1000, meaning the whole path will be something like `/run/user/1000`.

Note that the `$( ... )` syntax is used for command substitution, which allows the output of a command to replace the command itself.

### Checking the Size of `.xsession-errors`

The next thing to look at is how to check the size of the `.xsession-errors` file.

We can do that with the disk usage command:

```bash
du -k ~/.xsession-errors
```

The `-k` switch ensures that the output is in kilobytes. This will output something like:

```bash
240  /home/jim/.xsession-errors
```

As we are only interested in the first number (not the path), we can grab this using the `awk` command:

```bash
du -k ~/.xsession-errors | awk '{ print $1 }'
```

This outputs the size of `.xsession-errors` in kilobytes:

```
240
```

Finally, we can use the `test` command to check whether the file size is greater than a certain threshold. For example 1MB:

```bash
test $(du -k ~/.xsession-errors | awk '{ print $1 }') -gt 1024
```

Notice that once again we are using the command substitution syntax (`$( ... )`), to allow the output of both commands to replace the commands themselves.

This gives us something like this:

```bash
test 240 -gt 1024
```

The `test` command provides no output, but returns an exit status of 0 for "true" (test successful) and 1 for "false" (test failed). We can use this to our advantage in the next section.

You can also check this using the `$?` variable which represents the exit status of the previous command.

```bash
test 1 -gt 2; echo $?
1
```

```bash
test 1 -gt 0; echo $?
0
```

### Only Trigger Notification if File Size Exceeded

The final piece of the puzzle is to ensure that our notification is only triggered if the size of `.xsession-errors` is above a certain threshold. We can do this with the `&&` operator, like so:

```
<frequency> <command-1> && <command-2>
```

The right side of `&&` will only be evaluated if the exit status of the left side is zero (i.e. true).

<span id="solution">This gives us the following as our final solution:</span>

```bash
*/15 * * * * [ $(du -k .xsession-errors | awk '{ print $1 }') -gt 1024 ] && XDG_RUNTIME_DIR=/run/user/$(id -u) notify-send 'Hey' '.xsession errors is getting too big!'
```

Note that I am using square brackets as an alias for the `test` command and that I have set the cron job to run every 15 minutes (which is plenty). It is also not necessary to specify the complete path of the `.xsession-errors` file, as the cron job will be run in the user's home directory.

If the file size exceeds whichever threshold you have specified, a notification will be triggered every 15 minutes until the problem is addressed.

For added convenience, I have also [set up an alias in my shell](https://www.sitepoint.com/zsh-commands-plugins-aliases-tools/#15customaliasestoboostyourproductivity), so that when I type `x`, the `.xsession-errors` file is opened in my editor.

## Conclusion

In this post I have shown how to set up a cron job to automatically monitor the size of the `.xsession-errors` file and to trigger a notification if the file size becomes too big. I hope people find this useful. If you have any questions or comments, let me know below.
