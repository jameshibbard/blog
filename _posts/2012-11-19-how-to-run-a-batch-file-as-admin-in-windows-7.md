---
title: How to Run a Batch File as Admin in Windows 7
layout: post
permalink: /how-to-run-a-batch-file-as-admin-in-windows-7/
tags:
  - batch files
  - windows
excerpt_separator: <!--more-->
comments: false
---

This is quite a handy tip I picked up this week when I had a Windows batch file which I needed to run with administrator privileges.

Of course, you can find the file, right click it, select _Run as Administrator_ and do it that way, but if you need to run this file more than a few times, that soon gets to be a pain in the neck.

<!--more-->

So, the way to have this file run with elevated privileges every time is as follows:

1. Open Windows Explorer (Windows key + E) and locate the file in question.
2. Right click the file and select _Create Shortcut_.
3. Rename the shortcut to something sensible.
4. Right click on the shortcut and select _Properties_.

   ![Win 7 dialogue box displaying shortcut properties"](https://res.cloudinary.com/hibbard/image/upload/v1528967452/shortcut_properties.png "Shortcut properties")
5. In the window that now appears, make sure that the _Shortcut_ tab is selected.
6. Click on the _Advanced_ button.
7. Put a tick in the box that says _Run as Administrator_
8. Click _OK_ twice.

   ![Win 7 Run as Admin Dialogue"](https://res.cloudinary.com/hibbard/image/upload/v1528967624/run_as_admin.png "Run as admin")

Now, when you double-click the batch file, [UAC](http://en.wikipedia.org/wiki/User_Account_Control "User Account Control") will prompt you for administrative privileges.

You can also use this technique to always run the command prompt, or any other application for that matter, as administrator.

**Extra tip**: in Windows 7 you can start any program that you have pinned to the toolbar with administrator privileges, simply by holding <kbd>Ctrl</kbd> + <kbd>Shift</kbd> when you click on it.
