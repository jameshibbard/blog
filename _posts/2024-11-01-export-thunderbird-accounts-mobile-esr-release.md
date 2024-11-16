---
title: Export Thunderbird Accounts to Mobile When Stuck on an Older ESR Release
layout: post
permalink: /export-thunderbird-accounts-mobile-esr-release/
excerpt_separator: <!--more-->
tags:
  - linux
  - thunderbird
image_shadow: true

---

Mozilla recently launched [Thunderbird Mobile for Android](https://www.thunderbird.net/en-US/mobile/), and I was eager to give it a try. However, I am running Linux Mint 21.3, and installing Thunderbird from the repositories means you're stuck with version 115.x, which doesn’t include the "Export to Mobile" feature. This is only available as of Thunderbird Desktop 128.4.0 or Thunderbird Beta 132 and newer.

Manually setting up my many email accounts on mobile wasn’t something I wanted to deal with. Instead, I downloaded a newer portable version of Thunderbird, used that to export my settings to the mobile app, and then reverted to my distro’s ESR release. This post explains how you can do the same.

<!--more-->

> Note: While I am using Linux Mint, these instructions can be adapted for other operating systems. The steps may differ slightly based on your OS.

## 1. Downloading a Newer Thunderbird Version

First, visit the [Thunderbird website](https://www.thunderbird.net/en-US/download/linux/) and download a newer version that includes the "Export to Mobile" functionality. I selected the Thunderbird Extended Support Release (default option), which currently provides version 128.4.3.

Clicking *Download* will save a file named `thunderbird-128.4.3esr.tar.bz2` to your Downloads folder. Right-click the file and choose *Extract Here*.

![Screenshot of the Thunderbird Desktop download page showing options to select Locale, Release Channel (Thunderbird Extended Support Release), and Operating System, with a highlighted Download button.](https://res.cloudinary.com/hibbard/image/upload/v1731772422/thunderbird-mobile-export/01-thunderbird-mobile-export.png)

## 2. Locating and Preparing Your Profile

Next, locate the folder where Thunderbird stores its profiles. On Linux Mint, this is typically found at `home/<user-name>/.thunderbird`. Open this folder and note the subfolders it contains. Also, make a copy of `installs.ini` and `profiles.ini`.

You’ll also need to know where your current Thunderbird profile is stored. Unless you’ve changed it, it should be in the same folder. However, mine is located at `/home/jim/files/Thunderbird`, which I find handy for backup purposes. If you’re unsure of your profile’s location, open Thunderbird and navigate to _Help > Troubleshooting Information > Profile Directory > Open Directory_.

![Screenshot of Thunderbird's Troubleshooting Information page, showing application details such as version 115.16.0esr, build ID, operating system, and the profile directory with an 'Open Directory' button.](https://res.cloudinary.com/hibbard/image/upload/v1731772936/thunderbird-mobile-export/02-thunderbird-mobile-export.png)

## 3. Launching the New Version

Ensure that no other versions of Thunderbird are running, then launch the newly downloaded version. You can do this in the folder you just extracted, by either double-clicking the `thunderbird` executable or running `./thunderbird` from a terminal.

When the newer version of Thunderbird starts, it will create a new profile folder in `home/<user-name>/.thunderbird`. For example, mine was named `ee3snioc.default-esr`. Once this new profile folder is created, close Thunderbird. Then, rename the new profile folder by appending `--orig` to its name. I renamed mine to `ee3snioc.default-esr--orig`.

**IMPORTANT:** Before proceeding any further, back up your profile folder. Skipping this step may lead to a ["You have launched an older version of Thunderbird" error](https://support.mozilla.org/en-US/kb/unable-launch-older-version-profile) when reverting to your original ESR version.

![Screenshot of Thunderbird's account setup screen showing no messages found, a 'Support Us' section encouraging donations, and the 'Donate' button on the right.](https://res.cloudinary.com/hibbard/image/upload/v1731772936/thunderbird-mobile-export/03-thunderbird-mobile-export.png)

## 4. Creating a Symlink to Use Your Existing Profile

To use your existing profile with the new version, you need to create a symlink pointing Thunderbird to your current profile folder.

To create the symlink, open a terminal and use the following syntax:
`ln -s /path/to/your/profile /path/to/new/profile`

In my case, the command was:
`ln -s /home/jim/files/Thunderbird /home/jim/.thunderbird/c381xert.default-esr`.

With this done, restart the newer Thunderbird version. It will use your existing profile, and you can proceed with exporting your settings to the mobile app.

![File manager view of the .thunderbird folder, showing profile folders such as c381xert.default-esr and c381xert.default-esr--orig, along with files like installs.ini and profiles.ini.](https://res.cloudinary.com/hibbard/image/upload/v1731772422/thunderbird-mobile-export/04-thunderbird-mobile-export.png)

## 5. Exporting Settings to Thunderbird Mobile

To find the export button, click:
_≡ > Tools > Export for Mobile_, then follow the [instructions on the Mozilla site](https://support.mozilla.org/en-US/kb/thunderbird-android-import#w_export-thunderbird-desktop-settings) to complete the process.

![Screenshot of Thunderbird's 'Export for Mobile' settings page, showing options to export accounts to Thunderbird Mobile using a QR code, with selectable account options.](https://res.cloudinary.com/hibbard/image/upload/v1731772422/thunderbird-mobile-export/05-thunderbird-mobile-export.png)

## 6. Cleaning Up and Reverting to the ESR Version

Once everything has been set up on your mobile, close the newly downloaded version of Thunderbird. Then, delete the tar file, the extracted folder, and the original profile folder it created. Also, delete the symlink you made and your profile folder, as well as `installs.ini` and `profiles.ini`.

Finally, restore your profile folder, `installs.ini`, and `profiles.ini` from the backups you made earlier. Open Thunderbird, and you’ll be back on your distro's ESR release as though nothing ever happened.

---

I hope this tip helps someone. If you have any questions, leave a comment below, and I’ll be happy to answer them.
