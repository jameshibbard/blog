---
title: Fixing Cinnamon Gjs-CRITICAL Errors in Linux Mint After Upgrade
layout: post
permalink: /fixing-cinnamon-gjs-critical-errors-linux-mint-upgrade/
excerpt_separator: <!--more-->
tags:
  - cinnamon
  - linux
image_shadow: true

---

Yesterday, I upgraded from Linux Mint 22 to 22.1 Xia using the Update Manager. The process was smooth enough, but having upgraded my `.xsession-errors` file started filling up with `Gjs-CRITICAL` errors.

Here's how I solved the problem.

<!--more-->

## Quick Fix

For those of you looking for a quick fix, the problem was my workspace switcher applet. I stopped it filling `.xsession-errors` with junk by altering the `display()` method on the `WorkspaceOsd` class in `/usr/share/cinnamon/js/ui/workspaceOsd.js` to return immediately:

```js
var WorkspaceOsd = GObject.registerClass(
  class WorkspaceOsd extends Clutter.Actor {
    ...
    display(activeWorkspaceIndex, workspaceName) {
      return; // Add this line.
      ...
    }
  }
);
````

This effectively disabled the Workspace OSD (On-Screen Display), preventing it from rendering or animating during workspace switches.

If that solves your problem, great! If you'd like to understand more about what the problem was and what that line does, read on.

## So What is `.xsession-errors`?

`.xsession-errors` is a log file typically located in your home directory (`~/.xsession-errors`). It captures output from graphical applications and the X session, including warnings, errors, and debugging messages. If you are looking for this file but cannot see it, you may need to have Linux display hidden files (<kbd>Ctrl</kbd> + <kbd>h</kbd>).

While this might sound like a useful file to have, occasionally a rogue program might write lots and lots of errors to it, causing the file to balloon to many gigabytes in size. This can be inconvenient at best, or crash your operating system at worst.

> You can read about how to set up an early warning system to stop this happening here: [How to Prevent the .xsession-errors File From Growing Too Large](https://hibbard.eu/prevent-xsession-errors-file-growing-too-large/).

The problem in my case was caused by my workspace switcher applet.

![Screenshot of the Cinnamon Applets Manager showing various applets, including 'User Applet', 'Window List', 'Windows Quick List', 'Workspace Switcher', and 'XApp Status Applet'. The 'Workspace Switcher' applet is highlighted and selected for configuration.](https://res.cloudinary.com/hibbard/image/upload/v1737292196/workspace-switcher_skeo9e.png)

Every time I switched workspace, it was writing ca. 2000 lines of errors to my `.xsession-errors` file. Most of them looked like this:

```text
(cinnamon:3211): Gjs-CRITICAL **: 12:20:15.112: Object .Gjs_ui_workspaceOsd_WorkspaceOsd (0x5e6cf4bdd290), has been already disposed — impossible to access it. This might be caused by the object having been destroyed from C code using something such as destroy(), dispose(), or remove() vfuncs.
== Stack trace for context 0x5e6cf3617600 ==
#0   7ffd3805b250 b   /usr/share/cinnamon/js/ui/environment.js:156 (201847d7c150 @ 15)
#1   7ffd3805b360 b   self-hosted:221 (201847d70970 @ 267)
#2   7ffd3805bac0 b   /usr/share/cinnamon/js/ui/environment.js:156 (201847d70fb0 @ 625)
#3   7ffd3805c210 b   /usr/share/cinnamon/js/ui/environment.js:360 (201847d7c9c0 @ 23)
#4   5e6cf48b7118 i   /usr/share/cinnamon/js/ui/workspaceOsd.js:124 (251492f0f0b0 @ 108)
```

Not good!

As one can see from the stack trace, the problem occurs on line 124 in `/usr/share/cinnamon/js/ui/workspaceOsd.js`

```js
GLib.source_remove(this._timeoutId);
```

This line removes a previously set timeout, which is used to control the disappearance of the Workspace OSD. The workspace OSD is a visual indicator that appears briefly when switching workspaces, showing the active workspace name or number.

To understand how this code is being triggered, I traced the method containing it. Line 124 is part of the `_onTimeout()` method:

```js
_onTimeout() {
  GLib.source_remove(this._timeoutId);
  this._timeoutId = 0;
  this.ease({
    opacity: 0.0,
    duration: ANIMATION_TIME,
    mode: Clutter.AnimationMode.EASE_OUT_QUAD,
    onComplete: () => this.destroy(),
  });
  return GLib.SOURCE_REMOVE;
}
```

This method is called when the timeout expires, and as seen here, it destroys the `WorkspaceOsd` object. This aligns with the error message about accessing an already disposed object.

Searching the file revealed that `_onTimeout()` is invoked in the `display()` method via this line:

```js
this._timeoutId = GLib.timeout_add(GLib.PRIORITY_DEFAULT, DISPLAY_TIMEOUT, this._onTimeout.bind(this));
```

This is where the timeout is set. The `display()` method is responsible for initiating the animation and lifecycle of the Workspace OSD, making it the root cause of the issue. Thus, modifying `display()` to return immediately prevents the problematic chain of events.

```js
var WorkspaceOsd = GObject.registerClass(
  class WorkspaceOsd extends Clutter.Actor {
    ...
    display(activeWorkspaceIndex, workspaceName) {
      return; // Add this line.
      ...
    }
  }
);
```

You can edit the file in an editor of your choice. I used Sublime Text, but in the worst case, nano would also suffice.

```bash
sudo nano /usr/share/cinnamon/js/ui/workspaceOsd.js
```

## Restart Cinnamon

Once the change had been made I needed to restart Cinnamon.

You can do this by pressing <kbd>Alt</kbd> + <kbd>F2</kbd>, then typing "r" (for restart ) into the _Run a Command_ dialog box that appears.

Alternatively, you can type the following into the terminal:

```bash
cinnamon --replace &
```

Or log out and log back in again.

I hope this helps someone. If you have any questions, you can hit me up in the comments below and I will do my best to help.
